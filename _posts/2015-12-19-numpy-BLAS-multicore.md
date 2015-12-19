---
layout: post
title: How to make numpy work on several CPUs
description: "Make your numpy faster"
modified: 2015-12-19
---

Almost everybody now uses *numpy* as it is extremely helpful for data analysis.
However, oftentimes (if not almost always) numpy does not deliver at its full strength since it is installed in a very inefficient way - 
when it is linked with old-fashioned ATLAS and BLAS libraries which can use only 1 CPU core.

You might easily check if it is the case for you. To do this just create a simple test program:
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
Pay attention to *%CPU* column of that process: a value around 100 means that it is actually using only 1 CPU core.

If this is the case, you might want to significantly improve numpy performance. And, fortunately. this is very easy.


# Install OpenBLAS library
OpenBLAS is a very good library with various algorithms and functions of linear algebra which lies in the core of many modern data analysis methods.

However, to begin with, you need a fortran compiler, so install *gfortran* package, as *g77* compiler that you most probably have is incompatible with OpenBLAS.
{% highlight sh %}
apt-get install gfortran
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
make install
{% endhighlight %}
The default installation directory is */opt/OpenBLAS*. You may choose a different location, though:
{% highlight sh %}
make install PREFIX=/your/preferred/location
{% endhighlight %}


# Build the *right* numpy
First of all, get rid of the wrong numpy you already have
{% highlight sh %}
pip uninstall numpy
{% endhighlight %}

Then download numpy sources
{% highlight sh %}
pip install -d . numpy
{% endhighlight %}
This will create in the current directory a file with a name like *numpy-1.10.2.tar.gz*. 
Unzip it and enter the source directory.
{% highlight sh %}
tar xzf numpy-1.10.2.tar.gz
cd numpy-1.10.2
{% endhighlight %}

Create here a file *site.cfg* with the following content
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

Now build and install numpy
{% highlight sh %}
python setup.py build
python setup.py install
{% endhighlight %}

If there were no errors during compilation and installation and everything went just fine, 
run the test again to make sure that all CPU cores are now being used.
