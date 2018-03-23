---
layout: single
title:  "Getting CUDA with cuDNN"
date:   2018-02-18 16:10:00 +0800
categories: Guides
tags: cuDNN CUDA NVIDIA
---
In this guide I will show you how to install CUDA with cuDNN in your home
directory without invoking admin privileges. Unless stated otherwise, we
will be using this configuration for all of my guides.

# Assumptions
* Ubuntu OS 16.04
* NVIDIA GPU with driver installed

# What is
From NVIDIA:  
*CUDA® is a parallel computing platform and programming model developed by
NVIDIA for general computing on graphical processing units (GPUs). With CUDA,
developers are able to dramatically speed up computing applications by
harnessing the power of GPUs.*

*The NVIDIA CUDA® Deep Neural Network library (cuDNN) is a GPU-accelerated
library of primitives for deep neural networks. cuDNN provides highly tuned
implementations for standard routines such as forward and backward convolution,
pooling, normalization, and activation layers.*

In essence, if you are working on deep learning tools and require
GPU-enabled processes, you will need CUDA along with cuDNN.

# Why
The reason for a non-standard installation is two-fold:
1. You may not have admin privileges to install CUDA and cuDNN to system paths
in the conventional manner
2. Even if your sysadmin have already installed CUDA or cuDNN on your machine,
it may still be a good idea to install them on your own because repeating
research experiments by others sometimes require specific versions of CUDA and
cuDNN. Non-standard installations can thus grant you the ability to toggle
between various versions under different environments easily using a
virtualization manager such as [Anaconda][anaconda]

# Getting CUDA
The standard installation path for cuda is the versioned path
`/usr/local/cuda-x.x` where `x.x` reflects the version number. The standard
installation will also create a symlink `/usr/local/cuda` to that versioned
install path. You may check if a sysadmin had already installed cuda in your
machine via the command:
{% highlight bash %}
cat /usr/local/cuda/version.txt
#=> CUDA Version 8.0.61
{% endhighlight %}

Let's heave over to [NVIDIA's CUDA download page][download-cuda]
then select your system configurations and download the **runfile (local)** for
the CUDA version that you want. For this guide, I shall choose version 9.0
as the example.

Let's make the versioned cuda install directory under `$HOME` following
NVIDIA's convention of displaying the version in the dirname:
{% highlight bash %}
cd ~
mkdir cuda-9.0
{% endhighlight %}

I downloaded the runfile named `cuda_9.0.176_384.81_linux-run` into my
`~/Downloads` directory. Let's head over and install CUDA into the
versioned install directory:
{% highlight bash %}
cd ~/Downloads
sh cuda_9.0.176_384.81_linux-run --silent --toolkit --toolkitpath=~/cuda-9.0
{% endhighlight %}

Once the installation is done, we have to add the following environment
variables:
{% highlight bash %}
export CUDA_HOME=$HOME/cuda-9.0
export PATH=$CUDA_HOME/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
# For 32-bit OS: export LD_LIBRARY_PATH=/usr/local/cuda-9.1/lib${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
{% endhighlight %}
You may append them to `~/.bashrc` if you're running on your local machine,
`~/.bash_profile` if you're accessing a remote machine via ssh, or better
yet [save the environment varialbes][manage-environments] under a Conda environment!

We are done for CUDA!

# Getting cuDNN
Installing cuDNN is relatively hassle-free as it's simply about unpacking
cuDNN files into our `$CUDA_HOME` directory.

Download the NVIDIA cuDNN [here][download-cuDNN] corresponding to your CUDA
version. You'll be requried to sign up for a free developer account if you have
not already done so. For this guide I shall choose cuDNN version 7.0.4 as the
example.

On the download page click on the link that reads
"*Download cuDNN v7.0.4 (Nov 13, 2017), for CUDA 9.0*". It should activate an
accordion style dropdown displaying a couple of download links. We shall choose
the **Library for Linux** because the other modes of installation requires
admin privileges.

Lets download `cudnn-9.0-linux-x64-v7.tgz` into our `~/Downloads` directory
by clicking on the download link that reads "cuDNN v7.0.4 Library for Linux"

Once the download is complete, we shall head over to the downloads directory
and simply unpack the cuDNN files into our CUDA directory:
{% highlight bash %}
cd ~/Downloads
tar -zxvf cudnn-9.0-linux-x64-v7.tgz -C $CUDA_HOME --strip-components=1
{% endhighlight %}

That's it! We are done!

[anaconda]: https://anaconda.org/
[download-cuda]: https://developer.nvidia.com/cuda-downloads
[manage-environments]: https://conda.io/docs/user-guide/tasks/manage-environments.html#saving-environment-variables
[download-cuDNN]: https://developer.nvidia.com/cudnn
