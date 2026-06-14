---
layout: post
title:  "AI Image Generation from CFML in One Function Call"
date:   2026-06-14 20:00:00 +1000
categories: cfml lucee cloudflare ai image-generation
---

I wanted to generate images from CFML. Not call out to some heavyweight Python service, not stand up a GPU — just write a prompt in Lucee and get a picture back. So I built a small library that wraps [Cloudflare's Workers AI](https://developers.cloudflare.com/workers-ai/) image models, and the whole thing comes down to this:

```cfc
cf  = new cloudflareimages.CloudflareImages();   // creds from env
img = cf.generate( prompt = "a lighthouse on a rocky coast at sunset, painterly" );
img.toFile( expandPath( "./lighthouse.jpg" ) );
```

That call produced this:

![A lighthouse on a rocky coast at sunset](/assets/post/2026-06-14-cfml-ai-images-with-cloudflare/lighthouse.jpg "Flux 1 Schnell, one call")

## Try it right now

I put a public demo up — type a prompt, hit generate, get an image:

### [cfml-image-with-cloudflare.kukiel.dev](https://cfml-image-with-cloudflare.kukiel.dev)

![The demo page](/assets/post/2026-06-14-cfml-ai-images-with-cloudflare/demo-page.png "The demo — prompt in, image out")

It's capped at a handful of images a day so it stays free, but it's a real, live CFML app calling real AI models. Here are a few things people (well, me) have made with it:

![A gallery of generated images](/assets/post/2026-06-14-cfml-ai-images-with-cloudflare/gallery.png "A few generations — a samurai cat, a hot air balloon, a cozy bookshop")

## The annoying part it hides

Cloudflare gives you a bunch of text-to-image models, and they don't agree on how to answer. Most of the Stable Diffusion family hand you **raw image bytes**. Flux hands you **JSON with the image base64-encoded inside it**. If you call the API directly you end up writing two code paths and a pile of content-type sniffing.

The library makes that disappear. Whatever model you pick, you get back the same `GenerationResult`:

```cfc
img.getBinary();    // the raw bytes
img.toBase64();     // base64 (handy for a data: URI)
img.toFile( path ); // write it to disk
```

The ugly part lives in exactly one place — a response normalizer — so you never think about it. Swap `@cf/black-forest-labs/flux-1-schnell` for `@cf/stabilityai/stable-diffusion-xl-base-1.0` and your code doesn't change.

## More than just generate

It also does image-to-image and inpainting, and it can list the available models:

```cfc
// reimagine an existing picture
img = cf.imageToImage( prompt = "make it winter", image = expandPath("./summer.png") );

// paint something into a masked region
img = cf.inpaint( prompt = "a hat", image = photo, mask = maskBytes );

// what can I use?
models = cf.listModels();
```

When something goes wrong it throws **typed** exceptions, so you can actually react to *what* failed — a missing token (`ConfigError`), Cloudflare saying no (`APIError`, with its real message and code), a network blip (`TransportError`). The demo uses that to show a friendly "that prompt was blocked" message when the safety filter trips, instead of a raw stack trace.

## Testing without burning money

The bit I'm happiest with: the test suite never calls Cloudflare. The one place that touches the network — a tiny `Transport` component — is swappable, so the tests inject a fake that returns canned responses (raw bytes for SDXL, base64 JSON for Flux). The whole suite runs offline, in milliseconds, costs nothing, and still proves the SDXL-vs-Flux normalization actually works. There's one live smoke test that only wakes up if real credentials are present.

```
flowchart: your code → facade → (validate) → Transport (the only cfhttp)
           → Cloudflare → normalizer → GenerationResult → back to you
```

Each piece is one small CFC with one job. That's what makes it a drop-in: copy the folder, point it at a Cloudflare account, ask for a picture.

## Getting a key

You need a free Cloudflare account and a **Workers AI** API token (dashboard → AI → Workers AI → REST API). The free tier gives you 10,000 "Neurons" a day at no charge — enough for a few hundred images depending on size. A 512×512 image is cheap; 1024×1024 costs about 4× more, so the demo runs at 512 to stretch the free allowance.

## Grab the code

It's all open source — **[cloudflareimages on GitHub](https://github.com/kukielp/cfml-image-with-cloudflare)**. Clone it, copy the `cloudflareimages/` folder into your app, and you're away. The README has the full API, a Mermaid diagram of how the pieces fit, and the TestBox suite. Use it, fork it, send a PR, and please share it around — the CFML world could use more of this.

If you give the demo a spin, let me know what you make.
