---
layout: single
title:  "Installing Caffe2 with CUDA on Anaconda"
date:   2018-02-19 16:10:00 +0800
categories: Guides
tags: caffe2 CUDA cuDNN anaconda
---

The following guide shows you how to install install [Caffe2][caffe2] with CUDA
under the Anaconda 2 virtual environment without needing to use `sudo`. This
guide is meant for machines running on Ubuntu 16.04 equipped with NVIDIA GPUs
with CUDA support. i.e it assumes CUDA is already installed by a system admin.
As caffe2 is best supported on Python 2 at the time of this writing, this
installation guide is written for Python 2.7.

# Assumptions
* Ubuntu OS
* NVIDIA GPU
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

Let's create a virtual Anaconda environment called "caffe2".
{% highlight bash %}
conda create -n caffe2 python=2.7 anaconda
{% endhighlight %}
*You many of course use a different environment name, just be sure to adjust
accordingly for the rest of this guide*

After it prepares the environment and installs the defaults, activate the
virtual environment via
{% highlight bash %}
source activate caffe2
# to deactivate: source deactivate caffe2
{% endhighlight %}

Now let's install the necessary dependencies in our current caffe2 environment:
{% highlight bash %}
conda install -y future gflags glog lmdb mkl mkl-include numpy opencv protobuf snappy six cmake future
{% endhighlight %}

Let's clone the caffe2's repo and its submodules into our home directory.
{% highlight bash %}
cd ~
git clone --recursive https://github.com/pytorch/pytorch.git && cd pytorch
git submodule update --init --recursive
{% endhighlight %}

We shall avoid polluting the PyTorch source tree by building within a build
folder (yes within PyTorch root directory, not within Caffe2 directory!)
{% highlight bash %}
mkdir build && cd build
{% endhighlight %}

We shall now build the package using CMake with the following flags
{% highlight bash %}
cmake -DCUDNN_INCLUDE_DIR=~/cuda/include -DCUDNN_LIBRARY=~/cuda/lib64/libcudnn.so -DCMAKE_PREFIX_PATH=~/anaconda2/envs/caffe2 -DCMAKE_INSTALL_PREFIX=~/anaconda2/envs/caffe2 ..
{% endhighlight %}
* CMake variable  `CUDNN_INCLUDE_DIR` indicates where to find the `include` directory for your
cuDNN
* CMake variable  `CUDNN_LIBRARY` indicates where to find the `libcudnn.so` for your cuDNN
* CMake variable  `CMAKE_PREFIX_PATH` tells CMake to look for packages in your conda environment
before looking in system install locations (like `/usr/local`)
* CMake variable  `CMAKE_INSTALL_PREFIX` indicates where to install Caffe2 binaries such as
`libcaffe2.dylib` after Caffe2 has been successfully built, the default is
`/usr/local` which will require administrator privilege

Let's find out how many cores your machine has
{% highlight bash %}
cat /proc/cpuinfo | grep processor | wc -l
#=> n
{% endhighlight %}

Let's `make` the package efficiently by maximising the number of jobs for it.
General rule of thumb is to use 1 + n number of jobs where n is the output from
the previous command. i.e. number of cores. Mine was 24 so I run the following
{% highlight bash %}
make -j 25
{% endhighlight %}

After make is completed, we are now finally ready to install
{% highlight bash %}
make install
{% endhighlight %}

You'd think we're done, but not quite! For some reason python will not know
where to look for the caffe2 python modules. I'm not sure why is this and if you
do know how to fix it in the installation process, please let me know! We have
to point the `$PYTHONPATH` environment variable to our caffe2 build folder
{% highlight bash %}
export PYTHONPATH=$HOME/caffe2/build:$PYTHONPATH
{% endhighlight %}
However it will be tedious to type that everytime we activate our environment.
You may append that line to `.bash_profile` or `.bashrc` but some variables
such as `$PYTHONPATH` are potentially used in many environments and it could
lead to python import errors when the paths contain different modules sharing
the same name. For instance, both caffe and caffe2 contain a module named
'caffe'.

The solution to overcome this is to write a script to save our environment
variables within our environemnt so that they get loaded automatically every
time we activate our environment and get unset automatically when we deactivate
our environment. The following steps are an adaptation of this
[guide](https://conda.io/docs/user-guide/tasks/manage-environments.html#macos-and-linux)
stated in the official Anaconda documentation.

Let's enter our environment directory and do the following
{% highlight bash %}
cd ~/anaconda2/envs/caffe2
mkdir -p ./etc/conda/activate.d
mkdir -p ./etc/conda/deactivate.d
touch ./etc/conda/activate.d/env_vars.sh
touch ./etc/conda/deactivate.d/env_vars.sh
{% endhighlight %}

Edit `./etc/conda/activate.d/env_vars.sh` as follows:
{% highlight bash %}
#!/bin/sh

export PYTHONPATH=$HOME/caffe2/build:$PYTHONPATH
{% endhighlight %}

Edit `./etc/conda/deactivate.d/env_vars.sh` as follows:
{% highlight bash %}
#!/bin/sh

unset PYTHONPATH
{% endhighlight %}

Now let's reload the current environment to reflect the variables
{% highlight bash %}
source activate caffe2
{% endhighlight %}

We are now ready to test if caffe2 has installed correctly
{% highlight bash %}
cd ~
# To check if Caffe2 build was successful
python -c 'from caffe2.python import core' 2>/dev/null && echo "Success" || echo "Failure"
#=> Success

# To check if Caffe2 GPU build was successful
# This must print a number > 0 in order to use Detectron
python2 -c 'from caffe2.python import workspace; print(workspace.NumCudaDevices())'
#=> 1
{% endhighlight %}

[caffe2]: https://github.com/caffe2/caffe2
[anaconda-guide]: {% post_url 2018-02-18-getting-up-to-speed-with-anaconda %}
[cudnn-guide]: {% post_url 2018-02-17-getting-cudnn %}
