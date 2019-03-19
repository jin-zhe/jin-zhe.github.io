Now let's install the necessary dependencies in our current {{ include.env_name }} environment:
{% assign installs = include.installs | split: ',' %}
{% highlight bash %}
{% for install in installs %}
conda install {{ install }} -y{% endfor %}
{% endhighlight %}