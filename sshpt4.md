# SSH Security and Usability – Part 4 #

**Author Steve Robillard**

In [part 1](https://raspberrypise.tumblr.com/post/148032481829/ssh-security-and-usability-part-1) of this series we looked at how to create an SSH key pair (on windows) and configure your Pi to use passwordless logins. In [part 2](https://raspberrypise.tumblr.com/post/148401541394/ssh-security-and-usability-part-2) we showed how to create a key pair in Linux and use an ssh config file to make passwordless logins easier. [Part 3](https://raspberrypise.tumblr.com/post/148725075714/ssh-security-and-usability-part-3) showed how to display custom messages as part of the SSH login process and modify the command prompt when using SSH. In this week’s post we will begin our look at securing SSH. 

**Note:** this post assumes that you are using Raspbian Jessie. Though most of what we cover will be applicable to other distros.

## Physical Security ##

All security begins with physical security. If you cannot ensure the physical security of the client and server all other attempts to secure the system are for naught. Given enough time and access to the hardware an attacker can easily compromise a system. They could install a rootkit, a compromised version of the SSH server, a key logger or simply steal your private key. The Pi’s default login configuration (auto login and well known username), small size and use of an SD card only make this easier. Complete coverage of physical security is outside the scope of this post, however I do suggest that you research this further, especially if your server is located somewhere that you do not have physical control over (like a locked room in your home). I would also suggest that you configure your Pi to require the user to login. This can be configured using the raspi-config script. 

## Protecting your Keys ##

I said this in a previous part of this series, but it bears repeating **NEVER SHARE YOUR PRIVATE KEY**.

In parts one and two we created a key with a length of 2048 bits. To increase the security of your key you can increase the key length. To create a key with a bit length of 4096, either change the setting in PuttyGen

![The PuttyGen Bit Length Setting](http://giyhub.com)puttygenbitlength.png

or with the following command on Linux:

**ssh-keygen -t rsa –b 4096**

**Note:** You will need to upload your new key to the server for it to work (see part 1 for how to do this from Windows and part 2 for Linux instructions). After confirming the new key works, you should remove the old key from the servers** ~/.ssh/authorized_keys** file.  To remove the old key:

Open the **~/.ssh/authorized_keys** file in your editor:

**Nano ~/.ssh/authorized_keys**

Locate the correct key. Each key entry begins with **ssh-rsa**.  Match the text string that follows with the content of your public key, and delete the cooresponding lines. 

Save your changes:

**<kbd>Ctrl-o</kbd>** and **<kbd>Enter</kbd>**

and exit the editor:

**<kbd>Ctrl-x</kbd>**

To limit the potential damage that can result from a compromised private key you should:

Use different keys for different purposes. This ensures that if your private key is compromised that the damage is limited to a subset of all machines and services you use. For example if your AWS server login key is compromised the attacker does not also gain access to your GitHub account. 

## The SSH Agent ## 

We saw, in the first two parts of this series, how to use a key pair to replace passwords for SSH logins. The use of keys improves the security of your SSH logins; key based logins are harder to crack because of their length, compared to passwords. We also saw how to use an SSH agent to avoid having to repeatedly reenter the passphrase that protects our key. The SSH agent makes using passwordless logins easier, but does have security issues of its own. For example it is possible to discover the private key and passphrase by examining the memory associated with the agent process. So what steps can you take to minimize the risks?

- Unload the agent and keys when not needed (e.g. done using SSH or logging out of your account).  
- Never copy your private key to a machine where someone else has root.
- Never use an SSH agent on a computer someone else has root or administrator privileges on. 
- Never forward your agent connection. 

Note: Doing any of the previous three will essentially share your private key with anyone with root access to the box. However, there are also times when this may not be an option. I don’t know of a way to use a passphrase protected key without using Pageant. It may also not be possible on a computer or network supplied by your employer.  

**The Server’s Key Fingerprint**

One thing I glossed over in the first two parts of this series was the server’s key fingerprint. The first time you connect to a host you are presented with the server’s key fingerprint and asked if you want to continue to connect. 

![SSH Server Connection String Showing the Key Fingerprint](http://github.com)sshfingerprint.png

This fingerprint is an important security feature. It is how the server identifies itself to clients, and protects against man in the middle attacks. As such you should verify this fingerprint before connecting.

**Note:** the best way to do this is using the keyboard connected directly to the server you want to connect to, instead of after connecting via SSH. You may want to do this as part of your initial setup and record the key fingerprints.

To check the server’s fingerprint you can use one of the following commands.

**Note:** You can determine which key type is being used by examining the second line of the connection prompt as shown in the image above.
 
for an ecdsa key:

**ssh-keygen -lf /etc/ssh/ssh_host_ecdsa_key.pub**

for an rsa key:

**ssh-keygen -lf /etc/ssh/ssh_host_rsa_key.pub**

The output from the preceding command should match that presented by the connection prompt.

You can now type **yes** and complete the connection. Typing yes stores the hosts key in your **~/.ssh/known_hosts** file. 

In the event that a server’s key has changed (either due to a man in the middle attack or from non-threat events (e.g. rebuilding the server or regenerating the keys) you will see a warning like that seen below when trying to connect.
 
![Host ID Chaned Warning Message](http://github.com)hostidchangewarning.png

If you are sure this is the result of a server configuration change and not an attack. You can remove the old key entry with the following command:

**ssh-keygen -f "/home/pi/.ssh/known_hosts" -R lotus**

**Note**: change the path and hostname above to match your system. The exact command is normally contained in the warning message as seen in the image above.

and reconnect. Obviously, if the reason cannot be ascribed to server maintenance you should not continue connecting and notify the system administrator immediately.  When it comes to security, err on the side of caution and assume it is an attack. 

Next time we will continue looking at ways to harden your SSH server configuration.

