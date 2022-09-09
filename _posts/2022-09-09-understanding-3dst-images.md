---
layout: post
author: Daniel López Guimaraes
title: Understanding the Nintendo Anime Channel's 3DST images
---

Many years ago, I used to use the Nintendo Anime Channel application on my Nintendo 3DS. It's concept was simple: a service where you could watch some anime series that were licensed to Nintendo directly from your 3DS (and completely free).  

Sadly, [the service was shut down](https://www.nintendo.co.uk/Nintendo-3DS-Family/Download-Content/Nintendo-Anime-Channel/Nintendo-Anime-Channel-947961.html "Nintendo's official statement") the 31st October 2018, but in recent years, I've found about *homebrewing* consoles and all of its advantages, including the "revival" of services that were shut down by their original developers.

## Introduction
  
A few months ago, I searched about revivals of the Nintendo Anime Channel and found [this project](https://github.com/cooolgamer/Nintendo-Anime-Channel-Revival) on GitHub. Although it was possible to play your own videos using the Mobiclip's MOFLEX format (if you had access to Nintendo's 3DS development kit, as there are currently no encoders available to public), there wasn't any information about how the thumbnails worked (we only knew that they were in a 3DST format, by looking at the revival's project files).

## The investigation

My first approach was to do a Google search about these files. By doing that, I've found this [GBATemp forum post](https://gbatemp.net/threads/3dst-files-with-texture-as-first-seven-bytes.486126/) talking about these files and looking for a way to edit them. This post had a link to a 3DST file, but the URL wasn't working at all. Luckily, the URL was [archived](https://web.archive.org/web/20180905210456/https://dm13bvvnwveun.cloudfront.net/inazuma/thumbs/IE26-thumbnail.3dst) on the Wayback Machine.  
The post also had an answer including a modified version of Ohana3DS-Rebirth which *claimed* to support 3DST files, but when trying it, it errored out with `Invalid texture format on BCH!`.  

![The Ohana3DS error](assets/img/2022-09/ohana3ds-error.png)  

My next step was to look inside the application's files. By looking in the romfs of the app, we can see two interesting things:  

1. There is a `scripts` folder which contains **plain-text Lua files**.  
2. **(More importantly)** Inside the `sprites` folder, there are some 3DST files accompanied by their respective PNG files.  

!["scripts" folder](assets/img/2022-09/scripts-folder.png)!["sprites" folder](assets/img/2022-09/sprites-folder.png)

Now, we had a couple of 3DST sample files to test. Since it didn't seem like I was going to find more information, I decided to try and understand the format **by myself**.  

## 'Decoding' the format  

To try and decode the format, I used a *reverse-engineering* tool called `dnSpy` to de-compile the modified Ohana3DS executable. By looking at the code, it seemed that the program was looking for a header which contained the word `3DST` in hexadecimal.  
Using a hex editor, I saw that Nintendo Anime Channel's 3DST files had the word `texture` on them. Also, the program was looking for width, height and format variables at different hexadecimal spots than where they are.  
These spots are easy to locate, since the 3DST files from inside the application are square textures (which mean they have the same value), and the format was spottable by comparing the archived image from the ones that are bundled with the app.  

![3DST file in hexadeccimal](assets/img/2022-09/3dst-hexadecimal.png)

Finally, I had to set the image data offset, which was at 0x80. When I tried running the modified Ohana3DS again, it loaded the image correctly and showed me that:  

- The thumbnails are stored as **square images**, which are rescaled by the 3DS to fit the border.  
- The images are all saved **upside-down**. The application then flips the images back (tested with Citra).  

![Ohana3DS displaying a 3DST file](assets/img/2022-09/ohana3ds-success.png)

## Adding support for all formats  

All that was remaining is to implement all the image formats that the application supported. The 3DST textures from the app are only `rgba8`, but the thumbnail used a different format. By trial-and-error, I determined that it uses `etc1`.  
But that wasn't all, since `rgba8` uses hex 0x0 and `etc1` uses hex 0x4, so I tested manually every combination until the app stopped rendering the images (which meant that the format hex value wasn't implemented).  

**Fun fact**: while I was testing the different formats, I've managed to crash the application many times, which led to **panic** the system and forced me to restart the console.  

![The application error](assets/img/2022-09/anime-channel-error.jpg)

## Things to know

- The image resolutions must be a number that belongs to a power of two (it supports 256x256 or 512x256, but it will *fail* with 256x192 or 192x192).  
- For some reason, the Nintendo Anime Channel application doesn't have an image format assigned to the hex value 0x3.
- Recently, I found out that the modified Ohana3DS is available on [GitHub](https://github.com/CaptainSwag101/Ohana3DS-Rebirth) as a fork. It seems that it detects for Minecraft's 3DST files, which use a different implementation from Nintendo ones.

![Nintendo Anime Channel showing custom 3DST images](assets/img/2022-09/anime-channel-images.jpg)

Now, this discoveries are available on my GitHub and I've made PRs to [Ohana3DS-Rebirth](https://github.com/gdkchan/Ohana3DS-Rebirth/pull/78) and [Kuriimu2](https://github.com/FanTranslatorsInternational/Kuriimu2/pull/262). You can go to check them out if you want to check the source code!

This documentation is available to anyone and is licensed under the [MIT](https://opensource.org/licenses/MIT) License. Feel free to try and add 3DST images support to more programs!

Thanks for reading, and have a nice day!

