# Linux-DSLR-Webcam
How to turn your DSLR camera into a Web Cam usable on Linux.

## Why?
Imagine you are someone who only uses Linux and wants to stream to Twitch,YouTube, etc and you have a DSLR camera and you want to upgrade things so you decide to see if it is possible to make use of your nice expensive camera with your computer.

Well in this guide that is very much possible! 

### Step 1 (Update and Upgrade your system)
Now you would do this based on your respective Linux OS. In my case that is Debian based so.

```sudo apt update && sudo apt upgrade```

### Step 2 (Software Setup)
We need to install some software to get the camera functionality that we want. That is doing with the following command

Debian Based Systems

```sudo apt-get install gphoto2 v4l2loopback-utils v4l2loopback-dkms ffmpeg```

Arch Based Systems

```pacman -S gphoto2 v4l-utils v4l2loopback-dkms ffmpeg```

Fedora Based Systems

```sudo dnf install gphoto2 v4l2loopback ffmpeg```

### Step 3 (Loading Kernel Module)
We need to load the Video4LinuxV2 module in order to get the functionality that we need and that can be done with the following.

```sudo modprobe v4l2loopback exclusive_caps=1 max_buffers=2```

Loading the Kernel Module like this will require this to be done at every restart as this not is a persistent loading method. To ensure that the Kernel Module loads at system boot you need to do the following.

- Add the dslr-webcam module to the modules list
- Create the dslr-webcam.conf to modprobe.d
- Feed configuration to the dslr-webcam.conf

## Step 3.1 (Adding the dslr-webcam module to the start list)
To get the module to start with system boot we need to do the following.

```sudo vi /etc/modules```
That then brings up the following
```
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
```

At the bottom of that text field we need to add `dslr-webcam` this will then load the Kernel Module at system start and ensure that we can use the module.

## Step 3.2 (Creating the dslr-webcam.conf configuration)

We now need to make the configuration file for the module and we can do that with the following.

```sudo vi /etc/modprobe.d/dslr-webcam.conf```

Upon doing that we are greeted with a blank screen into which we need to type or paste the following

```
# Module options for Video4Linux, needed for our DSLR Webcam
alias dslr-webcam v4l2loopback
options v4l2loopback exclusive_caps=1 max_buffers=2
```

We can now save and exit with the knowledge that the module will start on system boot!

### Step 4 (Testing the Camera)

We should now be able to plug our camera into our system to check to see if it is functional. 

To test to make sure that the camera is being seen and being accessed correctly we do `gphoto2 --summary` That should then give something similar to the following output

![gphoto2 Camera Summary](https://user-images.githubusercontent.com/25136516/230117540-c0a6bb56-3f2e-4184-8dcf-be31c471e38e.png)

## Step 4.1 (Is the Camera actually talking back?)

We now need to confirm if the camera is actually talking back to us and that can be done with

```gphoto2 --capture-image-and-download```

This should then tell us if the camera is talking. Because you should then hear the shutter on the camera and then the command will tell you where it stored the image.

## Step 4.2 (Getting the camera ready for video streaming)

Now that we know that the camera is running correctly we need to find out the identity number assigned to the camera with the following command

```v4l2-ctl --list-devices |grep -A1 Dummy```

That then should give an output that is similar to the following

![gphoto2 camera dummy location](https://user-images.githubusercontent.com/25136516/230119838-73d241f0-463a-4fad-b7a0-e042bcf4e495.png)

On my system the camera is showing up as /dev/video0 this may differ on your system but this is where the camera is located on our system.

Once we know where the camera is we can setup the video streaming with this command


```gphoto2 --stdout --capture-movie | ffmpeg -i - -vcodec rawvideo -pix_fmt yuv420p -threads 0 -f v4l2 /dev/video0```
