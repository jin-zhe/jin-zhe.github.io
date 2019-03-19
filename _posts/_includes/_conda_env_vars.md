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
cd $CONDA_PREFIX
mkdir -p ./etc/conda/activate.d
mkdir -p ./etc/conda/deactivate.d
touch ./etc/conda/activate.d/env_vars.sh
touch ./etc/conda/deactivate.d/env_vars.sh
{% endhighlight %}

Edit `./etc/conda/activate.d/env_vars.sh` as follows:
{% highlight bash %}
#!/bin/sh

export PYTHONPATH={{ include.pythonpath }}:$PYTHONPATH
{% endhighlight %}

Edit `./etc/conda/deactivate.d/env_vars.sh` as follows:
{% highlight bash %}
#!/bin/sh

unset PYTHONPATH
{% endhighlight %}

Now let's reload the current environment to reflect the variables
{% highlight bash %}
source activate {{ include.env_name }}
{% endhighlight %}