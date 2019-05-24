Let's create a virtual Conda environment called "{{ include.env_name }}":
{% highlight bash %}
conda create -n {{env_name}} python={{ include.python_ver }}{% if include.meta_package %} {{ include.meta_package }}{% endif %}
{% endhighlight %}
*You many of course use a different environment name, just be sure to adjust
accordingly for the rest of this guide*

After it prepares the environment and installs the default packages, activate
the virtual environment via:
{% highlight bash %}
conda activate {{ include.env_name }}
# to deactivate: conda deactivate {{ include.env_name }}
{% endhighlight %}