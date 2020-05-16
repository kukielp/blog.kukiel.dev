---
layout: post
title:  "The repository 'http://ppa.launchpad.net....does not have a Release file."
date:   2020-05-15 10:28:34 +1000
categories: linux ubuntu debian
---
Ever followed a guide a little to blindly and added a repository and now expereincing issues such as:

{% highlight bash %}
The repository 'http://ppa.launchpad.net/{something}/archive/ubuntu focal Release' does not have a Release file.
{% endhighlight %}

## Solution: 

{% highlight bash %}
sudo add-apt-repository -r ppa:{reponame}
sudo apt update
{% endhighlight %}

For me this happened to be:

{% highlight bash %}
sudo add-apt-repository -r ppa:longsleep/golang-backports
sudo apt update
{% endhighlight %}


And you should be back in business.