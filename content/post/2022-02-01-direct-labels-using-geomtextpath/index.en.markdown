---
title: Direct labels using {geomtextpath}
author: edalfon
date: '2022-02-01'
slug: direct-labels-using-geomtextpath
categories:
  - R
tags:
  - geomtextpath
  - ggplot2
description: ~
image: https://i.imgur.com/5N3YjNz.png
math: ~
license: ~
hidden: no
comments: yes
---




This is a quick post to try out [`{geomtextpath}`](https://allancameron.github.io/geomtextpath/).

I have skimmed through the package's vignettes and it seems pretty cool. 
But what did catch my eye was mainly this example below.

<img src="https://i.imgur.com/7C2iHVS.png" width="480" />

And when I saw that, I thought that for me, the major use case for this package, would be direct labeling line plots. Perhaps it's a more convenient alternative. So let's see.

I would start with this quick-and-dirty line plot.

<img src="https://i.imgur.com/4CoXSEN.png" width="480" />

Reading the package's vignette, my thought was that one could just use `geom_textline()` and play with `vjust` and `hjust` to automagically direct 
label the lines. So let's try.

<img src="https://i.imgur.com/hYndZv5.png" width="480" />

Ok, good enough. Now just use `vjust` and `hjust` to put the labels towards the end of the line and prevent the text to overplot the line (and also remove the legend for color, which would not be needed anymore).

<img src="https://i.imgur.com/q9R23xZ.png" width="480" />

mmm, ok, it kinda works, but for this particular dataset, the data towards the end of the lines do not help and labels end-up being unreadable. 

So let's try at the beginning of the lines?

<img src="https://i.imgur.com/gXeTiYX.png" width="480" />

Yeah, a bit better. Far from perfect, though.

But that was the idea of this post. Just to take a look at the package and see how easy and effective would be to use it to direct label line plots.


