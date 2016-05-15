**Author: Steve Robillard**

**Suggested Tags:** raspberrypi, bash, command line, bash aliases

#  Improving Your Command Line Skills Part 2 #

In [part one](https://raspberrypise.tumblr.com/post/141758901139/improving-your-command-line-skills-part-1) of this series we looked at keyboard shortcuts and how to use man and TLDR pages. In this part, we will look at [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) aliases.

## Aliases ##
Aliases are a way to rename a function, create a command that is easier to remember and a way to save typing.
 
The syntax for an alias is:

**`alias alias_name="command_to_run"`**

Note: there is no space between the alias_name and the `=` sign, and none between the equal sign and the quotation mark. The only place you can have a space is after the word alias and within the quotation marks.

For example we can alias the 

**`sudo apt-get update && sudo apt-get –y upgrade`**

command, to the much shorter `uu` command, by entering the following:

**`alias uu="sudo apt-get update && sudo apt-get –y upgrade"`**
 
at the command line. Now you can type `uu` to update and upgrade your system. There is one problem with the above alias command, it is temporary. That means it will only last as long as the shell does. To make an alias permanent you can add it to your `~/.bashrc` file.
 
Add the following to the bottom of the file:


    #########
    # Aliases
    #########
 
    alias uu=”sudo apt-get update && sudo apt-get –y upgrade”



Save your changes and exit the editor. 

To enable the new alias enter the following:

**`source ~/.bashrc`**

The above command will reread the `.bashrc` file, and activate the new alias. The sharp eyed may have noticed that there are several aliases already present in the file. For those who missed them the following command will print them to the terminal:

**`grep alias ~/.bashrc`**

Some of the aliases are commented out (those preceded by a **#**). If you want to enable these just remove the #. So to enable the `ll` alias change:

**`#alias ll='ls -l'`**

To
 
**`alias ll='ls -l'`** 

and source the `.bashrc` file

**`source ~/.bashrc`**

You can see a list of all active aliases using the following command:

**`alias`**

To remove an alias either delete or comment out the line in your .bashrc file. 

## A few caveats ##

Bash aliases are extremely powerful, but there are a few cases where they should not be used (e.g. SSH connections and Git). There are SSH and git specific files (~/ssh/config and ~/.gitconfig respectively) where these should be created.  Using these program centric files provides increased security and integration. 

Another thing to be careful of with aliases is hiding an existing command. If you remember last week we looked at this tar command:

**`tar xfa tmux-2.1.tar.gz`**

I mentioned that this tar command was like unzip, but if we alias this as unzip:

**`alias unzip="tar xfa"`**

We will be overriding the default behavior of the existing unzip command. This can cause strange behavior that is difficult to debug. However, there are times when this can be helpful – see the top example below. To avoid this, type your intended alias name (unzip) at the command line. If you don’t see: 

**bash: youraliasname: command not found**

You should probably choose a different name for your alias (e.g. alias uzip ="tar xfa"). After adding this to your `~/.bashrc` file and sourcing the file, you can type uzip filename to extract any tar zip file (e.g. `uzip tmux-2.1.tar.gz`).

## Alias examples ##

A web search will turn up hundreds of useful Bash aliases, but here are a few I find useful.
  
**`alias la="ls -la"`**   	# complete directory listing
 
**`alias rm="rm -i"`**    	# safer deletion

**`alias df="df -h"`**    	# human readable sizes

**`alias top="htop"`**   		# run htop instead of top (you need to install htop first)

**`alias diff='colordiff'`** 	# colorize the diff output for easier viewing

If you have a favorite alias, please add it in a comment below.

Remember, if you aren’t sure what these commands do (e.g. rm = remove a file) or what the switches do (e.g. `–i` = interactive) you can read the man page for the command (i.e  `man command`).

## Additional Resources ##

Bash Reference Manual section on [aliases](http://www.fnal.gov/docs/products/bash/bashref.html#SEC61) 

In part three of this series we will look at Bash functions.

