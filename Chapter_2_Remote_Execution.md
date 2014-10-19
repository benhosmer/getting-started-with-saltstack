# Chapter 2: Remote Execution

This chapter will cover _Remote Execution_, which is the process of initiating commands on one, or many machines at once.

Here is a common scenario to help illustrate a common systems administrator's task to keep an eye on her company's servers:

Angie arrives for work on Monday morning and logs in to her computer. She is responsible for the general health and maintenance of the company's database server, web server, and a small file server. She likes to check if the servers stayed up and running over the weekend and begins the process of logging in to each one individually and running the `uptime` command. 

From a command line prompt on the database server, she issues this command:

`[angie@db]$ uptime`

And she receives this response:

`09:47:08 up 91 days,  3:23,  1 user,  load average: 0.00, 0.00, 0.00`

So far so good, now she moves on to the file server:

`[angie@fs]$ uptime`

`09:48:19 up 89 days,  2:21,  1 user,  load average: 0.00, 0.00, 0.00`

She then logs in the web server, but before she can check on it, she is interrupted by a fellow employee to remind her about a meeting.

She gets automatically logged out due to inactivity, and after logging back in:

`[angie@webserver]$ uptime`

`09:50:23 up  4:29,  1 user,  load average: 0.00, 0.00, 0.00`

The web server has only been running for a few hours.

This was quite a bit of redundancy and lost time just to check on each server and find out that there was a possible problem with the web server. She had to log in to each machine, and issue the same command three times. In this simple example, this process probably only took a few minutes, but imagine if this same systems administrator had one-hundred separate servers to check on?

Using SALT, Angie can log in to her _Master_ and issue this command:

`[angie@saltmaster]$ sudo salt '*' cmd.run "uptime"`

She receives this response:

    dbserver:
        09:47:08 up 91 days,  3:23,  1 user,  load average: 0.00, 0.00, 0.00
    fileserver:
        09:47:08 up 89 days,  2:21,  1 user,  load average: 0.00, 0.00, 0.00
    webserver:
        09:47:08 up  4:29,  1 user,  load average: 0.00, 0.00, 0.00

Again, this was quite a simple introduction to remote execution, and I've only demonstrated running a single command on three different servers, but as we go through this chapter, I'll point out some of the more powerful things that you can do with remote execution. Now that you have a general idea of what remote execution is, let's set up an environment with a _Master_ and one _Minion_.

These can be separate physical machines, or two virtual-machines. It really doesn't matter. If you followed along with Chapter 1 and used [Vagrant](http://vagrantup.com) you can use the same method and create two new virtual machines. We'll use one as a _Minion_ and _Master_ just to simplify things.

Feel free to choose whatever distribution you would like, but you'll find installation a little simpler if you stick with recent and updated distributions.

However you created these two machines, at this point you should have two separate systems, each up and running.

Again, we will use a fairly simple website hosting architecture with on minion acting as the webserver, and one acting as the master. Later, when we move to writing and testing our _State Files_ in Chapter 3, we'll use the `salt-call` command, but for this initial exercise I'd like to use two different machines.

## Set up Your Master
Choose one of your machines to act as the master. You can pick either one you want and install the `salt-master` package on this one. The master will require very little configuration at this point, especially for the remote execution tutorials. All you really need to do is simply note the ip address of this machine. Ensure that master is running. On CentOS and Ubuntu systems the `salt-master command`:

`$ service salt-master status` 

should return

`salt-minion (pid  1961) is running…`. 

If it isn't, go ahead and start it now with 
`$ service salt-master start` 

which will return :

`Starting salt-master daemon:  [  OK  ]`

## Configure the Minion

Now on your second machine, log in and install the `salt-minion` package using whatever is appropriate for your distribution.

On RedHat family systems the package-manger is yum by default:

`$ yum install salt-minion`

Or Debian systems:

`$ apt-get install salt-minion`

Located within the `/etc/salt` directory, you'll find a heavily commented `minion` configuration file.

Near line 11, you'll find the `master` entry `#master: salt`.

Change this to the ip address of your _master_ that you noted earlier: `master: 192.168.33.123`. Your ip might differ from the one I used in this example.

Near line 40, locate the `#id` entry. Here is where we can name our _minions_ something more human readable. You can use any naming scheme you like here because SALT is very flexible. Let's use a fictitious hostname for our webserver. Edit the line:

`id: webserver1.example.com`

Save the file and we're done. Feel free to read through the other entries if you'd like. We'll revisit these later when we get more in depth with targeting.

You can log out of your _minion_ now and log back in to your _master_.

In order to begin using SALT, you need to authenticate your _minion_ and accept its keys on the master.

Use the `salt-key` command for to list the accepted, unaccepted, and rejected keys for your minion:

`$ salt-key -L` will list all keys and will return:

`Key for webserver1.example.com accepted.`

You should see your recently configured _minion_ listed. You can accept its keys with:

`$ salt-key -a webserver1.example.com`

Which returns:

`Key for minion webserver1.example.com accepted.`

Now that our _master_ and _minion_ can authenticate with one-another securely, you have full-control over your _minion_. For our final step, let's test the connection with a ping request:

`$ salt 'webserver1.example.com' test.ping`

Which returns:

    webserver1.example.com:
        True

This means that you are connected to your _minion_! Congratulations.

We use the `salt` command from our master to invoke the myriad of SALT functions. Let's examine our `test.ping` command a little closer. It forms the basis of all of our remote-execution actions that will follow.

First, we used `salt` and followed it by a `target`, which, in this case, was `webserver1.example.com`, our _minion_ id. We then followed this by our command, which was `test.ping`.

This is an example of very fine-grained targeting. We only targeted one specific minion. In our case, we only have one to target at this point anyway. Later we'll explore the use of `grains` for more flexible targeting.

The formula for the `salt` command follows this pattern:

`$ salt` `<target>` `<action`

You can also use [regular expression](http://en.wikipedia.org/regex) matching in your targeting.

## Targeting with Grains

I alluded to _grains_ earlier but didn't really explain what they were. 

Consider our scenario of a simple web-hosting operation. Currently we only have one _minion_ being controlled by our _master_. I've deliberately kept this simple to begin with, but shortly we will add another minion. 

When you examined the _minion_ configuration of your during the initial installation and setup, we named it with an `id` entry in the minion configuration file. This works for a nice human-understandable name, but may not work well as we add minions, considering that we would need to type out long names to target them, and that we may need to target other system types or functions.

Create a second _minion_ so that you now have three total machines running. This one will be our database server in our fictional web-hosting setup.

Let's give this one the id `id: database1.example.com`

Look further down in the _minion_configuration file, and locate near line 51 where you'll see:

    # grains:
    #   roles:
    #     - webserver
    #     - memcache

Let's add our own role for our databse _minion_:

    grains:
      roles:
        - database

While you're there, don't forget to add your _master's_ ip address to this new _minion_ as well and make sure it is running.

And log back in to your first _minion_, the web-server, and add this to the _minion_ configuration:

    grains:
      roles:
        - webserver

Log out and then log back in to your _master_. You'll need to accept the new keys for your database _minion_ like you did previously for the webserver.

After doing this, check that you now have both with the `test.ping` command, but instead we can use a wild-card to target everything:

`$ salt '*' test.ping`

You should have seen both _minions_ respond back with:

    webserver1.example.com
        True
    database1.example.com
        True

If you don't, check that the _minion_ is running and if so, restart it.

Now that we have configured the _grains_ for each of your _minions_ we can target them by _roles_. Again, like the `id` for our _minions_ the _grains_ are very flexible in SALT. You can name them whatever suits your datacenter's architecture. You might not be hosting webservers and databases, but instead fileservers and hadoop clusters. Name them something that makes sense to you and your team.

In our fictional example, Angie has again arrived for work and wants to check the uptime of all of her webservers. She can now target them by role using the _grains_ system:

`$ salt -G 'role:webserver' cmd.run "uptime"`

and the database server:

`$ salt -G 'role:database' cmd.run "uptime"`

The `-G` flag indicates that we want to target a specific grain, in this case `role` followed by the specific role, `database` and `webserver`. We then told each minion to _run the command_ using `cmd.run` followed the command of `"uptime"`.

Again, the syntax follows:

`salt` `<target>` `<command>` with each target now having arguments along with each command.

We could issue any available command on each system by modifying the above example:

`$ salt -G 'role:database' cmd.run "reboot"`

You can get a listing of all available _grains_ by using the `grains.ls`:

`$ salt 'webserver1.example.com' grains.ls`

Or on all _minions_ with the wildcard:

`$ salt '*' grains.ls`

Which returns:

    …
    - ps
    - pythonpath
    - pythonversion
    - roles
    ...

On a default installation without any custom grains, here are the grains available to SALT automatically:

    - biosreleasedate
    - biosversion
    - cpu_flags
    - cpu_model
    - cpuarch
    - defaultencoding
    - defaultlanguage
    - domain
    - fqdn
    - gpus
    - host
    - id
    - ipv4
    - kernel
    - kernelrelease
    - localhost
    - manufacturer
    - mem_total
    - nodename
    - num_cpus
    - num_gpus
    - os
    - os_family
    - oscodename
    - osfullname
    - osrelease
    - path
    - productname
    - ps
    - pythonpath
    - pythonversion
    - saltpath
    - saltversion
    - serialnumber
    - server_id
    - shell
    - virtual


You can query the details of these grains using `grains.item`:

`$ salt 'webserver1.example.com' grains.items`

    pythonversion: 2.6.6.final.0
      roles:
          webserver
      saltpath: /usr/lib/python2.6/site-packages/salt
      saltversion: 0.15.1
      serialnumber: 0
      server_id: 813553791

And you can get the information for a specific grain item using  `grains.get`:

`$ salt '*' grains.get cpuarch`

    local:
        X86_64

And you query multiple items and values by adding additional grains:

`$ salt '*' grains.item saltversion os_family`

    os_family:
        RedHat
    saltversion:
        0.14.1

Obviously for only two machines, we don't gain much benefit, but imagine if we had five webservers and five database servers? Hopefully you've started to think about implementing targeting for your systems and the architecture for your organization. 

Also note, that these commands are issued to each _minion_ at the same time, so if one _minion_ is down, it won't stop any other _minions_ from returning. The timeout for a response is also configurable within the _master_ configuration.

## Package Installation Using Remote Execution

Later in Chapter 3, we'll use _State Files_ to install and configure our systems, but for more practice, and to explain the use of _remote execution_ we're going to install the necessary packages for our web-hosting systems.

Currently we have two _minions_ and one _master_. If you'll recall earlier, one _minion_ is our webserver, and one is our database server. We've merely named them and given them roles, but up until now, they are just stock operating systems available from your favorite distributions at this point.

Let's get some working systems now by installing the Apache Webserver on your webserver1.example.com _minion_ and MySQL on our database1.example.com.

We'll target them by role, but we could have just as easily targeted them by name as well. Using roles, we gain some abstraction if we were performing this on ten separate machines and save time by only typing one command.

`$ salt -G 'role:webserver' pkg.install "httpd"`

and for the database server:

`$ salt -G 'role:database1' pkg.install "mysql"

*Note:* I've used the package name of _httpd_ here, which is what the Apache web server is called in the default package manager for RedHat based systems. You'll need to replace the name of the package with the proper name for your operating system's distribution.

Take this real-world example that demonstrates a bit more of the power of  _grains_ in a more real-world scenario:

Suppose you were just notified of a security release for the Apache webserver from the CentOS base distribution, and you know you are using this on several of your webservers. You also are using Apache on some Debian based servers, but you don't want to update the package on them, since this security update only applies to Apache from the CentOS repos. Using _compound matching_ with _grains_ you can target only your webservers _and_ only those webservers that are CentOS or RedHat based systems like this:

`$ salt -C 'G@os_family:RedHat and G@roles:webserver' pkg.install "httpd"`

Using the `-C` flag I've instructed SALT to use compound targeting and by using _grains_ within my target, I targeted those systems that are RedHat based *and* have the _role_ of web server.

If you recall back to the introductory chapter, where I went over some basics of Python, 

Here is a sample return of this command:

    httpd:
        ----------
        new:
            2.2.15-28.el6.centos
        old:
            
    httpd-tools:
        ----------
        new:
            2.2.15-28.el6.centos
        old:
            2.2.15-26.el6.centos


## File Management

Most software that you install has configuration files associated it with it as well.

SALT has a solution for managing these files as well, the SALT File System.

Let's look at using the SALT File System now to manage a custom mysql configuration file, and distribute it to our minion.

You access the SALT File System by using the built-in _cp module_ like this:

`$ salt 'database1.example.com' cp.get_file salt://my.cnf /etc/my.cnf`

Let' break this command down now.

You've already seen the majority of this syntax already, but I'll reiterate here just for clarity.

The `salt` command should be recognizable now, as well as the target, in this case _database1.example.com_, our database server:

`$ salt 'database1.example.com'`

Next, `cp.get_file` uses the _cp_ module and the _get_file_ function.

Following that, `salt://my.cnf` is the location of the file on the _master_. Refer back to the _Master Configuration_ section in Chapter 0. You'll need to change your _master configuration_ now to tell SALT where your file roots are located.

This will be in the section containing _file_roots_, somewhere around line 256.

The default setting is most likely commented out, and looks like this:

    #file_roots:
    #  base:
    #    - /srv/salt

Simply delete the `#` symbols from the beginning of those lines to use `/srv/salt` as your file server location like this: 

    file_roots:
      base:
        - /srv/salt

After making these changes, it is a good idea to restart your _salt-master_ to ensure that the configuration changes are read.

`$ service salt-master restart`

Which should return:

    Stopping salt-master daemon:      [  OK  ]
    Starting salt-master daemon:      [  OK  ]

This tells SALT that we want to use the directory `/srv/salt` on our _master_ as the base for our file server.

The final section of our copy command contains the location on the _minion_ where we want to place our file, in this case `/etc/my.cnf`

Running this command now,

`$ salt 'database1.example.com' cp.get_file salt://my.cnf /etc/my.cnf`

will copy the local file, _my.cnf_ that is on your _master_ in `/srv/salt` to your minion(s) file-system at `/etc/my.cnf`.

and return

    database1.example.com
        /etc/my.cnf

SALT informs you that the file has been copied.

If we had multiple _minions_ that served as database servers, we could have just easily targeted them using the previously mentions _grains_ system like this:

`$ salt -G 'role:database' cp.get_file salt://my.cnf /etc/my.cnf`

or any other grains available to you.


## Testing Commands

Often you may not want to blindly implement a command, especially on a mission-critical production system. SALT offers a _test_ argument that you can supply to give you an idea of whether or not there are any hidden snags before initiating a command.

Simply add the `test=True` flag to your command, like this:

`$ salt '*' pkg.install "httpd" test=True`





