---
layout: post
title: How to make numpy use several CPUs
description: "Make your numpy faster"
tags: [python, numpy, BLAS, linux, multicore]
date: 2015-12-19
image:
  feature: abstract-10.jpg
---

Almost everybody now uses *numpy* as it is extremely helpful for data analysis.
However, oftentimes (if not almost always) numpy does not deliver at its full strength since it is installed in a very inefficient way - 
when it is linked with old-fashioned ATLAS and BLAS libraries which can use only 1 CPU core even when your computer is equipped with a 
multicore processor or even a few processors.

*This post is intended for Linux users only. Those from Windows world might need additiondal googling. 
The other option is to switch to scientific Python bundles like Anaconda (my choice), Canopy or some other.*

You might easily check if you are affected by the single core numpy problem. To do this just create a simple test program:
{% highlight python %}
import numpy as np
size = 10000
a = np.random.random_sample((size, size))
b = np.random.random_sample((size, size))
n = np.dot(a,b)
{% endhighlight %}

And run it as a background process
{% highlight sh %}
python test.py &
{% endhighlight %}

Afterwards run *top* to check the performance of your computer:
{% highlight sh %}
top
{% endhighlight %}
You will see a *python* process at the very top of your process list.
Now pay attention to the *%CPU* column of that process: a value around 100 means that it is actually using only 1 CPU core.

If this is the case, you might want to significantly improve numpy's performance. Fortunately. this is very easy.

# Check libraries
There are two quite different situations:

- you have some ATLAS/BLAS libraries already installed
- you don't have any libraries yet
To find out what you have check if your numpy is linked to BLAS
{% highlight sh %}
cd /usr/local/lib/python2.7/dist-packages/numpy/core
ldd multiarray.so
{% endhighlight %}
In earlier numpy versions (before 1.10) you have to check linkage of *_dotblas.so* (instead of *multiarray.so*), so you should do:
{% highlight sh %}
cd /usr/local/lib/python2.7/dist-packages/numpy/core
ldd _dotblas.so
{% endhighlight %}

The output will look something like:
{% highlight sh %}
        linux-vdso.so.1 =>  (0x00007fffe58a2000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f8adbff4000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f8adbdd6000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f8adba10000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f8adc68c000)
{% endhighlight %}

If there is no mention of *libblas.so* (like in the output above), then you have situation #2 - you don't have any BLAS library installed.
This means that your numpy uses its own internal library of linear algebra functions which is extremely slow. 
So you will get the greatest performance improvement, but at the expense of recompiling and reinstalling numpy.

If *libblas.so* is mentioned in your ldd output, you may just reassign a BLAS library which is fast and easy.

In any case, you need a better BLAS library first.

# Install OpenBLAS library
OpenBLAS is a very good library with various algorithms and functions of linear algebra which lies in the core of many modern data analysis methods.

However, to begin with, you need a fortran compiler, so install *gfortran* package, as *g77* compiler that you most probably have is incompatible with OpenBLAS.
{% highlight sh %}
sudo apt-get install gfortran
{% endhighlight %}

Now download OpenBLAS sources from Github
{% highlight sh %}
git clone https://github.com/xianyi/OpenBLAS.git
{% endhighlight %}

Enter the directory and compile sources
{% highlight sh %}
cd OpenBLAS
make FC=gfortran
{% endhighlight %}

When make has successfully finished, install the library
{% highlight sh %}
sudo make install
{% endhighlight %}
The default installation directory is */opt/OpenBLAS*. You may choose a different location, though:
{% highlight sh %}
sudo make install PREFIX=/your/preferred/location
{% endhighlight %}


# Reassign BLAS
If you had another BLAS library at the beginning, you need to make OpenBLAS library the preferred choice of all BLAS libraries installed.
{% highlight sh %}
sudo update-alternatives --install /usr/lib/libblas.so.3 libblas.so.3 \
	/opt/OpenBLAS/lib/libopenblas.so 50
{% endhighlight %}
Now run the test again and make sure that all CPU cores are now being used.
This is all you have to do. Now enjoy the full speed numpy.

# Build the *right* numpy
Those who did not have any BLAS libraries have to reinstall numpy.

First of all, get rid of the wrong numpy you already have.
{% highlight sh %}
sudo pip uninstall numpy
{% endhighlight %}

Then create a file *.numpy-site.cfg* in your home directory with the following content:
{% highlight sh %}
[default]
include_dirs = /opt/OpenBLAS/include
library_dirs = /opt/OpenBLAS/lib

[openblas]
openblas_libs = openblas
include_dirs = /opt/OpenBLAS/include
library_dirs = /opt/OpenBLAS/lib

[lapack]
lapack_libs = openblas

[atlas]
atlas_libs = openblas
libraries = openblas
{% endhighlight %}
If you have chosen a different location for your OpenBLAS installation, edit the paths accordingly.

And install numpy again
{% highlight sh %}
sudo pip install numpy
{% endhighlight %}
If there were no errors during compilation and installation and everything went just fine, 
run the test again to make sure that all CPU cores are now being used.

If you prefer a manual compilation/installation, like I often do, you may try the following approach.
First, download numpy sources.
{% highlight sh %}
pip install -d . numpy
{% endhighlight %}
This will create in the current directory a file named like *numpy-1.10.2.tar.gz* (the version will definitely change in future). 
Unzip it and enter the source directory.
{% highlight sh %}
tar xzf numpy-1.10.2.tar.gz
cd numpy-1.10.2
{% endhighlight %}

Now create *site.cfg* file (notice that the name is a bit different here) with the very same content as *.numpy-site.cfg* above.

And finally build and install numpy
{% highlight sh %}
python setup.py build
sudo python setup.py install
{% endhighlight %}
