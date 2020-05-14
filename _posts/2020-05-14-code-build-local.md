---
layout: post
title:  "Running AWS Code build locally"
date:   2020-05-14 16:38:34 +1000
categories: aws cd/cd codebuild docker
---
I've been working with code build for some time now, have even had to configure our own images for some Windows builds.  Recently I helped my friend Zain setup a pipeline, while leanring the ins and out of codebuild ( using scp in codebuild to be specfic ) we performed 100's of test builds.  While the sercice itself is quite cheap free is a good option to cheap.  When asked how can I cut the costs I suggested running codebuild locally which I had not done myself untill now.

There is a great artile on [AWS Code Build Locally](https://aws.amazon.com/blogs/devops/announcing-local-build-support-for-aws-codebuild/)

but I thought I'd repeat the process and provide a very simple app to demonstrate the process as well as using actual aws credentials to perform typical tasks such as syncing to s3 or accessing parmeter store.

There are some pre-requisist:

- Docker
- aws command line installed
- aws access key and secret ( if you need to access a native service not just compile and app )
- 10 gig hdd space ( yes the codebuild image is huge )
- about 30 mins to buidl the container

The Steps at a high level are:
1. Clone the codebuild docker image repo
2. Build the docker image
3. Pull the codebuild agent container
4. Download the aws helper script
and your good to go.

For those beginning what is the codebuild doing, well it's a build envrioment.  If you look in the docker file you'll see it's literally installing all the depancies you might need ( ok not all of them but lots of common ones ).

You pass your repo to the build enviroment alogn with the instructions in the buildspec.yaml and the code build agent will interperate the commands running your build steps.

{% highlight bash %}
 paulk@iMac#/tmp/cra-test$ codebuild_build.sh -i aws/codebuild/standard:4.0 -a /tmp -c ~/.aws
{% endhighlight %}

My current workign directory ( pwd )
{% highlight bash %}
 /tmp/cra-test
{% endhighlight %}

I place codebuild.sh in my path so I can call it anywhere
{% highlight bash %}
 codebuild_build.sh
{% endhighlight %}

Use -i to specify the build envioment image
{% highlight bash %}
 -i {imagename}
{% endhighlight %}

Use -a to specify the output directory.  for me /tmp works fine I don't need to keep the artifact
{% highlight bash %}
 -a /tmp
{% endhighlight %}

Use -c to specify the aws credentials file
{% highlight bash %}
 -c ~/.aws
{% endhighlight %}

You can see how the shell scripts helps as it puts together this:
{% highlight bash %}
Build Command:
docker run -it -v /var/run/docker.sock:/var/run/docker.sock -e "IMAGE_NAME=aws/codebuild/standard:4.0" -e "ARTIFACTS=/tmp" -e "SOURCE=/tmp/cra-test" -e "AWS_CONFIGURATION=/Users/paulk/.aws" -e "INITIATOR=paulk" amazon/aws-codebuild-local:latest
{% endhighlight %}

See a run below

![alt text](https://cdn.kukiel.dev/preact-build.gif "Preact Build")

## Result: 

![alt text](/assets/post/2020-05-14-code-build-local/s3.png "S3 Output")