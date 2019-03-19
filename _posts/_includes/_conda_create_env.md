Let's create a virtual Anaconda environment called "{{ include.env_name }}".
{% highlight bash %}
conda create -n {{env_name}} python={{ include.python_ver }} {{ include.channel }}
{% endhighlight %}
*You many of course use a different environment name, just be sure to adjust
accordingly for the rest of this guide*

After it prepares the environment and installs the defaults, activate the
virtual environment via
{% highlight bash %}
conda activate {{ include.env_name }}
# to deactivate: source deactivate {{ include.env_name }}
{% endhighlight %}