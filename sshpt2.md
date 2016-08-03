# SSH Security and Usability – Part 2 #
**Author Steve Robillard**

In [part 1]( https://raspberrypise.tumblr.com/post/148032481829/ssh-security-and-usability-part-1) of this series we looked at how to create an SSH key pair (on windows) and configure your Pi to use passwordless logins. In today’s post we cover how to generate a key pair on Linux and how to create a ./ssh/config file to make using SSH easier. 

For the purpose of this post I am going to assume that you have connected to your SSH server at least once already, and hence we won’t have to worry about debugging SSH server or connection issues. 

**Note:** Don’t skip todays post just because you don’t have a linux box to login from, because the steps used to generate a key are the same as needed to create a key for GitHub, and the SSH config file is useful with any  SSH login you may have (e.g. AWS or your webhost). 

## Generate a Key Pair ##

**Note:** Unlike Windows Linux will not require any additional packages be installed. From a command prompt you can generate a key pair with the following command:

**ssh-keygen -t rsa –b 2048**

This will generate a RSA key with 2048 bits. You can accept the default file name (<kbd>enter</kbd>) and location (**/home/your username/.ssh/id_rsa**).

Note: If this directory does not exist it will be created. 

Enter a passphrase and verify it (reenter it).

**Note:** Do not skip this step. It may make using your key easier, but it makes your keys less secure. 

Verify the directory permissions and ownership. This may seem like an unnecessary step, especially for newly created keys and directories, since the keys and folder are created if they don’t exist, but consider what happens if someone does a recursive **chmod** or **chown** or changed the default **umask**). Therefore I think verifying the details is just a case of due diligence. It can also save hours of troubleshooting the SSH connection issues that can result.

Switch to your home directory:

**cd ~**

Check the permissions and ownership:

**ls –la**

Your **~/.ssh** directory should have read, write and execute permissions for the owner only (i.e. **drwx------**), and the owner and group (i.e. the third and fourth columns) should match your username. 

Check the permissions and ownership of your key files.

Change to your **~/.ssh** directory (note the leading dot – indicating a hidden folder)

**cd .ssh**

List the file permissions and ownership:

**ls -la**

You should see two files. Your private key **id\_rsa** with only read and write permissions for the owner (i.e. **–rw-------**), and your public key **id\_rsa.pub** with read permissions for owner, group and other and write permissions for the owner (i.e. **–rw-r—r--**). Both should have your username in the owner and group fields (i.e. the third and fourth columns). 

Remember **NEVER** share or upload your private key.

## Copy the Public Key to the Your SSH Server ##

**Note:** I am assuming that you have already configured your server for passwordless logins. If you have not already done so see [part 1](https://raspberrypise.tumblr.com/post/148032481829/ssh-security-and-usability-part-1) of this series for complete instructions. To copy your key to the remote server you will need to temporarily enable password based logins first. 

On your SSH server open the **/etc/ssh/sshd_config** file in your editor:

**sudo nano /etc/ssh/sshd_config**

**Tip:** Append –c to the above command to enable the display of line numbers.

Locate the following line (line 52):
**#PasswordAuthentication no**

Change **no** to **yes**:

**PasswordAuthentication yes**

Save your changes:
<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Reload SSH:

**sudo service ssh reload**

Once you have made the above changes. You can connect via SSH using your username and password on the remote machine:

**ssh-copy-id user@192.168.0.100**

Replace *user* in the above command with your username on the remote machine, and the *IP Address* with the IP Address or hostname of the remote machine. 

So if you are trying to connect to your Raspberry Pi as the pi user you would use the following:

**ssh-copy-id pi@raspberrypi.local**

**Tip:** you can leave off the username if the username you are using is the same on both machines.

**Note:** if this is the first time you connect to this machine it will display the key fingerprint and ask you to confirm that you want to connect (more on this in the security post of this series).

When prompted enter your password (for the remote machine).
 
You should see something like the following:

**Number of key(s) added: 1<br>
Now try logging into the machine, with:   "ssh 'pi@raspberrypi.local'"<br>
and check to make sure that only the key(s) you wanted were added.**

**Note:** the username and hostname above may be different. 

End your SSH session:

**exit**

## Verify that the key Works ##

Start the SSH Agent:

**eval  \`ssh-agent\`**

**Note:** the backticks ( **`** )used in the previous command – on US keyboards this is located to the left of the number 1 key.

Add your key to the SSH Agent:

**ssh-add ~/.ssh/id_rsa**

The above assumes you used the default keyname and location when creating your key pair, if not substitute the correct path and file name in the above command. 

**Tip:** if you used the default name and location when creating the key pair you can omit the filename and path from the above command (i.e. **ssh-add**).

When prompted enter your private key’s passphrase. 

You should see a message that your identity was added to the agent.

Open a new connection to your server:

**ssh pi@raspberry.local**

You should be logged in automatically without being prompted for a password or username.

## Disable Password Authentication ##

**Note:** be sure the above works correctly before proceeding, especially if you do not have physical access to your Pi, or if it is running headless. Failure to do this can result in you being locked out of the Pi and unable to restore SSH access.

On your SSH server open the **/etc/ssh/sshd_config** file in your editor:

**sudo nano /etc/ssh/sshd_config**

**Tip:** Append –c to the above command to enable the display of line numbers.

Locate the following line (line 52):

**#PasswordAuthentication yes**

change **yes** to **no**:

**PasswordAuthentication no**

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Restart SSH:

**sudo service ssh restart**

Exit your SSH session:

**exit**

## Making Logins Easier With an SSH Config File ##

In my post on [Bash aliases](https://raspberrypise.tumblr.com/post/142215518744/improving-your-command-line-skills-part-2) I mentioned that aliases should not be used for SSH connections, as there was a better more specific method of shortening SSH commands. Instead you should create a SSH config file.
 
Right now we are logging in by starting the SSH Agent, adding our key to it and then using something like:

**ssh username@hostname**

To connect. With an SSH config file we could change that to:

**ssh hostalias** (i.e. ssh pi1)

**Note:** this may not seem like a big improvement, if you are only dealing with one or two servers, but if like me you are using SSH on multiple machines it makes life much simpler. It also makes working with non-standard SSH ports easier (I will revisit the topic of port numbers in the security part of this series). An example should make this clearer. 

Switch to you **~/.ssh** directory:

**cd ~/.ssh**

Create the file and open it in your editor:

**nano config**

Enter the following text:

<pre>Host YourHostAlias
    HostName YourHostnameOrIPAddress
    Port 22
    User YourUsername
    IdentityFile ~/.ssh/id_rsa
</pre>


Enter your desired host alias, IP Address or hostname, and username on the remote server. The port number and location of the identity file are set to their defaults. If these do not match your system change them as required. 

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

You should now be able to login with the following:

**ssh hostalias** 

So assuming you set your host alias to pi1 you would use:

**ssh pi1**

You would then be prompted for your passphrase. After entering it you should be logged in to your SSH server. No need to load your key to the SSH agent which is simpler and improves security. 

Setting up paswordless logins on a Mac is likely to be very similar to that done on Linux. However, I do not have a Mac, so if someone wants to cover how to set this up on a Mac, or verify the Linux steps work please drop a comment below and let us know.

Next time we will look at customizing the login message and command line prompt when using SSH.
