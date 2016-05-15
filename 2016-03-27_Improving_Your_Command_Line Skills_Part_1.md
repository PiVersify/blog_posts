**Author:** SteveRobillard

**Publish Status:** published 03/27/16

**Suggested Tags:** raspberrypi, bash, command line, man page, keyboard shortcuts

# Improving Your Command Line Skills Part 1 #

For those who are new to the Raspberry Pi and Linux, the command line can seem intimidating and cryptic.  In this first post I will cover man pages and keyboard shortcuts. In a subsequent post I will cover bash aliases and functions.

## Man Pages ##
Jacob001’s post [Tmux 101: Installing from Source](https://raspberrypise.tumblr.com/post/141348857424/tmux-101-installing-from-source), included this command: **tar xfa tmux-2.1.tar.gz**. For those new to the command line this can look like an eye chart and lead to more questions than answers. What is tar? What does it do? What does **xfa** mean? To answer these questions you can use the man command. Man stands for manual page and is the built in documentation for Linux commands. 

Let’s look at the man page for the tar command; enter the following **man tar**. From the output we can find answers to the questions above. Tar “is the GNU version of the tar archiving utility.” OK, but what is an archive utility? Looking at the description we can see that “tar stores and extracts files from a tape or disk archive.” That is more clear; it is essentially like unzip. From the Description, we know that the first argument is a function(s), and from the *Function Letters* and *Other Options* Sections we can decipher the functions used: 

- **x** extracts files from the archive,
- **f** specifies the file we are working with - tmux-2.1.tar.gz in this case, and
- **a** tells tar to use the file suffix (tar.gz in this case) to determine the compression algorithm needed to extract the files. 

To get more familiar with man give these examples a try: 
 
- **man man** (yes the man command has its own manual page).
- **man apt-get**.
- **man wget**.
- **man ls**.

The following keyboard shortcuts are useful for navigating within man pages:

- **up** and **down** arrows go up and down one line.
- **Ctrl+u** and **Ctrl+d** go up and down half a page.
- **Spacebar** to go down a page.
- **/** search forward for text.
- **?** search backward for text.
- **n** find next occurrence of searched word.
- **N** find previous occurrence of searched word.
- **g** go to top of document.
- **G** go to bottom of document.
- **q** quit man page.

There is a second help system called info that some prefer to man. To learn more about the info command try either **man info** or **info info**. 

I have worked with Linux for more than 20 years and I still refer to man pages regularly. Normally I know the command I need, but don’t remember the specific arguments or their order. A new tool called TLDR pages makes finding the answer to these questions much faster, quoting from the website “The [TLDR pages](http://tldr-pages.github.io/) are a community effort to simplify the beloved man pages with practical examples.” The website includes a live demo and installation instructions. Once installed or running the live demo running **TLDR tar** shows that we could have used **tar xzf  tmux-2.1.tar.gz** to extract the tmux-2.1.tar.gz file.

Some people say it is faster or easier to just google this information, but what do you do when you are trying to establish a network connection, or someplace without a network connection? Man still works in these cases. Man, also gives you documentation that matches the software installed on your system. These sometimes subtle differences can lead to hours of debugging.  

## Keyboard Shortcuts ##
Keyboard shortcuts are ubiquitous in Linux. Below are a few I find especially useful:

- **Ctrl-A** go to the beginning of the line.
- **Ctrl-E** go to the end of the line.
- **Ctrl-r** reverse search your history for a command.
- **cd** go to your home directory.
- **cd –** go to the previous directory (especially useful when umping back and forth between a pair of directories.
- **Ctrl-l** clear the screen (the same as clear).
- **sudo !!** run the last command with sudo.
- **Tab** auto complete command, file and folder names
- **Up arrow** previous command (walk back through the command history).
- **Down arrow** walk forward through the command history. 

There are dozens more, but these should give you a good start. You can get a complete list of shortcuts and their bindings using the following command: 

**`bind -P|grep "can be found"|sort | awk '{printf "%-40s", $1} {for(i=6;i<=NF;i++){printf "%s ", $i}{printf"\n"}}'`**

The above command comes from [Registered User](http://askubuntu.com/questions/444708/is-there-any-manual-to-get-the-list-of-bash-shortcut-keys)'s answer to this ask Ubuntu [question](http://askubuntu.com/questions/444708/is-there-any-manual-to-get-the-list-of-bash-shortcut-keys).
