**Author:** Jacobm001

**Publish Status:** draft

**Suggested Tags:** raspberrypi, raspbian, tmux

# Tmux 102: Getting to know Tmux.

[Tmux 101](https://raspberrypise.tumblr.com/post/141348857424/tmux-101-installing-from-source) discussed how to install Tmux from the source.  This week, we'll be focusing on Tmux's basic usage.

## Launching Tmux

### Open Tmux

Tmux is launched form a standard terminal. If you're in a desktop environment, you'll need to open a terminal, otherwise simply type the command `tmux`. At this point, you should see a terminal that looks similar to what you're accustomed to, with a yellow-green bar on the bottom with some writing on it. Congrats, you've started your first Tmux session!

Before we go any further, it's probably important to clarify the vocabulary that will be used in this post.

- **session:** An instance of Tmux. You can have multiple sessions running at the same time, but in most cases you will only ever interact with one of them at a time. Think of a session like an instance of a webrowser.
- **window:** In Tmux, a window is similar to a web browser's tab.
- **pane:** The actual interactive shell provided by Tmux. By default, each window has one pane, but the window can be divided into multiple panes. This can be though of like different content areas within a browser tab.

### What do all the labels mean?

Tmux manages to fit a surprising amount of information into a relatively small space. On the far left, we see a number (probably `0`) in brackets. This indicates our session name. By default, the session name is `0`. If this session is detached, and another one is created, the new session name will be auto incremented to 1.

Next we see a list of the windows, and their names. Each window is given an auto incremented number, and by default its name is whatever application is currently running on the focused pane. The asterisk (`*`) indicates that window `0:bash` is active.

### Using Tmux

As you may have guessed, you can start entering commands. If you run the command `uptime`, you'll see an output much like you'd normally expect. Where tmux gets interesting, is the pane and window functions mentioned earlier. To enter a command in TMux you need to use the *bind key*. The default bind key is `ctrl-b`. Once pressed, the next key entered will register a given command. 

For a simple example, we'll run two applications simultaniously. In the current pane, type `nano test.txt`. Type in "Hello, World!" and save the file by pressing `ctrl-o`. Now, in another case this might be a much bigger document. You may be writing something important (like this blog post), but need to check on something else (like the system load). So, to switch to another window, use the keys `ctrl-b` and then press `c` . 

You should now see that you have 2 windows, and the newest window is focused.
