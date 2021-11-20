---
title: "Inspection and Dissection: Calmer Internet - My First Web Extension"
date: 2021-08-18T15:18:16+01:00
draft: false
featuredImage: static/calmer-internet/banner.svg
author: Luke Briggs
rssFullText: true
linkToMarkdown: true
categories: [Project Release, Software Development]
description: A web extension that hides the elements of web pages that are best off hidden
---

> The product page is available [here](projects/calmer-internet)

# Brief Description
*Calmer Internet* is a web extension for Chrome, Firefox, and Edge that makes the internet a less infuriating place to be.
It removes comments, recommended content and various trending pages from Twitter, YouTube, and Instagram to try and relieve that doom-scrolling.

# Rationale
Since I finished stage 1 of University in June, I had a considerable amount of time off.
A lot of that time was spent in a state of not quite working and not quite relaxing.
I was stuck in a cycle of following the YouTube sidebar of bland content before going to Twitter and scrolling down the trending hashtags and comments, getting annoyed at every step.

Many social media platforms just throw various things at you in an attempt to just keep scrolling and clicking and engaging, before you know it you've spent an hour of your life on a soul-sucking site.

I didn't want to quit these sites entirely. 
There are truly incredible and insightful people on there like [Tom Scott](https://www.youtube.com/user/enyay), [exurb1a](https://www.youtube.com/c/Exurb1a), and [Ahoy](https://www.youtube.com/channel/UCE1jXbVAGJQEORz9nZqb5bQ).
Twitter is also good as a micro-blog feed for people I want to follow in the tech sector. 
The conclusion I came to is that I need to take control of what I see. 
I should only be consuming things I have actively sought out, because those are the things that I enjoy the most.
It also means that I can't just keep scrolling and watching forever, I will run out of content because I only see what I have subscribed to.
There is no 'one more click'

My initial solution was to have everything go to an RSS reader but most social media platforms don't play nicely with rss feeds and feed readers don't really play nicely with conveying threads, retweets and video content.
As a result, I turned to editing the sites themselves to provide a more focused experience.

# How it's made

This extension didn't start out as an extension. It started out as a set of rules for my ad-blocker. 
I'd use the eyedropper tool to select the *Trending* bar on Twitter just so I wouldn't be tempted to click it.
I did similar things with the trending tab on YouTube.
It then got to a point where I wanted to do more like remove the recommended sidebar and the video-wall end screens, so I decided I needed a web extension.

I have never made a web extension before, but I do have some experience in HTML and JavaScript so it wasn't too troublesome to get used to.
Most of what my extension does takes place in a 'content script'.
This is a javascript file that just loaded alongside the current page at load time.
The file then looks at the current URL and checks the page for any elements I've told it to remove.

```javascript
    // Adding Twitter elements to a list of elements to remove
    if (window.location.hostname === "twitter.com" || window.location.hostname === "mobile.twitter.com"){
        if(!settings["TwitterTrendingBar"]){
            elementsToRemove["trendingBar"] = document.querySelectorAll("div[aria-label='Timeline: Trending now']")
        }
        if(!settings["TwitterExploreLinks"]){
            elementsToRemove["exploreLinks"] = document.querySelectorAll("a[href='/explore']")
        }
        if(!settings["TwitterWhoToFollow"]){
            elementsToRemove["miscStyling"] = document.getElementsByClassName("css-1dbjc4n r-1867qdf r-1phboty r-rs99b7 r-1ifxtd0 r-1bro5k0 r-1udh08x")
            elementsToRemove["whoToFollow"] = document.querySelectorAll("aside[aria-label='Who to follow']")
        }
        if(!settings["TwitterTopics"]){
            elementsToRemove["topics"] = document.querySelectorAll("div[aria-label='Timeline: ']")
        }
    }
```

The script then attaches a listener that causes new checks every time the webpage is updated

```javascript
    function observeBodyRemove(){
        // Select the node that will be observed for mutations
        var targetNode = document.querySelector('body');

        // Options for the observer (which mutations to observe)
        const config = { attributes: true, childList: true, subtree: true };

        // Callback function to execute when mutations are observed
        const callback = function(mutationsList, observer) {
            elementsToRemove = getElementsToRemove();
            for(key in elementsToRemove){
                for (element of elementsToRemove[key]){
                    try {
                        element.remove();
                    }
                    catch(e){
                    }
                }
            }
        };

        // Create an observer instance linked to the callback function
        const observer = new MutationObserver(callback);

        // Start observing the target node for configured mutations
        observer.observe(targetNode, config);
    }
```

The only other JavaScript in the extension is in a 'background script'.
Whereas 'content scripts' are loaded alongside a webpage, background scripts are persistent and always run whenever the extension is loaded.
This means they are useful for doing things before a page is loaded. 
One feature of the extension is you can redirect any links to YouTube.com to youtube.com/feed/subscriptions, so you aren't bombarded by useless recommendations.
I initially had the redirect in a content script but that required the whole page to be loaded before redirecting away from it.

My current solution uses the WebRequest API to intercept a page request and load a new one instead.
This is also done be setting up a listener, such as this one:
```javascript
// YouTube
    chrome.webRequest.onBeforeRequest.addListener(
        function (details) {
            if(!settings["YTHomeRedirect"]) {
                return {redirectUrl: "https://youtube.com/feed/subscriptions"};
            }
        },
        {
            urls: [
                "*://*.youtube.com/"
            ],
            types: ["main_frame", "sub_frame", "stylesheet", "script", "image", "object", "xmlhttprequest", "other"]
        },
        ["blocking"]
    );
```

Finally, all that was left was to create a manifest.
I have great experience with manifests since [Pepys recently received builds for rpm, .deb, and PKGBUILD](https://github.com/LukeBriggsDev/Pepys).

```json
{
  "manifest_version": 2,
  "name": "Calmer Internet",
  "version": "1.1.0",
  "browser_specific_settings": {
    "gecko": {
      "id": "calmerinternet@lukebriggs.dev"
    }
  },
  "browser_action": {
    "default_icon": {
      "16": "icons/16.png",
      "24": "icons/24.png",
      "32": "icons/32.png"
    },
    "default_title": "Calmer Internet"
  },
  "icons": {
    "48": "icons/48.png",
    "96": "icons/96.png"
  },
  "content_scripts": [
  {
    "matches": [
      "*://*.youtube.com/*",
      "*://*.twitter.com/*", "*://twitter.com/*",
      "*://*.instagram.com/*"
    ],
    "js": ["calmer.js"]
  }
  ],
  "background": {
    "scripts":["background.js"]
  },
  "permissions": [
    "*://*.youtube.com/*",
    "*://*.twitter.com/*", "*://twitter.com/*",
    "*://*.instagram.com/*",
    "storage",
    "webRequest",
    "webRequestBlocking"
  ],
  "description": "Provides a calmer internet experience. Removes comments and various recommendation elements from Twitter, YouTube, Instagram.",
  "options_ui": {
    "page": "options.html"
  }
}
```

# Deployment

The extension is available on the Chrome Web Store, the Firefox Add-on Store, and the Microsoft Store.
WebExtensions are now pretty cross-compatible, so I use the exact same code-base across all browsers.
It would even support Safari as of Big Sur, but I do not have a Mac to upload it, nor do I want to fork out the $99 price of an Apple Developer account.

As ever, the project is also available in full on my [GitHub](https://github.com/LukeBriggsDev/calmer-internet)
