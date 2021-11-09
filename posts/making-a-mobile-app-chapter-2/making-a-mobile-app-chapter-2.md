---
title: "Making a Mobile App – Chapter 2: A Bad Worker Blames His Tools"
date: 2018-12-09T20:07:21
draft: false
featuredImage: static/postimages/6/phone.jpg
author: Luke Briggs
rssFullText: true
linkToMarkdown: true
categories: [Software Development, Making a Mobile App]
---
>“Don’t Panic.”
>
>Douglas Adams

Finding the right tool for the job might be the most important thing I’ve learned >in my short time in the world of development. The classic conversation everyone who wants to get into development goes something like this:

>New Guy: “What programming language is best.”
>
>Pro: “There isn’t one.”
>
>New Guy: “Ok. Well what programming language should I use for X.”
>
>Pro: “It depends.”
>
>New Guy: “Fine. Well what should I use if I want to use X for Y.”
>
>Pro: “Whatever you are comfortable with.”
>
>New Guy: “What do you mean comfortable. I’ve never programmed before,
>anyway its not as if a language is some sort of sofa. Hey! Where are you going!
> you can’t just leave me here!”
>
> Google: “Hello.”
>
> New Guy: “What.”
>
> Google: “I’m your new best friend.”

There are so many use cases and things you might want to develop that almost every instance of someone’s development is bespoke to what they want. This is something that you have to have experiences before you understand it which makes it more useful for a new developer to learn while doing rather than learning then doing.

I had to find the right tools for what I wanted. My requirements for this app were based off of the time constraints I had, the lack of experience I had and the needs I felt the app had:

1. Needed to be easy to port to different platforms.
2. Had to be easy to integrate design and programming elements together.
3. Needed to be able to make a professional and modern look.
   4.Didn’t require low level access

I settled on [React Native](https://facebook.github.io/react-native/) with the [Expo toolchain](https://expo.io/). I felt React was by far the best option I had. The only other options were using C# to make a Xamarin.Forms app which felt a bit like using a jackhammer to do teeth surgery or learning to use Java and Objective-C/Swift to do native development and use a completely different codebase, not only would I have to learn two languages and systems at once but also would need to fork out an initial cost of $99 for an apple developer account and develop on an Mac which I neither have nor want.

I’m using Expo because it means the app will be fully cross platform and I don’t need to interact with any of the awkward SDK’s and system specific stuff. Expo also provides a neat app that you run on your Android or iOS which connects to a local server with the app files and allows you to run your app within their app, no compiling, no apple developer account, no Mac. This made testing incredibly easy. Expo also handles keystoring for Android and all that fancy stuff.