**Author:** Jacobm001

**Publish Status:** published 03/20/2016

**Suggested Tags:** raspberrypi, raspbian, tmux

# Tmux 101: Installing from Source

For the first entry in our *[Tmux](https://tmux.github.io/) 101* series, we're going to learn how to install Tmux from source. 

## What is Tmux?

Tmux is a "terminal multiplexer". In short, it allows you to have multiple "windows" in the same terminal. If desired, the windows can then be split into "panels" that allow you to have mutliple terminal inputs visible at the same time. What's possibly the most useful feature though, is that Tmux allows you to leave software running in the background, even if you disconnect the session.

## Why install from the source? Isn't the Repo good enough?

In most cases, the repository is adequate for the average users needs. It is however, imperfect. The repo doesn't contain all the software that you may want to install (large as it is), and that software is often significantly behind the current version.  Now, there's often good reason for this. Software in the repository has been pretty thoroughly tested and you can be reasonably sure it's not going to break your system. Both of those factors take time, and mean that the repository can't necessarily guarantee the most recent version of the software in question.

If the software in question is missing features that may be found in a more current version, you may want to install from source.  In this case, the repository has version `1.7`, but the most recent release of tmux is `2.1`. 

## Installing:

### 1. Installing Dependencies

The first thing we'll need to do is install some dependencies. As the homepage specifies, tmux relies on the libraries, [libevent](http://libevent.org/) and [ncurses](http://invisible-island.net/ncurses/). We can install these by using the commands[^1]:

	sudo apt-get update
	sudo apt-get upgrade
	sudo apt-get install libevent-dev libncurses5-dev

### 2. Getting tmux

Now that the dependences are installed, we can download the source code for tmux. Download and unzip the source using the commands[^2]:

	wget https://github.com/tmux/tmux/releases/download/2.1/tmux-2.1.tar.gz
	tar xfa tmux-2.1.tar.gz
	cd tmux-2.1

In case you're unfamiliar with what you just did, `wget` downloaded an archive file containing the tmux source code. `tar` is then used to extract the files[^2]. Finally, `cd` changes the pwd to location where we extracted our files.

### 3. The configuration step

At this point, we're in the directory that contains the source files for tmux. While all these files are important to the eventual process, we're really only concerned with one file, `configure`. Running this script will check for dependencies, and assuming all are met, create a [Makefile](https://en.wikipedia.org/wiki/Makefile). Run this script with the command `./configure`.

Since we've already installed the required dependencies, it should finish without errors on a standard Raspbian installation. 

### 4. Compile and install Tmux

If you view the directory using `ls` you should now see a Makefile. To compile tmux, type the command `make`. This will take a significant amount of time. Go grab a cup of coffee, some pi, whatever... be patient. 
 
In case you're familiar with the `-j` parameter, do **not** use this with tmux. `-j` allows for multiple cores to be used when compiling software, which normally reduces compile time. The makefile produced by the configure script does **not** support parallel compilation, and will cause it to throw an error.

Once it's finished, the software can be installed by typing `sudo make install`. Notice the `sudo` at the beginning of the command. Installing software in this manor includes administrative rights, just as if you had done it with `apt-get`.

### 5. Test and clean up

The last thing to do is to refresh the terminal session. Assuming you're using `bash` (the default terminal shell), this can be done by either using the command `source ~/.bashrc` or by simply closing and reopening the terminal window. Now if you type `tmux`, you should see a new tmux session pop up. If it does, everything works, and we can begin cleaning up the extra files. You can close Tmux by typing `exit`.

Finally, we just need to clean up our mess. Use the command `cd ..` (assuming you didn't close the window in the previuos step) to get to the parent directory. We can remove both the archive file, and the source directory that we no longer need by typing `rm -Rf tmux*`. 

## Next week

Now that Tmux is successfully installed, feel free to browse some of its documentation. Next week's post will focus on Tmux's basic commands and session management.

## Additional Resources
- [Tmux](https://tmux.github.io/)
- Tmux [Documentation](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man1/tmux.1?query=tmux&sec=1)

[^1]: These commands work on Raspbian and most Debian based operating systems. If you're running a different OS, like Arch, you'll need to use that distribution's package manager instead.

[^2]: The flags as follow are:
- `x`: extract
- `f`: read from a file
- `a`: automatically detect the type of archive (`gz`, `bzip2`, `lzma`, etc)
