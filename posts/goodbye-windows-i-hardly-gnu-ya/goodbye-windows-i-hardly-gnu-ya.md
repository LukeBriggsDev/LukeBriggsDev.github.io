---
title: "Goodbye Windows, I Hardly GNU ya"
date: 2021-02-07T21:12:30
draft: false
featuredImage: static/postimages/13/desktop.png
author: Luke Briggs
rssFullText: true
linkToMarkdown: true
categories: [Computing, Personal History]
---
**Preface: I should state that this is not an evangelical sermon, Linux isn't for everyone but it is for me. This is merely a detailing of why I have chosen to switch Operating System**

I have, over the course of the past year, consumed pretty much every interpretation and adaptation of Sherlock Holmes out there, from the original works to tangentially linked shows such as the excellent House MD; one thing I have come to understand is that you can only deduce correctly if your initial assumptions are correct. Logic is useless if its underlying axioms are unstable.

So we turn to my current assumption when I use computers "Everything must be done in Windows". I have used Windows all my life, all the software I need is on Windows, and all other operating systems are either anaemic in their UX and feature set or are incapable of running on my hardware. It just stands to reason, or does it?

Lets look at that first point, 'I have used Windows all my life'. That is not a very good reason, and the weakest of all. It is indeed true but it shouldn't have any bearing on choice of Operating System, it is an unquestioned ideology rather than any valid argument.

The second axiom, 'all the software I need is on windows', was probably true for quite a long time. For instance, if I wanted to play any windows games I would have been bang out of luck in days gone by, but tools such as [Lutris](https://lutris.net/) and the strides made in [WINE](https://www.winehq.org/) have made that side of things perfectly capable for my needs. Needless to say Mac is straying further away at this point with their move to ARM so the assumption still holds true for them. Outside of gaming, all the apps I *need* are actually available on Linux. My academic work in Computer Science lends itself perfectly to Linux. All the IDEs I use (the JetBrains suite) are available and the terminal is actually more conducive to my studies compared to the limited Unix-like aliases in the Windows Powershell. The only notable exception on Linux is the lack of the Microsoft Office Suite, but I was quite staggered when I realised that I don't actually have any need for it. My long form writing is now done in LaTeX ([see my reasons for that here](posts/the-laypersons-guide-to-latex)), and I don't use Excel or PowerPoint. Should the need ever arise, there is the native [LibreOffice](https://www.libreoffice.org/) suite, and the cross platform [Google Docs](https://docs.google.com). And if compatibility is a necessity then Microsoft also now offers a web app version of the suite.

The second point links to the third, that other operating systems have terrible UX. The king of UX is obviously MacOS, but that obviously doesn't support my hardware or the ability to run the occasional windows game. I had always perceived Linux as having terrible UX because I always just viewed through the lens of someone who has used Windows their whole life. I always just assumed (once again question your assumptions) that how a Linux distro shipped was how it was. I thought I would have to be stuck with the bland, Ubuntu default GNOME desktop and icons with their washed out colours and pre-iOS 6 style realism. I was completely wrong about this and now we are going to look at where Linux, my new Operating System, is actually better for my needs.



# The Linux way of thinking

Linux is about what Computers were built to do, and it is about the things that us Computer people love. GNU/Linux is a whole software ecosystem that is far greater than what Windows could ever hope for and only rivaled by what Apple has cultivated on its platforms. Unlike Apple, however, GNU/Linux also offers the user complete freedom in every aspect, and there are levels to this. Yes there are demigods that are compiling their own kernels but because there is such a community driven approach, other people have performed every possible layer of abstraction. Over the decades other people have been putting rungs at all levels of the ladder for other people to step on. Because Linux is built for the community by the community, it doesn't fight me.

If I wanted to change the interface on windows, I would need to install several third party apps, each using their own design language and frameworks all while eating up huge amounts of RAM as they do so. In Linux its as easy as choosing an overall desktop environment you like and customisation is welcomed with open arms as a basic human right. Surprisingly, Linux also has an incredibly consistent design language because of how its built. In Windows you have all sorts of frameworks from many decades that all conflict. You'll go from a UWP app with its tile based design ideas, and then open a WPF app that has a reasonable flat aesthetic and some okay ideas, before finally finding one of your applications uses WinForms with all of its tabs and ugly green loading bars. Almost everything on Linux using either the Gtk or Qt GUI libraries, and because of this everything looks consistent. Most Desktop Environments also let you either make or download your own themes for these graphics libraries. As a result you can make all the colours, buttons and menus in the ENTIRE Linux ecosystem look how you want.

The Linux way is for people who care about these things. If you care about things like the same apps using the same frameworks, or the fact that you can use package managers to allow apps to auto-update and avoid redundant dependencies, or how you have a whole community of people who's only incentive is to make good software.

# Technical Details

**Distribution**: [Manjaro](https://manjaro.org/)

**Desktop Environment**: [KDE Plasma](https://kde.org/plasma-desktop/)

**Theme**: Breeze Dark

**Icon Set**: [Reversal](https://github.com/yeyushengfan258/Reversal-icon-theme)

**Background**: ([Imgur](https://i.imgur.com/DUXXqM2.jpg)) ([Mirror](static/postimages/13/themagichour.png))

## Why Manjaro?

I chose Manjaro, at first by design and then retroactively I realised that I would have had to go with something like it anyway. I wanted a distribution that didn't come with the usual GNOME look, something that either looked excellent at first glance or had extensive customisation capabilities, and something that was pretty light on what it came with. I ended up narrowing it down to [Manjaro](https://manjaro.org/) and [Elementary OS](https://elementary.io/). In the end I chose Manjaro between the two because Elementary seemed slightly far behind in its update cycle compared to the Ubuntu it is based off and its never fun to start off with something knowing there is a major version incredibly close. It turned out in the end that Manjaro was really my best choice between the two anyway thanks to its easy switching between Kernels. I currently need to be running the latest experimental kernel for my touch pad to work.

