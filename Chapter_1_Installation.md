# Chapter 1: Installation and Configuration

## Installation

Since SALTSTACK is available for a wide-range of operating systems, I'll detail briefly what is needed to install SALT and then refer you to the [official] documentation for further information.

SALTSTACK is available in most major linux distribution's repositories. SALTSTACK is also available for FreeBSD, Windows, Solaris, and Ubuntu Phone as well!

The recommended way to install SALTSTACK is to use your package manager. You also can install SALTSTACK from source by cloning the [SALTSTACK Github] repositories. Instructions are available in the repositories regarding the dependecies required prior to installation from source.

Often, you may want to use a newer version of SALT than is available currently from your distributions repositories. An easy way to install the dependencies first is to use your package manager to install whatever version of SALTSTACK is available, and then build the version you want from source.

SALT also has a fully automated shell-script that makes it even easier to install, if you have access to an outside network.

Using wget, you can install SALT with one command: `wget -O - http://bootstrap.saltstack.org | sudo sh`

You can find more information about [SALT Bootstrap] on the project page.

## Configuration

Throughout this book, I'll often demonstrate a SALT command using the _salt-call_ command and also describe it using the _salt_ command too. _salt-call_ is often used during testing and it allows you to use SALT on a single machine without setting up a master.

You can very quickly get a basic SALTSTACK system running by installing just the salt-minion within a virtual-machine. This is a great way to test your state files and learn SALT without needing to set up a separate master and minion. You can also run SALT with the master and minion on the same machine if you like.

### Master Configuration

Throughout the SALTSTACK documentation, and this book, the _master_ is referred to as the main controlling machine that instructs the _minions_. _Minions_ are other individual machines that receive commands from the master. Your configuration files, known as state files, exist on the master as well as any other files that you want to transfer to the _minions_.

For the remainder of this book, I would recommend creating a new virtual-machine with your favorite operating system to use as a testing environment.

Using your package manger, or _salt-bootstrap_, install the _salt-master_ package now.

In a typical linux-based SALTSTACK installation, you'll find the configuration files in `/etc/salt` and the _master_ configuration file appropriately named `master` within that directory. A _minion's_ configuration file will be named `minion` within the same `/etc/salt` folder.

The default configuration file is well-documented and you can probably leave it as it is. You may want to read through it though just to familiarize yourself with the various options available.

Now that the _salt-master_ has been installed, you may need to start it. Depending on your operating system, you may need to use `systemctl salt-master start` or `service salt-master start`.

## Minion Configuration

For this section of this book, go ahead and install the `salt-minion` using your package manager on the same machine that you installed the `salt-master` package on.

Before starting it, you'll need to make a few edits to the _minion_ configuration file located within `/etc/salt/minion` first.

Again, this configuration file is well documented.

Open the `minion` file in your favorite text-editor and locate the line `master:`. This line tells the _minion_ where the _master_ is located.

Since you are using the same machine for both the _minion_ and the _master_, locate the ip address of your machine and add it. Your configuration file should look something like this now:

'' `master: 192.168.1.33`

Most likely your ip address will be different.

Now start the `salt-minion` using whatever method your system supports. I'll use `service salt-minion start` for this example.

Now with both the minion and master started and running you have successfully installed SALTSTACK! You are now ready to take control of your infrastructure with a powerful and fast remote execution and configuration management system. 

If you were using two separate systems, you could now log out of your minion and probably never need to log in to it again. After the next section on _Salt Keys_, you'll have full-control of your _minion_ from your master.

## Salt Keys

Before you can do anything with a newly installed _minion_, you'll need to accept its keys on the _master_ first.

This is accomplished with the `salt-key` command.

Again, for this section your _minion_ and _master_ are the same machine, but later when you begin to actually use SALT, they will be on different machines. The `salt-key` command would be intitiated on the _master_

Log in to your master if you aren't already, and type the following:

`salt-key -L`

This command will list all of the accepted, rejected, and pending keys within your _master_.

Hopefully you see a pending key for your newly installed _minion_.

## Conclusion

You've now completed the installation of SALT and configured your systems to communicate with one-another. You should have a working SALT system to continue to learn on.

You can test the connection now with the `test.ping` command like this:

`# salt '*' test.ping`

Which should return:

	local:
		True

If you used `salt-call`, or:

	yourminionname:
		True

if you're using a _master_ and _minion_ configuration.

