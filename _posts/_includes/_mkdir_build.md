We shall avoid polluting the {{ include.repo_name }} source tree by building
within a build folder
{% highlight bash %}
mkdir build && cd build
pwd #=> ~/{{ include.repo_name }}/build
{% endhighlight %}