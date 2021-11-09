---
title: "Making a Mobile App – Chapter 4: Fast and Flash"
date: 2018-12-10T21:57:21
draft: false
featuredImage: static/postimages/8/desk.jpg
author: Luke Briggs
rssFullText: true
linkToMarkdown: true
categories: [Software Development, Making a Mobile App]
---
> “A common mistake that people make when trying to design something
> completely foolproof is to underestimate the ingenuity of complete fools.”
>
> Douglas Adams

As this app will be for a business, it has to serve some purposes. First it has to present the company as a professional entity and also fit with its branding to provide not only a consistent experience but to also strengthen that companies place in your head, making them more likely to go to them rather than anyone else. Next it has to serve a point, it’s all well and good having some fancy spiel about company ethos and dedication to the craft but you want the user to have a reason to not only to install your app but to also keep using it. Finally, it has to be straightforward to use and foolproof. Well, nothing is foolproof proof. Well, nothing is foolproof.

#### Branding
The company colours were red, dark grey and white. It therefore makes sense for the app to only use the colours red dark grey and white, seems simple enough.

![red app](static/postimages/8/redapp.png)

Eww, a red pastel background with black text and mismatch white buttons and icons. It looks like a tomato soup can just exploded in the microwave. It’s all about complimentary colours and keeping things clean. The main colour of any app or website shouldn’t be something outrageous or garish, you want to present professionalism and minimalism. It’s good to have an accent colour that pops and has your brand stand out but if you end up with too much it is both nauseating and ugly.

![black app](static/postimages/8/blackapp.png)

Ahh Much better. Character is nice but flamboyance is very easily annoying

#### Purpose
Like I say, your app needs a purpose otherwise what’s the point?

![menu](static/postimages/8/menu.png)
![times](static/postimages/8/times.png)

As you can see the app act doesn’t revolutionise e-commerce but what it does do is act as a hub for everything involved with this company. You can click a button and google maps opens up with the locations for shops and restaurants already plugged in and ready to plan a route. You can call to book a table for a restaurant and there are galleries of menus and images to see what products and dishes are on offer (As a side note if you are making a ReactJS app in Expo then [react-native-image-view](https://www.npmjs.com/package/react-native-image-view) is a godsend, it is a bit fiddly to get working but it means you can keep using Expo without having to delve into Android files and java SDKs).

#### Foolproofness and Expandability
To make your app accessible you have to ensure it is something easy to navigate and intuitive to use. Since this is my first mobile app I felt it was more of a case of making sure it doesn’t look out of place compared to the modern selection of mobile apps that supermarkets and resteraunts have on offer. It uses clear icons and a navigation bar at the bottom which is the current style. I wanted the user to not be confused upon opening the app but feel comfortable in using it and having a smooth experience.

One of the brilliant things about React Native is the modules are not only easy to use but also offer a great deal of wiggle room to do what you want. For example, creating new screens is as easy as adding a function to that screens javascript add sticking a call to it in a main navigator script; this allows the room to expand quite quickly once you’ve got a baseline set giving scalability without what some projects have which is a feeling of trying to balance on one leg while trying to solve a rubik’s cube.
