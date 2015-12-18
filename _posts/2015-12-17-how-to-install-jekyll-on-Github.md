---
layout: post
title: How to setup a Jekyll blog on Github Pages
description: "The start"
modified: 2015-12-17
---

Today, *17 Dec 2015*, I have created this blog. It is powered by [Jekyll](https://jekyllrb.com/) with [Hpstr theme](https://github.com/mmistakes/hpstr-jekyll-theme)
and hosted on Github Pages.
This is a simple how-to procedure:

# Prerequisites
You need to have Ruby, Ruby Development Kit and Python already installed on your computer

# Create a local Jekyll installation
* Install [Jekyll](https://jekyllrb.com/) and a few useful gems
{% highlight sh %}
gem install jekyll
gem install jekyll-sitemap
gem install jekyll-paginate
gem install jekyll-gist
gem install pygments.rb
{% endhighlight %}
Note that *pygments.rb* requires python's *pygment* package, so you might need install it in the first place
{% highlight sh %}
pip install pygments
{% endhighlight %}

* Clone [Hpstr repo](https://github.com/mmistakes/hpstr-jekyll-theme)
{% highlight sh %}
git clone https://github.com/mmistakes/hpstr-jekyll-theme.git
{% endhighlight %}

* Rename a directory *hpstr-jekyll-theme* to whatever you like

* Build the site
{% highlight sh %}
jekyll build
{% endhighlight %}

* If you are using Windows you might encounter an SSL certificate verify error during the build process.
In order to fix it download [cacert.pem](http://curl.haxx.se/ca/cacert.pem) and put it a good place, e.g. in *%RUBY_DIR%\cert* directory
(most likely you don't have it, so create it) and then set a required env variable
{% highlight sh %}
set SSL_CERT_FILE=D:\Ruby\cert\cacert.pem
{% endhighlight %}
Use [control panel](http://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/environment_variables.mspx?mfr=true) to make this setting permanent.

* Now run build again and afterwards run *serve* to check it on your local machine
{% highlight sh %}
jekyll serve
{% endhighlight %}
This will run a web server and you may visit [http://localhost:4000] to check if everything is OK.

# Create a Github Pages site
* In your Github account create a repo for your Github Pages site. It must have a name *your-account-name*.github.com

* Connect your local repo to your Github repo
{% highlight sh %}
git remote rm origin
git remote add origin https://github.com/YOU/YOU.github.com.git
{% endhighlight %}
Instead of *YOU* use your account name.

* Commit and push your site to Github
{% highlight sh %}
git add -u
git commit -m "Initial commit"
git push -u origin master
{% endhighlight %}

* Check your site at https://*YOU*.github.io
