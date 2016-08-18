# Securing your Raspberry Pi with Let's Encrypt
**Author: Jacobm001**

A lot of people use their Raspberry Pis to host a personal website for a variety of reasons. Unfortunately, lack https encryption. Without that layer of security, even a well developed CMS like [Drupal](https://www.drupal.org/), [Wordpress](https://wordpress.org/), or [Grav](https://getgrav.org/) are highly vulnerable to network snooping, man in the middle attacks, and a variety of other concerns.

In the past, SSL certificates were expensive and difficult to maintain. Because of this, few sites used https if it wasn't absolutely necessary. In reality, a lot of sites that **[should](http://codebutler.com/firesheep/)** have used https didn't until relatively recently.

We're not going to go into detail on how [public key encryption](https://en.wikipedia.org/wiki/Public-key_cryptography) works here. However, you should know that to make an https connection, you need a **trusted** third party to take part in the communications between your host and client devices. This is where the part about costing money comes in, and how a barrier to entry has recently been removed.

[Let's Encrypt](letsencrypt.org) is a fairly new organization that provides trusted, 3rd party certificates for free. Not only do they make what was once an expensive product free, they also offer some easy, automated tools that make the process simple. At this point, Let's Encrypt is listed as a trusted source in all major web browsers.

## Assumptions

If you're reading this, I'm assuming that you are already running a web server of some kind and a domain name. If you're not, you may find [this](https://raspberrypise.tumblr.com/post/144922590364/setting-up-a-lamp-stack-on-a-raspberry-pi-part-1) article helpful.

## Using Let's Encrypt

The Raspberry Pi isn't an officially supported device, but the script is written in Python, and both nginx and apache2 are supported, so you shouldn't have any problems with it. Let's Encrypt will auto install any dependencies you're missing and then give you a fairly simple prompt to follow.

Note: you will see one more screen than I have to display. Since both of my machines have letsencrypt previously installed, my administrative email address is already stored.

1. Download certbot-auto: `wget https://dl.eff.org/certbot-auto`
2. Make the certbot-auto script executable: `chmod a+x certbot-auto`
3. Run the script `sudo ./certbot-auto`
4. Select which of your active domains you want to enable https for. It defaults to all sites. ![](letsencrypt_dialog0.png)
5. Select if you want to do *Easy* or *Secure* mode. Easy will allow users to enter your site through both http and https. Personally, I see no reason to allow http for anything, so I always select the *Secure* option. ![](letsencrypt_dialog1.png)
6. Once that's done, you should see a screen telling you that you've succeeded. It will also give you a link you can use to test your new certifications if you wish. ![](letsencrypt_dialog2.png)
7. Finally, you'll see a short outro telling you a little bit about the application. As always, consider donating to your favorite open source applications. ![](letsencrypt_dialog3.png)

Once you've reached this part, you should be done. Try going to `yoururl.com` (replace with your real domain). You should automatically be redirected to `https://yoururl.com`. You'll need to do this again in about two and a half months, as the certificate expires every three. This can be automated, but that comes with certain security trade offs. 

#### Further Reading:

If you would like a simpler explanation as to how public key encryption works, I'd recommend taking a look at [this](https://medium.com/@vrypan/explaining-public-key-cryptography-to-non-geeks-f0994b3c2d5#.zdquvwopu) article. It's slightly over simplified, and cuts out the part about the trusted middle man, but is a good overview none the less.

