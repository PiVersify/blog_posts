# SSH Security and Usability – Part 5 #
**Author Steve Robillard**

In the previous post in this series we covered:

- [Passwordless logins on Windows](https://raspberrypise.tumblr.com/post/148032481829/ssh-security-and-usability-part-1),
- [Passwordless logins on Linux](https://raspberrypise.tumblr.com/post/148401541394/ssh-security-and-usability-part-2),
- [Creation of a custom login messages and SSH Prompt](https://raspberrypise.tumblr.com/post/148725075714/ssh-security-and-usability-part-3), and
- [Securing our keys and agent](https://raspberrypise.tumblr.com/post/149429965524/ssh-security-and-usability-part-4).

In today’s post we will look at hardening the server by changing the server’s configuration. 

## SSH Server Configuration ##

We can improve the security of our SSH server by modifying its configuration file **/etc/ssh/ssh_config**. We have already seen an example of this when we setup passwordless logins. Today we will look at changing the port number, logging, limiting logins and forwarding.

**Note:** To prevent being locked out of the server, due to a configuration error, you should open a second connection, or better still, have physical access to the server via a monitor and keyboard. 

**Tip:** If not using [etckeeper](https://raspberrypise.tumblr.com/post/146996830774/tracking-your-systems-configuration-changes-with), you should make a copy of the file (**/etc/ssh/sshd_config**) before editing using the following command: 
  
**mv /etc/ssh/sshd_config /etc/ssh/sshd_config.bak** 

### Change the Port Number ###

By default SSH listens on port 22. If your SSH server is exposed to the internet, it is only a matter of time, before you start to see probes of your SSH server in your logs. You do proactively monitor your logs don’t you? You can’t eliminate this entirely, but you can eliminate many automated scans by changing the port SSH listens on. 

**Note:** this is a case of [security through obscurity]( https://en.wikipedia.org/wiki/Security_through_obscurity), a determined attacker will be able to locate the new port via a thorough reconnaissance. This, however, is not the only measure we will rely on to protect our SSH server, and is primarily used to improve the signal to noise ratio of our logs. 

To change the port number:

open the **/etc/ssh/sshd_config** file in your editor:

**sudo nano –c /etc/ssh/sshd_config**

**Tip:** the **–c** flag enables the display of line numbers.

Locate the following (line 5):

**Port 22**

And change it to: 

**Port 2332**

**Note:** you can change this to any unused port. You can see which ports are in use with the following command: 

**netstat –lat**

**Note:** many tutorials suggest port numbers 222 and 2222; I avoid these as they are becoming more common in automated probes.

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Restart the SSH service:

**sudo service ssh restart**

Test the new config by opening a new connection to the server.

**Note:** don’t forget to update the port number in putty or your **~/.ssh/config** file, and modify your firewall as needed. 

Update your **/etc/services** file. This file maps services to ports and protocols.

Note: this will be needed when we firewall our SSH server in the next post. 

open the /etc/services file in your editor:

**sudo nano –c /etc/services**

Locate the following line:

**ssh  22/tcp  # SSH Remote Login Protocol**

and change it to:

**ssh  2332/tcp  # SSH Remote Login Protocol**

**Note:** there is also an entry for udp. This is historical and does not need to be changed.

c

**Tip:** If you have more than one network interface (e.g. Ethernet and WiFi as on the Pi 3) and are using a static IP, you can limit which interface SSH listens on by uncommenting and editing the **Listen Address** rule at the top of the **/etc/ssh/sshd_config** file (e.g. **ListenAddress 192.168.1.240**). 

### Logging ###

By default SSH logs to the AUTH facility of syslog, with log level info. To log failed login attempts and to make troubleshooting SSH issues easier you should change this to DEBUG. If your SSH server is exposed to the internet you may be surprised at just how many probes you receive.

To change the logging level:

open the **/etc/ssh/sshd_config** file in your editor:

**sudo nano –c /etc/ssh/sshd_config**

Locate the following (line 24):

**LogLevel INFO**

And change it to: 

**LogLevel DEBUG**

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Restart the SSH server:

**sudo service ssh restart**

Logs are useless if no one reads or monitors them, but reading hundreds of lines of logs is not only tedious, but risks missing import events in a sea of data. One way to monitor your logs for important events (e.g. logins and the use of sudo) is via logwatch. Logwatch will monitor your logs for events of interest and email you a report once a day. 

To install logwatch:

**sudo apt update**<br />
**sudo apt install logwatch**

You can verify logwatch works by doing a manual run:

**logwatch**

**Note:** this will print the report to the screen rather than emailing it. 

Note: You may also need to change this line:

**/usr/sbin/logwatch --output mail**  

to:

**sudo /usr/sbin/logwatch --mailto pi@localhost**

in the **/etc/cron/daily/00logwatch** file.

Beyond the scope of this post, there is much more you can do with your logs, to make them more useful and improve security. You can:


- Import them into [splunk](http://www.splunk.com/) which will allow searching and querying your log files. 
- Send them to an external logging service like [papertrail](https://papertrailapp.com/) or [loggly](https://www.loggly.com/). This will keep a copy that cannot be easily rewritten to hide an attack. Given enough time an attacker can bypass this, but it will be hard to hide the earliest steps taken by the attacker – which are often the most important. 
- Archive them to a cloud service like Amazon’s [AWS](https://aws.amazon.com/) S3 or Glacier.

### Limiting Logins ###

By default SSH permits root logins. This should be disabled. You will still have access to the root account by logging in as a normal user and using sudo to gain root privileges. 

To disable root logins:

open the **/etc/ssh/sshd_config** file in your editor:

**sudo nano –c /etc/ssh/sshd_config**

Locate the following (line 31):

**PermitRootLogin without-password**

And change it to: 

**PermitRootLogin no**

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Restart the SSH server:

**sudo service ssh restart**

You can also allow or deny logins to specific users and groups using the DenyUsers, AllowUsers, DenyGroups, AllowGroups settings. These rules are processed in the order listed. I create a second admin user on my Pi – with all the same permissions and groups and grant them access via SSH, then block the pi user. I then assume anyone trying to login as the pi user is an attacker and block them. This approach has two benefits it can be identified easily in the logs and can be automated.
 
To limit which users can login via SSH:

open the **/etc/ssh/sshd_config file** in your editor:

**sudo nano –c /etc/ssh/sshd_config**

Add these lines to the end of the file:

**DenyUsers pi**<br />
**AllowUsers srobillard**

changing the username to match your setup.

**Note:** you can go further and limit the IP address they can connect from by adding the and **@** and IP to the rule (e.g. **AllowUsers srobillard@192.168.2.255** or to permit a subnet **AllowUsers srobillard@192.168.2.***)

**Note:** you could likewise add DenyGroups and AllowGroups rules to the end of this file.

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Restart the SSH server:

**sudo service ssh restart**

**Tip:** you can rate limit connections and sessions which can slow down brute force attempts and help ameliorate a DOS attack. For more information see the MaxSessions and MaxStartups entries in the man page (**man sshd_config 5**).

### Forwarding ###

SSH can forward a graphical application, or a port. These features can be very useful, but if unneeded they should be disabled. 

To disable forwarding:

open the **/etc/ssh/sshd_config** file in your editor:

**sudo nano –c /etc/ssh/sshd_config**

Locate the following line (line 64):

**X11Forwarding yes**

And change it to:

**X11Forwarding no**

Locate the following line (line 65):

**X11DisplayOffset 10**

Add this line after the above line:

**AllowTcpForwarding no**

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Restart the SSH server:

**sudo service ssh restart**

### Disconnect Idle Sessions ###

SSH allows you to set an idle timeout period (in seconds), after which idle sessions will be disconnected. 

To disconnect idle sessions after five minutes:

open the **/etc/ssh/sshd_config** file in your editor:

**sudo nano –c /etc/ssh/sshd_config**

and add the following lines to the end of the file:

**ClientAliveInterval 300
ClientAliveCountMax 0**

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Restart the SSH server:

**sudo service ssh restart**

I have not mentioned several other security related settings (e.g. PermitEmptyPasswords, HostBasedAuthentication and IgnoreRHosts), because recent versions of SSH have safe default values. To see all the settings available and their default values consult the man page (**man sshd_config 5**)
 
Next time we will look at network security.
