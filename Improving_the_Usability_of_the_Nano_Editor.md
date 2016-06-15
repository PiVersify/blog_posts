# Improving the Usability of the Nano Editor #

Every Linux user will have their favorite editor (please no holy wars about the best editor – it is a personal choice); mine is Nano. It may not be as powerful as Emacs or Vim, but it also doesn’t require remembering arcane keyboard shortcuts, or come with a Mount Everest learning curve either. I have used both Emacs and Vim, but Nano is still my editor of choice – partially because it is very similar to the Vax editor (EDT) I used in university.
 
The default installation leaves a lot of room for improvement. By default it doesn’t show line numbers or cursor position, jumps when scrolling through a file, making it hard to scan the text, and lacks syntax highlighting. We can solve all of these problems with a ~/.nanorc file. 

Note you can copy and rename the system wide config file (/etc/nanorc) to your home directory and edit or uncomment the features you want to enable, but I prefer creating the file from scratch with only the changes I need reducing the noise and clutter. 

To begin switch to your home directory

**cd** 

create an empty .nanorc file:
 
**touch .nanorc**

then open it with nano

**nano .nanorc**

To enable line numbers and cursor position add the following:

<pre>
### Enable line numbers 
set const
</pre>

Lines that begin with # (e.g. ### Enable line numbers are comments) and are ignored by nano. But they can help when trying to remember that set const is for line numbering - which is not exactly the clearest or most semantic choice.

To enable line by line scrolling instead of the chunk by chunk default behavior, add: 

<pre>
### Enable smooth scrolling 
set smooth
</pre>

Nano comes with several language specific syntax files (e.g python, php, html etc.). By default these are installed in the /usr/share/nano directory. You can link to these files from your .nanorc file by including the following: 

<pre>
### Enable syntax highlighting for .sh files
include /usr/share/nano/sh.nanorc
</pre>

However, there are a few drawbacks to this approach. First they are global shared files; which means that any change you make (e.g. changing the color of keywords) will affect all users who have linked to this file. You may love pink highlighted keywords, but others may not share your avant-garde color choices. This may not be that big a deal on the Pi – where multiple user setups are less common, but should be avoided. Second, on other systems where you do not have admin privileges you may not be able to edit these files. Lastly, the default files provide no support for several common languages (e.g. javascript, lua, puppet). 

A better approach would be to maintain your own copy of the nano syntax highlighting files. Thankfully github user scopatz https://github.com/scopatz/nanorc has made this as easy as cloning a repository and including the files in your .nanorc file. First we will need to install git:

**sudo apt-get update<br />
sudo apt-get install git<br />**

Switch to your home directory:

**cd**

Clone the repository:

**git clone  https://github.com/scopatz/nanorc.git ~/.nano**

if you included:

<pre>
### Enable syntax highlighting 
include /usr/share/nano/sh.nanorc
</pre>

From above, change it to:

<pre>
### Enable syntax highlighting 
</pre>

Save the file:

**Ctrl-O** and  **Enter** 

And exit nano:

**Ctrl-x**

To Include the syntax highlighting files enter the following at the command propmpt:

**find ~/.nano/ -iname "*.nanorc" -exec echo include {} \; >> ~/.nanorc**

This will append an include line for each syntax file in your .nano directory. 

The complete file will now contain the following:

<pre>
### Enable line numbers
set const

### Enable smooth scrolling
set smooth

###### Enable syntax highlighting
include /home/pi/.nano/yum.nanorc
include /home/pi/.nano/pkgbuild.nanorc
include /home/pi/.nano/json.nanorc
include /home/pi/.nano/keymap.nanorc
include /home/pi/.nano/xresources.nanorc
include /home/pi/.nano/makefile.nanorc
include /home/pi/.nano/erb.nanorc
include /home/pi/.nano/markdown.nanorc
include /home/pi/.nano/pov.nanorc
include /home/pi/.nano/Dockerfile.nanorc
include /home/pi/.nano/zshrc.nanorc
include /home/pi/.nano/yaml.nanorc
include /home/pi/.nano/cmake.nanorc
include /home/pi/.nano/ledger.nanorc
include /home/pi/.nano/vi.nanorc
include /home/pi/.nano/po.nanorc
include /home/pi/.nano/javascript.nanorc
include /home/pi/.nano/sed.nanorc
include /home/pi/.nano/nginx.nanorc
include /home/pi/.nano/privoxy.nanorc
include /home/pi/.nano/rpmspec.nanorc
include /home/pi/.nano/git.nanorc
include /home/pi/.nano/mutt.nanorc
include /home/pi/.nano/email.nanorc
include /home/pi/.nano/gitcommit.nanorc
include /home/pi/.nano/ocaml.nanorc
include /home/pi/.nano/dot.nanorc
include /home/pi/.nano/coffeescript.nanorc
include /home/pi/.nano/rust.nanorc
include /home/pi/.nano/lua.nanorc
include /home/pi/.nano/conky.nanorc
include /home/pi/.nano/html.nanorc
include /home/pi/.nano/vala.nanorc
include /home/pi/.nano/php.nanorc
include /home/pi/.nano/man.nanorc
include /home/pi/.nano/css.nanorc
include /home/pi/.nano/csharp.nanorc
include /home/pi/.nano/peg.nanorc
include /home/pi/.nano/inputrc.nanorc
include /home/pi/.nano/js.nanorc
include /home/pi/.nano/gentoo.nanorc
include /home/pi/.nano/mpdconf.nanorc
include /home/pi/.nano/puppet.nanorc
include /home/pi/.nano/sls.nanorc
include /home/pi/.nano/fish.nanorc
include /home/pi/.nano/tex.nanorc
include /home/pi/.nano/nanorc.nanorc
include /home/pi/.nano/zsh.nanorc
include /home/pi/.nano/ini.nanorc
include /home/pi/.nano/python.nanorc
include /home/pi/.nano/haml.nanorc
include /home/pi/.nano/systemd.nanorc
include /home/pi/.nano/glsl.nanorc
include /home/pi/.nano/lisp.nanorc
include /home/pi/.nano/awk.nanorc
include /home/pi/.nano/swift.nanorc
include /home/pi/.nano/cython.nanorc
include /home/pi/.nano/tcl.nanorc
include /home/pi/.nano/groff.nanorc
include /home/pi/.nano/java.nanorc
include /home/pi/.nano/patch.nanorc
include /home/pi/.nano/perl.nanorc
include /home/pi/.nano/xml.nanorc
include /home/pi/.nano/scala.nanorc
include /home/pi/.nano/pkg-config.nanorc
include /home/pi/.nano/sql.nanorc
include /home/pi/.nano/arduino.nanorc
include /home/pi/.nano/perl6.nanorc
include /home/pi/.nano/kickstart.nanorc
include /home/pi/.nano/sh.nanorc
include /home/pi/.nano/go.nanorc
include /home/pi/.nano/ruby.nanorc
include /home/pi/.nano/c.nanorc
include /home/pi/.nano/fortran.nanorc
include /home/pi/.nano/apacheconf.nanorc
include /home/pi/.nano/haskell.nanorc
include /home/pi/.nano/asciidoc.nanorc
include /home/pi/.nano/conf.nanorc
include /home/pi/.nano/colortest.nanorc
include /home/pi/.nano/asm.nanorc
include /home/pi/.nano/reST.nanorc
</pre>

To test your changes reopen the .nanorc file:

**nano .nanorrc**

You should now have line numbers and cursor position indicated above the menu at the bottom of the screen and nicely highlighted code. If you arrow down through the file it should scroll one line at a time. 

I have only touched the surface of the customizations nano supports. You can view the full list of options with the man command:

**man nanorc**

If Python is your primary language you may want to look at the following options tabsize, tabstospaces and autoindent. If you are a mouse centric user you may want to enable the mouse option.

## Additional Resources ##
[Nano Tips and tricks](https://www.if-not-true-then-false.com/2009/tuning-nano-text-editor-with-nanorc/) https://www.if-not-true-then-false.com/2009/tuning-nano-text-editor-with-nanorc/

A two part series on nano shortcuts and syntax highlighting [part 1](https://freethegnu.wordpress.com/2007/06/23/nano-shortcuts-syntax-highlight-and-nanorc-config-file-pt1/) and [part 2](https://freethegnu.wordpress.com/2007/06/23/nano-shortcuts-syntax-highlight-and-nanorc-config-file-pt2/)
