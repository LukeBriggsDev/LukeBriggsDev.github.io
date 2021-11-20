---
title: "The Layperson’s Guide to LaTeX"
date: 2021-01-06T22:06:22
draft: false
featuredImage: static/postimages/11/rainbow.jpg
author: Luke Briggs
rssFullText: true
linkToMarkdown: true
categories: [Computing, Tutorial]
description: "Pretty prinitng"
---
As part of my degree I will have to write many words into many documents and submit them all as part of assignments.

There was a time when we were young, stupid and thought that the peak of document formatting was WordArt, a drop shadow, and rainbow II. We would stick borders on our .pub files and feel superior if our PowerPoints had a dissolve transition. As we age past primary school we begin to conform to the world’s sensibilities that Comic Sans is not an adequate typeface and having every colour on the spectrum is a way to actually guarantee some of your text will be unreadable.

Since we are forced to conform, we might as well do it to perfection, and you can only go so far in a tool that still thinks that word processors need a background ‘water droplet’ texture fill.

# What is this LaTeX and Why is it Not Pronounced Like That?

If you want a tool that focuses on the nicest looking documents without caving into the design requirements of 7 year olds, you have to go back to a time before texture fills. Actually you have to go back to a time before computers could even show pictures. Nice looking documents (i.e had legible fonts) had to be obtained using specialised typesetters that cost as much as a house, either that or you’d use a literal printing press – this was the time in which TeX was created. LaTeX is built on TeX and is its more approachable step-son. LaTeX uses a mark-up language to design its document with the idea being that you focus on the content while the engine works out the best formatting.

# Isn’t This a Bit of a Faff?

It depends on your personality. If you are happy with documents that are only adequate, then continue in your mediocrity. But for those of us who seek perfection and see neuroplasticity as a fundamental attribute, LaTeX offers a sterling reward for your efforts.

# How Good Does it Look?

In the days when people weren’t made of pixels I spent the time making my own word template. It had wonderful serif headings; all the styles used Word’s tools so it did as much of the heavy lifting as possible; and all the font sizes were made just right.

The results from Word are as follows:
![full word document](static/postimages/11/full.png)

It looks *okay*. It looks far better than what some people create in Word. Even creating something okay looking feels like a hack though. Having nice paragraph spacing underneath headings required my to individually change all the line spacing options by hand. Anyone who has ever tried to implement code into a word document also knows that it will require you to sacrifice your firstborn.

Now, for a LaTeX document:
![full latex](static/postimages/11/fulllatex.png)

No time spent messing with templates, no changing font sizes, and it probably took me less time to make a document that looks even better. The best thing about LaTeX is that because everything is done programmatically, it can have an integration that is mind boggling. For instance, if you want to insert a segment of a python script in your document you dont actually need to copy and paste bits into your .tex file. You can just tell it where the script is, give it the line numbers and it will display and format it all for you.

# Where Do I Begin?

This is for those of a Windows disposition

First install a distribution of LaTeX called [MiKTeX](https://miktex.org/download). MiKTeX has everything you need and will make the whole experience as easy as possible.

Our code editor is called TeXworks, so open that up and lets write some LaTeX.

LaTeX follows a syntax of \command[option]{parameter} and we start a document off by selecting the type of document we want to create, and then starting and ending said document. There are a number of [document classes](https://en.wikibooks.org/wiki/LaTeX/Document_Structure#Document_classes) but article is the one recommended for most documents.

```latex
% Start of Document (Comments are denoted by a percentage at the start of the line)
\documentclass{article}
\begin{document}
I'm so pretty
\end{document}
```
And we get:
![empty latex](static/postimages/11/empty.png)

Currently it looks like a note left by a serial killer with access to a typewriter so lets stick our name on it.

The title, date, and author of your document all have dedicated tags in LaTeX. They are \title \date and \author. We then tell LaTeX to show all these on the screen by using \maketitle

```latex
% Start of Document (Comments are denoted by a percentage at the start of the line)
\documentclass{article}
 
\begin{document}
 
\title{My Pretty Document}
\author{J. Smith}
\date{23 November 1963}
\maketitle
 
\end{document}
```
Output:
![title page](static/postimages/11/empty.png)

You can hear groups of your preferred gender(s) running to throw themselves at you as we speak.

Now for some content. Once again there are various different types of [sectioning](https://en.wikibooks.org/wiki/LaTeX/Document_Structure#Sectioning_commands) you can use but we will use the pretty straight forward \section

```latex
% Start of Document (Comments are denoted by a percentage at the start of the line)
\documentclass{article}
\begin{document}
 
\title{My Pretty Document}
\author{J. Smith}
\date{23 November 1963}
 
\maketitle
 
\section{This is a subtitle}
I can just write text under the section tag and it will all be made nice
 
\section{This is my second subtitle}
I just add another section tag and it will become the second section in the document. Pretty cool!
 
\end{document}
```

Output:

![content](static/postimages/11/content.png)

Positive gasps emanate from the crowd at the sight of such serifs. You will notice that LaTex automatically added heading numbers and changed the weight, face, and size of the type to better suit a heading. [There are all sorts of things you can do](https://en.wikibooks.org/wiki/LaTeX) that I haven’t covered like how it can do bibliographies for you and organise images and figures correctly. There are also things that are beyond my knowledge like drawing your own 3d graphics within source. All of it can be taken one step at a time. If you don’t know something, look it up, learn how to do it, and implement it – it’s all pretty approachable.

LaTeX isn’t perfect in its implementation, only its possible output. For instance, the answer to the question ‘how do I insert an svg’ is ‘use a pdf instead’ but I hope I have given you a taste of a better alternative to Word, even if it does introduce the possibility that your reports can have syntax errors.

# Further Notes

If you are writing your reports in LaTeX like me then you will need the correct bibliography formatting. LaTeX offers a few styles by default but I have learned there is no such thing as a standard and my University (Newcastle) uses [its own guidelines](https://libguides.ncl.ac.uk/managing/harvard). The process for creating your own style involves both a command line tool and further tweaking using a bespoke programming language using REVERSE POLISH NOTATION, which is what happens if Christopher Nolan got his hands on a compiler. To save my fellow students time, I created a style for Newcastle’s bibliography style and [it is hosted on GitHub](https://github.com/LukeBriggsDev/Newcastle-BibTeX).

