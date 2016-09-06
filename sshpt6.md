# SSH Security and Usability – Part 6 #
**Author Steve Robillard**

- [Passwordless logins on Windows](https://raspberrypise.tumblr.com/post/148032481829/ssh-security-and-usability-part-1),
- [Passwordless logins on Linux](https://raspberrypise.tumblr.com/post/148401541394/ssh-security-and-usability-part-2),
- [Creation of a custom login messages and SSH Prompt](https://raspberrypise.tumblr.com/post/148725075714/ssh-security-and-usability-part-3), and
- [Securing our keys and agent](https://raspberrypise.tumblr.com/post/149429965524/ssh-security-and-usability-part-4).
- [SSH server configuration](https://raspberrypise.tumblr.com/post/149800066154/ssh-security-and-usability-part-5)

In today’s post we will look at network security using UFW, Fail2ban and TCP Wrappers.

## UFW ##

UFW stands for uncomplicated firewall, and is a simple front end to iptables. You can allow or deny network traffic based on source, destination, host, port, protocol and service. 

**Note:** there is a GUI for ufw called gufw. It can be installed with the following command: **sudo apt install gufw**. However, its use will not be covered in this post.

To install ufw:

**sudo apt install ufw**

Unlike most services (e.g. Apache, Etckeeper) ufw is not enabled by default after installation. 

**Note:** To prevent being locked out of the server, due to a configuration error, you should perform these actions using a keyboard and monitor connected directly to the server. 

To enable ufw:

**sudo ufw enable**

**Note:** The enable command warns you that your SSH connection may be disrupted by this and asks you to confirm the action. If this happens you will need physical access to login and reestablish SSH access. 

UFW comes with a default profile, which blocks most incoming traffic and provides a good starting point. You can check the status of ufw and view the default rules with the following command:

**sudo ufw status verbose**

However, this default profile will not allow connections on the custom SSH port (2332) we set in the last post. 

To allow SSH traffic to the new port you can use the following command:

**sudo ufw allow 2332**

Check the new rule:

**sudo ufw status verbose**

**Note:** ufw will automatically generate two rules one for IPv4 and one for IPv6.

This rule will work, but it opens a bigger hole than is necessary. When it comes to security you want to apply every possible constraint to limit your attack surface. One problem with the above rule is that it allows UDP connections to our custom SSH port (2332). As I mentioned in the last post SSH only uses TCP. 

Remove the previous rule:

**sudo ufw delete allow 2332**

**Note:** to delete an existing rule add "**delete**" after ufw to the command you used to add the rule, or use the following command to list the rules by number:

**sudo ufw status numbered**

and then remove the rule with the following command: 

**sudo ufw delete #**

Replacing the **#** in the above command with the number of the rule you want to delete.

You can verify the change with the following command: 

**sudo ufw status verbose**

We can replace the previous rule with:

**sudo ufw allow 2332/tcp**

and verify it with: 

**sudo ufw status verbose**

You should now be able to establish a new SSH connection to your server using port 2332. If you only need access to your server from a specific IP address or range of IP’s you can replace the above rule with one that specifies which IP address or range of addresses can connect.

To allow a specific IP address:

**sudo ufw allow from 192.168.1.85 to any port 2332 proto tcp**

And for a range of IP’s:

**sudo ufw allow from 192.168.1.0/24 to any port 2332 proto tcp**

Change the above rules to match your network configuration. 

**Note:** unlike the previous rules this will only create one rule (IPv4). If you need a rule to support IPv6 you will need to create it yourself, by replacing the IPv4 address or range in the above command with the proper IPv6 address or range.

We can protect against brute force login attacks, by limiting the number of connections made to our server in the last thirty seconds, from the same IP address.  Because, we are using passwordless logins this rule’s main value is in limiting the noise sent to the log files by an automated attack. To limit connection attempts to our SSH server, add the following rule to ufw:

**sudo ufw limit proto tcp from any to any port 2332**

This will allow no more than 6 connections in a 20 second time period. 

## Fail2ban ##

Fail2ban monitors your log files for signs of an attack and dynamically adds firewall rules to block the attacker. It also removes those rules after a set period of time.  

To install fail2ban:

**sudo apt install fail2ban**

Fail2ban comes with a default config file (**/etc/fail2ban/jail.conf**). Because this file can be modified by a subsequent package update, it is suggested that you create a **/etc/fail2ban/jail.local** file and add your local config changes to the new file. 

Create the /etc/jail.local file:

**sudo nano /etc/fail2ban/jail.local**

and add the following to it:

```
# Fail2Ban configuration file.
# Comments: use '#' for comment lines and ';' for inline comments

[DEFAULT]
# "ignoreip" can be an IP address, a CIDR mask or a DNS host. Fail2ban will not
# ban a host which matches an address in this list. Several addresses can be
# defined using space separator.
ignoreip = 192.168.1.0/24

# "bantime" is the number of seconds that a host is banned.
bantime  = 900

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime = 600
maxretry = 3

# Destination email address used solely for the interpolations in
# jail.{conf,local} configuration files.
destemail = pi@localhost

#
# ACTIONS
#

# email action. Since 0.8.1 upstream fail2ban uses sendmail
# MTA for the mailing. Change mta configuration parameter to mail
# if you want to revert to conventional 'mail'.
mta = mail

#
# JAILS
#

[ssh]
enabled  = true
port     = ssh
filter   = sshd
action = iptables[name=SSH, port=ssh, protocol=tcp]
logpath  = /var/log/auth.log

[ssh-ddos]

enabled = true
port = ssh
filter = sshd-ddos
action = iptables[name=SSH, port=ssh, protocol=tcp]
logpath = /var/log/auth.log
maxretry = 10
```

**Note:** change the above IP range, email address and MTA as needed to match your settings. 

This file and the jail.conf file it is based on are well commented, but there are a few things to note in the above:

- At present fail2ban only supports IPv4. IPv6 support is expected in the next major release.
- ignoreip = 192.168.1.0/24 this will prevent fail2ban from blocking traffic from your local network, You can alternatively supply a single IP address. 
- If you did not edit your **/etc/services** file after changing the port SSH listens on. The above will not work. Either change the **/etc/services** file or replace **port = ssh** with **port = ####**, where **####** is your SSH port. 

**Note:** you only need to include the settings you want to override to the jails.local file. Default options will be taken from the jails.conf file. 

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Restart fail2ban:

**sudo service fail2ban**

Verify it is working:

**sudo tail -f /var/log/fail2ban.log**

**Tip:** If you want to test that fail2ban is working correctly, banning and unbanning connections, you can edit the **/etc/fail2ban/jail.local** file and comment out (insert a “**#**” at the beginning of the line) the following line: **ignoreip = 192.168.1.0/24**. Then try connecting to your server with a nonexistant username. If you tail the logs you should see a warning for each unsuccessful connection and a ban after several unsuccessful attempts. Once the bantime  expires (15 minutes)  you should see an unban message in the log, and be able to connect again – with your regular username. 

## TCP Wrapper ##

TCP wrappers are used to restrict access to TCP services based on host name, IP address, or network address.  Its configuration is controlled by two files: **/etc/hosts.allow** and **/etc/hosts.deny**. When a connection request for a TCP service (like SSH) is received the service checks for a rule allowing the connection in **/etc/hosts.allow**, if it finds one it allows the connection. If a matching rule is not found it checks the **/etc/hosts.deny** file and if no matching rule is found it allows the connection. 

**Note:** Because hosts.allow is checked before hosts.deny, the allow rules have higher precedence. Each file is checked from the top down and only the first matching rule is applied. 

Best practice is to [whitlist]( https://en.wikipedia.org/wiki/Whitelist) connections (i.e., deny everything and add explicit exceptions where necessary). To deny everything Open the **/etc/hosts.deny** file in your editor:

**sudo nano /etc/hosts.deny**

and add the following to the end of the file:

**ALL: ALL**

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

To add an explicit exception open the /etc/hosts.allow file:

**sudo nano /etc/hosts.allow**

And add the following to the end of the file:

**sshd: 192.168.1.**

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Change the partial IP address above to match your network configuration. 

**Note:** the trailing dot above which will allow SSH connections from the IP range 192.168.1.0-192.168.1.255. You can also whitelist a single IP (by using a complete IP address (e.g. 192.168.1.85).
 
**Tip:** If you have physical access to the server, you can test your changes by commenting out, or temporarily removing the changes to the **/etc/hosts.allow** file and trying to establish an SSH connection to your server. Remember to uncomment or reapply the changes to the **/etc/hosts.allow** file after testing. 

**Note:** You may be wondering why you need to bother with TCP wrappers if you configured SSH to only allow specific users, from specific IP’s, and added a firewall rule that includes a from clause. This is a well-known security principle called [defense in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)), and is designed to provide redundancy in case of a failure or a successful attack against a part of the system. 

While I have only covered using these services to protect SSH, all of them can be used to protect more than just SSH. I suggest you read the man page and configuration file for each to learn how to protect the rest of your network services. 

Next time we will look at the crypto used by SSH.
