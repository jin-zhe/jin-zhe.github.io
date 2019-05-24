---
layout: single
title:  "Getting up to speed with Conda"
date:   2018-02-18 16:10:00 +0800
categories: Guides
tags: Anaconda Conda Miniconda
---
> "[Conda][conda] is an open source package management system and environment
  management system that runs on Windows, macOS and Linux. Conda quickly
  installs, runs and updates packages and their dependencies. Conda easily
  creates, saves, loads and switches between environments on your local
  computer. It was created for Python programs, but it can package and
  distribute software for any language."

Why does this matter you ask? In research, it is often the case that different
experiments and projects requires very different environments to execute and so
a system to organize, manage and switch between different environments is
curcial. In addition, it is also often the case in academic or industrial
research labs that users are not granted admin access to GPU servers. Since you
won't have the ability to do `sudo apt-get`, the installation of AI tools
is impossible without virtual environments.

You might be familiar with [Virtualenv][virtualenv] which is another popular
virtual environment manager for Python. However Virtualenv is really only
meaningful if you have `sudo` access, a shortcoming which Conda does not have.
Conda's virtualization is also much more *extensive* than Virtualenv. For
instance you can install [CMake][cmake] in a Conda environment and start
building tools from source. You really don't have much reasons to use Virtualenv
once you're on Conda.

Conda is a package management system that is delivered via 2 software suites,
namely [Miniconda][miniconda] and [Anaconda][anaconda]. Anaconda is meant to be
a complete software suite for data science and so comes with more than 720
open-source packages out-of-the-box. This means that it takes up a whopping 3 GB
just to install! In comparison Miniconda is concise, lightweight and meant for
users who don't mind individually downloading and installing required packages.

I personally feel that Anaconda is very bloated with unecessary packages and 
many users have also experienced slow executions after using it for awhile. For
that reason, I shall advocate for the use of Miniconda which will be used in the
remaining sections of this guide. Since both Anaconda and Miniconda is built
upon conda, their usage syntax is identical and you could easily adapt this
guide for Anaconda if you so wish.

I shall walk you through installing Miniconda and get you up to speed with the
most common and frequently used commands. Since Python 2 is reaching its end of
life in 2020, I shall install Conda in Python 3 for the sake of continual
relevance of this guide.

## Installation
Let's install **Miniconda 3** via the download page [here][miniconda].
The "3" simply means it's for Python 3 so don't download the one meant for
Python 2! Anaconda packages also use this naming convention.

My downloaded file was a shell script called `Miniconda3-latest-Linux-x86_64.sh`.
Let's run it
{% highlight bash %}
sh Miniconda3-latest-Linux-x86_64.sh
{% endhighlight %}

At the end of installation, the installer will prompt if you would like to
initialize conda for your shell by running `conda init`. I suggest you do that.
By doing so, Miniconda will add the following lines to your `~/.bashrc`:
{% highlight bash %}
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/jinzhe/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/jinzhe/miniconda3/etc/profile.d/conda.sh" ]; then
        . "/home/jinzhe/miniconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/jinzhe/miniconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
{% endhighlight %}

The above adds new environment variables to your shell so that you may start
running conda commands on the terminal. After conda initialization is done,
restart your shell by:
{% highlight bash %}
exec $SHELL
{% endhighlight %}

While this may work well on a local machine, it's not ideal if Conda was
installed on a remote server we `ssh` into. This is because `~/.bashrc` is
only automatically executed for interactive non-login shells and not for login
shells. In light of this, `~/.bash_profile` is a much better candidate to store
this environment variable export command.

In other words, if you're running on `ssh`, I'd suggest you cut
and paste that snippet of export code from `~/.bashrc` to `~/.bash_profile`.
After you're done, be sure to run `source ~/.bash_profile`.

Finally, be sure to run a first update right after installation
{% highlight bash %}
conda update conda
{% endhighlight %}

We are done with installation!

## Quick Start
Let's start by creating a virtual environment!
{% highlight bash %}
# conda create --name <environment-name> <meta-package>
conda create -n magical_schoolbus python=2.7 anaconda
{% endhighlight %}
* `-n`, `--name` flag indicates the name for the environemt we wish to create
* `python=2.7` indicates our Python version for this environment
* The trailing `anaconda` in the end indicates that we wish to initialize our
environment with the 'anaconda' meta-package. This is entirely optional!

Once the new environment is prepared, let's clean up the installation cache
{% highlight bash %}
conda clean --all
{% endhighlight %}

We can see a list of all the environments we have created via
{% highlight bash %}
conda info --envs
{% endhighlight %}

Since we have only created one environemnt, we should only see
`magical_schoolbus`. We are now ready to hop onboard!
{% highlight bash %}
source activate magical_schoolbus
{% endhighlight %}

Let's take a look at which are the programmes currently installed in our
environment
{% highlight bash %}
conda list
{% endhighlight %}

It appears that `CMake` isn't in the list. Let's check if it's available for
download. To search for packages to install, you may either search
[conda cloud][anaconda] on your browser or via the command line like so
{% highlight bash %}
conda search cmake
{% endhighlight %}

Horray! CMake is available! We may proceed to install it in our current virtual
environment
{% highlight bash %}
# conda install --channel <channel-name> --name <environment-name> <package-name>
conda install cmake -y
{% endhighlight %}
* The `-y`, `--yes` flag simply automates the 'yes' approval from you to install
the packages
* If you wish to install in another virtual environment, simply use the `-n`,
`--name`  flag
* `-c`, `--channel` indicates additional channel to  search  for  packages if
you do not wish to install from default packages

To uninstall a package, simply do
{% highlight bash %}
conda uninstall some_package
{% endhighlight %}

Finally when our work is complete, we may leave the virtual world like so
{% highlight bash %}
source deactivate
{% endhighlight %}

To destroy an environment, input command
{% highlight bash %}
conda remove --name myenv --all
{% endhighlight %}
OR via its alias
{% highlight bash %}
conda env remove --name myenv
{% endhighlight %}

To rename an environment, simply clone the current environment into another
with the desired name and thereafter removing the old environment. Note that
this is a potentially time and space consuming operation
{% highlight bash %}
source deactivate # be sure to exit virtual environment first!
conda create --name new_name --clone old_name
conda remove --name old_name --all
{% endhighlight %}

That's all folks! Be sure to check out the official [docs][docs] for the
latest and complete documentation.

## Important caveats
Referenced from [here](https://caffe2.ai/docs/faq.html#why-is-caffe2-not-working-as-expected-in-anaconda)  
There are 3 common ways people create their Conda environments:

### Method 1
{% highlight bash %}
conda create -n some_env
{% endhighlight %}
**DO NOT DO THIS** this, as this creates a brand new env but does NOT install Python or
setuptools into the env. This means that if you activate your new env with
`source activate some_env` immediately after you create it, then your
root env `base` will still be activated. Many build tools rely on the Python
package `distutils` to know where to install Python files. If you use this
method, then `distutils` will still point to the `base` Python instead of your
Python env, which might conflict with other flags passed to cmake system.

### Method 2
{% highlight bash %}
conda create -n some_env anaconda
{% endhighlight %}
**DO NOT DO THIS** either! This will install **all** base Anaconda packages into
your new env. This is a lot of packages, most of which you will not use. This
wastes a lot of memory and takes a long time to install. This also makes it
harder to install later packages, as dependency resolution is complicated by all
the existing packages.

### Method 3
{% highlight bash %}
conda create -n some_env python=3
{% endhighlight %}
**ALWAYS DO THIS**! This will create a new env and install only the minimal
packages, including Python setuptools and `distutils`. If you use this to create
all your conda envs and never install packages into your root env then you will 
have a much better experience with Conda.

[anaconda]: https://www.anaconda.com/
[cmake]: https://cmake.org/
[conda]: https://docs.conda.io/en/latest/index.html
[docs]: https://conda.io/docs/
[miniconda]: https://docs.conda.io/en/latest/miniconda.html
[virtualenv]: https://virtualenv.pypa.io/en/latest/
