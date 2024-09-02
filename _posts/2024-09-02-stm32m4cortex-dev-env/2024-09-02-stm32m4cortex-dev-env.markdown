---
layout: post
title:  "STM32 Cortex-M4 Docker Based Development Enviroment"
date:   2024-09-02 12:08:38 -0400
categories: jekyll update
---
Recently, I have been working on a side-project of developing an RTOS like operating system on the STM32 Nucleo F303RE development board, which is based on the M4-Cortex Chipset.

I have been working on this project with an old friend from college named [Nick Schneider](https://github.com/nick-a-schneider).

As we were deciding on how to setup the development enviroment for this project, Nick brought up the possibility of using docker for compiling our source code to the appropriate .bin or .elf format.

This setup worked well for awhile. We still needed our gdb and openocd tools locally which required some syncing of enviroment variables for the program paths to be invocated from PowerShell. 

From further research however, I found that it is possible to completely work from the docker container, and to be able to have compiling, git/source control tracking, and debugging capabilities derived from this enviroment as well.

I will be showing the proper procedure for utilizing a containerized docker enviroment on a Windows 10 host, some steps will not be needed if a Linux based host is used.

![A sample layout for how docker itself works with something as simple as gcc](https://visualgdb.com/w/wp-content/uploads/2016/01/docker.png)

# Programs needed for Windows Hosts
### WSL
Ensure that you have [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) installed and enabled for use with docker with the Windows host. I currently use Ubuntu Image 22.04.3 LTS, but it should not matter which distro you use (to my knowledge at least).

### Docker
Install [Docker](https://www.docker.com/) on your Windows host, and ensure that docker is enabled for use with WSL and not the Windows hypervisor

### Visual Studio Code
Install [VSCode](https://code.visualstudio.com/) on your Windows host, ensure that you have also downloaded the docker extension in VSCode as well. This will be used in order to attach a workspace to your docker container.

### usbipd
[usbipd](https://github.com/dorssel/usbipd-win) is needed in order to bind usb devices to be used through WSL.


# Creating a Dockerfile
This part will be fairly simple and straightforward, I will be borrowing this process from one of my own projects and listing the steps needed below...

{% highlight Dockerfile %}
FROM ubuntu:23.10

# update and install basic tools
RUN apt-get update && \
    apt-get install -y \
    curl \
    udev \
    wget \
    nano \
    build-essential \
    symlinks \
    expect \
    git 
#Copy current git repo into project directory
RUN git clone https://github.com/JGoard/jocktos.git && cd ..

# install emulator and C build tools
RUN apt-get install -y \
    qemu-system \
    gcc-arm-none-eabi \
    gdb-arm-none-eabi \
    libnewlib-arm-none-eabi
RUN apt-cache policy gcc-arm-none-eabi

# install usb dev dependencies and openocd
RUN apt-get install --no-install-recommends -y \
libhidapi-hidraw0 \
libusb-0.1-4 \
libusb-1.0-0 \
libhidapi-dev \
libusb-1.0-0-dev \
libusb-dev \
libftdi-dev \
libtool \
usbutils \
openocd \
# install make tools
make \
cmake \
automake \
pkg-config \
autoconf \
texinfo 

RUN cd ..
# OpenOCD talks to the chip through USB, so we need to grant our account access to the FTDI.
COPY openocd.cfg /usr/local/share/openocd/openocd.cfg  
RUN /lib/systemd/systemd-udevd --daemon && udevadm control --reload-rules

#OpenOCD talks to the chip through USB, so we need to grant our account access to the FTDI.
EXPOSE 3333
EXPOSE 4444
EXPOSE 6666
CMD ["tail", "-f", "/dev/null"]
{% endhighlight %}

You may edit this dockerfile however you would like, we have added a doxygen build system to this dockerfile, which may not be needed for your project

# Building a Dockerfile into an Image

Now that the dockerfile has been created, you must now build a docker image. The docker image will allow for containers to be created from this image. The container is what we will be attached to for our development enviroment.

To build this docker image, you must run the following commands in your local powershell window...

{% highlight Powershell %}
docker build -t **docker_image_name**:latest .
{% endhighlight %}

As you have the image built, now comes the moment of truth. Connecting a USB device, and launching a container...

# Attaching a USB Device to WSL using usbipd

With your STM32 board, you must find the VID and PID combination for this device. This [Link](http://www.linux-usb.org/usb.ids) provides the defined VID:PID combinations that should work for any STM32 Dev board with a built in debugger.

For this example, I will be using my personal STM32F303RE Development Board which utilizes ST-LINK/V2.1 onboard.

{% highlight Powershell %}
VID:PID Combinatsion -> 0483:374b // ST-LINK/V2.1 VID:PID Number
{% endhighlight %}

## Binding and Attaching USB Device to WSL

This step only needs to be performed once on the local host system.

From the usbipd help file...

{% highlight Powershell %}
Registers a single USB device for sharing, so it can be attached to other machines. Unless the
  --force option is used, shared devices remain available to the host until they are attached to
  another machine.
{% endhighlight %}

Now, I can run the following command when my STM32 board is connected to the host machine...

{% highlight Powershell %}
usbipd bind --hardware-id 0483:374b
{% endhighlight %}

This should be successful.

Attaching the USB device to usbipd and passing it through to WSL should be a similar command. Ensure the device is connected to the local host.

{% highlight Powershell %}
usbipd attach --wsl --hardware-id 0483:374b
{% endhighlight %}

Your device should now be successfully attached to WSL.

## Launching a Docker Container 

Launching a container is a straightforward enough process...

You must launch the container in '--privileged' and '-i' for privileged access and interactive mode. 

NOTE: '--privileged' should be used carefully to avoid giving too much external access

{% highlight Powershell %}
docker run -i --expose=3333 --expose=4444 --expose=6666 --privileged *image_name*:latest sh
{% endhighlight %}

This should now launch a container with the appropriate permissions in order to utilize openocd and gdb with your board and debugger of choice.

## Attaching VSCode to Container






