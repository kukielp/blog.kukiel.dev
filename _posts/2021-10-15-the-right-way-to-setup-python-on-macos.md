---
layout: post
title:  "The right way to setup Python on Macos"
date:   2021-10-15 16:28:34 +1000
categories: macos python developer
---

I use Python more and more regularly and each time I set up a new machine I go through the same steps, I forget a step and am left scratching my head, this time I decided to document them.

## Problem Statement:
- I want to use multiple versions of Python locally
- I do not want to override the default version of Python on the Mac in-case it's required some something else.
- I do not want to edit symlinks to the python binaries

## Solution: 
The solution is pyenv.  Pyenv allows me to install and manage as many version as python as I need and flip between them where required, it uses a convention to allow my terminal to use a version on Python installed under my home directory rather then over writing the global install.

In this example I am setting up my 2020 M1 MacBook pro, by default the Python version is:

{% highlight bash %}
 ~  python --version                                                                         
Python 2.7.16
{% endhighlight %}

and
{% highlight bash %}
 ~  python3 --version                                                                           ok
Python 3.8.9
{% endhighlight %}

I really want Python 3.10 for [Pygame](https://www.pygame.org/news), and 3.9 for [AWS Lambda](https://aws.amazon.com/lambda/), this is how you can accomplish that.

##  Installation
To accomplish this we first start with [Homebrew](https://brew.sh/), If your reading this your likely familiar, if not Homebrew is a software manager that is priamrliy used for MacOs ( though I do use it on linux ) to install the software I require with ease.

You can install HomeBrew in a terminal like so:
{% highlight bash %}
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
{% endhighlight %}

Next we update and install pyenv
{% highlight bash %}
brew update
brew install pyenv
{% endhighlight %}

{% highlight bash %}
cat <<EOT >> ~/.zshrc 
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/shims:$PATH"
export PATH="$PYENV_ROOT/bin:$PATH"
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi
EOT
{% endhighlight %}

Now I am use zsh if you are using bash that should read:
{% highlight bash %}
cat <<EOT >> ~/.bash_profile 
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/shims:$PATH"
export PATH="$PYENV_ROOT/bin:$PATH"
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi
EOT
{% endhighlight %}

Now we have to reload our terminal
{% highlight bash %}
source ~/.zshrc
#or for Bash
source ~/.bash_profile
{% endhighlight %}

We now have pyenv correctly installed and ready for application.

Let's install Python 3.10 and set it as the local version of Python to use for this terminal session.

I have Xcode installed, if you don't then you may need to follow these steps first

{% highlight bash %}
xcode-select --install
{% endhighlight %}

Then install Python 3.10
{% highlight bash %}
pyenv install 3.10.0
{% endhighlight %}

OK, let's check it out did it work?
{% highlight bash %}
python --version
{% endhighlight %}

{% highlight bash %}
 ~  python --version                                                                         
Python 2.7.16
{% endhighlight %}

No, not yet, we still need to enable it, do so for this session by running:

{% highlight bash %}
~  pyenv local 3.10.0
#now test
~  python --version                                                                         
Python 3.10.0
{% endhighlight %}

Great!  Now how do we set that as the "global" default?

{% highlight bash %}
~  pyenv global 3.10.0
#now test
~  python --version                                                                         
Python 3.10.0
{% endhighlight %}

Open a new terminal session and you'll see that it's still 3.10.0

## Additional Python versions

For this session we actually want Python 3.9.6, to accomplish this install 3.9.6 like so:

{% highlight bash %}
pyenv install 3.9.6
{% endhighlight %}

Then:

{% highlight bash %}
~  pyenv local 3.9.6
#now test
~  python --version                                                                         
Python 3.9.6
{% endhighlight %}

Perfect!  Nice and clean and ready for development!