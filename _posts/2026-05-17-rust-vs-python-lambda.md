---
layout: post
title:  "We Cut Our Lambda Bill by 33% Switching to Rust — And AI Wrote the Code"
date:   2026-05-17 10:00:00 +1000
categories: aws lambda rust python benchmark cdk serverless
---

We ran 50,000 event registration records through identical Lambda functions — one in Python, one in Rust — processing CSV and JSON files from S3 into DynamoDB. Rust completed the work in 12-15 seconds. Python took 20-21 seconds. Same memory (1024MB), same architecture (ARM64/Graviton), same data, same DynamoDB tables.

That's a **33% reduction in compute cost** with zero loss in functionality. And the traditional argument against Rust ("it's too hard to write") no longer holds when AI tools can generate production-quality Rust as easily as Python.

## Try It Yourself

This is a simulated version of the real benchmark dashboard. Click **Run All Tests** and watch Rust vs Python process 50,000 records side-by-side. Change the memory tier to see how CPU allocation affects each runtime differently.

<div style="position:relative;width:100%;padding-top:0;margin-bottom:2rem;">
<iframe src="/assets/post/2026-05-17-rust-vs-python-lambda/demo.html" style="width:100%;height:900px;border:1px solid #2f3336;border-radius:12px;" frameborder="0" allowfullscreen></iframe>
</div>

The timings above are based on real benchmark data from live Lambda invocations. The demo runs locally in your browser so you can experiment without hitting my AWS account.

[Full source code on GitHub](https://github.com/kukielp/lambda-compare)

---

## The Experiment

### What We Built

A full-stack benchmark comparing Python and Rust Lambda performance on a realistic workload:

- **50,000 event registration records** across 30 music concerts
- **11 fields per record**: registration ID, event details, attendee info, ticket type, payment method, seat section
- Data stored in S3 as both **CSV (7.8 MB)** and **JSON (17.1 MB)**
- Each Lambda reads from S3, parses the file, and batch-writes to DynamoDB
- Real-time progress streaming via WebSocket API Gateway — every 1,000 records sends a timing update back to the browser
- A **verification Lambda** confirms both tables contain identical data after each run

### The Architecture

```
Browser (WebSocket) → API Gateway → Lambda → S3 + DynamoDB
                    ← progress messages ←
```

Both Lambdas are configured identically:
- **1024 MB memory** (ARM64 Graviton)
- **On-demand DynamoDB** (no throttling variable)
- **Batch writes of 25 items** with retry logic for unprocessed items
- Same IAM permissions, same region, same data

The only difference is the runtime: Python 3.12 managed runtime vs. Rust on `provided.al2023` custom runtime.

Infrastructure is deployed with **AWS CDK (Python)**. The entire stack — S3 bucket, 2 DynamoDB tables, 5 Lambda functions, WebSocket API Gateway with 6 routes — deploys in a single `cdk deploy`.

---

## The Results

### Raw Numbers (1024 MB / 0.58 vCPU)

| Test | Total Time | S3 Download | Parse Time | Avg per 1,000 Records |
|------|-----------|-------------|------------|----------------------|
| **Rust + CSV** | 14,740ms | 238ms | 140ms | 295ms |
| **Rust + JSON** | 12,416ms | 272ms | 167ms | 248ms |
| **Python + CSV** | 21,194ms | 207ms | 469ms | 424ms |
| **Python + JSON** | 20,205ms | 288ms | 170ms | 404ms |

### What Stands Out

**1. Parse performance is where Rust dominates on CPU-bound work**

CSV parsing: Rust finishes in 140ms. Python takes 469ms. That's **3.3x faster** — the `csv` crate in Rust is compiled and zero-copy where possible, while Python's `csv.DictReader` has interpreter overhead on every field.

Interestingly, JSON parsing is almost identical between the two (167ms vs 170ms). Python's `json` module is implemented in C and highly optimized for this exact workload — deserializing a large array of flat objects. Rust's `serde_json` is fast, but the gap vanishes when both are essentially calling into optimized native code.

**2. DynamoDB writes dominate — and compress the gap**

The bulk of execution time is DynamoDB `BatchWriteItem` calls. This is network I/O, not CPU. Both runtimes are waiting on the same DynamoDB service at the same latency. This is why we see a 1.5x gap rather than the 5-10x gap you'd see in pure CPU benchmarks.

This is actually the realistic scenario for most Lambda workloads: you're orchestrating AWS service calls, not crunching numbers. The performance gain from Rust is real but bounded by I/O.

**3. The overhead adds up**

Rust processes each batch of 1,000 records in ~280ms. Python takes ~400ms. That 120ms difference per batch is consistent — it's the accumulated overhead of Python's interpreter, object creation, `Decimal` conversion, and garbage collection.

At 50,000 records, that's 6+ seconds of pure runtime overhead.

**4. Memory tier changes the picture dramatically**

Try switching the demo above to 128 MB. At the smallest memory tier (0.07 vCPU), Python takes nearly **5 minutes** to process 50k records. Rust finishes in **23 seconds**. That's a 12x difference — because at 128MB, Python is CPU-starved: the interpreter overhead that's invisible at 1024MB becomes the dominant bottleneck when you have 8x less CPU.

This is the most practical finding: **Rust lets you run at lower memory tiers**. If Rust can do the job at 128MB where Python needs 1024MB, you're saving 8x on GB-seconds before you even factor in execution speed.

---

## The Cost Analysis

### Per-Invocation Cost (1024 MB)

Lambda pricing formula: `(Memory in GB) × (Duration in seconds) × $0.0000166667`

| Runtime | Duration | Cost per Invocation |
|---------|----------|-------------------|
| Rust (avg) | 13.6s | $0.000227 |
| Python (avg) | 20.7s | $0.000345 |
| **Savings** | **7.1s** | **$0.000118 (34%)** |

### At Scale

| Events/Day | Python Monthly | Rust Monthly | Annual Savings |
|-----------|---------------|-------------|---------------|
| 1 million | $0.21 | $0.14 | $0.85 |
| 100 million | $20.70 | $13.60 | $85 |
| 1 billion | $207 | $136 | $852 |

The honest take: for a single I/O-bound Lambda, the dollar savings are modest. But Lambda workloads rarely come alone. Most production systems have dozens of data-processing functions — ETL pipelines, event ingestion, webhook processing, file transformers. Multiply across a fleet and the savings compound to thousands per year.

The bigger win is often **right-sizing memory**. If Rust can run your workload at 256MB where Python needs 1024MB, you save 4x on the memory dimension alone — independent of execution speed.

### The Hidden Savings: Duration-Based Timeout Costs

When a Python Lambda occasionally hits the 15-minute timeout on large files, you pay for the full 15 minutes AND have to retry. A Rust Lambda with 30% less execution time gives you significantly more headroom before hitting timeout boundaries.

---

## Cold Starts: The Other Performance Win

We used warm Lambdas for fair comparison, but it's worth noting:

- **Rust cold start** (provided.al2023, ARM64): ~50-80ms
- **Python cold start** (managed runtime, with boto3): ~300-500ms

For latency-sensitive workloads (API backends, real-time processing), Rust's cold start advantage is arguably more important than its throughput advantage. A 6x faster cold start means fewer users experience the "first request" penalty.

---

## The AI Angle: Why This Changes Everything

### The Traditional Argument

"Rust is faster, but it takes 3x longer to write. The developer time cost outweighs the infrastructure savings."

This was true in 2023. It is not true in 2025.

### What Actually Happened

The Rust Lambda in this project — including AWS SDK integration, CSV/JSON parsing, DynamoDB batch writes with retry logic, WebSocket progress streaming, and error handling — was generated by Claude in a single session. The Python Lambda took the same amount of time to generate.

**Development time for both: effectively identical.**

The entire project — both Lambdas, the CDK infrastructure, the streaming UI, the data generator, the verification system — was built in a single afternoon with AI assistance.

### The Code Comparison

**Python — batch write:**
{% highlight python %}
with table.batch_writer() as batch:
    for record in records:
        record["amount_paid"] = Decimal(str(record["amount_paid"]))
        batch.put_item(Item=record)
{% endhighlight %}

**Rust — batch write:**
{% highlight rust %}
let write_requests: Vec<WriteRequest> = items
    .iter()
    .map(|r| {
        WriteRequest::builder()
            .put_request(
                PutRequest::builder()
                    .item("registration_id", AttributeValue::S(r.registration_id.clone()))
                    .item("event_id", AttributeValue::S(r.event_id.clone()))
                    // ... remaining fields
                    .build()
                    .expect("valid put request"),
            )
            .build()
    })
    .collect();
{% endhighlight %}

Yes, the Rust code is more verbose. No, that doesn't matter when AI generates it. What matters is:
- It compiles to a single static binary (~15MB)
- It runs 33% faster
- It uses less memory at peak
- Cold start is under 100ms

### The New Calculus

| Factor | Before AI (2023) | After AI (2025) |
|--------|------------------|-----------------|
| Rust dev time | 3-5x Python | 1x (AI generates both) |
| Rust expertise needed | Senior systems engineer | Prompt engineering |
| Maintenance burden | Higher (borrow checker, lifetimes) | Similar (AI handles refactors) |
| Runtime cost | 30-60% less | 30-60% less |
| Cold start | ~50-100ms | ~50-100ms |

The barrier to Rust was never the language — it was the learning curve. AI eliminates that curve.

---

## When to Use Rust vs. Python for Lambda

### Use Rust When:
- **High-volume data processing** — the 33% cost reduction compounds
- **Latency-sensitive paths** — cold starts matter, p99 matters
- **CPU-bound computation** — parsing, transformation, serialization
- **Low-memory configurations** — Rust thrives where Python starves
- **Long-running Lambda** — more headroom before the 15-minute timeout

### Stay with Python When:
- **Rapid prototyping** — still faster to iterate locally without compile steps
- **Heavy AWS SDK usage with minimal processing** — if 95% of your Lambda is `await sdk_call()`, the runtime barely matters
- **Team maintenance** — if your team knows Python and won't use AI for Rust maintenance
- **Lambda Layers and shared code** — Python's ecosystem for Lambda layers is more mature

### The Sweet Spot

Keep Python for orchestration Lambdas (Step Functions glue, simple API handlers) and move data-heavy Lambdas to Rust (file processing, ETL, stream consumers, batch operations).

---

## Reproducing This Benchmark

The entire project is in a single CDK stack:

{% highlight bash %}
# Prerequisites
rustup install stable
brew install zig
cargo install cargo-lambda

# Build Rust Lambda
cd lambdas/rust_processor
cargo lambda build --release --arm64

# Generate test data
python data/generate_data.py

# Deploy
cd cdk
python -m venv .venv && source .venv/bin/activate
pip install aws-cdk-lib constructs
cdk deploy --outputs-file outputs.json

# Update UI config and run
node update_config.js
cd ui && python -m http.server 8080
{% endhighlight %}

Open `http://localhost:8080`, click "Warm Up", then "Run All Tests". Watch the progress stream in real-time. Click "Verify Data" to confirm both tables match.

---

## Conclusion

Rust Lambda functions cost 33% less than Python for the same workload. The performance gap is bounded by I/O (DynamoDB writes) rather than CPU, so for pure computation the savings would be even larger.

The traditional trade-off — faster runtime vs. harder development — no longer exists. AI tools generate production-quality Rust Lambda code as easily as Python. The compile step adds 60 seconds to your deploy pipeline. That's it.

If you're running data-processing Lambdas at any meaningful scale, Rust is now the rational economic choice. The barrier is gone. The savings are real. The data is identical.

Ship it in Rust.
