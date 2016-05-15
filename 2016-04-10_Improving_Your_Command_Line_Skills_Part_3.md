# Improving Your Command Line Skills Part 3 #
**Author: Steve Robillard**

**Suggested Tags:** raspberrypi, bash, command line, bash functions

In the [part one](https://raspberrypise.tumblr.com/post/141758901139/improving-your-command-line-skills-part-1) of this series we covered man pages and keyboard shortcuts. In [part two](https://raspberrypise.tumblr.com/post/142215518744/improving-your-command-line-skills-part-2) we looked at Bash aliases. Today we will cover Bash functions.

## Functions ##
Bash functions are the most powerful technique we will look at. They allow you to group a series of commands into a single command, and can be called from the command line just like the Linux functions you are already familiar with (i.e. man, ls, cd). 

The syntax to create a bash function is: 

<pre>function functionname()
{
  Command 1
  Command 2
  …
}</pre>

How many times have you changed directory and immediately listed the contents (cd and ls)? Let’s see how a function can make this a little easier. 

Add the following to your ~/.bashrc file:

**function cdls() { cd "$@" && ls; }**

Save your changes and exit the editor. 

To enable your new function enter the following:

**source ~/.bashrc**

Now let’s try our new function:

**cdls /etc**

You should now be in the **/etc** directory (you can verify this by typing **pwd** or looking at the prompt), and should see a listing of the directories contents. 

In parts one and two we looked at the following command:

**tar xfa tmux-2.1.tar.gz**

from Jacob’s post on [Installing TMUX from source](https://raspberrypise.tumblr.com/post/141348857424/tmux-101-installing-from-source). Tar is one of those commands I can never remember which switches (i.e xfa) I need to use. We saw in part 1 how man and TLDR pages could help, and in part two we created an alias (i.e. alias uzip =” tar xfa”) to extract tar zipped files. The only problem was that it only worked for tar zipped files. We would need to create an aliases for each additional file type we wanted to work with (e.g. .rar, .zip).  Let’s see how using a function we can extract almost any archive file format with one easy to remeber command.
 
Add the following to your ~/.bashrc file:


<pre># credit: http://nparikh.org/notes/zshrc.txt
# Usage: extract <file>
# Description: extracts archived files / mounts disk images
# Note: .dmg/hdiutil is Mac OS X-specific.
extract () {
    if [ -f $1 ]; then
        case $1 in
            *.tar.bz2)  tar -jxvf $1                        ;;
            *.tar.gz)   tar -zxvf $1                        ;;
            *.bz2)      bunzip2 $1                          ;;
            *.dmg)      hdiutil mount $1                    ;;
            *.gz)       gunzip $1                           ;;
            *.tar)      tar -xvf $1                         ;;
            *.tbz2)     tar -jxvf $1                        ;;
            *.tgz)      tar -zxvf $1                        ;;
            *.zip)      unzip $1                            ;;
            *.ZIP)      unzip $1                            ;;
            *.pax)      cat $1 | pax -r                     ;;
            *.pax.Z)    uncompress $1 --stdout | pax -r     ;;
            *.Z)        uncompress $1                       ;;
            *)          echo "'$1' cannot be extracted/mounted via extract()" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}</pre>

Save your changes and source your ~/.bashrc file. 

Let's try out our new extract function. Create a new directory and cd into it:

**mkdir Test && cd Test** (see below for a function that does this). 

Download the tmux source file:

**wget https://github.com/tmux/tmux/releases/download/2.1/tmux-2.1.tar.gz**

Now uncompress it with our new extract function:

**extract tmux-2.1.tar.gz**

if you list the directory contents you should see a new tmux-2.1 folder. The same as if you had used **tar xfa tmux-2.1.tar.gz**,  or the uzip alias from part two **uzip tmux-2.1.tar.gz**. The biggest advantage is that you don’t need to remember a different set of commands or alias for other archive formats. For example the exact same command will work for zip files. 

Download a zip file:
 
**wget https://github.com/tmux/tmux/archive/master.zip**

and unzip it with the extract command:

**extract master.zip**

You should now have a new tmux-master directory.

## Function examples ##

A web search will return dozens of useful Bash functions, but here are some I find useful.

###Make a directory and immediately cd into it:###

<pre>function mcd () {
    mkdir -p $1
    cd $1
}</pre>

usage: **mcd newdirectoryname**

###Change directory and list all files:###

<pre>function cdll() { cd "$@" && ls -la; }</pre>

usage: **cdll newdirectoryname**

###Create an archive (this is the inverse to extract):###
<pre># roll - archive wrapper
# usage: roll <foo.ext> ./foo ./bar
roll()
{
  FILE=$1
  case $FILE in
    *.tar.bz2) shift && tar cjf $FILE $* ;;
    *.tar.gz) shift && tar czf $FILE $* ;;
    *.tgz) shift && tar czf $FILE $* ;;
    *.zip) shift && zip $FILE $* ;;
    *.rar) shift && rar $FILE $* ;;
    *) echo "'$1' is not a valid archive type"
  esac
}</pre>

usage: **roll compressedfilename filesordirectorytoarchive**

e.g. **roll master.tar.gz ./tmux-master/***

## Additional Resources ##
 
Bash Reference Manual section on [functions](https://www.gnu.org/software/bash/manual/html_node/Shell-Functions.html)

Bash Function [Tutorial](
http://ryanstutorials.net/bash-scripting-tutorial/bash-functions.php)

In the final part of this series we will explore a technique to manage and version control all path changes, environment variables, alias, and functions in a clean way.
