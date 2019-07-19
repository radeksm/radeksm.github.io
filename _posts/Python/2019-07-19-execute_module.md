---
layout: post
title: Execute Python modules and compressed modules
tags:
    - Python
author: Radosław Śmigielski
---

Why?
====
It was somehow mysterius to me how commands like below wotk?

How does Python interpreter know where to find executable code?

  ```
  echo '{"json":"obj"}' | python3 -m json.tool
  python3 -m zipfile -c /tmp/zip.zip /etc/fstab
  python3 -m venv /tmp/venv
  python3 -m unittest
  ```

So after a little bit of googling I found
[PEP-0441](https://www.python.org/dev/peps/pep-0441/) which says:

 > Python has had the ability to execute directories or ZIP-format archives
 > as scripts since version 2.6 [1]. When invoked with a zip file
 > or directory as its first argument the interpreter adds that directory
 > to sys.path and executes the __main__ module.

How?
====

Executable module
-----------------

Let's start with **python -m module.name** option. Official Python3 doc says:
 > Searches sys.path for the named module and runs the corresponding .py file
 > as a script.

Unfortunately that doc sentenence doesn't specify what is that
"corresponding .py file". And as it tunedned out it needs to be **__main__.py**
file, which is an module entry point.
So here is the example of empty module which can be called as executable.

Two files in **module** directory:
{% highlight bash %}
$ ls -l module/
total 12
-rw-rw-r--. 1 radek radek  65 Jul 19 16:06 __init__.py
-rw-rw-r--. 1 radek radek  96 Jul 19 16:05 __main__.py
{% endhighlight %}

And files content as follow:
{% highlight python %}
# file: module/__init__.py
print("I am package {} from file {}".format(__name__, __file__))

# file: module/__main__.py
if __name__ == "__main__":
    print("I am package {} from file {}".format(__name__, __file__))
{% endhighlight %}

So now, let's execute our new module:
{% highlight bash %}
$ python -m module
I am package module from file module/__init__.py
I am package __main__ from file /tmp/module/__main__.py
{% endhighlight %}

Compressed executable module
----------------------------
This may be a little bit similar to Java .jar files and should make modules
distribution easier.

Let's compress our module, I am going to use **zipfile** module but it can
be zip in any other way.
{% highlight bash %}
$ python3 -m zipfile -c  module.zip module
$ python3 -m zipfile --list compressed_module.zip
File Name                                             Modified             Size
__init__.py                                    2019-07-19 16:06:20           65
__main__.py                                    2019-07-19 16:05:20           96
{% endhighlight %}

Now we can execute that compressed module:
{% highlight bash %}
$ python3 compressed_module.zip
I am package __main__ from file compressed_module.zip/__main__.py
{% endhighlight %}
And it works!
