**Author:  Steve Robillard**

In the precious three parts of this series we covered [keyboard shortcuts, Man and TLDR pages](https://raspberrypise.tumblr.com/post/141758901139/improving-your-command-line-skills-part-1), [Bash aliases](https://raspberrypise.tumblr.com/post/142215518744/improving-your-command-line-skills-part-2) and [Functions](https://raspberrypise.tumblr.com/post/142634263599/improving-your-command-line-skills-part-3). In this final part I will present a way of managing all the additions we have made to the .bashrc file. 

## Managing the .bashrc file ##

Over the course of this series we have made several additions to our .bashrc file, and it is starting to get a little long and unwieldly. Manually making these changes only gets worse when we have multiple machines, and want to maintain a consistent environment across all of them. My ideal solution to this problem would be automated, group changes into atomic units and version controlled. 

Automation will make maintaining a consistent environment across machines easier, and avoid the redundancy and errors that come with a manual process. Atomic changes allow us to apply only those changes that are appropriate on a per machine basis (e.g. applying the path changes and environment variables needed by Java only on machines where Java is installed). Some may argue that version control is overkill for these types of files; but Git provides the same benefits here as on larger software projects, tracking changes, coordination, cooperation and the ability to experiment without fear. 

In researching this problem I came across several solutions:

- [Versioning the home directory with Git](https://www.digitalocean.com/community/tutorials/how-to-use-git-to-manage-your-user-configuration-files-on-a-linux-vps),
- Keeping dotfiles (e.g. .bashrc, .nano) in a [subdirectory and symlinking](http://blog.smalleycreative.com/tutorials/using-git-and-github-to-manage-your-dotfiles/), and 
- Creating a [conf.d like directory](http://chr4.org/blog/2014/09/10/conf-dot-d-like-directories-for-zsh-slash-bash-dotfiles/). 

Keeping the home directory in Git causes problems with subdirectories and maintenance of the .gitignore file. Similarly, using a Git checkout to a subdirectory for dotfiles (e.g. .bashrc, .nano, .gitconfig) requires the creation of lots of symlinks to make things work. Neither of these options are what I consider clean and both moved the maintenance issue elsewhere instead of eliminating it. 

In contrast using a conf.d style directory allows us to:

- All changes are kept in a single directory and can cleanly be versioned, without needing symlinks, 
- Requires only a minimal one time change to the .bashrc file (which can be automated with Puppet or a shell script – though that is beyond the scope of this post).
- Can be updated/synced with a simple Git pull, 
- Allows grouping of changes into atomic units. 

## How to do it ##

Switch to your home directory:  

**cd**

Create the directory to hold our files:

**mkdir .bashrc.d**

Edit your ~/.bashrc file:

**Nano .bashrc**

Add the following to the end of the file:

    # Load all files from .bashrc.d directory
    if [ -d "$HOME/.bashrc.d" ] && [ "$(find "$HOME/.bashrc.d/" -maxdepth 1 -name "*.sh" | wc -l)" -gt 0 ]; then
      for file in $HOME/.bashrc.d/*.sh; do
        source $file
      done
    fi


Save your changes and exit the editor (**ctrl + o, ctrl + x**). 

Now any shell script (.sh) file you add to the new directory will be included when your .bashrc file is sourced (i.e. when you login, open a new terminal or source it).
 
If you added last week’s extract function to your .bashrc file remove it, and in the .bashrc.d folder create a new file called extract.sh:

**cd .bashrc.d
touch extract.sh**

Now edit the file:

**nano extract.sh**

and paste the following text:

    # credit: http://nparikh.org/notes/zshrc.txt
    # Usage: extract 
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
    }

Save your changes and exit the editor (**ctrl + o, ctrl + x**).
 
Now source your .bashrc file:

**source ~/.bashrc**

and test the extract function:

**wget https://github.com/tmux/tmux/releases/download/2.1/tmux-2.1.tar.gz
**extract tmux-2.1.tar.gz**

Once everything is working you can create a git repository to track changes. Switch to the .bashrc.d directory:

**cd .bashrc.d**

initialize the repository:

**git init**

add your files to the repository:

**git add .**

and commit: 

**git commit –am “initial commit”**

creating a github account and pushing to github is left as an exercise for the reader. Once that is done adding the extract function to a new machine just requires:

- editing your .bashrc file, 
- cloning the remote git repo, and 
- sourcing your .bashrc file. 

You can add additional alaiases and functions in a similar way. In a future post I will share some additional customizations I use for SSH, and the Bash history. 

Note: while I did not come up with this solution, I did make a few enhancements to the original code:

- Renamed the directory, 
- Removed the code needed to support other shells, and 
- Added a guard check for an empty directory. 

