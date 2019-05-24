---
layout: single
title:  "Installing Caffe with CUDA in Conda"
date:   2018-02-28 16:10:00 +0800
categories: Guides
tags: Anaconda Caffe Conda CUDA cuDNN Miniconda
---
{% assign env_name= "caffe" %}
{% capture pythonpath %}$HOME/{{ env_name }}/python{% endcapture %}

The following guide shows you how to install install [Caffe][caffe] with CUDA
under the Conda virtual environment.

## Assumptions
* Ubuntu OS
* NVIDIA GPU with CUDA support
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
{% include_relative _includes/_conda_install.md packages="lmdb openblas glog gflags hdf5 protobuf leveldb boost opencv cmake numpy=1.15" %}
{% include_relative _includes/_conda_install.md packages="doxygen" channel="conda-forge" %}
{% endhighlight %}

{% include_relative _includes/_git_clone.md env_name=env_name url="https://github.com/BVLC/caffe.git" repo_name=env_name %}

{% include_relative _includes/_mkdir_build.md repo_name=env_name %}

We shall now build the package using CMake with the following flags
{% highlight bash %}
cmake -DBLAS=open -DCUDNN_INCLUDE=$CUDA_HOME/include/ -DCUDNN_LIBRARY=$CUDA_HOME/lib64/libcudnn.so -DCMAKE_PREFIX_PATH=$CONDA_PREFIX -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX -DCMAKE_CXX_FLAGS="-std=c++11" ..
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
* CMake variable  `CPU` indicates whether or not to use CPU-only installation
* Also see the [list of the available make flags][cmake-flags] and their default
values

{% include_relative _includes/_make.md all="all" makes="pycaffe" %}

Now we run test
{% highlight bash %}
make runtest
{% endhighlight %}

{% include_relative _includes/_pythonpath.md pythonpath=pythonpath %}

{% include_relative _includes/_conda_env_vars.md pythonpath=pythonpath env_name=env_name %}

We are now ready to test if caffe has been installed correctly with CUDA
{% highlight bash %}
cd ~
# To check if Caffe build was successful
python -c 'import caffe; caffe.set_mode_gpu()' 2>/dev/null && echo "Success" || echo "Failure"
#=> Success
{% endhighlight %}

[caffe]: http://caffe.berkeleyvision.org
[conda-guide]: {% post_url 2018-02-18-getting-up-to-speed-with-conda %}
[cudnn-guide]: {% post_url 2018-02-17-getting-cudnn %}
[cmake-flags]: https://github.com/BVLC/caffe/pull/1667
