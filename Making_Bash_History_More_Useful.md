# Making Bash History More Useful #
One of the most useful command line features is the Bash history. Commands you enter at the command line are stored in a file called ~/.bash_history. To view the contents of your Bash history use the following command:

**History**.

The first column is the history number and the second is the command.

<pre>
320  startx
321  ping 192.168.1.243
322  df
323  du
324  df -h
325  ssh pi@alfa.local
</pre>

To rerun one of the commands type a bang(!) and the command number. To rerun the ping command use the following: 

**!321**.

You can also rerun past commands by using the up and down arrow to scroll through past commands. To rerun the ping command hit the up arrow once and then hit enter.

By default Raspbian limits the history file to 1000 commands and 2000 lines. However, more than once I needed to rerun a command and it was no longer in my history. With a few changes we can increase these limits or even remove them.

These limits are controlled by the HISTSIZE and HISTFILESIZE variables and are set in your ~/.bashrc file. The HISTSIZE variable controls the number of commands to remember. Setting this to 0 will prevent commands from being stored in history. Setting this to a number less than 0 will save every command (unlimited). Setting this to any other number will allow storing up to that number of commands. 

The HISTFILESIZE variable sets the maximum number of lines the file can contain. It will remove older entries to prevent the number of lines exceeding this value. Similar to HISTSIZE, a 0 value will truncate the file to zero lines (no history). Negative values disable truncation, and all other numbers truncate the file to the specified number of lines. 

So we could set both HISTSIZE and HISTFILESIZE to -1 and have unlimited history. But as the old adage goes there is no such thing as a free lunch. As the file grows it consumes more resources. On a memory and CPU constrained system like the Pi this could become an issue. So instead let’s start by increasing the limits to 10,000 commands and 20,000 lines. To change these limits open your ~/.bashrc file:

**cd**  
**nano .bashrc**

locate the following lines:

HISTSIZE=1000<br />
HISTFILESIZE=2000,

and change them to:

HISTSIZE=10000<br />
HISTFILESIZE=20000.

Save the file `ctrl-o` and exit `ctrl-x` the editor. Reload the ~./bashrc file `source ~/.bashrc` to update the values. 

This is better but only delays the length of time before we begin losing older commands from history. To prevent this and avoid the performance hit of setting an unlimited history size, we can keep a full list of commands in a file that is not loaded automatically when Bash starts. To do this we can add the following to our .bashrc file.

**cd**
**nano .bashrc**

copy and paste the following to the bottom of the file:

<pre>
# Bash eternal history
    # --------------------
    # This snippet allows infinite recording of every command you've ever
    # entered on the machine, without using a large HISTFILESIZE variable,
    # and keeps track if you have multiple screens and ssh sessions into the
    # same machine. It is adapted from:
    # http://www.debian-administration.org/articles/543.
    #
    # The way it works is that after each command is executed and
    # before a prompt is displayed, a line with the last command (and
    # some metadata) is appended to ~/.bash_eternal_history.
    #
    # This file is a tab-delimited, timestamped file, with the following
    # columns:
    #
    # 1) user
    # 2) hostname
    # 3) screen window (in case you are using GNU screen)
    # 4) date/time
    # 5) current working directory (to see where a command was executed)
    # 6) the last command you executed
    #
    # The only minor bug: if you include a literal newline or tab (e.g. with
    # awk -F"\t"), then that will be included verbatime. It is possible to
    # define a bash function which escapes the string before writing it; if you
    # have a fix for that which doesn't slow the command down, please submit
    # a patch or pull request.
    PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND ; }"'echo -e $$\\t$USER\\t$HOSTNAME\\tscreen $WINDOW\\t`date +%D%t%T%t%Y%t%s`\\t$PWD"$(history 1)" >> ~/.bash_eternal_history'

    # Turn on checkwinsize
    shopt -s checkwinsize

    #Prompt edited from default
    [ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u \w]\\$ "

    if [ "x$SHLVL" != "x1" ]; then # We're not a login shell
        for i in /etc/profile.d/*.sh; do
      if [ -r "$i" ]; then
          . $i
      fi
  done
    fi
fi

# Append to history
# See: http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO/x329.html
shopt -s histappend
</pre>

Save the file `ctrl-o` and exit `ctrl-x` the editor. Reload the ~./bashrc file to reload the new configuration. 

This will save commands to a file (~/.bash_eternal_history). You can test the changes by entering a few commands:

**ls –la**<br />
**ping 8.8.8.8**<br />
**cd /etc**<br />
**ls –la**<br />
**cd**<br />

Now check the contents of the new file with the following command:

**cat ~/.bash_eternal_history**

You should see something like the following:

<pre>
2780    pi      Jaguar  screen  05/31/16 17:40:17 2016 1464730817       /home/pi 2053  ls -la
2780    pi      Jaguar  screen  05/31/16 17:40:25 2016 1464730825       /home/pi 2054  ping 8.8.8.8
2780    pi      Jaguar  screen  05/31/16 17:40:30 2016 1464730830       /etc 2055  cd /etc/
2780    pi      Jaguar  screen  05/31/16 17:40:33 2016 1464730833       /etc 2056  ls -la
2780    pi      Jaguar  screen  05/31/16 17:40:35 2016 1464730835       /home/pi 2057  cd
</pre>

If you implemented the system for managing your .bashrc file I described in [part 4](https://raspberrypise.tumblr.com/post/143121276514/improving-your-command-line-skills-part-4) of my Improving Your Command Line Skills Series, you can simply create a new file (e.g. bash_eternal_history.sh) in your .bashrc.d directory with the code from above, and reload the .bashrc file.

Below are several useful commands for working with this new file:

**tail ~/.bash_eternal_history**          # list the last 10 lines of the file<br />
**grep ~/.bash_eternal_history “jira”**   # list all lines include “jira”<br />
**wc –l .bash_eternal_history**       # count the number of lines in the file.<br />
 
Next time you need to remove a PID file to restart a program, or restart a program that doesn’t use the standard service command you’ll be able to find and rerun the necessary command. 

Additional Resources

*[How to use Bash History Commands and Expansions](https://www.digitalocean.com/community/tutorials/how-to-use-bash-history-commands-and-expansions-on-a-linux-vps)<br />
*[The Definitive Guide to Bash Command Line History](http://www.catonmat.net/blog/the-definitive-guide-to-bash-command-line-history/)<br />
*[Understanding Bash History](http://www.symkat.com/understanding-bash-history)
