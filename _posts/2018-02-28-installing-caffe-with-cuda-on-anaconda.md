---
layout: single
title:  "Installing Caffe with CUDA on Anaconda"
date:   2018-02-28 16:10:00 +0800
categories: Guides
tags: caffe CUDA cuDNN anaconda
---
The following guide shows you how to install install [Caffe][caffe] with CUDA
under the Anaconda 2 virtual environment.

# Assumptions
* Ubuntu OS
* NVIDIA GPU with CUDA support
* Anaconda 2 (see installation instructions [here][anaconda-guide])
* CUDA (*installed by system admin*)

# Specifications
This guide is written for the following specs:
* Ubuntu 16.04
* Python 2.7
* CUDA 8
* cuDNN v7.1
* Anaconda 2
* OpenCV3

# Guide
First, get cuDNN by following this [cuDNN Guide][cudnn-guide].

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
conda install lmdb openblas glog gflags hdf5 protobuf leveldb boost opencv cmake numpy=1.15 -y
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
export ENV_PATH=$HOME/anaconda2/envs/caffe
cmake -DBLAS=open -DCUDNN_INCLUDE=$CUDA_HOME/include/ -DCUDNN_LIBRARY=$CUDA_HOME/lib64/libcudnn.so -DCMAKE_PREFIX_PATH=$ENV_PATH -DCMAKE_INSTALL_PREFIX=$ENV_PATH -DCMAKE_CXX_FLAGS="-std=c++11" ..
{% endhighlight %}
* CMake variable `BLAS=open` indicates that we would like use OpenBLAS instead of the default
which is ATLAS
* CMake variable `CUDNN_INCLUDE` indicates where to find the `include` directory for your
cuDNN
* CMake variable `CUDNN_LIBRARY` indicates where to find the library path for your cuDNN
* CMake variable  `CMAKE_PREFIX_PATH` tells CMake to look for packages in your conda environment
before looking in system install locations (like `/usr/local`)
* CMake variable  `CMAKE_INSTALL_PREFIX` indicates where to install Caffe binaries
* CMake variable  `DCMAKE_CXX_FLAGS` indicates which C++ compiler version to use
* Also see the [list of the available make flags][cmake-flags] and their default
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
make pycaffe
make all -j 25
make install
make runtest
{% endhighlight %}

After make is completed, we are now finally ready to install
{% highlight bash %}
make install
{% endhighlight %}

You'd think we're done, but not quite! For some reason python will not know
where to look for the caffe python modules. I'm not sure why is this and if you
do know how to fix it in the installation process, please let me know! We have
to point the `$PYTHONPATH` environment variable to our caffe folder like so
{% highlight bash %}
export PYTHONPATH=$HOME/caffe/python:$PYTHONPATH
{% endhighlight %}
However it will be tedious to type that everytime we activate our environment.
You may append that line to `.bash_profile` or `.bashrc` but some variables
such as `$PYTHONPATH` are potentially used in many environments and it could
lead to python import errors when the paths contain different modules sharing
the same name. For instance, both caffe and caffe2 contain a module named
'caffe'.

The solution to overcome this is to write a script to save our environment
variables within our environment so that they get loaded automatically every
time we activate our environment and get unset automatically when we deactivate
our environment. The following steps are an adaptation of this
[guide](https://conda.io/docs/user-guide/tasks/manage-environments.html#macos-and-linux)
stated in the official Anaconda documentation.

Let's enter our environment directory and do the following
{% highlight bash %}
cd $CONDA_PREFIX
mkdir -p ./etc/conda/activate.d
mkdir -p ./etc/conda/deactivate.d
touch ./etc/conda/activate.d/env_vars.sh
touch ./etc/conda/deactivate.d/env_vars.sh
{% endhighlight %}

Edit `./etc/conda/activate.d/env_vars.sh` as follows:
{% highlight bash %}
#!/bin/sh

export PYTHONPATH=$HOME/caffe/python:$PYTHONPATH
{% endhighlight %}

Edit `./etc/conda/deactivate.d/env_vars.sh` as follows:
{% highlight bash %}
#!/bin/sh

unset PYTHONPATH
{% endhighlight %}

Now let's reload the current environment to reflect the variables
{% highlight bash %}
source activate caffe
{% endhighlight %}

We are now ready to test if caffe has been installed correctly with CUDA
{% highlight bash %}
cd ~
# To check if Caffe build was successful
python -c 'import caffe; caffe.set_mode_gpu()' 2>/dev/null && echo "Success" || echo "Failure"
#=> Success
{% endhighlight %}

[caffe]: http://caffe.berkeleyvision.org
[anaconda-guide]: {% post_url 2018-02-18-getting-up-to-speed-with-anaconda %}
[cudnn-guide]: {% post_url 2018-02-17-getting-cudnn %}
[cmake-flags]: https://github.com/BVLC/caffe/pull/1667
