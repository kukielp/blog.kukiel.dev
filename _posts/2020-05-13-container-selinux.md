---
layout: post
title:  "Requires: container-selinux >= 2:2.74"
date:   2020-05-13 16:28:34 +1000
categories: linux centos docker
---
I had an issue recently installing docker on Centos-7

{% highlight bash %}
Error: Package: containerd.io-1.2.10-3.2.el7.x86_64 (docker-ce-stable)
    Requires: container-selinux >= 2:2.74
{% endhighlight %}

## Solution: 

Visit: 
[http://mirror.centos.org/centos/7/extras/x86_64/Packages/](http://mirror.centos.org/centos/7/extras/x86_64/Packages/)

Find the most recent version ( unless you need a specific version ).  At the time that was:

container-selinux-2.119.1-1.c57a6f9.el7.noarch.rpm

Install the package
{% highlight bash %}
     sudo yum install http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.119.1-1.c57a6f9.el7.noarch.rpm
{% endhighlight %}

Be aware if you are behind a proxy, you may need to edit the yum.conf file ( /etc/yum.conf ) described below:

{% highlight bash %}
    # The proxy server - proxy server:port number 
    proxy=http://{myproxyserver}:3128 
    # The account details for yum connections ( if required )
    # proxy_username=yum-user 
    # proxy_password=qwerty
{% endhighlight %}

Alternativly ( assuming correct http_proxy is set):

{% highlight bash %}
    # download locally
    wget http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.119.1-1.c57a6f9.el7.noarch.rpm
    # install
    sudo yum -i container-selinux-2.119.1-1.c57a6f9.el7.noarch.rpm
{% endhighlight %}