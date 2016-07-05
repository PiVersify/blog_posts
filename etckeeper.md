#Tracking Your System’s Configuration Changes with Etckeeper#

**Author: Steve Robillard**

Note: this post assumes a basic knowledge of git. If you are unfamiliar with git I suggest you work through one of the git tutorials available online, see the additional resources section below. 

Etckeeper is a utility that allows us to version control the contents of the /etc directory, using git, mercurial, darcs, or bzr. We will use git in this post. So why would we want to version control the /etc directory? 

The /etc directory is the main configuration directory on Linux (though not the only place that configuration data is kept), and DevOps best practices suggest tracking configuration changes like you would your source code. Also, when something breaks the most likely cause is the most recent change. With version control we can see exactly what was changed, when and by whom (including changes made by apt).
 
Similarly, how many times have you wished, after hours of working to get something working, that you could return the system to a known good state, and start fresh? For example, look at the number of networking questions that resulted from users following older tutorials to setup a static IP, after the Pi Foundation introduced a new non-standard network configuration in May of 2015.  With version control we could have made and tested these changes in isolation, and if necessary discard or roll them back, returning the system to a known good state. 

## Installation ##

Update and upgrade the system:

**sudo apt-get update && sudo apt-get upgrade**

Install git:

**sudo apt-get install git**

Configure git for the root account: 

**sudo git config --global user.name "Your Name" <br />
sudo git config --global user.email youremailaddress**

Configure git for your normal account. While not necessary for etckeeper this ensures that it is configured when you use git for your personal projects, you do version control your code don’t you? You need to switch to your home directory before doing this, and switch back to /etc after.

**git config --global user.name "Your Name" <br />
git config --global user.email youremailaddress**

Install etckeeper:

**Sudo apt-get install etckeeper**

Because git is already installed this will not only install etckeeper, but initialize the database and make an initial commit of the files in /etc.

You can check the status of the repository with the following commands:

**cd /etc <br />
sudo git status**

you should see the following:

**On branch master<br />
nothing to commit, working directory clean**

Note: if you run a git command in /etc without sudo you will see an error message like the following:

**fatal: Not a git repository (or any of the parent directories): .git**

There are a couple of configuration changes I suggest you make.

Open the config file with nano:

**sudo nano /etckeeper/etckeeper.conf**

locate the following line:

**#AVOID_DAILY_AUTOCOMMITS=1**

And change it to (uncomment – remove the #):

**AVOID_DAILY_AUTOCOMMITS=1**

This will prevent code from being automatically committed once a day. I feel that all changes should be explicit and include appropriate commit messages. Something that is not possible with auto commits. Changes made by apt will be automatically committed, but you explicitly asked it to run and hence commit.

Locate the following line:

**#AVOID_COMMIT_BEFORE_INSTALL=1**

And change it to:

**AVOID_COMMIT_BEFORE_INSTALL=1**

This will prevent the system automatically committing before an apt-get install. My reasoning is largely the same as above. If you have uncommitted changes when running apt-get install the install will fail, and you will need to commit the changes before rerunning apt.

Save the changes:

**<kbd>ctrl-o</kbd>** and **<kbd>enter</kbd>**

Then exit the editor:

**<kbd>ctrl-x</kbd>**

If you now check the status of the repository you will see that the etckeeper.conf file has been modified. We now need to stage it and commit the file to the repository. Because this is not a new file we can do this in one step with the following command:

**sudo git commit –am “disable auto commits”**

To further protect this data we can push it to a remote repository like [github](http://github.com) or [bitbucket](http://bitbucket.org). Both offer free accounts but github charges for private repositories. For security reasons this repository should not be publicly accessible. Creating an account and new repository on these services is outside the scope of this article, but once you have done this you can add the remote repository and push changes to your remote repository. 

Note: you will need to create an SSH key for the root user, and add it to your github or Bitbucket account for the following to work, both of the above services have documentation explaining how to do this.

Start by switching to the /etc directory:

**cd /etc**

add the remote repository:
 
**sudo git remote add origin yourrepository’sURL**

push all the changes to the remote repository:

**sudo git push -u origin master**

check the status to confirm it worked:

**sudo git status**

You should see something like the following:

**On branch master<br />
Your branch is up-to-date with 'origin/master'.<br />
nothing to commit, working directory clean**

Etckeeper can be configured to automatically push changes to the remote repository, however, I don’t suggest you do this, since it will still require you entering the passphrase for your SSH key, or adding it to the SSH agent. You did protect your SSH key with a passphrase didn’t you?

Remember etckeeper is only a wrapper around git, so all of the standard git tools and commands are available, branching, logs, stash, diff etc. 

You now have a safer environment to modify and configure your system, and the knowledge that you can always return your system to a kn own good state. Additionally in the event of SD card corruption a simple git pull can recover all of the configuration changes you made to your system's etc. directorty. 
 

##Additional Resources##

- [Codecademy's Git Course](https://www.codecademy.com/learn/learn-git)
- [Bitbucket](http://bitbucket.org})
- [Github](http://github.com)
- [Git succinctly](https://www.syncfusion.com/resources/techportal/ebookconfirm/git/sitevisitors)
- [Pro Git](https://git-scm.com/book/en/v2)

