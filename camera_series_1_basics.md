**Author**: angussidney

**Publish status**: Draft

**Suggested Tags**: raspberrypi, raspbian, cameramodule

#Camera Module Part 1: Getting started

This is the first article in a 3 part series about using the Raspberry Pi Camera Module in various situations. This article is about using the basic functions of the camera in both Bash and Python; future articles will show you how to capture slo-mo, timelapse, and maybe even stream video to another device. 

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

The basic command for taking a picture in Bash is as follows:

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

Here are some other useful commands:

    # Specify the resolution
    camera.resolution = (640, 480) 
    # Start or stop the preview. Use Ctrl+C to stop the preview manually
    camera.start_preview()
    camera.stop_preview()
    