---
title: "Inspection & Dissection: This Site!"
date: 2021-02-03T16:39:25
draft: false
featuredImage: static/postimages/12/index.png
author: Luke Briggs
rssFullText: true
linkToMarkdown: true
categories: [Project Release]
---
Look at me with a fancy website. We're about to get meta as we discuss how this site you're exploring right now came to be.

# Why have a website?

I want something to point to when someone asks 'so what have you done?'. I also need it for when I ask myself the same thing. Sometimes I wonder how much I have actually done in my spare time, and a site like this helps me to remember that I haven't wasted *all* of my spare time on Minecraft.

# Why make your own website?

Here in Computer Science land: men are real men, women are real women, and small furry creatures from Alpha Centauri are real small furry creatures from Alpha Centauri; thanks to these huge revelations in the field of existentialism we take it upon ourselves to walk the road already taken.

The real reason is I ought to have side-projects, more than I used to. I have more time, not just because of COVID but also because Computer Science is now the only academic subject I have to pay attention to, whereas most of my time in Sixth Form was spent on Maths and Further Maths. It made reasonable sense for the first project to be a platform for my other projects and this is that platform.

# Where did you start?

I knew I wanted to make a website with a Flask back end for 2 reasons:
1. I wanted a back-end that could enable things like logging in and writing blog posts all within the site.

2. I vehemently hate JavaScript and would prefer to do as much with python as possible. In fact this whole site doesn't have any JavaScript. You can inspect element and see that everything front end is done through CSS and HTML; this is certain to change but I will hold out for as long as I can.

For the uninitiated, websites have 2 parts: A front end and a back end. In a simple form, the front end is everything you see (HTML, CSS, JS) and the backend is everything you don't (Databases, Form requests, etc). Flask is a python framework that allows the said language to be used to serve all your requests.

I had used Flask in the past whilst following the [Harvard CS50x](https://cs50.harvard.edu/x/2021/) course back in 2019 to make a fake stocks app (💎👐). This time round, to re-familiarise myself I followed [this excellent tutorial](https://flask.palletsprojects.com/en/1.1.x/tutorial/).

After you are let loose on your own (especially in web dev) I found you have to have goals and a purpose to what you are doing otherwise it'll never happened. I persevered through the trial-and-error process that is CSS

I used a couple of sites for reference when I was making the design for the sight. These were the sights of: my new friend [George Wood](https://gwood.dev), the blog of [Casey Liss](https://www.caseyliss.com/), the blog of [Marco Arment](https://marco.org), and [Hypercritical](https://hypercritical.co) (No prizes for guessing which podcast I listen to).

# What extra things have you done?

I'm honoured at your presumption that I go above and beyond in the call of duty. The main thing I have done is make all my posts in markdown. Yes, all my posts are entirely written in markdown which makes me able to write nicely structured documents quickly. The raw text of the markdown is stored as a regular text field in the sqlite database but at the stage when the backend grabs the field from the database, jinja parses it as html using the python module [mistune](https://mistune.readthedocs.io/en/latest/intro.html). To spice the formatting up even further, since I will be probably be using code snippets, I added the syntax highlighter [pygments](https://pygments.org/). In fact here is the code for the highlight renderer being rendered in it:

```python
import mistune
from pygments import highlight
from pygments.lexers import get_lexer_by_name
from pygments.formatters import html


class HighlightRenderer(mistune.Renderer):
    def block_code(self, code, lang=None):
        if lang:
            lexer = get_lexer_by_name(lang, stripall=True)
            formatter = html.HtmlFormatter(style='monokai', noclasses=True)
            return highlight(code, lexer, formatter)
        return '<pre><code>' + mistune.escape(code) + '</code></pre>'

```

It goes without saying that this website will continue to be updated as I change my mind on a few things and wish to add more functionality. On my list of todos are:

* A switch between light mode and dark mode
* A way for people to follow the blog (maybe through RSS)
* A way to better list out blog posts (currently all are listed on one page which will probably become cumbersome when they grow in number)

In the meantime I hope you come back and see if I can offer you any informative content.
