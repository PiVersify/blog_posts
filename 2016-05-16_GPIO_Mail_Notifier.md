Creating a GPIO Mail Notifier
==

So, you have a shiny new Pi, some LEDs, and too much free time. What should you do? How about make a device that will light-up for you when you have some unread emails? That's easy! But, what if you want to know what type of emails you have *before* you even open up your mail client? Well, that's a bit more difficult. It's not impossible, though, and I'll show you how to do it quickly!

Pre-requisites:
--
First, you will need:
* A Raspberry Pi (Any model will do!)
* The RPi.GPIO Python Library
* A text editor
* 3 LEDs
* 3 Resistors to not kill your LEDs
* A G-Mail account
* An internet connection on your Pi. I prefer ethernet, but you can use Wi-Fi no problem!

Step 1:
--

First of all, you'll have to do some things in your G-Mail account. If you have 2-Factor Authentication on, you'll need to make sure you generate an app password to use with this little script. After you've done that, or if you don't even have 2-Factor on, you'll need to create three labels that you'll want to keep an eye on. For example, you can make the labels `Work`, `School`, and `Mom`. Then, you'll have to set-up filters in your account so that emails get sorted into those labels according. For example, you can set-up a filter that sorts all mail from `*@boringwork.email` to go into the `Work` category.

Step 2:
--

Alright, here's the fun part. the script. You'll have to modify a couple of the lines yourself to fit the labels you made, but it's fairly straightforward. Copy and paste the entire script below to a new file named `mail-indicator.py`:

```python
#Usage:
#rpi-mailindicator.py -u GMAIL-ADDRESS -p GMAIL-PASSWORD
#optional arguments: -r RED_LIGHT_PIN -y YELLOW_LIGHT_PIN -g GREEN_LIGHT_PIN

import sys, imaplib
import RPi.GPIO as g
from time import sleep

red_pin, yellow_pin, green_pin = 2, 3, 4

if '-u' not in sys.argv:
    print "You must supply a username!"
    exit(1)
  
if '-p' not in sys.argv:
    print "You must supply a password!"
    exit(1)
  
for i in sys.argv:
    if sys.argv[i] == "-u": USERNAME = sys.argv[i+1]
    if sys.argv[i] == "-p": PASSWORD = sys.argv[i+1]
    if sys.argv[i] == "-r": red_pin = int(sys.argv[i+1])
    if sys.argv[i] == "-y": yellow_pin = int(sys.argv[i+1])
    if sys.argv[i] == "-g": green_pin = int(sys.argv[i+1])

#Setup the GPIO header as needed/desired  
g.setmode(g.BCM)
g.setup(red_pin, g.OUT, pull_up_down=g.PUD_DOWN)
g.setup(yellow_pin, g.OUT, pull_up_down=g.PUD_DOWN)
g.setup(green_pin, g.OUT, pull_up_down=g.PUD_DOWN)
g.output(red_pin, 0)
g.output(yellow_pin, 0)
g.output(green_pin, 0)

#Now, let's get signed into G-Mail...
mail = imaplib.IMAP4_SSL('imap.gmail.com')
mail.login(USERNAME, PASSWORD)


while True:
    mail.select('work')
    green_count = mail.search(None,'UnSeen')
    mail.select('school') #Change this to whatever category you want for the yellow light
    yellow_count = mail.search(None, 'UnSeen')
    mail.select('mom') #Change this whatever category you want for the red light
    red_count = mail.search(None, 'UnSeen')
    if red_count == 0: g.output(red_pin, 0)
    else: g.output(red_pin, 1)
    if yellow_count == 0: g.output(yellow_pin, 0)
    else: g.output(yellow_pin, 1)
    if green_count == 0: g.output(green_pin, 0)
    else: g.output(green_pin, 1)
    sleep(0.5)
```

Finally! Execution!
--

So, lastly, you'll want to make this thing run. I recommend running it in a `screen` or `tmux` session so that you can detach it and leave it running in the background, but that's up to you. To run, simply type `sudo python mail-indicator.py -u your.email@gmail.com -p YourSuperSecretPassword`. That's it! Hope you enjoyed it!
