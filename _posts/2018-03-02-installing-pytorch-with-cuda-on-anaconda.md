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
conda update mkl # Update mkl package in root env to prevent this error later on: https://github.com/pytorch/pytorch/issues/6068#issuecomment-377226963
conda create -n pytorch python=3.6
{% endhighlight %}

For our non-standard installation of cuDNN, we need to tell pytorch where to
look for `libcudart` via the environment variable `$LD_LIBRARY_PATH`. Let's
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
export LD_LIBRARY_PATH=$CUDA_HOME/lib64
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

# Install optional dependencies: add LAPACK support for the GPU
conda install -c pytorch magma-cuda90 # or magma-cuda80 if CUDA 8
{% endhighlight %}

Let's clone the pytorch repo into our home directory:
{% highlight bash %}
cd ~ && git clone --recursive git@github.com:pytorch/pytorch.git && cd pytorch
{% endhighlight %}

We are now ready to install pytorch via the very convenient installer in the
repo:
{% highlight bash %}
CUDNN_LIB_DIR=$CUDA_HOME/lib64/ \
CUDNN_INCLUDE=$CUDA_HOME/include/ \
MAX_JOBS=25 \ # Please use: "cat /proc/cpuinfo | grep processor | wc -l" + 1
python setup.py install
{% endhighlight %}

We are now ready to test if pytorch has been installed correctly with CUDA

{% highlight bash %}
## For a quick test:
cd $HOME
# Check if Pytorch was installed
python -c 'import torch' 2>/dev/null && echo "Success" || echo "Failure"
#=> Success

# To check if Caffe2 GPU build was successful
python -c 'import torch; print(torch.cuda.is_available())'
#=> True

## For a comprehensive test:
cd $HOME/pytorch
bash test/run_test.sh
{% endhighlight %}

If you are also installing [torchvision][torchvision]:
{% highlight bash %}
cd $HOME/pytorch
git clone https://github.com/pytorch/vision
cd vision
python setup.py install
{% endhighlight %}

[pytorch]: http://pytorch.org/
[anaconda-guide]: {% post_url 2018-02-18-getting-up-to-speed-with-anaconda %}
[cudnn-guide]: {% post_url 2018-02-17-getting-cudnn %}
[torchvision]: https://github.com/pytorch/vision
