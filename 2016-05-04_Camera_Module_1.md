**Author**: angussidney

**Publish status**: Published May 04, 2016

**Suggested Tags**: raspberrypi, raspbian, cameramodule

#Camera Module Part 1: Getting started

This is the first article in a series dedicated to using the Raspberry Pi Camera Module in various situations. This article is about using the basic functions of the camera in both Bash and Python; future articles will show you how to capture slo-mo, timelapse, and maybe even stream video to another device. 

This article assumes you have a [Raspberry Pi Camera Module](https://www.raspberrypi.org/products/camera-module/) and a Pi with a camera port (that is, any model except the Zero). A stand for the camera is also highly recommended.

##Enabling the Camera

This article assumes that you are running the latest version of Raspbian. To make sure you are using the latest version, perform the following command:

    $ sudo apt-get update && sudo apt-get upgrade

The camera module is not always enabled by default. To enable it, open the configuration application using the following command:

    $ sudo raspi-config

Go to the Enable Camera Module option. Select 'Enable' and press enter. When you are done, press finish and let the Pi reboot.

Throughout this article you will be shown how to control the camera with both Bash (Linux command line) or Python. If you wish to use Python, you will need to install the [picamera](https://github.com/waveform80/picamera) module using the relevant command:

    $ sudo apt-get install python-picamera     # Python 2.x
    $ sudo apt-get install python3-picamera    # Python 3.x

and use this code at the start of your program to import the module:

    import picamera
    camera = picamera.PiCamera()


##Capturing still images

###Bash/Command line

The basic command for taking a picture in Bash is `raspistill`. You use it like this:

    $ raspistill  -o test.png
    
The `-o` flag specifies that you want to save/output the image, `test.png` can be replaced with whatever filename or path you like. If you are using a stand, you may find that the image is upside down. You can use the `-vf` and `-hf` flags to flip the image vertically or horizontally, respectively.

Other useful flags:

 - `-w px`: The width of the image. Replace `px` with width in pixels. Max 2592.
 - `-h px`: The height of the image. Replace `px` with height in pixels. Max 1994.
 - `-t secs`: The delay before the image is taken. Replace `secs` with time in seconds. Defaults to 5 seconds if not present.
 - `-k`: Press enter every time you wish to take a photo. To exit, press X followed by Enter. Note that it overwrites the existing file rather than creating a new one.
 
###Python
 
This is the basic command for capturing images with Python. Don't forget to add the import and PiCamera commands to the top of your script!

    camera.capture('test.png')
    
Like Bash, we can flip our image if it is upside down (make sure these commands go before the capture command):

    camera.hflip = True
    camera.vflip = True

Finally, always close the camera before ending the script to clean up:

    camera.close()

Here are some other useful settings:

 - Specify the resolution in pixels along the x and y axis.
       camera.resolution = (x, y) 
 - Start or stop the preview. Use `Ctrl+C` to stop the preview manually.
       camera.start_preview()
       camera.stop_preview()
 - Insert a delay before/between captures. Replace `s` with seconds.
       from time import sleep
       sleep(s)


##Capturing Videos

###Bash/Command line

To capture videos, we use the `raspivid` command. Like `raspistill`, we use the `-o` flag and a filename to output our video:

    raspivid -o vid.mp4

This will output a 5-second video encoded in `h.264`. You can play it by running:

    omxplayer vid.mp4

Like `raspistill`, we can use `-vf` and `-hf` to flip the video, `-w` and `-h` to specify the resolution, and `-k` to capture multiple videos. However, `raspivid` also presents us with two new options:

 - `-t ms`: Time to record for. Replace ms with milliseconds.
 - `-fps frames`: Frames per second to record. Replace frames with fps.

###Python

We can use the following commands to start or stop a recording:
    
    camera.start_recording('vid.mp4')
    camera.stop_recording
    
To wait between starting and stopping the recording, we can use the `wait_recording` command (where s is seconds):

    camera.wait_recording(s)

Note that `wait_recording()` is different to `time.sleep()`. `wait_recording()` will throw an exception and stop if any errors are found (e.g. not enough space), while `time.sleep()` will continue *even if* errors are found, potentially causing even more problems.

##Disabling the red light

You may notice that whenever the camera is recording, a red LED is turned on. If you want this turned off (e.g. you are using the Pi as a wildlife cam or the light is reflecting off shiny objects), [edit your Pi's](http://elinux.org/R-Pi_configuration_file#How_to_edit_from_the_Raspberry_Pi) `config.txt` to add the following line:

    disable_camera_led=1

##Further Reading

 - Run the `raspistill` and `raspivid` commands without any parameters to find all flags and settings, including changing the brightness, exposure, contrast, saturation, etc.
 - Read the [picamera documentation](https://picamera.readthedocs.org/en/release-1.10/index.html) for additional examples and settings for controlling the camera via Python
 - The Raspberry Pi website has [documentation and tutorials](https://www.raspberrypi.org/documentation/usage/camera/README.md) for both [Bash](https://www.raspberrypi.org/documentation/usage/camera/raspicam/README.md) and [Python](https://www.raspberrypi.org/documentation/usage/camera/python/README.md).