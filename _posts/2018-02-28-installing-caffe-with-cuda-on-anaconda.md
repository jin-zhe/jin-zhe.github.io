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

# Specifications
This guide is written for the following specs:
* Ubuntu 16.04
* Python 2.7
* CUDA 8
* Anaconda 2
* OpenCV3

# Guide
Find out your CUDA version by running the following command
{% highlight bash %}
cat /usr/local/cuda/version.txt
#=> CUDA Version 8.0.61
{% endhighlight %}

Download the NVIDIA cuDNN [here][download-cuDNN] corresponding to your CUDA
version. You'll be requried to sign up for a free developer account if you have
not already done so. Be sure to download via the link that reads something like
`cuDNN v7.0.5 Library for Linux`. Do not download the `.deb` version as you'll
need admin privilege to install that. The file I downloaded was named
`cudnn-8.0-linux-x64-v7.tgz` and we are going to extract its contents into the
home directory
{% highlight bash %}
# cd your_download_directory
tar -zxvf cudnn-8.0-linux-x64-v7.tgz -C ~/
{% endhighlight %}
This should create a `~/cuda` directory.

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
conda install lmdb openblas glog gflags hdf5 protobuf leveldb boost opencv cmake -y
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
cmake -DBLAS=open -DCUDNN_INCLUDE=~/cuda/include/ -DCUDNN_LIBRARY=~/cuda/lib64/libcudnn.so -DCMAKE_PREFIX_PATH=~/anaconda2/envs/caffe -DCMAKE_INSTALL_PREFIX=~/anaconda2/envs/caffe ..
{% endhighlight %}
* CMake variable `BLAS=open` indicates that we would like use OpenBLAS instead of the default
which is ATLAS
* CMake variable `CUDNN_INCLUDE` indicates where to find the `include` directory for your
cuDNN
* CMake variable `CUDNN_LIBRARY` indicates where to find the library path for your cuDNN
* CMake variable  `CMAKE_PREFIX_PATH` tells CMake to look for packages in your conda environment
before looking in system install locations (like `/usr/local`)
* CMake variable  `CMAKE_INSTALL_PREFIX` indicates where to install Caffe binaries
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
make all -j 25
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
variables within our environemnt so that they get loaded automatically every
time we activate our environment and get unset automatically when we deactivate
our environment. The following steps are an adaptation of this
[guide](https://conda.io/docs/user-guide/tasks/manage-environments.html#macos-and-linux)
stated in the official Anaconda documentation.

Let's enter our environment directory and do the following
{% highlight bash %}
cd ~/anaconda2/envs/caffe
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
[download-cuDNN]: https://developer.nvidia.com/cudnn
[cmake-flags]: https://github.com/BVLC/caffe/pull/1667