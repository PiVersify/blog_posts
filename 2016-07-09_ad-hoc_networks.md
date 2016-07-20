# Ad Hoc Networking with Raspberry Pis

The current trends in [Internet of Things](https://en.wikipedia.org/wiki/Internet_of_things) lets the users connect many things over the Internet. This makes it simpler to obtain a lot of useful information without having to physically approach the devices. You can connect these so called *Smart-Objects* over a Wireless Medium like your WiFi and play around with the Objects. This blog post would help the new __IoT__ Enthusiasts connecting their Raspberry Pis over WiFi without a Router. 

[Ad-Hoc Networks](https://en.wikipedia.org/wiki/Wireless_ad_hoc_network) are decentralized networks, this means you do not need a central Router or Access-Points like the __Infrastructure Networks__. In a nutshell, every device (here, Raspberry Pis) is its own *Boss*. The great thing about being Ad-Hoc is the whole network becomes very flexible. 

An important fact for __IoT__ is that as the devices are going to be very large in quantity, and in order to provide connectivity to all of them, __IPv6 Addresses__ are preferred over __IPv4 Addresses__. This post also supports using __IPv6 Addresses__ on Raspberry Pis.

## Requirements

* __Hardware__ : Raspberry Pi 2 Model B

* __Firmware__ : Debian Wheezy 7.10 / 7.11

* __USB WiFi Adapter__ : The current post supports using [LogiLink WL0145A](http://logilink.de/Suche/WL0145A) adapters. In general any USB Adapters with __Ralink RT5370 Driver__ in them.

An easy way to determine this is when the Adapter is plugged in the Raspberry Pi and on the Terminal:

    $ lsusb | grep -i "ralink"
    Bus 001 Device 004: ID 148f:5370 Ralink Technology, Corp. RT5370 Wireless Adapter

* __Power Supply__ : WiFi Adapters generally need a proper power adapters. In general __2 A__ current ratings are essential.

Generally, such Ratings are preferred:

> Input: 100-240 V ~ 50/60 Hz 0.3A Output: 5 V - 2.1A / 5.2 V - 2.1A

[[images/setup1.jpg]]

## Setup

1. Firstly backup your *original* interfaces file (Better be safe than sorry!)

        $ cp /etc/network/interfaces /etc/network/interfaces.backup

2. Keep your Ethernet setting the way you like, for any Firmware/System Updates. Change your WiFi settings.

    
        #your /etc/network/interfaces file

        auto lo
        iface eth0 inet dhcp

        auto wlan0
        allow hot-plug wlan0


3. Add the `ipv6` Kernel Module on startup.

        $ sudo nano /etc/modules

        ## your /etc/modules file

        # add ipv6 module here
        ipv6


4. Enable __SSH__ if not already configured.

        $ sudo raspi-config

    * In __Advanced Options__ -> __SSH__ -> __Enable__


5. Automatically connect to the Pi to an Ad-Hoc network on Pi Bootup.

		$ sudo nano /etc/rc.local

		## inside your rc.local file

		adHocNetwork() {

			echo "Creating AdHoc Network"
			echo "Setting Network Parameters"

			ifconfig wlan0 down
			iwconfig wlan0 channel 6 essid name-of-network mode ad-hoc
			ifconfig wlan0 up
		}

		adHocNetwork
		exit 0
the `name-of-network` can be the name of your choice, `channel` can be value between __1 to 11__ but please take care of any Access-Points near you. Do not take the same channel number as the Access-Points. 

	__Hint__ : Use any one of the following [1, 6, 11] channel numbers if other two are occupied. This is beneficial to provide less intereference

6. Reboot your Pi.

		$ sudo reboot



Once the Pi has been rebooted check your `wlan0` settings using the following:

	$ ifconfig wlan0
	wlan0 Link encap:Ethernet  HWaddr aa:bb:cc:dd:ee:ff  <===== Your MAC Address
    inet6 addr: fe80::axbb:ccff:fedd:eeff/64 Scope:Link  <==== Your Link-Layer Address is up (derived from MAC addr)
    UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

Here we are using [IPv6 Link-Local Addresses](https://en.wikipedia.org/wiki/Link-local_address#IPv6). This addresses are unique to every WiFi Adapter connected to every Pi.

Now continue these steps on your other Pis and create a cool Ad-Hoc network !!

## Connectivity

If you used __IPv4__ and __DHCP__ before, you could only ping the devices if they have the IP Addresses from DHCP Server. But this Ad-Hoc Network needs no such DHCP server.

You can ping the Pis using the following command:

    $ ping6 -I wlan0 fe80::axbb:ccff:fedd:eeff

You should be able to ping your Pis easily now!

You can now move your Pis without worrying about cables since your are wirelessly connected. No need to worry if your DHCP Server fails, you don't need one!

## Ease of Usage

The __IPv6 Link-Local Addresses__ are bit tedious to remember hence it is easier to use a __mDNS__ service.

1. change your system conf settings

		$ sudo nano /etc/sysctl.conf

		## add following lines
		net.ipv6.conf.all.autoconf = 1

		# add for wlan0 interface 
		net.ipv6.conf.wlan0.autoconf = 1

2. Install `avahi-daemon` on Pis

		$ sudo apt-get install avahi-daemon

3. Edit the following files for configuring your mDNS avahi-daemon

		$ sudo nano /etc/avahi/avahi-daemon.conf

		## change this line from `no` to `yes`
		use-ipv6 = yes


		$ sudo nano /etc/nsswitch.conf

		## in hosts section:

		files mdns_minimal [NOTFOUND=return] dns mdns

		$ sudo avahi-daemon -r

4. Change your Pi name:

		$ sudo nano /etc/hosts

		127.0.0.1       localhost
		::1             localhost ip6-localhost ip6-loopback
		fe00::0         ip6-localnet
		ff00::0         ip6-mcastprefix
		ff02::1         ip6-allnodes
		ff02::2         ip6-allrouters

		127.0.1.1       raspberrypi

	Change the Last line `127.0.1.1 raspberrypi` to `127.0.1.1  millenialfalcon`

5. Change the Hostname of Pi:

		$ sudo echo 'millenialfalcon' > /etc/hostname

	Commit the changes using :

		$ sudo /etc/init.d/hostname.sh


Finally reboot your Pi and ping you *Millenial Falcon* from other device using the following:

	$ ping6 -I wlan0 millenialfalcon.local

## Control of Pis

You can now run your work on these Pis and most common functionality with Pis are __SSH__ and __SCP__.

Unfortunately the __mDNS__ names are unavailable for the __SSH__ and __SCP__.

### __SSH__

In order to avoid typing your Pi's password everytime, it preferred to make all your Pis in the network Headless. 

The Official Raspberry Pi [Documentation for Headless SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/passwordless.md) will help you estabilish a passwordless entry to your Pi.

__Hint__ : After the SSH-Key is generated you can share the key directly with the Pi using :

	$ ssh-copy-id pi@fe80::axbb:ccff:fedd:eeff%wlan0

You will be asked to give the password once and then next time you will SSH into the Pi without a password.

	$ ssh pi@fe80::axbb:ccff:fedd:eeff%wlan0

__NOTE__: please remember to use `%wlan0` at the end of __IPv6 Address__ that is important.

### __SCP__

__SCP__ is used to share files between your Pis.

In order to send a file to a Pi:

	$ scp myFile pi@[fe80::axbb:ccff:fedd:eeff%wlan0]: /home/pi/your/folder

__Note__: You need the `[]` around your IPv6 Addresses or else there will be errors.


Hope this blog post covers the majority of things. Good Luck with creating fun Projects in Ad-Hoc Networks
