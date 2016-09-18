---
layout: post
title: Preparing Windows Linux Subsystem for data scientists
description: "Your python programs are realy slow unless you use Numba. Part 2."
tags: [WSL, linux]
date: 2016-08-25
image:
  feature: abstract-11.jpg
comments: true
---

I do a lot of data science stuff: reading and writing large data files, running some complicated  and resource-consuming software (like Apache Spark), programming in several languages simultaneously (R, Python, JavaScript, Scala), quick prototyping in Jupyter notebooks. Some libraries I use do not work on Windows, so I need Linux as well.

Not surprisngly, soon after [Windows Subsystem for Linux (WSL)](https://msdn.microsoft.com/en-us/commandline/wsl/about) was announced I tried it as I got sick and tired of all that VM stuff. [The installion procedure](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide) was pretty straightforward. However, soon I bumped into problems with some packages and particularly Jupyter notebooks. So I had to remove WSL completely. Eventually I found out the best installation process which leads to a problem-free configuration where everything works smoothly.

By the way, I use the very same flow for ordinary Linux servers (with a couple of exceptions I will mention later).

# How to install and uninstall WSL
After enabling WSL you might need to reinstall it (for instance, when you get into a vicious circle of non-compatible packages or when something does not work and you don't know why). You can do that from a usual Windows command prompt with _lxrun.exe_

To remove WSL and all your Linux files, just type a simple command:

{% highlight sh %}
lxrun /uninstall /full
{% endhighlight %}
You will see a confirmation prompt (unless you use _/y_ option) and after a few minutes WSL will be uninstalled and all your files will be deleted.

After that you can re-install WSL:

{% highlight sh %}
lxrun /install /y
{% endhighlight %}

# Installing system packages
I usually start with installing compilators, git and some useful libraries.

{% highlight sh %}
sudo apt-get install build-essential gfortran
sudo apt-get install zlib1g-dev liblapack-dev libatlas-base-dev libopenblas-dev libhdf5-dev libedit-dev
sudo apt-get install git
{% endhighlight %}


Due to WSL peculiarities, some network applications do not work properly. Unfortunately, [Jupyter is one of them](https://github.com/Microsoft/BashOnWindows/issues/185). Fortunately, there is a fix.

{% highlight sh %}
sudo add-apt-repository ppa:aseering/wsl
sudo apt-get update
sudo apt-get install libzmq3 libzmq3-dev
{% endhighlight %}

As you might guess I also [use numba quite a lot](/tags/#numba), so I need LLVM. The problem is that LLVM is developing quite quickly and new releases often break API compatibility. Currently LLVM 3.7 is required for `numba`. But it is not available in the official repository for Ubuntu 14.04 which lies at the core of WSL. That is why we need some manual editing.

{% highlight sh %}
cd /etc/apt/sources.list
sudo cat > llvm.list << EOF
deb http://apt.llvm.org/trusty/ llvm-toolchain-trusty main
deb-src http://apt.llvm.org/trusty/ llvm-toolchain-trusty main
# 3.7
deb http://apt.llvm.org/trusty/ llvm-toolchain-trusty-3.7 main
deb-src http://apt.llvm.org/trusty/ llvm-toolchain-trusty-3.7 main
# 3.8 
deb http://apt.llvm.org/trusty/ llvm-toolchain-trusty-3.8 main
deb-src http://apt.llvm.org/trusty/ llvm-toolchain-trusty-3.8 main
# 3.9 
deb http://apt.llvm.org/trusty/ llvm-toolchain-trusty-3.9 main
deb-src http://apt.llvm.org/trusty/ llvm-toolchain-trusty-3.9 main
EOF
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 15CF4D18AF4F7421
sudo apt-get update
sudo apt-get install llvm-3.7
sudo ln -s /usr/bin/llvm-config-3.7 /usr/bin/llvm-config
{% endhighlight %}
What we do here is pretty simple:

- add additional sources for LLVM packages
- add a new key
- update info about available packages
- install LLVM 3.7
- create a symlink so that llvm-config is referencing the right version.

# Installing Python packages
First of all, we need `pip` and python development package with C headers and other stuff for building custom packages.

{% highlight sh %}
sudo apt-get install python-pip python-dev
{% endhighlight %}
The rest will be installed through `pip`, so I just list the packages:

- cython
- virtualenv
- numpy
- numexpr
- blosc
- scipy
- tables
- pandas
- sklearn
- statsmodels
- enum34
- llvmlite
- numba
- jupyter

And now you can start jupyter to check it:

{% highlight sh %}
jupyter notebook --no-browser
{% endhighlight %}
Connect to [http://localhost:8888](http://localhost:8888) (your settings might  differ), and create or open a notebook to run some code. If the kernel does not crash immediately, then everything is fine.

Take into account that this is the baseline installation. You can now install some other great software like `xgboost`, `tensorflow`, `mxnet`, `neon`, etc. I don't need these in some projects, that is why they are not a part of the baseline.

# To sum up
WSL is a great tool, despite some current constraints (like graphics and networking). Nevertheless, as a data scientists you can use WSL to your advantage and it will be of a great help especially if you need to live in both worlds (Windows and Linux).
