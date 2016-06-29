# An Introduction to Crontab

**Author: Jacobm001**

## What is Crontab?

According to its manpage, `crontab` *is the program used to install, deinstall, or list the tables used to drive the cron daemon*. In plain English, `crontab` is the tool used to schedule applications to be run via the cron daemon.

## What should I use it for? 

You can use a `crontab` entry anytime you have a task that needs to be run in a routine manner. Sometimes that's a script that runs at a specific time interval, a recurring entry, or when the Raspberry Pi's state changes. Examples could include things like running a command every time the Raspberry Pi starts, or reading a sensor every 5 minutes.

Another advantage of contrab is the command issues will be run regardless of whether you're logged in or not. That means you don't need to worry about turning your tiny program into a full fledged daemon, or running an additional program like screen or tmux in the background.

## Using Crontab

You can open your crontab by using the command `crontab -e`. If you've never opened it, you'll be prompted for a favorite text editor. Personally, I'd recommend `nano` for this task. Once chosen (if it hadn't been already), you'll see something like:

    # Edit this file to introduce tasks to be run by cron.
    #
    # Each task to run has to be defined through a single line
    # indicating with different fields when the task will be run
    # and what command to run for the task
    #
    # To define the time you can provide concrete values for
    # minute (m), hour (h), day of month (dom), month (mon),
    # and day of week (dow) or use '*' in these fields (for 'any').#
    # Notice that tasks will be started based on the cron's system
    # daemon's notion of time and timezones.
    #
    # Output of the crontab jobs (including errors) is sent through
    # email to the user the crontab file belongs to (unless redirected).
    #
    # For example, you can run a backup of all your user accounts
    # at 5 a.m every week with:
    # 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
    #
    # For more information see the manual pages of crontab(5) and cron(8)
    #
    # m h  dom mon dow   command

Like most Unix configuration files, `#` indicates that the following line is a comment. That makes the beginning of this file unnecessary, but I highly recommend you keep it around as a reference. It's much more convenient than constantly going back and forth from the man pages.

As you can see in this example, you can issue a complete command (like `tar`) or you can reference a file. Personally, most of my tasks tend to be on the longer side. That makes them difficult to manage in the crontab. Instead, I store them in a hidden directory in my user's home directory named `/home/pi/.cron_scripts`.

## An Example

For this example we're going to archive my website's directory and then push it to a remote location.

### Step 1: Create a new script

Here, we'll write the little program that handles the archiving and pushing of our files. Create a new script by running `nano ~/.cron_scripts/backup.sh`. Into that file put:

    #!/bin/usr/sh
    tar cfa ~/backup.tar.xz /var/www/html
    scp ~/backup.tar.xz user@host.com:/backups/backup.tar.xz
    rm ~/backup.tar.xz
    

Exit nano (`Ctrl-o Ctrl-x`) and then make it executable with the command `chmod +x ~/.cron_scripts/backup.sh`.

### Step 2: Test the script

It may sound silly to test such a simple script, but trust me, troubleshooting here is far easier than at any other step in this process. If you script doesn't work here, it's going to be much more difficult to find out while something is going wonky later. Run the script with `~/.cron_scripts/backup.sh`. Once it's done, check and make sure your directory made it where it was supposed to go.

### Step 3: Add your script to Cron

At this point, you need to choose how often you want to backup your website. For this case, I'll pick a truly arbitrary number and go with weekly backups on Tuesdays at 6:31 in the evening. That gives us an `h` value of 18 (24 hour format), an `m` value of 31, and a `dow` value of 3. Everything else is a `*`. That gives us the line:

    31 18 * * 3 /home/pi/.cron_scripts/backup.sh

Notice that I did not use home directory shortcut, `~`. When using crontab, it's best to always use full paths, rather than relative.

## Other Useful Crontab Commands

There are some commands that can be used in place of the standard time format. `@reboot` will run the command any time the Raspberry Pi is restarted, and there's a shortcut for `@hourly`, `@daily`, `@weekly`, `@monthly`, `@annually`, and `@yearly`.  As always, there's more to his tool than what can be covered here. Play with cron, and spend some time looking at the man page. You never know what useful things you might find in the manual!