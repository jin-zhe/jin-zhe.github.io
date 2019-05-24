---
layout: single
title:  "Installing Caffe2 with CUDA in Conda"
date:   2018-02-19 16:10:00 +0800
categories: Guides
tags: Anaconda Caffe2 Conda CUDA cuDNN Miniconda
---
{% assign env_name = "caffe2" %}
{% assign repo_name = "caffe2" %}
{% capture pythonpath %}$HOME/{{ repo_name }}/build{% endcapture %}

# Deprecation warning
[Since May 2008, Caffe2 has been merged in PyTorch](caffe2-merge). To install
the lastest version of Caffe2, simply get [PyTorch][pytorch]. The
instructions for installing PyTorch can be accessed [here](pytorch-guide).

The following guide is kept here for posterity.

---

The following guide shows you how to install install [caffe2][caffe2] with
CUDA under Conda virtual environment. This guide is meant for machines running
on Ubuntu 16.04 equipped with NVIDIA GPUs with CUDA support. i.e it assumes CUDA
is already installed by a system admin.

## Assumptions
* Ubuntu OS
* NVIDIA GPU
* Conda (see installation instructions [here][conda-guide])
* CUDA (*installed by system admin*)

## Specifications
This guide is written for the following specs:
* Ubuntu 16.04
* Python 2.7
* CUDA 8
* cuDNN v7.1
* Miniconda 2
* OpenCV3

## Guide
First, get cuDNN by following this [cuDNN Guide][cudnn-guide].

{% include_relative _includes/_conda_create_env.md env_name=env_name python_ver="2.7" %}

Now let's install the necessary dependencies in our current {{ env_name }} environment:
{% highlight bash %}
{% include_relative _includes/_conda_install.md packages="future gflags glog lmdb mkl mkl-include numpy opencv protobuf snappy six cmake" %}
{% endhighlight %}

{% include_relative _includes/_git_clone.md env_name=env_name url="https://github.com/caffe2/caffe2.git" repo_name=repo_name %}

{% include_relative _includes/_mkdir_build.md repo_name=repo_name %}

We shall now build the package using CMake with the following flags
{% highlight bash %}
cmake -DCUDNN_INCLUDE_DIR=$CUDA_HOME/include -DCUDNN_LIBRARY=$CUDA_HOME/lib64/libcudnn.so -DCMAKE_PREFIX_PATH=$CONDA_PREFIX -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX ..
{% endhighlight %}
* CMake variable  `CUDNN_INCLUDE_DIR` indicates where to find the `include` directory for your
cuDNN
* CMake variable  `CUDNN_LIBRARY` indicates where to find the `libcudnn.so` for your cuDNN
* CMake variable  `CMAKE_PREFIX_PATH` tells CMake to look for packages in your conda environment
before looking in system install locations (like `/usr/local`)
* CMake variable  `CMAKE_INSTALL_PREFIX` indicates where to install Caffe2 binaries such as
`libcaffe2.dylib` after Caffe2 has been successfully built, the default is
`/usr/local` which will require administrator privilege
* CMake variable  `CPU` indicates whether or not to use CPU=only installation

{% include_relative _includes/_make.md %}

{% include_relative _includes/_pythonpath.md pythonpath=pythonpath %}

{% include_relative _includes/_conda_env_vars.md pythonpath=pythonpath env_name=env_name %}

We are now ready to test if caffe2 has installed correctly
{% highlight bash %}
cd ~
# To check if Caffe2 build was successful
python -c 'from caffe2.python import core' 2>/dev/null && echo "Success" || echo "Failure"
#=> Success

# To check if Caffe2 GPU build was successful
# This must print a number > 0 in order to use Detectron
python2 -c 'from caffe2.python import workspace; print(workspace.NumCudaDevices())'
#=> 2
{% endhighlight %}

[caffe2]: https://caffe2.ai
[pytorch]: https://github.com/pytorch/pytorch
[caffe2-merge]: https://caffe2.ai/blog/2018/05/02/Caffe2_PyTorch_1_0.html
[conda-guide]: {% post_url 2018-02-18-getting-up-to-speed-with-conda %}
[cudnn-guide]: {% post_url 2018-02-17-getting-cudnn %}
[pytorch-guide]: {% post_url 2018-03-02-installing-pytorch-with-cuda-in-conda %}
