---
layout: single
title:  "Installing Caffe with CUDA on Anaconda"
date:   2018-02-28 16:10:00 +0800
categories: caffe CUDA cuDNN
---
The following guide shows you how to install install [Caffe][caffe] with CUDA
under the Anaconda 2 virtual environment.

# Assumptions
* Ubuntu OS
* NVIDIA GPU with CUDA support
* Anaconda 2 (see installation instructions [here][anaconda-guide])
* CUDA (*installed by system admin*)
Find out your CUDA version by running the following command
{% highlight bash %}
cat /usr/local/cuda/version.txt
#=> CUDA Version 8.0.61
{% endhighlight %}

# Specifications
This guide is written for the following specs:
* Ubuntu 16.04
* Python 2.7
* CUDA 8
* Anaconda 2
* OpenCV3

# Guide
Let's create a virtual Anaconda environment called "caffe".
{% highlight bash %}
conda create -n caffe python=2.7 anaconda
{% endhighlight %}

After it prepares the environment and installs the defaults, activate the
virtual environment via
{% highlight bash %}
source activate caffe
{% endhighlight %}

Now let's install the necessary dependencies in our current caffe environment:
{% highlight bash %}
conda install openblas glog gflags hdf5 protobuf leveldb cmake -y
conda install -c menpo opencv3 -y
{% endhighlight %}

Let's clone the caffe's repo into our home directory.
{% highlight bash %}
cd ~ && git clone --recursive https://github.com/BVLC/caffe && cd caffe
{% endhighlight %}

We shall avoid polluting the caffe source tree by building within build a folder
{% highlight bash %}
mkdir build && cd build
{% endhighlight %}

We shall now build the package using CMake with the following flags
{% highlight bash %}
cmake -DBLAS=open ..
{% endhighlight %}
* `-DBLAS=open` indicates that we would like use OpenBLAS instead of the default
which is ATLAS
* See the [list of the available make flags][cmake-flags] and their default
values

Let's find out how many cores your machine has
{% highlight bash %}
cat /proc/cpuinfo | grep processor | wc -l
#=> n
{% endhighlight %}

Let's `make` the package efficiently by maximising the number of jobs for it.
General rule of thumb is to use 1 + n number of jobs where n is the output from
the previous command. i.e. number of cores. Mine was 24 so I run the following
{% highlight bash %}
make all -j 25
{% endhighlight %}

After make is completed, we are now finally ready to install
{% highlight bash %}
make install
{% endhighlight %}

We are now ready to test if caffe has installed correctly
{% highlight bash %}
cd ~
# To check if Caffe2 build was successful
python -c 'import caffe' 2>/dev/null && echo "Success" || echo "Failure"
#=> Success
{% endhighlight %}

[caffe]: http://caffe.berkeleyvision.org
[anaconda-guide]: {% post_url 2018-02-19-getting-up-to-speed-with-anaconda %}
[cmake-flags]: https://github.com/BVLC/caffe/pull/1667