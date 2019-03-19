Let's clone {{ include.env_name }}'s repo and its submodules into our home directory.
{% highlight bash %}
cd ~
git clone --recursive {{ include.url }}
cd {{ include.repo_name }}
git submodule update --init --recursive
{% endhighlight %}