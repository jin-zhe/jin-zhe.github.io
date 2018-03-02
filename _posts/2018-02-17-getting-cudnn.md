---
layout: single
title:  "Getting cuDNN"
date:   2018-02-18 16:10:00 +0800
categories: Guides
tags: cuDNN CUDA NVIDIA
---
This may be a too short to justify a stand-alone post but since many of my
guides requires this bit of instruction, I felt it was necessary to abstract it
out separately.

In this guide I will show you how to get the right version of cuDNN and where to
place it. Unless stated otherwise, we will be using this configuration for all
of my guides.

The stadard path for cuDNN installation is `/usr/local/cuda`. However, we will
be *installing* cuDNN in our `$HOME` directory. The reason for this is two-fold:
1. You may not have admin privileges to install cuDNN in the conventional manner
2. Existing research project repositories sometimes work only on specific
releases of cuDNN so this method allows you to switch between different versions
easily along with a virtualization manager such as [Anaconda][anaconda]. 

# Assumptions
* Ubuntu OS
* NVIDIA GPU with CUDA support
* CUDA (*installed by system admin*)

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
# cd <your_download_directory>
tar -zxvf cudnn-8.0-linux-x64-v7.tgz -C ~/
{% endhighlight %}

This should create a `~/cuda` directory. Note the directory is called 'cuda'
instead of 'cudnn'. If you are installing a separate release of cuDNN, simply
unpack it into a separate directory of your naming e.g. `~/cudnn_v4`.

That's it! We are done!

[anaconda]: https://anaconda.org/
[download-cuDNN]: https://developer.nvidia.com/cudnn