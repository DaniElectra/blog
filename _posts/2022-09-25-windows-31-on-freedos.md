---
layout: post
author: Daniel López Guimaraes
title: Running Windows 3.1 Enhanced mode on FreeDOS 
---

The [FreeDOS](https://freedos.org) operating system is an open-source version of DOS. One of it's approaches is to be a drop-in replacement for Microsoft's MS-DOS. Currently, almost every (if not every) DOS program works flawlessly and without any modifications whatsoever, except for one: Microsoft Windows.  
Currently, you can run every version of Windows as long as it doesn't require 386 enhanced mode. For example, if you try to run Windows 3.1 enhanced, you will get errored out telling you that there is protected software running, and telling you to close it.

## Introduction  
This problem is caused (among other causes) by the GEMMIS specification. From what I understand, it tells how memory should be managed by the various memory managers available for DOS. [Jemm](https://github.com/Baron-von-Riedesel/Jemm/issues/5), which is the memory manager used by FreeDOS, doesn't currently support GEMMIS.  
However, on 19th August 2021, a commit was made to the [FreeDOS kernel](https://github.com/FDOS/kernel/commit/9186e6c5ed1ab58bf1dc0497bacc352d3d758703) which added support for Windows 3.x 386 Enhanced mode. Sadly, this approach still doesn't work on Windows for Workgroups and it requires Jemm **not** to be loaded, but it's a great step!  
In this post, I will talk about me trying to get Windows 3.11 (not WfW) on FreeDOS using **real hardware**!

## The hardware
The machine I will be using for this experiment is a *Compaq Armada 100S* laptop from 1999. It's hardware specifications are:  

- Processor: AMD K6-2+ 533MHz  
- Graphics: Trident Cyberblade i7  
- RAM: 64MB  
- Hard drive: A *clicky* 5GB IDE drive  
- I/O: VGA, parallel, audio ports and a single **USB** port on the back  

See more detailed specifications in the image below:
![Specifications of the Compaq laptop](assets/img/2022-09/compaq-specs.jpg)  

The laptop, as we can see on the image, has currently installed *Damn Small Linux* (DSL), using the Linux 2.4 kernel, so we will need to erase it.  

## Installing FreeDOS  
Since I was installing DOS on a laptop that originally came with a DOS-based version of Windows (Windows 98), I thought the installaton would be easy and straightforward. However, the laptop will only boot from a floppy or a CD. Since I don't have a lot of CDs available, I will need to use a Plop Boot Manager and install FreeDOS from USB.  
When trying to boot to FreeDOS, it didn't load completely. The laptop would simply **hang** at a random point whether it's on boot or in the installation GUI.  

![FreeDOS failing to load](assets/img/2022-09/freedos-boot-usb.jpg)

I tried different approaches to fix (or skip) the error, like using different images of FreeDOS, and the most successful of them was using the FreeDOS environment of [Rufus](https://rufus.ie), where I could get FreeDOS running.  
The solution that lead me to installing FreeDOS on the laptop was to install it first on a VM, and copy the files to the laptop (not before reformatting the drive to FAT32) using DSL from USB, which worked completely fine.  

![File copying using DSL](assets/img/2022-09/copying-files.jpg)  

After transferring the files to the machine, I booted into FreeDOS from Rufus and installed the MBR (`fdisk /mbr #`) and the bootloader (`sys x:`) for DOS. When I unplugged everything and rebooted the machine, the FreeDOS installation booted from the drive with no errors!

![Installing the MBR and bootloader](assets/img/2022-09/set-bootloader.jpg) ![FreeDOS booting from the hard drive](assets/img/2022-09/freedos-boot.jpg)

## Installing and using Windows 3.11  
The installation of Windows 3.11 was easier than FreeDOS. Before proceeding with the installation, I will explain the requirements to make Windows 3.1 work on FreeDOS:  

### Requirements for Windows 3.1x Enhanced on FreeDOS

- You need to **compile** the kernel by yourself and add the *Windows 3.1* parameter, as the code that activates Windows 3.1x support won't be enabled by default on the kernel.  
- You can have HimemX running, but you **cannot** load Jemm, as it will complain about protected software running and tell you to close it.  
- Before running Windows 3.1x, you **must** run the `share` command and add the following parameter to the Windows `system.ini` file:  

```ini
[386Enh]
InDOSPolling=TRUE
```

I started the installation with Jemm running, as the GEMMIS detection isn't applied during the installation. The setup went flawlessly, and it installed without any errors whatsoever (I only needed to rewrite the `autoexec.bat` and `config.sys` files to the FreeDOSS counterparts `fdauto.bat` and `fdconfig.sys`).  

![The Windows 3.11 installaton GUI](assets/img/2022-09/win31-setup.jpg) ![The Windows 3.11 installation finish GUI](assets/img/2022-09/win31-setup-2.jpg)  

After rebooting in Safe mode (to prevent Jemm loading) and loading `share`, I could start Windows 3.11 in 386 Enhanced mode!  

![Windows 3.11 Enhanced mode on FreeDOS](assets/img/2022-09/win31-boot.jpg)

**Fun fact:** I tried using the Super VGA driver that Windows 3.11 has for 256 colors and 800x600 resolution and worked correctly without any modifications!  

![Windows 3.11 SVGA on FreeDOS](assets/img/2022-09/win31-custom-boot.jpg)  

## Final  
Since there is some interest on the Internet about running Windows 3.1x Enhanced under FreeDOS, I will try to post my pre-compiled kernel files, so that you don't need to compile them. But first, I need to recompile them, since they are outdated to the latest commit in GitHub.  

Thanks for reading, and have a nice day!

