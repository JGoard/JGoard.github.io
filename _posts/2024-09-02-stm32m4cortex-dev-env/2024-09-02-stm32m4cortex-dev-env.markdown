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

![A sample layout for how docker itself works with something as simple as gcc](docker-layout.jfif)

# Installing for Windows Hosts
### WSL
Ensure that you have [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) downloaded and enabled for use with docker with the Windows host. I currently use Ubuntu Image 22.04.3 LTS, but it should not matter which distro you use

### Docker
Install [Docker](https://www.docker.com/) on your Windows host, and ensure that docker is enabled for use with WSL and not the Windows hypervisor

### Visual Studio Code
Install [VSCode](https://code.visualstudio.com/) on your Windows host, ensure that you have also downloaded the docker extension in VSCode as well. This will be used in order to attach a workspace to your docker container.

### usbipd
[usbipd](https://github.com/dorssel/usbipd-win) is needed in order to bind usb devices to be used through WSL.

Follow these instructions or else debugging through the docker container will not work as intended. You will still be able to compile software, and use QEMU for container emulation






Jekyll also offers powerful support for code snippets:

Just a basic forever loop...

{% highlight c %}
int main(int argc, char *argv[])
{
  while(1)
  {
    printf("Hello World!");
  }
  return 0;
} 
{% endhighlight %}


