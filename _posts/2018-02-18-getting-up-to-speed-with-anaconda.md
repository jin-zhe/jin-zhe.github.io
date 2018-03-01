---
layout: single
title:  "Getting up to speed with Anaconda"
date:   2018-02-18 16:10:00 +0800
categories: anaconda conda
---
[Anaconda][anaconda] is a freemium open-source software for Python and R and it
is widely used in scientific computing. It provides a virtual environment
manager and it also comes with its own package management ecosystem conda. Why
does this matter you ask? Be it in academic or industrial research labs, it is
often the case that you are not granted admin access to GPU servers. Since you
won't have the ability to do `sudo apt-get`, the installation of AI tools can be
quite nerve-racking.

Anaconda's virtualization is much more *encompassing* than say Python's
virtualenv. For instance you can install [CMake][cmake] in an Anaconda
environment and start building tools from source. It goes without saying that
you no longer have much reasons to use Python's virtualenv once you're on
Anaconda.

I shall walk you through installing Anaconda and get you up to speed with the
most common and frequently used commands. Since *most* scientific packages out
in the wild offers the best support and stability for Python 2.7, I shall assume
that version of Python in this guide.

# Installation
Let's install **Anaconda 2** via the download page [here][download-anaconda].
The "2" simply means it's for Python 2 so don't download Anaconda 3 that is
meant for Python 3!

My downloaded file was a shell script called `Anaconda2-5.0.1-Linux-x86_64.sh`.
Let's run it
{% highlight bash %}
bash Anaconda-latest-Linux-x86_64.sh
{% endhighlight %}

Be sure to run a first update right after installation
{% highlight bash %}
conda update conda
{% endhighlight %}

We are done!

# Quick Start
Let's start by creating a virtual environment and installing CMake in it!
{% highlight bash %}
# conda create --name <environment-name> <python-packages>
conda create -n magical_schoolbus python=2.7 anaconda
{% endhighlight %}
* `-n`, `--name` flag indicates the name for the environemt we wish to create
* `python=2.7` indicates our Python version for this environment
* The trailing `anaconda` in the end indicates that we wish to initialize our
environment with the 'anaconda' meta-package. This is entirely optional!

Once the new environemnt is prepared, let's clean up the installation cache
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

# NOTE
After installation, Anaconda will automatically add the following to your
`~/.bashrc`
{% highlight bash %}
# added by Anaconda2 installer
export PATH="$HOME/anaconda2/bin:$PATH"
{% endhighlight %}
While this may work well on a local machine, it's not ideal if Anaconda was
installed on a remote server we `ssh` into. This is because `~/.bashrc` is
only automatically executed for interactive non-login shells and not for login
shells. In light of this, `~/.bash_profile` is a much better candidate to store
this environment variable export command.

TL;DR, if you're running on `ssh`, I'd suggest you cut
and past that bit of export code from `~/.bashrc` to `~/.bash_profile`. After
you're done, be sure to run `source ~/.bash_profile`

[anaconda]: https://www.anaconda.com/
[cmake]: https://cmake.org/
[docs]: https://conda.io/docs/