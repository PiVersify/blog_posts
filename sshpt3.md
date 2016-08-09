# SSH Security and Usability – Part 3 #
**Author Steve Robillard**


In [part 1](https://raspberrypise.tumblr.com/post/148032481829/ssh-security-and-usability-part-1) of this series we looked at how to create an SSH key pair (on windows) and configure your Pi to use passwordless logins. In [part 2](https://raspberrypise.tumblr.com/post/148401541394/ssh-security-and-usability-part-2) we showed how to create a key pair in Linux and use an ssh config file to make passwordless logins easier. In today’s post we will look at how to display custom messages as part of the SSH login process and modify the command prompt when using SSH. 

**Note:** this post assumes that you are using Raspbian Jessie.

There are two points in the SSH connection process where you can add your own messages, before the login prompt and after a successful login. Anyone who has administrated multiple boxes has at one time logged into the wrong box and wasted time diagnosing a problem or looking for a file on the wrong machine. One of the things I do to avoid this is add an ASCII text banner to my SSH logins. 

## Display a Custom Message before Login ##

The first step is to enable this feature in the **/etc/ssh/sshd_config** file:

**sudo nano /etc/ssh/sshd_config**

**Note:** the **d** in the file name.

Locate the following line(line 72):

**#Banner /etc/issue.net**

And change it to:

**Banner /etc/issue.net**

**Note:** remove the leading pound sign (#). In most Linux configuration files any line that begins with a **#** is considered a comment and ignored. Therefore, removing the # sign is often referred to as uncommenting the line.

Save your changes:

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Next we need to edit the **/etc/issue.net** file.

**sudo /etc/issue.net**

This file currently contains the following text:

**Raspbian GNU/Linux 8** 

**Note:** I don't find this that helpful so I remove the default text:

<kbd>**Ctrl-k**</kbd>

You can add whatever text you like to this file. On a multiuser or internet accessible box I would add a legal disclaimer and warning regarding unauthorized access and the logging policy. On my internal machines I add ASCII text of the hostname. For example Triumph (I name my machines around automobile brands).

![Triumph Ascii text](http://)

You can generate your own ASCII text using this [website]( http://patorjk.com/software/taag/#p=display&f=Graffiti&t=Type%20Something%20). If ASCII art is more to your liking this Pi Foundation [forum thread]( https://www.raspberrypi.org/forums/viewtopic.php?p=78678#p78678) has several examples of Raspberry themed ASCII art.
  
Once you are happy with the contents save your changes: 

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

Reload the SSH configuration:

**sudo service sshd reload**

## Test Your Changes ##

Exit your SSH session:

**Exit**

And reconnect. You should see your custom message at the top of the screen. 

**Note:** you may see an extra line similar to this: Using username “pi” if you are using putty/superputty. This comes from putty and is not controlled by your Pi. 

## Display a Custom Message after Successful Logins ##

To change the message that is displayed after a successful login, edit the **/etc/motd** file.

**sudo nano /etc/motd**

**Note:** This file is static and will not evaluate any included code.

Make your changes to this file - I usually just remove this text. 

Save your changes: 

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

**Test Your Changes** 

Exit your SSH session:

**Exit**

And reconnect. You should see your custom message after logging in. 

**Note:** by default you will see the last login time and machine used to connect. For security reasons I suggest leaving this. It can alert you that a security breach may have occurred. 

**Modifying the Command Prompt for SSH **

Again, anyone who has worked with multiple machines has entered the correct command on the wrong machine. In the case of SSH you might think you are running the command on your local machine instead of on the remote machine or vice versa – with potentially disastrous results. To help avoid this I modify the text and color of my SSH prompts. I color my regular SSH user logins yellow and append “ ssh” to the prompt. 

**Note:** I use the default green prompt for local non root logins, and red for the root prompt.

To do this edit your **~/.bashrc** file.

**nano  ~/.bashrc**

append the following to the bottom of the file:

**<pre>
\### Customize the Bash prompt when accessed via SSH
if [ -n "$SSH_CLIENT" ]; then text=" ssh"
   export PS1='\[\e[1;33m\]\u@\h:\w${text}$\[\e[m\] '
fi
</pre>**

**Note:** If you don’t like yellow you can replace the **33** above with one of the other [Bash foreground color codes](http://misc.flogisoft.com/bash/tip_colors_and_formatting).

Save your changes: 

<kbd>**Ctrl-o**</kbd> and <kbd>**Enter**</kbd>

and exit the editor:

<kbd>**Ctrl-x**</kbd>

**Note:** if you implemented the conf.d style [.bashrc management]( https://raspberrypise.tumblr.com/post/143121276514/improving-your-command-line-skills-part-4) method I presented in my Command Line Skills Series, you can create a new file with the text from above in your .bashrc.d directory.

## Test Your Changes ##

Exit your SSH session:

**Exit**

And reconnect. You should have a yellow prompt and have ssh appended to the end of it.

Next time we will begin looking at how to protect your SSH logins and hardening the SSH server.
