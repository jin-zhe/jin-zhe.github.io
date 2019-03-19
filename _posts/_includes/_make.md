{% assign num_cores = 24 %}
{% assign makes = include.makes | split: ',' %}

Let's find out how many cores your machine has
{% highlight bash %}
cat /proc/cpuinfo | grep processor | wc -l
#=> n
{% endhighlight %}

Let's `make` the package efficiently by maximising the number of jobs for it.
General rule of thumb is to use 1 + n number of jobs where n is the output from
the previous command. i.e. number of cores. Mine was {{ num_cores }} so I run the following
{% highlight bash %}
make{% if include.all %} all{% endif %} -j {{ num_cores | plus: 1 }}{% for make in makes %}
make {{ make }}{% endfor %}
{% endhighlight %}

After make is completed, we are now finally ready to install
{% highlight bash %}
make install
{% endhighlight %}