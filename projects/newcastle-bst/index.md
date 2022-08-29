---
title: "Newcastle-bst"
date: 2021-07-10T10:33:04+01:00
draft: false
featuredImage: static/postimages/full-release-newcastle-bst/bibtex.png
author: Luke Briggs
rssFullText: true
categories: [Project Release]
description: Harvard referencing style as recommended by Newcastle University
---

> [CTAN](https://ctan.org/pkg/newcastle-bst)
> [GitHub](https://github.com/LukeBriggsDev/Newcastle-BibTeX)


This package provides a [BibTeX](https://ctan.org/pkg/BibTeX) style to format reference lists in the [Harvard at Newcastle](https://libguides.ncl.ac.uk/managing/harvard) style recommended by Newcastle University. It should be used alongside [natbib](https://ctan.org/pkg/natbib) for citations.

## Installation
The required style file is available from [GitHub](https://github.com/LukeBriggsDev/Newcastle-BibTeX) and [CTAN](https://ctan.org/pkg/newcastle-bst). You can use the style by copying it into your working directory containing your `.tex` file. You can also add it to your bst directory in your tex path to use it without having to copy it over each time.

## Using the style
To use the style include this in your preamble:
```tex
\usepackage{natbib}
\usepackage[UKenglish]{isodate}
\bibliographystyle{newcastle}
```

Also remember to specify your `.bib` file at the end of the document:
```tex
\bibliography{file}
```

The easiest way to create .bib files for this style is through exporting entries from a reference manager such as [Mendeley](https://www.mendeley.com/).
However, some parts are not available through this (such as titleaddon for computer programs).
If you notice any discrepancies between generated references and the recommended styles then please raise this on [GitHub](https://github.com/LukeBriggsDev/Newcastle-BibTeX/issues)

## License
Copyright 2021 Luke Briggs
This work consists of the documented `newcastle.bst` file.

The text files contained in this work may be distributed and/or modified under the conditions of the LATEX Project Public License (LPPL), either version 1.3c of this license or (at your option) any later version.

This work has had no input from Newcastle University and is done entirely in order to help other students create bibliography quicker.

This work is ‘maintained’ (as per LPPL maintenance status) by Luke Briggs.