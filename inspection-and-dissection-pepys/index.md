# Inspection and Dissection: Pepys - A Straightforward Markdown Editor


> The product page is available [here](/pepys)

# Brief Description

Pepys is a GUI journaling application built using Python bindings for the Qt framework.
It is available as an installer for Windows and a Flatpak on Linux.

# Background

I started journaling around February last year, it was probably the most prescient thing I have ever done.
It wasn't long before we all went into self-imposed isolation and journaling provided a good way to express my thoughts.

My first posts were made in one big Word document.
Journals need to be as future-proof as possible, and I thought at the time that since entire governments are probably reliant on MS Word that it won't be going away any time soon.
It wasn't a good experience though. 
Having to type the date each time was tiresome, and I thought it would be a formatting mess should I use an Operating System without MS Word (something which would later happen when I spent several months using Linux).

I next moved on to [Notion](https://notion.so).
An excellent productivity application that can be used to do all sorts from notes to spreadsheets to databases.
It's template, tagging and calendar features made it a great prospect for writing my journal in. 
The entries were made in Markdown so would be viewable on any computer for as long as we still used ASCII and Unicode, seems pretty future proof.
The trouble was, it made journal writing a chore.
To write an entry I'd have to open up this electron application that took an *age* to open and slowly click through web links, waiting for pages to load, before I could get to writing entries.
I wanted a native application that would allow me to write journal entries in markdown, with a date-based file chooser.
I found no application that fitted all this criteria that worked on Linux and Windows, they all seemed Mac only (I am envious of the great ecosystem Apple cultivates among its development community).
So I took it upon myself to write my own.

# A Rocky Start

The first idea for a Journal application first came to me in June 2020 but it would take nearly a year before I would find the time and the courage to actually start it.

My only experience with a GUI application that wasn't made in a dedicated game engine was [Dice-Jack](/inspection-and-dissection-dice-jack) which was a version of blackjack that used Dice instead of cards and was made over a few days for an assignment as part of my Sixth Form Computing course.
I knew I couldn't use dotNet since I wanted it to be able to run on Linux as well as Windows.
My next thought was Java, since that was the language I was learning at University at the time and seemed to fit all my requirements.

I managed to get a very early text editor working in Java.
At that point, the idea was to have all the styling done in real time as you typed.
For example, if I typed a `#` then I would set the font size you type in to that of a title.

I was running into issues that were perfectly fixable but I was never truly comfortable in Java.
I will absolutely use Java in the future but it just didn't seem right with this project. 
With Java I had the choice between the ancient swing and JavaFX.
I have never got to grasps with the whole split GUI framework where some bits are in code and some bits are in XML files.
All the GUI frameworks are going this way so I will have to get my head around it eventually but this was not the project for this.

# A Walk in the Dark

After bouncing of Java frameworks I had to go with what I know, Python.
Python was the first language I learned and I've been using it for coming up 4 years now.
The big players in the Python GUI game is Gtk, Qt, wxWidgets, and Tkinter. 
Tkinter and wxWidgets are the more simple of the four with the majority of applications being built in either Gtk or Qt.
In the end I settled with Qt because it seemed to have better look and feel across different systems, and Gtk seemed to want me to use build systems that I was unfamiliar with.
I first broke ground with the first [git commit](https://github.com/LukeBriggsDev/Pepys/commit/66f42549e53db7a43224d317be2191b2000e0d94) being pushed at 20:29 BST on April 1st.
This was the first commit to the repo that would become the final release, this came at the end of several weeks of the aforementioned bouncing around.

## Syntax Highlighting
<figure>
<img alt="An early version of the edit pane" src="/static/postimages/inspection-and-dissection-pepys/pepys-april7-2021.png">
<figcaption>An early version of the edit pane</figcaption>
</figure>
My primary focus was in getting markdown source highlighting in the text window.
Things like bold, italic, and strike-through came rather quickly but became incredibly slow with large amounts of text.
My initial solution to the syntax highlighting was a naive one.
I would be performing regex searches and applying text character formatting across the document on each key press.
The approach was okay on small documents, but the larger the document the more text would be searched through and it would become impossible to type.
The first breakthrough came when I discovered a Qt widget called QSyntaxHighlighter which provided methods that would allow text to be broken up into 'blocks'.
I put all my regular expressions into a QSyntaxHighlighter and used its blocking mechanism to allow for typing to not be slowed down on large documents.

## HTML Preview
<figure>
<img alt="An early version of the view pane - note the switch from menu bar to tool bar" src="/static/postimages/inspection-and-dissection-pepys/pepys-april9-2021.png">
<figcaption>An early version of the view pane</figcaption>
</figure>
Alot of the early work went into refining my markdown regular expressions, making sure they formatted the correct parts and matched the output reasonably.
It wasn't long before I had to turn my attention to the HTML preview.
The preview pane was initially just a QTextBrowser that supported a limited subset of HTML.
When the preview button was clicked the markdown would be converted to html using [mistune](https://mistune.readthedocs.io/en/latest/).
I had used mistune previously on the [first iteration of this website](/inspection-and-dissection-this-site) so a lot of that could be copied and pasted from my work there.

The fact that only a subset of HTML was supported soon became an issue and I had to switch the preview to a full on web engine (the reason why the application is so large).
The web engine was such a double edged sword.
Without it: previews wouldn't match exports, math equations wouldn't be supported, many markdown elements couldn't be rendered properly (tables, inline html).
But with it, the application swells in size to over 100MB download and even larger when fully installed.
I wish I could resolve this alas I am too tired and too unskilled to be able to implement a solution.

## The Killer Feature

It was around this time that I went into exploring [pandoc](https://pandoc.org/).
I was amazed by its power and I will undoubtedly utilise it in many future projects.
Pandoc is a program written in haskell that uses its own flavour of markdown to provide a *huge* amount of outputs for document conversion.
I knew I had to implement this in some way to provide some way of providing a wide range of export option.
Then I will have truly realised my need of having entries been in a future proof format, why have 1 future proof format when you can have every format under the Sun?

## Calendar File Selector

I knew that my main method of interacting with files should be through a calendar.
Qt provides a reasonably good calendar widget that saved alot of time.
The way I keep track of the files in a user's journal is by having a very rigid folder structure which Pepys gets pointed to.
The journal directory has a structure of `YYYY\MM\DD\YYYY-MM-DD\YYYY-MM-DD.md` This makes it very easy to search for all the entries by date.

## Release

<figure>
<table>
<tr>
<th>
<img alt="Very Old logo" src="/static/postimages/inspection-and-dissection-pepys/pepysoldlock.png" width="256px">
</th>
<th>
<img alt="Old logo" src="/static/postimages/inspection-and-dissection-pepys/pepysold.png" width="256px">
</th>
<th>
<img alt="New logo" src="/static/postimages/inspection-and-dissection-pepys/pepysnew.png" width="256px">
</th>
</tr>
</table>
<figcaption>A comparison of different logos I went through</figcaption>
</figure>

I have went into my woes over releasing software [here](/flatpak-instructions-not-included) but it is safe to say that this is an are where I learned an awful lot.
In the end Pepys was released on Linux via flatpak and on Windows via an NSIS installer [available to download on GitHub](https://github.com/LukeBriggsDev/Pepys/releases/).
There was no Mac release because I do not own a Mac no have sufficient experience with Macs to be comfortable with creating installers and sufficiently testing them for correctness.

# A Retrospective

This whole process has been the longest I have ever spent on a project, it was probably the largest project I have ever made, and there were alot of firsts.
It was the first application I made that wasn't a game, it was the first proper GUI application I have made, and it was the first time I had to make installers by hand.
The process was arduous at times and I felt like giving up at some points. 

I sit here now after releasing it and I think "if I'd only done this" or "it would be so much better if..." but I can't think like that.
If I had spent as long as possible on each feature till it was perfect then the software would never be released.
I have learned that software development is an iterative process in 2 ways.
The first way is in relation to an individual piece of software.
You develop the software over time, slowly improving and adding until a finished product stumbles over the finish line.
The second way is at a macroscopic level.
Each huge project your begin, each journey you embark on, brings a new iteration on what went before.
The next GUI project I make *will* be better.
The next installer I make *will* be better.
And this excites me.

I suppose it has taken me until the end of my first year of University to understand what learning is really about.
Learning isn't about passing tests, although schools may tell you otherwise. 
Learning isn't even about knowing stuff, that is just a by-product.
Learning is about being better, about knowing you *will* be better, and about being excited about that.
I am excited about what I will create next, how I will iterate on what I have learned, and how I can surprise myself.

# Special Thanks
All of this took an awful lot of effort so I would like to give special thanks to the following people and projects for there valuable insights.

[Apostrophe](https://gitlab.gnome.org/World/apostrophe)
: An excellent markdown editor that shows how to make a properly good writing experience and led me down the Pandoc path. Also inspired some of the regex used.

[Humboldt University Optical Metrology Group](https://github.com/hermitdemschoenenleben/linien)
: Without whom I'd still be wrestling with compilers in Flatpak

[Lucy Marsten](https://akaru.me/) ([Github](https://github.com/noneuclideanmotion))
: Who provided a second set of eyes and motivated me to actually get stuff done.

[Jan Grulich](https://github.com/FedoraQt/adwaita-qt)
: For showing how to implement Adwaita colours in Qt

