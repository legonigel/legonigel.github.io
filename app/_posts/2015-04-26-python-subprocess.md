---
date: 2015-04-26 2:30:00
layout: post
title: 'Subprocessing in Python'
description: 'Getting python to play nicely with interactive subprocesses'
tags: ['Python','Multiprocessing']
suggested_tweet:
  hashtags: []
  related: []
---

I recently spent some time trying to get Python to play nicely with an interactive subprocess. By interactive, I mean that the subprocess is a command line program which waits for input from the user, then acts on that input and prints back out to the command line. I read a lot of documentation and looked for examples as to how to make this work, but couldn't really find anything complete. After some trial and error, I came to the result I will share with you.

Note: This is written for python 2.7, for the [Subprocess32](https://pypi.python.org/pypi/subprocess32/) module (a backport of the subprocess module from python 3.2)

## Introduction to Python subprocesses

Python subprocesses can be super useful. A lot of people use them to execute simple shell commands
(such as [this Stack Overflow post](http://stackoverflow.com/questions/12605498/how-to-use-subprocess-popen-python)
or [this other post](http://stackoverflow.com/questions/2715847/python-read-streaming-input-from-subprocess-communicate)).
If the application runs and returns quickly, then  [`subprocess.call`](https://docs.python.org/3.2/library/subprocess.html#subprocess.call) is probably the best way to call the subprocesses. In other related situations `subprocess.check_call` or `subprocess.check_output` are similar alternatives. But all of these wait on the program to return. In order to run a concurrent subprocess, it is necessary to use subprocess's [`Popen`](https://docs.python.org/3.2/library/subprocess.html#subprocess.Popen).

## Popen usage

`Popen` is normally used where the command is run, then the output is read from it using the [`Popen.communicate`](https://docs.python.org/2/library/subprocess.html#subprocess.Popen.communicate) method. This method allows us to pass data into the standard input of the subprocess and read from the subprocess's standard output and standard error. The issue with the communicate method is that, similar to the methods discussed above, it waits for the process to terminate. This is no good for an interactive process.

## Interactive Popen

Now we can finally arrive to what I want, an interactive subprocess. This is one part that the docs don't really discuss well. In order to do this, we need to make the subprocess with pipes.

{% highlight python %}
    ps = subprocess32.Popen(['command', 'argument'], stdout=subprocess32.PIPE, stderr=subprocess32.PIPE, stdin=subprocess32.PIPE)
{% endhighlight %}

This makes it so that the process is created with pipes and will run until it terminates. Unlike using `communicate` to pass data to the subprocess, pipes will not wait on the process to terminate before returning, however they will wait for data before returning. This means that if you expect the subprocess to output something, but it doesn't then your program will hang. For my application this was a problem.

By setting file status flags on the stdout and stderr of the program, we can use nonblocking reads. This is accomplished with the use of [fcntl](https://docs.python.org/2/library/fcntl.html).

{% highlight python %}
fcntl.fcntl(ps.stdout.fileno(), fcntl.F_SETFL, os.O_NONBLOCK)
fcntl.fcntl(ps.stderr.fileno(), fcntl.F_SETFL, os.O_NONBLOCK)
{% endhighlight %}

Finally we can get to where we can read from the stdout and stderr at our leisure.

{% highlight python %}
    stdout = ps.stdout.read()
stderr = ps.stderr.read()
{% endhighlight %}

But there is an issue. If there is no data in the stdout or stderr buffer then python throws an `IOError: [Errno 11] Resource temporarily unavailable`. For my purpose, I ignore any errors during the read from stdout and stderr and just assume there was no output. So my reads now look like this:

{% highlight python %}
    stdout = ''
stderr = ''
try:
    stdout = self.ps.stdout.read()
except Exception:
    pass
try:
    stderr = self.ps.stderr.read()
except Exception:
    pass
{% endhighlight %}

## Example

Finally, an example program with everything:

{% highlight python %}
# Import necessary modules
import os, fcntl, subprocess32

# Make the subprocess
ps = subprocess32.Popen(['command', 'argument'], stdout=subprocess32.PIPE, stderr=subprocess32.PIPE, stdin=subprocess32.PIPE)

# Fix the pipes to be nonblocking
fcntl.fcntl(ps.stdout.fileno(), fcntl.F_SETFL, os.O_NONBLOCK)
fcntl.fcntl(ps.stderr.fileno(), fcntl.F_SETFL, os.O_NONBLOCK)

while True:
    # Main loop
    stdout = ''
    stderr = ''

    # Read from stdout
    try:
        stdout = ps.stdout.read()
    except Exception:
        pass

    # Read from stderr
    try:
        stderr = ps.stderr.read()
    except Exception:
        pass
{% endhighlight %}

Of course there is one thing we still need to do: [Write](https://docs.python.org/2/library/stdtypes.html#file.write) to stdin. This is easy.

{% highlight python %}
ps.stdin.write('some_command_to_the_subprocess\n')
{% endhighlight %}

Don't forget to put the newline at the end of the write so the program sees that you actually pressed return after writing the line. As the docs for write point out, you may also need to `flush` after writing, depending on the buffering.
