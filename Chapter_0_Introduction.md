# Chapter 0: What is SALTSTACK?

Before words like _cloud_, _paas_, and _load-balancer_ were a part of our everyday speech, most systems were silo-ed machines that were managed by one person, or a small team. Many organizations had one or two large, high-powered servers for their entire company. 

Now we often see businesses that own fifty or more servers. With virtual-machines it isn't uncommon to find a company with a thousand servers.

If you can imagine the amount of administrative overhead required just to maintain security updates alone on one-thousand servers, you might also start to wonder if there is a solution that exists for this very thing. Even the mundane task of setting up _another_ web server can be automated by the very tool named in the title of this book: SALTSTACK.

SALTSTACK is an open source software project licensed under the Apache license. The code is available from [github.com/saltstack/salt](http://github.com/saltstack/salt). SALTSTACK Inc. is the company behind SALTSTACK and it was founded by Thomas Hatch, the original creator of SALTSTACK.

This book is for the new or seasoned System Administrator looking to manage and configure multiple servers more easily. These can be virtual-machines such as an Amazon EC2 instances, Rackspace Virtual-Machines, or bare-metal physical machines. As long as you have root access, you can probably use SALT with your provider.

You will need some basic knowledge of your own filesystem, the packaging system in use with your organization, and some simple command-line commands. If you've programmed before, you'll benefit as well and you might have an easier time understanding some of the syntax, but prior programming knowledge if definitely not a requirement to use SALSTACK.

This book will cover the installation and use of SALTSTACK to manage linux servers.

SALT does however support installation and management on almost any system imaginable though. This includes Windows, FreeBSD, Solaris and many others. Be sure to check the specific documentation from [docs.saltstack.com](http://docs.saltstack.com) for specific installation and use with your particular operating system. You can even use SALT installed on one type of operating system to manage systems with different operating systems installed. Do note however, that the commands must be available for the particular system you are initiating them on. A Linux specific command even though you are using SALT on a linux machine, won't be available, for example on a Windows based system.

I won't cover those systems here, since SALT works generically regardless of what system you have it installed on. This is one of the beauties of SALSTACK. Regardless of what type of system or systems you plan on managing, the syntax and commands are the same. There are a few operating system specific functions, but those are well-documented in the official [SALSTACK Documentation](http://docs.saltstack.com). These are an excellent resource that are updated as new features are added.

I am purposely keeping most of the examples in the following pages quite simple and concise. This book isn't meant to cover everything you would ever need to know about SALTSTACK, but more to ease you in to using it and get you running in as little as a few evenings or even a day.

I'll use an example system of an Apache and MySQL based web server operation to walk you through installing packages, managing services, and adding users. We'll also see how to check the health of these systems and initiate software updates after we automate their installation.

SALTSTACK can be considered to be two tools in one: Remote execution, and Configuration Management. This is what separates it from some of the other configuration management tools available. Both are also intertwined with one-another and you get both when you install SALTSTACK.

I'll start by quickly explaining how you can install SALTSTACK for your particular operating system. Whenever possible, I'll refer to the official documentation available from the project for the sake of brevity. You should refer to the documentation as well, since the project has bug fixes and contributions on a daily basis.

One of the most important aspects of making an open source software project successful is the community around this project. SALTSTACK is no exception and has an excellent community built around it. The official SALTSTACK site, [salstack.org](http://salstack.org), hosts the documentation for the project that includes many examples.

You can also communicate with the SALTSTACK community in the #salt channel from freenode. If that last sentence makes no sense, a simple search for `irc - internet relay chat` should get you going with an irc client so that you can exchange real-time messages with others.

Finally, I'd like to mention briefly the software that makes up SALT and also mention that I will use the terms SALT and SALTSTACK interchangeably here.

SALTSTACK is written in Python and by default uses [YAML](http://yaml.org) syntax for configuration files. You don't need to know any Python to use SALTSTACK though, so don't worry if you aren't a developer. YAML also is very simple to use and is meant to be easy for humans to read. SALTSTACK also by default uses Jinja for its templating needs. Again, don't worry if you don't know what these are, I'll explain everything when we get there. For now, I just want to introduce it and show you what it looks like:

## YAML
For more detailed information about YAML, be sure to read up on it at the official YAML site, located at [yaml.org](http://yaml.org). I'll give a brief rundown here about what it is and how it relates to SALTSTACK. 

YAML is intended to be a human-readable, but machine-parseable syntax. What this means, is that by requiring a small level of formatting constraints, it makes text clear enough that humans and computers can both interpret its meaning.

Let's use this human language sentence to build our fictional YAML file that will install the Apache Webserver:

`Using the system's package manager, install the httpd package.`

The equivalent command-line syntax that you would type from your command prompt for this would be:

`# yum install httpd` 

***Note, this is for RedHat based systems. The Debian equivalent would be:*** 

`# apt-get install apache2`

This is a short YAML snippet, using SALT specific functions and commands to do the exact same thing:

Using the YUM specific name of `httpd` for the package to install:

    apache:
      pkg:
        - name: httpd
        - installed

And for Debian-based systems since the package is called `apache2`:

    apache:
      pkg:
        - name: apache2
        - installed

Without even being familiar with YAML, you can probably figure out just from reading it what will happen.

Let's look at this in plain-language:

The first entry, `apache`, is the name of the our SALTSTACK command. This could easily have been `web-server` as well and can be used for quick reference as to what the state does. I chose to call it `apache` to name it after the Apache Webserver package, which is what this snippet installs. 

The second entry, `pkg`, is SALTSTACK specific and denotes the package state. 

Next is the name of the package to install, either `httpd` or `apache2` depending on what package manager you are using.

The fourth and final line says to call the `installed` function from the package state.

Once you start writing and reading YAML, you'll find it to be quite simple and elegant.

This handles the human-readable portion of files for configuration management, but what about the machine-readable requirements?

YAML enforces a few constraints. In order to make this machine-readable, one is that each nested item is preceded by two spaces.

The following would result in an error:

    httpd:
    pkg:
    - installed

    local:
        Data failed to compile:
    ----------
        Name httpd in sls apache is not a dictionary
    ----------
        Name pkg in sls apache is not a dictionary

Each sub-line within a function must be preceeded by two spaces. 

When you begin writing your own state files, if you get errors like the one above, first check your syntax and spacing in your state file.

The simplest way to become familiar with SALT YAML specific syntax, is to browse through the documentation for whatever state you happen to be using.


## Jinja

You can read more about Jinja at the official Jinja site located at [jinja.pocoo.org](http://jinja.pocoo.org/). I'm going to briefly introduce it here and show you how you can use it to reduce the amount of typing you do, and make your files much more succinct.

Jinja is a templating language used to replace one block of text with another. It also allows the use of some limited logic making it even more powerful than just a simple substitution.

To illustrate how you might use a templating language, suppose you were creating a list of log entries and at the bottom of every page of this report you needed to insert your company's name, address, and telephone number.

Our sample logs might look like this:

    Sep 26 16:56:24 dbserver yum[24646]: Installed: libunwind-1.1-2.el6.x86_64
    Sep 26 16:56:24 dbserver yum[24646]: Installed: gperftools-libs-2.0-11.el6.3.
    x86_64
    Sep 26 16:56:24 dbserver yum[24646]: Installed: libmongodb-2.2.6-1.el6.x86_64
    Sep 26 16:56:24 dbserver yum[24646]: Installed: snappy-1.0.5-1.el6.x86_64

    Joe's Publishing 18 S. Main Street Birmingham, AL

For one page, it probably doesn't take much effort to copy and paste the name and address at the bottom. What if your report was fifty pages long though?

You could assign a _variable_ to represent the name and address and then substitute the long address with something quicker to type like `nameaddress` instead. I used something similar when writing this book. Instead of typing SALTSTACK and SALT over and over again, I set up a text variable that was much shorter so that every time I added the characters `-sts` or `-ss`, they were replaced by SALTSTACK and SALT respectively.

Templated, the previous sentence might look like this:

        Instead of typing -sts and -ss over and over again...every time I added the...they were replaced by -sts and -ss respectively.

What if your report also needed to contain the date and time that was generated, and this report is generated three times a day?

By the time you finished adding the date and time to all fifty pages, it would probably be time to start over and generate the next report.

You could again use a _variable_ to represent the date and time in your template and populate it automatically. Think of a variable as something static that doesn't change that represents something that is dynamic and does change.

At the end of your log file, you could insert a marker to insert the following instead:

`{{nameaddress}}`

`{{date}}`

`{{time}}`
	
When your logs were generated this would have the effect of inserting:

`Joe's Publishing 18 S. Main Street Birmingham, AL`

`23 Oct. 2014`

`13:00:01`

This table illustrates the substitutions of the Jinja variables with their plain-text equivalent:

|            | Name/Address         | Date         | Time |
|------------|----------------------|--------------|----------|
| Jinja      | {{nameaddress}}      | {{date}}     | {{time}} |
| Plain Text | Joes' Publishing.... | 23 Oct. 2014 | 13:00:01 | 

In the previous introduction to YAML, you may have noticed the difficulty that you might face later if you attempted to install the Apache Webserver on RedHat and Debian systems. Without Jinja, you would need to maintain two different state files for each system because the names for the Apache Webserver packages are different depending on which package manager you happen to be using.

Thankfully, SALTSTACK has a way to make this much easier and simpler using Jinja.

    {% if grains['os'] == 'Ubuntu' %}
    apache: apache2
    {% elif grains['os'] == 'RedHat' %}
    apache: httpd
    {% endif %}

You tell Jinja to act on a block of text by enclosing it within `{` and `}` symbols. For simple text replacement, you use two `{{` and `}}`.

The snippet above also contains a bit of logic to help set the name for apache as either httpd or apache2. Don't worry right now about the grains. We'll cover that more in depth in later chapters.

Again, in plain-language, the snippet above says essentially this:

`If the operating system is Ubuntu, set the name of apache to apache2, else if the name of the operating system is RedHat, set the name of apache to httpd.`

The previous two examples form the basis of our configuration files and these patterns will be used over and over again to adapt your states.

## Working with SALT Locally

For the examples and to ease learning, I'll show you how to install SALT using a Virtualbox virtual-machine within a single Linux server. You don't have to connect SALT to any other machines, and all commands and files are stored on that one machine. You can use the `salt-call` command in place of the `salt` command to use what is referred to as a _masterless_ configuration.

## SALT Specific Terms
I just introduced a SALT term, _masterless_, so now would be a good time to define some terms that you'll find common throughout SALT documentation, and this book.

SALT refers to the _Master_ as the server that would issue commands to all of your other servers. Those other servers are referred to as _Minions_. SALT accomplishes this very efficiently and quickly in an asynchronous manner. If one _Minion_ fails to respond, the command continues so that you don't have to wait for each to respond individually. The _SALT Master_ is another term that is interchanged with _Master_ and refers to the same thing. The ability to continue operations even if one or more servers aren't responding is another area that sets SALT apart from some other configuration management tools. 

SALT's configuration files are called _states_. These files by default are written in YAML and are where you specify your specific configurations, such as packages to install, and user accounts to create for each _minion_. Using the SALT filesystem, you can manage actual files and distribute them to each of your _minions_. The _top_ file is like a roadmap where you can specify which _states_ to apply to each minion. Multiple _states_ can be applied to each _minion_.

This is a sample tree of SALT's directory structure:

    /srv/salt/
    ├── apache
    │   ├── httpd.conf
    │   └── init.sls
    ├── groupmaker
    │   └── init.sls
    ├── hostnames
    │   ├── init.sls
    │   └── mage.conf
    ├── stagerpm
    │   ├── init.sls
    │   └── stage-1.0.0-dev.x86_64.rpm
    ├── npm
    │   └── init.sls
    ├── top.sls
    └── vim
        └── init.sls

Notice that the `top.sls` is located within the base of the `/srv/salt` directory. This is important and we'll get in to this file later. I've chosen to create directories for each _state file_ with their name for easier recognition. You don't _have_ to do this.

SALT is quite flexibile and doesn't enforce very many naming conventions. I happen to find it much simpler to have my _state files_ organized with directory names. You could have an `apache.sls` file instead and omit the directory if you chose to do so, but as you can see for example in the `apache` directory, there is  an `httpd.conf` file. The `/srv/salt` directory could quickly get littered with multiple files that don't relate to each other.

I haven't explained the _pillar_ yet, but I do want to point out that the structure is quite similar to the _state file_ layout:   
 
    /srv/pillar/
    ├── top.sls
    └── users.sls

_Grains_ are metadata fragments about individual _Minions_, and you can define your own. In the Jinja example that we just looked at, I accessed the _os_ grain, which returns information about the operating system in use on a particular _Minion_. _Grains_ also contain things like processor architecture, ip addresses, and kernel information.

The last term you'll see here is _Pillar_. Think of _Pillar_ data as kind of a global variable where you can set anything you want and make it reusable across _States_. Referring back to the Jinja example, I've set a _Pillar_ entry for the name of the Apache Webserver. We'll look at this a bit later when we get to the Configuration Management chapter. Within different linux operating systems for example, the Apache webserver for Debian-based systems is called apache2, but for the Red Hat family is called httpd. You can write generic _States_ that will correctly choose the correct name by utilizing the _Grains_ system within your _Pillar_ data. Don't worry at this point if this doesn't make much sense, just keep in mind the premise that _Pillar_ is a custom variable system. 

## A quick note about Python
Earlier I told you that you don't have to have any knowledge of Python, or any other programming language for that matter to successfully and effectively use SALT. I still stand by my word! 

I do want to give a very brief overview of some common Python data structures that will help you in writing and understanding your states, and troubleshooting when they don't quite work as you expected. If you've used other scripting languages like Ruby, you can probably safely skip this section on Python. I am only introducing these here because SALT is very Pythonic in its syntax and operations. 

Bare with me, this isn't a book about Python and this will be very brief, but it is important to understand how SALT interprets and collects, stores, and interprets information from your state files and commands. 

Because SALT is written in Python, many errors that get returned to you are returned by Python itself. Having some cursory knowledge of the way Python references data, can help you interpret what might be wrong with your syntax.

A typical Python error will look like this:

    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    NameError: name 'bananas' is not defined

This is what Python refers to as a _traceback_ and what others will ask you for if you ask for help. Start reading from the bottom and follow the errors up to the top for a hint as to what went wrong. You'll often see this when you encounter a bug, or supply SALT with an improper syntax.

Here is another, more complicated error produced by bug in SALT that was fixed in a later release:

    Traceback (most recent call last):
      File "/usr/bin/salt-call", line 11, in <module>
        salt_call()
      File "/usr/lib/python2.6/site-packages/salt/scripts.py", line 76, in     
        client.run()
      File "/usr/lib/python2.6/site-packages/salt/cli/__init__.py", line 255, 
    run
        caller.run()
      File "/usr/lib/python2.6/site-packages/salt/cli/caller.py", line 129, in 
        ret = self.call()
      File "/usr/lib/python2.6/site-packages/salt/cli/caller.py", line 71, in 
        ret['return'] = self.minion.functions[fun](*args, **kwargs)
      File "/usr/lib/python2.6/site-packages/salt/modules/yumpkg.py", line 
    group_install
        group_detail = group_info(group)
      File "/usr/lib/python2.6/site-packages/salt/modules/yumpkg.py", line
    group_info
        if group.name.lower() == groupname.lower():
    AttributeError: 'NoneType' object has no attribute 'lower'    

This error was produced by the `pkg.group_install` module:

`$ salt '*' pkg.group_install "Development Tools"`

when I attempted to use my package manger, yum, to install the "Development Tools" package.

If you read from the last entry on the traceback, and then follow up to the next line above it, notice `File "/usr/lib/python2.6/site-packages/salt/modules/yumpkg.py", line…`

This gives a good indication of the SALT module that produced the error. In this example, the `yumpkg.py`. This gives you a good starting point for diagnosing a good portion of the errors you might encounter, and should be included in any bug reports and correspondence with the community. This gives others a good point to begin helping you troubleshoot.

### Lists in Python

One of Python's most powerful and attractive features is its ability to quickly collect arbitrary collections of information. A basic tenet of this are Python's lists.

You define a list in python like this:

`fruits = ['apples', 'bananas', 'oranges']`

`fruits` is the name of the variable, which is a list that contains the names of three fruits: `apples`, `bananas`, and `oranges`.

The order of items in a list isn't arbitrary, and can actually be accessed by an index, starting with 0.

If we used Python to print our list of _fruits_:

`print fruits`

Our list would appear like this:

`['apples', 'bananas', 'oranges']`

We can access individual items by specifying an their index:

`print fruits[0]`

Would result in:

`apples`

### Dictionaries in Python
Dictionaries are another data type in Python, but they differ from lists in that you can define the index for them, instead of each item being assigned a positional integer.

Python distinguishes dictionaries from lists with `{` and `}`.

For example, this is a simple dictionary containing names of people and their ages:

`people = {'joe': 30, 'carl': 28, 'mary': 29}`

We can view the entire collection now:

`print people`

Which results in:

`{'joe': 30, 'carl': 28, 'mary': 29}`

We can now access the items in our dictionary by using each item's key:

`print people['joe']`

Which shows that _joe_ is thirty years old:

`30`

Your _State_ files will be translated from YAML syntax into lists and dictionaries that are then interpreted by SALT (and Python) and used to initiate commands and supply data to those commands.

You can learn more about Python data structures from the official [Python](http://python.org) website. I'm not going to go any further into Python, but I want you to visualize Python lists as we begin to write some _State_ files.

### Boolean Operators in Python

Python offers plain-word operators like and, or, and not for boolean operations. For example, Suppose I had a box of mixed fruit containing 10 apples, 5 oranges, and 15 bananas. I asked you to tell me how many pieces of fruit that you have that are apples and bananas. We have 30 pieces in total, but we only have 25 pieces that are apples and bananas.

Our simple python demonstration might look like this (you can try this yourself from a Python prompt):

    >>> apples = 10
    >>> oranges = 5
    >>> bananas = 15
    >>> total_fruit = apples + bananas + oranges
    >>> total_fruit
    >>> 30
    >>> apples and bananas
    >>> 15
    >>> if apples == 10 and oranges == 5:
    ...     print "True."
    >>> True.
    >>> if apples == 10 and not oranges == 10:
    ...     print "I have 10 apples and don't have 10 oranges."
    >>> I have 10 and apples and don't have 10 oranges.
    >>> if apples == 10 or oranges == 15:
    ...    print "Still true!"
    >>> Still true!

We'll touch on boolean operations in SALT when we examine the _grains_ system and when we look at _pillar_ as well when we put _jinja_ to use and add logic and abstraction to our configurations. For now, I just want to introduce boolean operations in case you haven't seen them before. 


## Other SALT Based Tools

There are other SALTSTACK tools that I won't cover in this book, but I would like to mention them here simply to give you an idea of the other tools available.

### SALTY-VAGRANT
I won't cover the use of [Vagrant](http://vagrantup.com) or [SALTY-VAGRANT](http://github.com/saltstack/salty-vagrant) here, but this is an excellent tool to check out and one you will find very convenient for testing new states prior to deploying them to production. Vagrant allows you to manage very small virtual-machines and instantly reset them whenever you want. SALTY-VAGRANT is a Vagrant plugin that allows you to automatically install SALT and deploy your state files within a Vagrant virtual-machine. This can save hours of troubleshooting since you start with a fresh base every time you use it.

### SALT-CLOUD

SALT-CLOUD implements the ability to manage multiple cloud-providers accounts and seamlessly integrate SALT with these providers. The benefit is that you can deploy SALT based servers within multiple providers and use generic state files. For example, using SALT-CLOUD, you can automate the creation and management of separate servers within Amazon Web Services, and Linode at the same time. You can deploy your hadoop cluster on two different providers without needing to have any knowledge of each provider's different APIs.

