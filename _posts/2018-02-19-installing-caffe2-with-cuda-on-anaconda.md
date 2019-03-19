---
layout: single
title:  "Installing Caffe2 with CUDA on Anaconda"
date:   2018-02-19 16:10:00 +0800
categories: Guides
tags: caffe2 CUDA cuDNN anaconda
---
{% assign env_name = "caffe2" %}
{% assign repo_name = "pytorch" %}
{% capture pythonpath %}$HOME/{{ repo_name }}/build{% endcapture %}

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

{% include_relative _includes/_conda_create_env.md env_name=env_name python_ver="2.7" channel="anaconda" %}

{% include_relative _includes/_conda_install.md env_name=env_name installs="future gflags glog lmdb mkl mkl-include numpy opencv protobuf snappy six cmake" %}

{% include_relative _includes/_git_clone.md env_name=env_name url="https://github.com/pytorch/pytorch.git" repo_name=repo_name %}

{% include_relative _includes/_mkdir_build.md repo_name=repo_name %}

We shall now build the package using CMake with the following flags
{% highlight bash %}
cmake -DCUDNN_INCLUDE_DIR=~/cuda/include -DCUDNN_LIBRARY=~/cuda/lib64/libcudnn.so -DCMAKE_PREFIX_PATH=$CONDA_PREFIX -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX ..
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
#=> 1
{% endhighlight %}

[caffe2]: https://github.com/caffe2/caffe2
[anaconda-guide]: {% post_url 2018-02-18-getting-up-to-speed-with-anaconda %}
[cudnn-guide]: {% post_url 2018-02-17-getting-cudnn %}
