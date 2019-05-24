---
layout: single
title:  "Installing PyTorch with CUDA in Conda"
date:   2018-03-02 16:10:00 +0800
categories: Guides
tags: Anaconda Caffe2 CUDA cuDNN Miniconda PyTorch
---
{% assign env_name = "pytorch" %}
{% assign repo_name = "pytorch" %}
{% capture pythonpath %}$HOME/{{ repo_name }}/build{% endcapture %}

The following guide shows you how to install [PyTorch][pytorch] with CUDA
under the Conda virtual environment.

## Assumptions
* Ubuntu OS
* NVIDIA GPU with CUDA support
* Conda (see installation instructions [here][conda-guide])
* CUDA (*installed by system admin*)

## Specifications
This guide is written for the following specs:
* Ubuntu 16.04
* Python 3.6
* CUDA 9.0
* cuDNN v7.1
* Miniconda 3
* OpenCV3

## Guide
First, get cuDNN by following this [cuDNN Guide][cudnn-guide].

Then we need to update `mkl` package in base environment to prevent
[this issue][mkl-issue] later on.
{% highlight bash %}
conda update mkl
{% endhighlight %}

Let's create a virtual Conda environment called "pytorch":
{% include_relative _includes/_conda_create_env.md env_name=env_name python_ver="3" %}

Now let's install the necessary dependencies in our current PyTorch environment:
{% highlight bash %}
# Install basic dependencies
{% include_relative _includes/_conda_install.md packages="cffi cmake future gflags glog hypothesis lmdb mkl mkl-include numpy opencv protobuf pyyaml=3.12 setuptools scipy six snappy typing" %}

# Install LAPACK support for the GPU
{% include_relative _includes/_conda_install.md packages="magma-cuda90" channel="pytorch" %}
{% endhighlight %}
* We specified `pyyaml=3.12` because newer versions will be incompatible with
  [Detectron][detectron], should you use it with Caffe2.
  See [this issue][detectron-issue]
* For LAPACK support, install `magma-cudaxx` where xx reflects your cuda
  version, for e.g. 91 corresponds to cuda 9.1

{% include_relative _includes/_git_clone.md env_name=env_name url="git@github.com:pytorch/pytorch.git" repo_name=repo_name %}

Before we begin manually compiling the binaries, we need to first assign some
environment variables.

Firstly, for our non-standard installation of cuDNN, we
need to tell PyTorch where to look for `libcudart` via the environment variable
`$LD_LIBRARY_PATH`. If you have followed my [cuDNN Guide][cudnn-guide]
you would have assigned this to be:
{% highlight bash %}
export LD_LIBRARY_PATH=$CUDA_HOME/lib64
{% endhighlight %}

Next we need to tells CMake to look for packages in our Conda environment
before looking in system install locations:
{% highlight bash %}
export CMAKE_PREFIX_PATH=$CONDA_PREFIX
{% endhighlight %}

We are now ready to install pytorch via the very convenient installer in the
repo:
{% highlight bash %}
CUDNN_LIB_DIR=$CUDA_HOME/lib64/ \
CUDNN_INCLUDE=$CUDA_HOME/include/ \
MAX_JOBS=25 \ 
python setup.py install
{% endhighlight %}
* To determine assignment for MAX_JOBS, please use the number that is one more
  than the output from `cat /proc/cpuinfo | grep processor | wc -l`

We are now ready to test if PyTorch has been installed correctly with CUDA

To check if PyTorch was installed successfully:
{% highlight bash %}
# Basic test:
cd ~ && python -c 'from caffe2.python import core' 2>/dev/null && echo "Success" || echo "Failure"
#=> Success

# For a comprehensive test:
cd $HOME/pytorch
python test/run_test.py
{% endhighlight %}

To check if GPU build was successful:
{% highlight bash %}
# Check number of GPUs visible to PyTorch:
python -c 'import torch; print(torch.cuda.is_available())'
#=> 2

# See initial output from the following to ensure GPU is used:
cd $HOME/pytorch
python caffe2/python/operator_test/activation_ops_test.py
{% endhighlight %}

### Torchvision
If you are also installing [torchvision][torchvision]:
{% highlight bash %}
cd $HOME/pytorch
git clone https://github.com/pytorch/vision
cd vision
python setup.py install
{% endhighlight %}

[detectron]: https://github.com/facebookresearch/Detectron
[pytorch]: http://pytorch.org/
[torchvision]: https://github.com/pytorch/vision
[detectron-issue]: https://github.com/facebookresearch/DensePose/issues/216
[mkl-issue]: https://github.com/pytorch/pytorch/issues/6068#issuecomment-377226963
[conda-guide]: {% post_url 2018-02-18-getting-up-to-speed-with-conda %}
[cudnn-guide]: {% post_url 2018-02-17-getting-cudnn %}
