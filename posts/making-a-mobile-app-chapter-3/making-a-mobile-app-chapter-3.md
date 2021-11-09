---
title: "Making a Mobile App – Chapter 3: The Setup"
date: 2018-12-10T20:14:17
draft: false
featuredImage: static/postimages/7/drawingboard.jpg
author: Luke Briggs
rssFullText: true
linkToMarkdown: true
categories: [Software Development, Making a Mobile App]
---

>“He attacked everything in life with a mix of extraordinary genius and naive >incompetence, and it was often difficult to tell which was which.”
>
>Douglas Adams

Setting up your React/Expo app is fairly straightforward but does require using the command line and some basic knowledge. You can find further detail [here](https://expo.io/learn) but I’ll summarise it now.

Firstly you will need to download and install [Node.js](https://nodejs.org/en/) This is quite self explanatory as it has an actual wizard thus not requiring any command line interaction. Node is necessary for almost anything running local javascript. Next you’ll need to open the command line (found on windows by typing ‘cmd’ into the start menu) and run this command:

```console
npm install expo-cli --global
```

You will then be asked to create an Expo account. Next you want no navigate to the directory where you want to store your app, this is done in windows by typing ‘cd’ followed by the directory. For example :

```console
cd C:\Users\<Your Username>\Documents\app
```

Then run the commands:

```console
expo init my-new-project
cd my-new-project
expo start
```

‘expo start’ will be the command you run in the directory whenever you want to run your Expo app. I would also recommend starting with the ‘tabs’ template as it places some very useful files and means you don’t need to set up your own navigation bars. After you have ran the start command your browser will open up with a QR code. Scan the QR code in the Expo app ([Google Play](https://play.google.com/store/apps/details?id=host.exp.exponent&hl=en_GB) and [iOS](https://itunes.apple.com/us/app/expo-client/id982107779?mt=8)) and behold your first expo app. If you have experience in ReactJS, you can dive into the files with your text editor of choice (I prefer [VS Code](https://code.visualstudio.com/)) and follow the instructions both in the app and consulting the [documentation](https://docs.expo.io/versions/v31.0.0/workflow/up-and-running).  If you don’t have experience, I would recommend using the [Codeacademy](https://www.codecademy.com/learn) course on ReactJS and Javascript. Also remember Google is your best friend, if you don’t know something don’t be afraid to google it.