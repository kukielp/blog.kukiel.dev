---
layout: post
title:  "Use AWS CDK to deploy a S3 bucket and use 53 to point DNS to the site!"
date:   2020-06-08 08:28:34 +1000
categories: AWS CDK S3 Route53
---
![Header](/assets/post/2020-06-08-AWS-cdk-s3-route53/header.png "Header")

Static sites are commonplace, often I'll knock something up as a proof of concept and to share that with my friends I'll make a public s3 bucket and send the link, quick and easy.  If I need/want this for a longer period of time I typically create a subdomain and point that to the bucket.

In this example we will:
- Create a bucket in s3 with CDK
- Setup the bucket to allow hosting
- Set the default document
- Deploy a sample html file to the bucket
- Look up a root hosted zone
- Create a new DNS record in an existing zone that points to a s3 bucket

# Requirements:
- AWS CDK
```npm install -g aws-cdk```
- Node.js - [Download Link](https://nodejs.org/en/download/)
- An AWS account
- Local credentials ( unless using Cloud 9 )
- Your Account ID ( required for Route 53 changes )
- A Zone in Route53

{% highlight bash %}
    # make a new folder for the project
    mkdir bucket.website.com
    # cd to the folder
    cd bucket.website.com
    # Initialise a cdk app
    cdk init app --language typescript
    # install the 3 additional modules we will need
    npm install @aws-cdk/aws-s3 --save-dev
    npm install @aws-cdk/aws-s3-deployment --save-dev
    npm install @aws-cdk/aws-route53 --save-dev
{% endhighlight %}

We are now ready to code.  Let's begin creating the files and folders.

{% highlight bash %}
    # Create a folder for the Program, this is my convention the program may have more parts
    # for now it's just static so create just those folders
    mkdir -p Program/static
    # Populate the index.html file with some content
    echo "S3 Hosting with CDK" > Program/static/index.html
{% endhighlight %}

Our example Program is ready to deploy.

open "lib/{stack-name}.ts" and let's code some infrastructure.

This is very straightforward, remember when creating buckets that the bucket name should match the domain name you intend to point at the bucket.

{% highlight js %}
    //Create the public S3 bucket 
    const publicAssets = new s3.Bucket(this, 'example-qr', {
      bucketName: 'example.url2qr.me',
      publicReadAccess: true,
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      websiteIndexDocument: 'index.html',
    });
{% endhighlight %}

Be aware that although the removalPolicy is set to destroy, when deleting this stack this will fail unless you remove the files in the bucket.  If the bucket is empty when the stack is deleted the bucket will also be deleted.

Deploy the static code to the bucket.
{% highlight js %}
    // Static Code into Bucket.
    const deployment = new s3Deployment.BucketDeployment(
        this,
        'deployStaticWebsite',
        {
            sources: [s3Deployment.Source.asset('./program/static')],
            destinationBucket: publicAssets,
        }
    );   
{% endhighlight %}

I have an existing zone I want to add a subdomain to ( url2qr.me ).  The first step is to look this up.  You can look this up by hostZoneId or zoneName.  For readability here I have used domain name. [Docs](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-route53.HostedZoneProviderProps.html)

{% highlight js %}
    //Lookup the zone based on domain name
    const zone = route53.HostedZone.fromLookup(this, 'baseZone', {
      domainName: 'url2qr.me'
    });
{% endhighlight %}

Next we create a new entry in Route53 and point the entry to the "bucketWebsiteDomainName" of the s3 bucket we created.  In this example it's "example.url2qr.me".

{% highlight js %}
 //Add the Subdomain to Route53
    const cName = new route53.CnameRecord(this, 'test.baseZone', {
        zone: zone,
        recordName: 'example',
        domainName: publicAssets.bucketWebsiteDomainName
    });
{% endhighlight %}

The complete code should look like so:

{% highlight js %}
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';
import * as s3Deployment from '@aws-cdk/aws-s3-deployment';
import * as route53 from '@aws-cdk/aws-route53';

export class AwsCdkS3Stack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    //Create the public S3 bucket 
    const publicAssets = new s3.Bucket(this, 'example-qr', {
      bucketName: 'example.url2qr.me',
      publicReadAccess: true,
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      websiteIndexDocument: 'index.html',
    });

    // Static Code into Bucket.
    const deployment = new s3Deployment.BucketDeployment(
      this,
      'deployStaticWebsite',
      {
        sources: [s3Deployment.Source.asset('./program/static')],
        destinationBucket: publicAssets,
      }
    );   

    //Lookup the zone based on domain name
    const zone = route53.HostedZone.fromLookup(this, 'baseZone', {
      domainName: 'url2qr.me'
    });

    //Add the Subdomain to Route53
    const cName = new route53.CnameRecord(this, 'test.baseZone', {
      zone: zone,
      recordName: 'example',
      domainName: publicAssets.bucketWebsiteDomainName
    });
  }
}
{% endhighlight %}

We will also have to pass in the accountID and default region. Open "bin/{stackname}.ts" and set the values.  I have them set as env variables, you can hardcode them for example purpose as well.

{% highlight js %}
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { AwsCdkS3Stack } from '../lib/aws-cdk-s3-stack';

const app = new cdk.App();
new AwsCdkS3Stack(app, 'AwsCdkS3Stack', {
    env: {
        account: process.env.CDK_DEFAULT_ACCOUNT,
        region: process.env.CDK_DEFAULT_REGION
    }
});
{% endhighlight %}

We are ready to deploy
{% highlight bash %}
  cdk bootstrap
  cdk deploy
{% endhighlight %}

Once complete run: 
{% highlight bash %}
   curl example.url2me.com
{% endhighlight %}

and we should see the contents of the index.html file.

![Curl](/assets/post/2020-06-08-AWS-cdk-s3-route53/curl.png "Curl")

On in a browser.
![safari](/assets/post/2020-06-08-AWS-cdk-s3-route53/safari.png "safari")

Full repo: [https://github.com/kukielp/aws-cdk-s3](https://github.com/kukielp/aws-cdk-s3)


