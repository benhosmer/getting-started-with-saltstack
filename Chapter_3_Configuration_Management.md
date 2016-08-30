# Chapter 3: Configuration Management

As you've seen in the previous chapter, SALT can be used to run remote commands on any _minion_ that it is connected to, and manage files as well.

This can be very useful for things like querying their status, updating operating systems, and installing software.

In the simple examples before, we only installed two packages: the Apache web server, and the mysql database server. 

Often you'll need to install PHP along with a multitude of libraries for even a simple content management system like Drupal or Wordpress.

If you can imagine, your commands will become quite lengthy and hard to recall or troubleshoot when using remote execution to install something like a LAMP server.

This is where configuration management steps in and extends SALT even further.

Configurations are stored in simple text-files called _state files_. These files are written using the _YAML_ syntax, making them easy for humans to understand. 

This allows you to script the provisioning and installation of your particular systems. You can start from a bare-bones operating system, and have a complete web server up and running completely automatically.

For this next section, you may want to start with two fresh minions and configure them to work with your _master_

## Installing Packages Using State Files

We'll continue our fictional story of Angie the Systems Administrator, but this time she was instructed to set up a new Web and Database Server to host her company's new website.

She's done this before, and has gone through the tedium of installing Apache, MySQL, PHP and the other PHP extensions needed for their content management system. She has a working web server to use as a guide and with some configuration files for each.

Let's run down the packages and files she will need for this:

The Database Server requires this: 

1. MySQL:
    - A custom `my.cnf` file to configure the database server.

The Webserver will require this:

1. PHP:
    - A `php.ini` file to configure php.

2. php-mysql:
    - A PHP library to utilize the MySQL database with PHP.

As you probably see, our simple configuration is starting to become more complex. Complex enough that we probably don't want to enter each command separately on the command-line either.

Since you previously configured your `file_roots` in Chapter 2, you've got the basic layout for storing your state files already. If you haven't, go back now and review that section since we'll be using it now.

### The Webserver

Let's start with installing Apache, and copying the php.ini file to its proper place on our web server _minion_.

Within your _master's_ `/srv/salt` directory, create a new directory called `apache`.

Now in your favorite text editor, create an `init.sls` file in the `/srv/salt/apache` directory.

This _state file_ will contain all of the instructions we need for installing and configuration the Apache web server software. The name of the directory is arbitrary, in this case `/srv/salt/apache`, but it is a good idea to name it something that is easy for you to remember and that describes the instructions contained within it.

Add the following to this new file:

    httpd:
    [2]pkg.installed

In the example above, I've added a `[2]` to denote spaces before the `pkg. installed` directives for clarity. The actual entry won't contain this and will look like this:

    httpd:
      pkg.installed

This is _YAML_ syntax, which is sensitive to the spacing in front of each directive. Just remember to always include two spaces before your directives.

This is a very specific directive, in that it refers to the Apache web server package as _httpd_. This state would only be useful on a RedHat based system, since the package is often called _apache2_ in Debian based systems.

As your systems become more complex, you'll find it easier to refer to your state directives in a more logical and human comprehensible fashion, like this:

       apache:
          pkg:
            - name: httpd
            - installed

You can name each directive something that makes sense to you, and then use `name` to refer to the specific item. In this example, I've changed the name of the directive simply to _apache_, after the web server software that it installs.

Save this file and test it now with

`$ salt 'webserver1.example.com` state.sls apache`

The syntax should start becoming more familiar, and I won't go over the command or targeting this time.

I have introduced a new function though, the `state.sls` function. This is useful for testing, and for executing a single state file on a _minion_ or multiple _minions_.

Following the `state.sls` function is the name of the state file you want to execute, in this case our `apache` state file.

Executing this now should result in the following:

    webserver1.example.com:
    ----------
        State: - pkg
        Name:      httpd
        Function:  installed
            Result:    True
            Comment:   The following packages were installed/updated: httpd.
            Changes:   httpd: { new : 2.2.15-29.el6.centos
    old : 
    }

Your versions might differ, since they have probably been updated since the writing of this book.

**Note**
You can install multiple packages at once with the `pkgs` list like this:

    java-packages:
      pkg.installed
        - pkgs:
          - java-1.7.0-openjdk
          - haveged
          - tomcat


## Managing Files with State Files

The second part of our web server setup involves copying the system specific configuration files for the Apache web server to the minion.

For this example, we'll use an `httpd.conf` file for Apache. 

Place this file within the `/srv/salt/apache` folder.

If you closed your `/srv/salt/apache/init.sls` file, reopen it now. We're going to add directives to manage our custom `httpd.conf` file now.

We accomplish this with the built-in `file` module, and the `managed` function included with SALT.

After the apache entry in your `init.sls` file, add the following:

    apache-config:
      file:
        - managed
        - source: salt://apache/httpd.conf
        - name: /etc/httpd/httpd.conf

Save your `init.sls` file now and execute your state:

`# salt 'webserver1.example.com' state.sls apache`

    webserver1.example.com:
    ----------
        State: - file
        Name:      /etc/httpd/httpd.conf
        Function:  managed
            Result:    True
            Comment:   File /etc/httpd/httpd.conf updated
            Changes:   diff: New file
                       
    ----------
        State: - pkg
        Name:      httpd
        Function:  installed
            Result:    True
            Comment:   Package httpd is already installed
            Changes:

## Using the grains System within State Files
Building upon this, we can use the _grains_ system in our state files to target multiple operating systems.

Use jinja syntax to add grains to your state file like this:

    apache:
      pkg:
      {% if grains['os'] == 'Ubuntu' %}
        - name: apache2
      {% elif grains['os_family'] == 'Redhat' %}
        - name: httpd
      {% endif %}
        - installed

This gives us a much more generic and reusable state file, that can be used for Debian and RedHat based systems.

Save this file now, and let's test it again:

`# salt 'webserver1.example.com' state.sls apache`

If you're using a Debian based system, the `apache2` package should have been installed, or for RedHat, the `httpd` package.

## The top files

Up to this point, we've applied our states manually using the `state.sls` function. As you can imagine though, this could become tedious if you wanted to create several identical webservers on different machines, or have multiple states applied to one machine.

In the context of SALT's state system, the `top.sls` file maps states to minions. It is located in the `/srv/salt` directory. In the next section, I'll talk about `pillar`, but for now, it also works in a similar fashion as the state system's top file, but maps pillar values to minions.

The `top.sls` should exist on the _master_, or the _minion_ if you're using SALT in a _masterless_ configuration. Either way, the machine you are initiating the `salt` or `salt-call` on should have the `top.sls` and _state files_ present.

Our previous example installed the webserver, php and packages on one machine. This likely isn't ideal for performance or fail-over, and we still
need a database as well.

We'll set our directory up like this:

    /srv/salt/
    ├── nginx
    │   ├── nginx.conf
    │   └── init.sls
    ├── mysql/
    │   └── init.sls
    ├── users/
    │   └── init.sls
    ├── top.sls

To better emulate production, our system will have two servers. One will be the webserver with NGINX and PHP installed. The other will be our Database server with MariaDB installed. This is in the _dev_ environment, so we'll also let other developers login to troubleshoot bugs. We'll manage them through a `users` state. This also allows us to create reusable states that can be applied across environments.

To complete this exercise, you'll need two separate minions. One with the id of `webserver.dev` and one with an id of `db.dev`.

Let's look at the `top.sls` file we'll need to accomplish this:

    base:
      '*':
        - users
      'webserver.dev':
        - nginx
      'db.dev':
        - mysql

With the

    base:
      '*':
        - users

we've ensured _every_ machine has the `users` state applied to it. This let's us create and maintain one `users` state and apply it to multiple machines without any additional changes.

Next, the `'webserver.dev'` minion gets the `nginx` state applied to it. This targets the _minion_ by its id, `webserver.dev`. Finally, we can apply the `mysql` state to the `db.dev` minion.

*@TODO*
- Explain using `highstate` now that we have a top file
- Explain using `test=True` to ensure everything works as expected

## Pillars
*Placeholder for Pillar Section*

## Troubleshooting
*Placeholder for tips for troubleshooting*
- `salt-call state.show_sls mystate`
- `salt-call state.show_highstate`
- Explain using `--log-level debug`
