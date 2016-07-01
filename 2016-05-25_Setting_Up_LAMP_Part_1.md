Setting up a LAMP stack on a Raspberry Pi - Part 1:
==
Local Intranet Site
--

The Raspberry Pi is great, we all know that. But, what is it great for? I have found that it can be a really efficient, low-cost, personal web server. With this running, you can host anything from a blog, to a personal page, to an online résumé without worrying about your data being stored on somebody else's servers. This is the first in a two part series, and will cover setting up a local web-server accessible through your local network only. It will get your server up off the ground, but it won't dive into hardening it against attacks or opening it up to the world. In part 2, we will go over how to make your server accessible over the internet, as well as how to make it secure and stop attackers from getting in. Let's start in on setting up your LAMP stack.

Prerequisites
--

So, wait, what exactly *is* a LAMP stack? Well, LAMP is an acronym for **L**inux, **A**pache, **M**ySQL, **P**HP. These are platforms commonly used for hosting websites of different designs. Linux is the base operating system, Apache is the actual webserver that serves content, MySQL is a database system, and PHP is a language for generating and serving dynamic content. A LAMP stack is a combination of these technologies to get the skeleton of a webserver down. Everything else can be built on top of this once you decide what you want. To get this all started, you'll need some basic materials:
* A Raspberry Pi board - The newer boards are, of course, better suited to this because of their higher specs, but the older boards will work too; especially if you plan on just hosting a simple site without much dynamic content.
* An SD card with enough space - Again, this depends on what you plan on doing with your server. An 8GB card should be a good middle ground if you want to just experiment with different things
* An ISO or image file of the OS you plan to use - Since you're setting up a webserver, I would suggest you [download](https://www.raspberrypi.org/downloads/raspbian/) a headless version of Raspbian. Headless means that it was designed to not be hooked up to a graphical display. This will give you more storage space, as none of those bulky graphical applications are installed.
* An Internet connection - This is self-explanatory ;p
* SSH access to the Pi - You can use PuTTY on Windows, SSH on Linux or Mac

L is for Linux
--

**Note:** If you already have a version of Raspbian installed, you can skip this section entirely and move down to "A is for Apache".

Alright, so after you've downloaded your preferred distribution of Raspbian, you'll need to get it installed. We won't be covering NOOBS here, as that is a fairly straight-forward process. Instead, we'll assume you downloaded a file that ends in .iso or .img. These types of files are called disk images, and they hold a copy of the entire operating system in them. If you're on Windows, you'll want to use a tool called [Rufus](https://rufus.akeo.ie/) to copy the image onto your card. The standard guide suggests using wind32diskimager. This is a good tool, too. However, I have had issues with it in the past as far as it not copying over the entire image and exiting early. Also, Rufus has a portable version which requires no installation. Ultimately, it is up to you which tool you choose. However, if you're on Linux or Macintosh, you'll want to open up a terminal. You'll need to figure out what device file your SD card is identified by. Usually, it will be `/dev/mmcblk0`, but it might be different depending on your system. To see a list of attached devices, you can use `lsblk`. If your card is split into multiple partitions, you might see entries like `mmcblk0p1` or `mmcblk0p2`. That is okay, but you don't want to use those. When issuing the next command, you'll want to make sure that you are using the top-level identifier (the one without a `p` in it):

    sudo dd if=/path/to/raspbian.img of=/dev/mmcblk0
    
Please note: It is ***vitally important that you do not make any errors in typing the above command***. `sudo` is always a dangerous tool to work with because of the fact that it gives you super-user access to the system. Additionally, `dd` can be very dangerous if accidentally specify the wrong input or output files. So, please, do not copy paste. Instead, double check the device your card is using and the path to your image file. I will not be blamed or take fault if you make an error and break your system. You have been warned.

After your image file is copied to your card, go ahead and pop the card into your Pi and power it on. Then, you'll have to SSH into it. I'm going to assume here that you know the basics of accessing a Pi through SSH. After you log-in, you'll want to make sure you take the vitally important step of changing the default password. You can do this by simply running `passwd`.

A is for Apache
--

Now that you've logged in, you'll want to get started on installing quite a few things. Since all installations require administrator rights, we're going to go ahead and drop down into a `root` prompt.

    sudo -i
    
This is the danger zone, so be careful. You are now in a log-in shell for the user `root`. Now you don't have to type `sudo` in front of everything, so that's a bit more convenient. However, it is also dangerous, as stated above. Just be mindful. If you'd like to stay safe, then **don't** drop into this mode and just append `sudo` to the beginning of each of the commands from here on out.

The first thing you need to install is the Apache webserver. This part of the walkthrough is pretty straight forward and doesn't require much.

    apt-get update && apt-get upgrade
    apt-get install apache2
    
That's it! To make sure it's working, open up a web browser and type in the IP of your Pi (the one you used for establishing the SSH connection). You should see a page that says "It works!". Apache is now installed. However, you don't want to just display this page, right? Let's go a head and set-up an Apache virtual host to start serving your own content.

    mkdir /var/www/html/mysite #You can change "mysite" to anything you want
    chown -R www-data: /var/www/html/mysite #This makes it so that the directory is owned by the webserver user
    chmod -R 755 /var/www/html/mysite #This sets more secure permissions on the folder
    
Now, you'll want to create an actual config file for Apache to use. First, disable this default site, as we will be making changes to it.

    a2dissite 000-default
    service apache2 reload
    
Next, open up the file `/etc/apache2/sites-available/000-default.conf in your favorite text-editor. Delete everything that's in there and copy/paste the contents below and make the necessary modifications, or compare the current contents to what I have below and see for yourself how to make the changes:

```apache
<VirtualHost *:80>

        #ServerName is the domain name that will trigger this config file. We will go into this more in part 2. For now, you can set it to whatever you want, as you will be accessing this site through IP only.
        ServerName yourdomain.tld
        #ServerAlias is optional. Again, we will cover more of this in part 2.
        ServerAlias www.yourdomain.tld
        #ServerAdmin is the email address of the server administrator. This won't work quite yet, but we'll leave it in here so that we can talk about how to make it work in part 2
        ServerAdmin you@email.com
        #DocumentRoot is the folder where your website files are stored.
        DocumentRoot /var/www/html/mysite

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

Save that file as is. Now, tell Apache to use that site:

    a2ensite 000-default
    service apache2 reload
    
Now, open up your web browser and try navigating to your pi's IP again. Since you're no longer using the original file, you'll just see a plain page listing the nonexistent files. You can add your web site files into here later on, and they'll be displayed.

M is for MySQL
--

MySQL is a database system. If you plan on hosting a blog, a shop, or something like ownCloud, you'll need a database back-end. To install the base of it, you'll need to:

    apt-get install mysql-server
    
During this set-up process, you'll be asked to enter a root password. This is a new password, not the password for the `root` user of Raspbian. Go ahead and enter any secure-yet-memorable password you like. Then, you'll need to lock down the installation of MySQL so that it's harder for hackers to get in there and mess with things.

    mysql_secure_installation

This will prompt you for the MySQL root password you just set. If it asks if you wish to change, go ahead and hit `N` as you've already set a custom password. Then, it will ask you some more questions. It's okay to press `Y` for each of these. This process removes some test users and other default settings which could be used to exploit your server if left around.

P is for PHP
--

Finishing up our journey! Again, this part of the guide is pretty easy.

    apt-get install php5 php-pear php5-mysql
    service apache2 restart
    
This installs the PHP5 base, the PHP extension/application repository, and the MySQL extension so that PHP can work with MySQL. To make sure it works, go ahead and open a blank file in your favorite text editor again. You can safely copy and paste the following excerpt without any modifications:

```php
<?php
phpinfo();
?>
```

Save that as `/var/www/html/mysite/info.php`. Then, open up your browser and go to `http://your.pi.ip/info.php` and you should see the PHP info page that tells you information about your system. You're done! After verifying that PHP works, it will be very wise of you to delete the `info.php` file you just created, as it gives away a lot of information. If it's still there when we get this opened up to the internet, an attacker will have most of what they need to know about your system to exploit it.

Finished!
--

You did it! This web-server can now serve any type of file to computers in your local network. This set-up would be good for a home intranet site, or to provide you some sort of local gateway to controlling your connected devices. Stay tuned for part 2, in which we will cover hardening the server and making it accessible to the world!
