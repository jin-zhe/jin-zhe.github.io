You'd think we're done, but not quite! We have to point the `$PYTHONPATH`
environment variable to our caffe folder like so
{% highlight bash %}
export PYTHONPATH={{ include.pythonpath }}:$PYTHONPATH
{% endhighlight %}