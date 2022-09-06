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
image: ~
math: ~
license: ~
hidden: no
comments: yes
---

This is a quick post to try out [`{geomtextpath}`](https://allancameron.github.io/geomtextpath/).

I have skimmed through the package's vignettes and it seems pretty cool. 
But what did catch my eye was mainly this example below.


```r
library(geomtextpath)
```

```
## Warning: package 'geomtextpath' was built under R version 4.1.3
```

```
## Loading required package: ggplot2
```

```
## Warning: package 'ggplot2' was built under R version 4.1.3
```

```r
library(ggplot2)
ggplot(iris, aes(x = Sepal.Length, colour = Species, label = Species)) +
  geom_textdensity(size = 6, fontface = 2, hjust = 0.2, vjust = 0.3) +
  theme(legend.position = "none")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-1-1.png" width="672" />

And when I saw that, I thought that for me, the major use case for this package, would be direct labeling line plots. Perhaps it's a more convenient alternative. So let's see.

I would start with this quick-and-dirty line plot.


```r
WDI::WDI(country = c("COL", "CL")) |> 
  ggplot() +
  aes(x = as.character(year), y = NY.GDP.PCAP.KD, 
      group = country, color = country) +
  geom_line()
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-2-1.png" width="672" />

Reading the package's vignette, my thought was that one could just use `geom_textline()` and play with `vjust` and `hjust` to automagically direct 
label the lines. So let's try.


```r
WDI::WDI(country = c("COL", "CL")) |> 
  ggplot() +
  aes(x = as.character(year), y = NY.GDP.PCAP.KD, 
      group = country, color = country) +
  geom_textline(aes(label = country))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Ok, good enough. Now just use `vjust` and `hjust` to put the labels towards the end of the line and prevent the text to overplot the line (and also remove the legend for color, which would not be needed anymore).


```r
WDI::WDI(country = c("COL", "CL")) |> 
  ggplot() +
  aes(x = as.character(year), y = NY.GDP.PCAP.KD, 
      group = country, color = country) +
  guides(color = "none") +
  geom_textline(aes(label = country), vjust = -0.5, hjust = 1)
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-4-1.png" width="672" />

mmm, ok, it kinda works, but for this particular dataset, the data towards the end of the lines do not help and labels end-up being unreadable. 

So let's try at the beginning of the lines?


```r
WDI::WDI(country = c("COL", "CL")) |> 
  ggplot() +
  aes(x = as.character(year), y = NY.GDP.PCAP.KD, 
      group = country, color = country) +
  guides(color = "none") +
  geom_textline(aes(label = country), vjust = -0.5, hjust = 0)
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-5-1.png" width="672" />

Yeah, a bit better. Far from perfect, though.

But that was the idea of this post. Just to take a look at the package and see how easy and effective would be to use it to direct label line plots.


