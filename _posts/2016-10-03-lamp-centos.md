---
title: "How to install LAMP Stack On CentOS 6"
categories:
  - blog
tags:
  - Linux_Basics
toc: true
---

This is a simple and straight forward tutorial to install and configure LAMP stack `(Linux, Apache, MySQL, PHP)` on a CentOS 6 system.

Before proceeding with further steps, please make sure that you have access to shell as a root or sudo user.

### Installing Apache

`Apache` is one of the most common and reliable Webserver. It helps you to serve files over `http/https` protocols.

- Install Apache using:
{% highlight bash %}
yum install httpd -y
{% endhighlight bash %}

- Start/Enable Apache using:

{% highlight bash %}
service httpd start
{% endhighlight %}

- Test and verify the Installation:

Open port 80 in `iptables`:
{% highlight bash %}
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
{% endhighlight %}

Open browser and type the URL `http://localhost` and It will show you apache default page.


### Installing mysql-server:

{% highlight bash %}
yum install mysql-server -y
{% endhighlight %}

- To set root password for `MySQL`:

{% highlight bash %}
/usr/bin/mysql_secure_installation

you will be prompted to enter current root password and you can leave it blank by pressing 'Enter'. 

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Then you will be prompted to provide new root password, there you can provide a custom strong password.

And you will have some more Yes/No questions, you can proceed with providing "Y" for all of them.

By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
... Success!

By default, MySQL comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!
{% endhighlight %}

- Start Mysql server using:
{% highlight bash %}
service mysqld start

{% endhighlight %}


### Installing PHP

`PHP` is the well known scripting language used to create dynamic web pages. To install PHP on your system,

{% highlight bash %}
yum -y install php
{% endhighlight %}

- If you want any custom php modules, you can find the same using:
{% highlight bash %}
yum search php *
{% endhighlight %}

- You can choose the exact name from the list and install it using:
{% highlight bash %}
yum install <module-name>
{% endhighlight %}

- To make this in effect restart the apache using:
{% highlight bash %}
service httpd restart
{% endhighlight %}

Yess...! you have successfully installed LAMP stack in your Server.

- To make apache and mysql run as soon as the machine boots, run the following commands:
{% highlight bash %}
chkconfig httpd on
chkconfig mysqld on
{% endhighlight %}

{% highlight bash %}
vi /var/www/html/info.php
{% endhighlight %}

- And insert the following code and save it:
{% highlight bash %}
<?php phpinfo(); ?>
{% endhighlight %}

- Test: Go to your browser and enter the following url: `http://1.2.3.4/info.php`
