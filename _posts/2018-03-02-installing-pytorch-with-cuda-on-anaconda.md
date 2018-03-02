---
layout: single
title:  "Installing pytorch with CUDA on Anaconda"
date:   2018-03-02 16:10:00 +0800
categories: Guides
tags: pytorch CUDA cuDNN anaconda
---
The following guide shows you how to install [pytorch][pytorch] with CUDA
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

Let's create a virtual Anaconda environment called "pytorch":
{% highlight bash %}
conda create -n pytorch python=2.7 anaconda
{% endhighlight %}

For our non-standard installation of cuDNN, we need to tell pytorch where to
look for `libcudart.` via the environment variable `$LD_LIBRARY_PATH`. Let's
tell Anaconda to set that every time we activate 'pytorch' and unset that
everytime we deactivate the environment

Let's enter our environment directory and do the following:
{% highlight bash %}
cd ~/anaconda2/envs/pytorch
mkdir -p ./etc/conda/activate.d
mkdir -p ./etc/conda/deactivate.d
touch ./etc/conda/activate.d/env_vars.sh
touch ./etc/conda/deactivate.d/env_vars.sh
{% endhighlight %}

Edit `./etc/conda/activate.d/env_vars.sh` as follows:
{% highlight bash %}
#!/bin/sh

export LD_LIBRARY_PATH=$HOME/cuda/lib64
{% endhighlight %}

Edit `./etc/conda/deactivate.d/env_vars.sh` as follows:
{% highlight bash %}
#!/bin/sh

unset LD_LIBRARY_PATH
{% endhighlight %}

We are now ready to activate the virtual environment:
{% highlight bash %}
source activate pytorch
{% endhighlight %}

Now let's install the necessary dependencies in our current pytorch environment:
{% highlight bash %}
export CMAKE_PREFIX_PATH="$(dirname $(which conda))/../" # [anaconda root directory]

# Install basic dependencies
conda install git numpy pyyaml mkl setuptools cmake cffi typing

# Add LAPACK support for the GPU
conda install -c pytorch magma-cuda80 # or magma-cuda90 if CUDA 9
{% endhighlight %}

Let's clone the caffe's repo into our home directory:
{% highlight bash %}
cd ~ && git clone --recursive git@github.com:pytorch/pytorch.git && cd pytorch
{% endhighlight %}

We are now ready to install pytorch via the very convenient installer in the
repo:
{% highlight bash %}
export CUDNN_LIB_DIR=$HOME/cuda/lib64/
export CUDNN_INCLUDE_DIR=$HOME/cuda/include/
python setup.py install
{% endhighlight %}

We are now ready to test if pytorch has been installed correctly with CUDA
{% highlight bash %}
bash test/run_test.sh
{% endhighlight %}

[pytorch: http://pytorch.org/
[anaconda-guide]: {% post_url 2018-02-18-getting-up-to-speed-with-anaconda %}
[cudnn-guide]: {% post_url 2018-02-17-getting-cudnn %}