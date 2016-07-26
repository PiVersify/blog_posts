# SSH Security and Usability – Part 1 #
**Author Steve Robillard**

I have ten Pi’s, an Ubuntu notebook and server at home and about a dozen cloud and webhosting accounts. Most of these machines are accessed exclusively via SSH. Since so much of my work is done via SSH, I have collected a set of customizations that make SSH both more secure and usable. In this series I will show you how to:

- configure SSH to use passwordless logins (from Windows and Linux),
- simplify SSH connections with an SSH config file,
- harden SSH,
- display a custom message on SSH logins, and
- customize the SSH prompt.

## Configure Passwordless Logins from Windows ##

For the purpose of this post I am going to assume that you have connected to your Pi via SSH at least once already, and hence we won’t have to worry about debugging SSH server or connection issues. 

To replace passwords we will use a public private key pair. This makes SSH logins more secure, due to their length, and with the right tooling and setup can be easier to use as well, a rare instance of a security and usability win win.

## The Software ##

We will need an SSH client, a way to generate a public private key pair and an SSH agent. My tools of choice are PuTTY (SSH Client), PuTTYgen (Key generator) and Pagenat (SSH authentication agent). You can download either the individual tools (PuTTY, PuTTYgen and Pagenat), or the entire suite of tools. including PSCP (SCP client) as a ZIP file or Windows installer package form the [download page[(http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html). I suggest downloading the full suite, as it ensures that all the tools are the same version and hence compatible. I will be using the windows installer package (MSI) for this post. 

**Note:** while the most recent version is beta 0.67 this is mature software that has been around for many years.
 
Download the [Windows installer package](https://the.earth.li/~sgtatham/putty/latest/x86/putty-0.67-installer.msi), and install it.

## Generate a Key Pair ##

From the start menu launch PuTTYgen (it is in a group called PuTTY).

![](Puttygen1.png)

Click Generate (the defaults are fine **Type of key to generate: SSH-2 RSA** and **Number of bits in a generated key: 2048**)

![](Puttygen2.png)

You will see a message instructing you to move your mouse over the blank area of the Keygen window to generate some randomness. When it finishes it will display the key and its fingerprint. 

![](Puttygen3.png)

Enter a passphrase and confirm it (this is like a password for your key and should not be skipped) You will need to remember this phrase to access and use it. I suggest storing it in a password database like [Keepass](keepass.info) or [LastPass}(http://lasypass.com)

Click **Save public key** give it a name (i.e. PiKey.pub) and save it to your users folder (i.e. c:/users/your_username). 

**Note:** If you have MS Publisher installed .pub files may be associated with Publisher and when clicked on will attempt to open with Publisher. You can open the file by right clicking on it and using notepad.
 
I keep all my keys in a .ssh folder (e.g. i.e c:/users/srobillard/.ssh). 

**Note:** If this folder does not already exist you can create it by naming the folder **.ssh.** (note the trailing dot). It will be removed by Windows Explorer. Thanks to Denny on [superuser.com]( http://superuser.com/questions/64471/create-rename-a-file-folder-that-begins-with-a-dot-in-windows) for the solution to the naming problem.

Click **Save private key** give it a name (i.e. PiKey – PuTTYgen will append the .ppk file extension) and save it to the same folder as your private key.

Close PuTTYgen

## Copy the Public Key to Your Raspberry Pi ##

Remember NEVER share or upload your private key (this is why I use the .pub file extension to name my public keys).
 
Launch PuTTY

![](Putty1.png)

Enter the **Host Name (or IP address)** of your Pi.
Optional, but recommended (save the connection details by entering a name in the **Saved Sessions** text box. You can also save the username by clicking on **Data** from the menu on the left and entering the username in the **Auto-login username** testbox. Click on **Session** at the top of the menu, and click **Save**. 

![](Putty2.png)

Click **Open** and enter your password when prompted.

Check for an **.ssh** directory in your home directory (note the leading dot):

**ls –la ~**

If this folder does not exist create it with the following command:

**mkdir .ssh**

Check the folder permissions:

**ls –la ~/.ssh** 

If you just created this file the permissions are almost certainly wrong. This directory should only have read write access for the owner (i.e. **drwx------**). To change the folder permissions use the following command:

**chmod 700 ~/.ssh**

Verify the new permissions with another: 

**ls –la**

Change directory to the new .ssh directory

**cd .ssh**

Create an authorized_keys file:

**nano authorized_keys**

Open your public key file with notepad (PiKey.pub). 

Select all of the contents: <kbd>**Ctrl-A**</kbd>, and copy it to the clipboard <kbd>**Ctrl-C**</kbd>.

Switch back to PuTTY

**Note:** If the **authorized_keys** file already exists and is not empty (paste the new content on a separate line).

Paste the contents of the clipboard (Right Mouse Click).

Remove the following from the start of the line:

**---- BEGIN SSH2 PUBLIC KEY ----Comment: "rsa-key-20160726"**

**Note:** The number in quotes will be different for your key. 

Add **ssh-rsa** to the beginning of the line, if it is not present. 

From the end of the line remove the following:

**---- END SSH2 PUBLIC KEY ----**

It should look like this and be all one line when complete:
ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEApLYUqh7KWXz1GhMIre4heJEJrakBgKjDBbLoKALLEMEzG1ppVnykWZa2qK2bYunrXngF0H9UHV2kX+Zji+fHoe6MWHqAWbodw5o7V/bvFjFxK3YDHzlQk8EFjYKQ2+hLVuYDvaxaXYtF/9YZLP7EUKcN2ch3ODD6KIKmMdT

**Note:** it wraps here because of page width.

Save the file:
 
<kbd>**Ctrl-o** and **Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Check the file permissions on the authorized_keys file:
 
**ls –la**

They should be **-rw--—-—**. If not use the following to change them:

**chmod 644 authorized_keys**

and verify the change with: 

**ls -la**

## Modify the SSHD Configuration  ##

Open the **/etc/ssh/sshd_config** file in your editor

**sudo nano /etc/ssh/sshd_config** 

**Tip:** Append –c to the above command to enable the display of line numbers.

locate the following lines (line 31 in my file):

**RSAAuthentication yes<br>
PubkeyAuthentication yes<br>
\#AuthorizedKeysFile     %h/.ssh/authorized_keys**

And change them to (uncomment the third line):

**RSAAuthentication yes<br>
PubkeyAuthentication yes<br>
AuthorizedKeysFile     %h/.ssh/authorized_keys**

Save your changes:

<kbd>Ctrl-o</kbd> and <kbd>Enter</kbd>

and exit the editor:

<kbd>Ctrl-x</kbd>

Restart SSH:

**sudo service ssh restart**

Exit your putty session: 

**Exit**

## Verify that the key Works ##

Launch Pageant from the start menu (PuTTY group)

Right click the icon in the system tray

![](http://pageant-icon.png)

And select **Add Key**

Navigate to the location of your private key and click **Open**. 

and enter your passphrase.

Relaunch PuTTY and your Saved Session. 

Click **Open** to connect. 

You should **not** be prompted for a password or username if you stored it above).

## Troubleshooting ##
If you are prompted for a password, check the following: 

- Did you copy and edit the public key correctly,
- Did you correctly edit the sshd_config file,
- Did you restart the SSH daemon after modifying the config file,
- Did you add the correct key to pageant and is it still running?

## Disable Password Authentication ##

**Note:** be sure it works correctly before proceeding, especially if you do not have physical access to your Pi, or if it is running headless. Failure to do this can result in you being locked out of the Pi and unable to restore SSH access.

Launch PuTTY and login into your Pi.

Open the **/etc/ssh/sshd_config** file in your editor

**sudo nano /etc/ssh/sshd_config**

Locate the following line (line 52):

**#PasswordAuthentication yes**

Remove the **#** and change **yes** to **no**:

**PasswordAuthentication no**

Save your changes:

<kbd>Ctrl-o</kbd> and <kbd>Enter</kbd>

and exit the editor:

<kbd>Ctrl-x</kbd>

Restart SSH:

**Sudo service ssh restart**

Exit your putty session: 

**Exit**

Verify Everything Works as Expected

Exit Pageant (right click the icon and choose exit).

Try to connect using Putty. You should see the following:
 
**Disconnected: No supported authentication methods available (server sent: publickey).**

Great that means passwords are not used for authentication. Now add your private key to Pageant, and try to connect with PuTTY. You should be automatically logged into your account. Congratulations, your SSH logins no longer accept passwords and your Pi is a little more secure. 

**Tip:** You may want to look into [SuperPuTTY]( https://github.com/jimradford/superputty). It is a wrapper for putty that adds several additional features including tabs and when closing your SSH session it does not require relaunching PuTTY.

Next Time we will look at how to setup passwordless SSH logins from Linux. 

Setting up paswordless logins on a Mac is likely to be very similar to that done on Linux. However, I do not have a Mac, so if someone wants to cover how to set this up on a Mac, or verify the Linux steps work please drop a comment below and let us know.
