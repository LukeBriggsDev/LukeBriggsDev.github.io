---
title: "Shiny New Things"
date: 2021-11-11T15:48:20
draft: false
featuredImage: static/postimages/shiny-new-things/mac_desktop.jpg
author: Luke Briggs
rssFullText: true
categories: [Computing, Software Development, Personal History]
description: An update on some changes I've made on both client and site. Also a dive into a custom static site generator
---

## New Hardware
The RISC revolution claims a new victim.
On October 25th I became the owner of my first Apple computer, my first non-x86 desktop, and what is already the best computer I have ever used.


### Past devices
I [moved away from Windows about 9 months ago](posts/goodbye-windows-i-hardly-gnu-ya/) and I really haven't looked back.
A few things changed in that gestation period.
Firstly, I settled on Fedora 34 as my final distro: GNOME 40 made some changes I liked and also my bluetooth audio devices worked better.
Secondly, and more importantly, my priorities shifted.
September saw the return of in-person University lecturing and some changes had to be made.


<figure>
<img src="static/postimages/shiny-new-things/legion5.jpg" alt="Lenovo Legion 5"></a>
<figcaption>&copy; <a href="https://www.trustedreviews.com/">Trusted Reviews</a></figcaption>
</figure>


My previous computer was a Lenovo Legion 5, it had a efficient (but still power hungry) Ryzen 5 4600h, and a dedicated Nvidia GPU.
The laptop was 15.6", weighed 2.4kg and would require obnoxious fans to spin up if it even thought you might do something demanding.
The laptop was a symptom of being purchased at the beginning of stay-at-home orders by someone who overestimated the amount of games they would be playing.


When it came to in-person teaching, it also was a bit cumbersome to use the machines on-site.
At the beginning of the term, the University decided move away from using local software to having Azure Virtual Desktops instead, with the PCs acting as thin clients.
Even with gigabit networking and decent hardware, virtual desktops still have a strange smell to them.
The main issue was the fact that they are still in the teething phase and IT departments are not quick to react.
Not every virtual desktop has exactly the same software, and there is no way to know which desktop you are going to get.
One day I logged onto a virtual desktop and it had all the software I needed except git.
My only solution was installing a portable version of git *into* my OneDrive so I could guarantee a git install on each machine.
Virtual desktops also made python virtual environments a hassle since every time I would do work on my personal machine, all the symlinks from the AVD would be broke and I'd have too recreate it.


**So here were my new requirement:**
* A CPU at least as powerful as my previous laptop
* 16GB RAM (8GB was too little when doing development alongside local virtual machines)
* No dGPU (the weight, battery, and noise overhead is just awful)
* Screen larger than 13", ideally 14.5 - 15"
* Portable, ideally around 1.5kg


<br />

**Like-to-haves:**
* 120Hz screen (Legion 5 had it and it made things very nice)
* 512GB hard drive (Legion 5 had 256GB and I was pushing up against it)
* A iGPU good enough to play basic 3d games at full speed


There was a selection of PC laptops that met some of my requirements. 
However, a lot of them put the higher RAM behind the models with a dGPU, which is an instant turn-off.


The incredible performance of last year's M1 chips got my head pointed towards Cupertino.
They were efficient, quiet, had a strong GPU, blazing CPU and were bundled with a first-party Unix OS.
Last year's MacBooks did not meet my requirements for the case however. A 13" screen is too small to be doing comfortable development work on.


So I sat eagerly awaiting the November Apple Event, hoping that rumors of a more powerful 14" MacBook Pro was on the horizon.


The announced computers met every single one of my criteria, the price was steeper than I had been anticipating but I had some disposable income and I had fallen slightly in love.


### The Mac Factor
I took delivery of my base-model 14-inch MacBook Pro (M1 Pro 8/14) on day one, and have used it every day in the 2 weeks since.
The true reason why I went slightly over-budget is due to the fact that these computers are the first in a while to be truly exciting to me.
A whole new instruction set platform in the desktop space is not something that has happened in my computing lifetime and it just awakens the computer nerd inside of me.
Speed *and* battery life, graphics *and* little fan noise, to mention nothing of the fact that Apple devices have the most high-quality feel of any laptop.


The software enthusiast inside of me also loves the idea of being one of the first on a new platform.
Not all the open source apps I use have been compiled for ARM yet, I could be the one to do it!
Heck, I have my own apps to port!


## New Site
If you have a look around you may also notice a slight redecoration.
This is iteration ⅠⅠⅠ of my site.


- The first was a [dynamic site served through my own flask web server](posts/inspection-and-dissection-this-site),
- The second was a [static site generated through Hugo](/posts/where-hugo-i-go/),
- This iteration is a [static site generator of my own devising](https://github.com/LukeBriggsDev/VictorSSG)


I made the change because my last site looked pretty cluttered (and I felt that my design skills have improved since my first attempt).
I also wanted a nicer way to list my projects.


The process of developing a static site generator in python was much more straightforward than I thought.
It only took around 4 days to get to a point where everything looked nice on the outside.
The codebase needs some cleanup and it isn't entirely ready for anyone other than me to use it but it works.


The program on initialisation creates a few directories to place stuff (`content`, `projects`, `static`, etc) as well as a config file for certain things like the navigation bar.
The built website gets put into a `public` directory.
The building process itself starts by copying the static directory tree into the public directory.
A python library called [mistune](https://pypi.org/project/mistune/) converts the markdown files in `content` (along with YAML metadata headers) to HTML, [pygments](https://pypi.org/project/Pygments/) is also used for syntax highlighting. 
The converted html posts are then passed to myy custom [jinja2](https://pypi.org/project/Jinja2/) templates that have all the styling and base markup.
There are a couple of other things like auto-generated social links on the homepage and a so-basic-its-almost-broken built-in HTTP server for viewing webpages locally.


I will probably tweak a few things over time but I think I've settled on the principle design, at least for a while.


The SSG is named Victor as a tongue-in-cheek nod to Hugo, I also quite like that it is also the name of Mr. Freeze which kind of makes sense for a static site generator.


[The source code is available on GitHub](https://github.com/LukeBriggsDev/VictorSSG)