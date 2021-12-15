---
title: Image quality in rmarkdown & co.
author: edalfon
date: '2017-12-05'
categories:
  - R
tags:
  - ggplot2
  - knitr
  - rmarkdown
slug: image-quality-in-rmarkdown-co
hidden: no
comments: yes
image: 'index.en_files/figure-html/unnamed-chunk-3-1.png'
---



This is just a quick reminder on how to deal with the frustration of 
far-from-perfect `ggplot2` images in `rmarkdown` and related formats (e.g. bookdown, blogdown, xaringan).


# The issue

Motivation is this.


```r
diamonds %>%
  ggplot(aes(x = carat, fill = cut)) + 
  geom_density(color = NA) +
  facet_wrap(vars(cut), ncol = 1) +
  scale_x_log10() +
  ggthemes::theme_tufte() +
  theme(legend.position = "none") 
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-1-1.png" width="70%" />

The plot probably does not make sense at all. And I did not spend a second 
thinking about its design, color palette, etc. But this plot illustrates two
frequent issues:

* Need to fix aspect-ratio and overall, deal with the shape and size of the plot.
* Image-quality is really far from perfect. Just compare it with the same plot 
  as rendered in RStudio Viewer (I am not in the mood to show the comparison 
  here, though).

# Towards a better plot

## Useful refs

Just some links to follow, when you need to bruch up on these things

- [The canonical / authoritative source](https://r4ds.had.co.nz/graphics-for-communication.html#figure-sizing)
- [Well, the rmarkdown cookbook of course](https://bookdown.org/yihui/rmarkdown-cookbook/graphical-device.html)
- [Tips and tricks about all this. Check it out](http://zevross.com/blog/2017/06/19/tips-and-tricks-for-working-with-images-and-figures-in-r-markdown-documents/)
- **EDIT:** [Making the case to use `cairo`](https://rmflight.github.io/post/nicer-png-graphics/)
- **EDIT:** [A recent and useful post discussing these issues](https://benjaminlouis-stat.fr/en/blog/2020-05-21-astuces-ggplot-rmarkdown/)
- [And an old SO question that repeatedly pop up in my search](https://stackoverflow.com/questions/18884778/poor-resolution-in-knitr-using-rmd)
- [A frequently-seen post with details on exporting plots to meet a journal standards](https://danieljhocking.wordpress.com/2013/03/12/high-resolution-figures-in-r/)
- [Good reference on how to deal with fonts, ..., if and when you want to go out of the standard](https://www.andrewheiss.com/blog/2017/09/27/working-with-r-cairo-graphics-custom-fonts-and-ggplot/)


## Fixing aspect ratio

I just follow the authoritative advice and set defaults for `fig.width`,
`fig.asp` and `out.width`. Now in the specific chunk y modify the `fig.asp`.
Usually, the trick is just to render the plot in RStudio Viewer and adjust there
until satisfied and then right click -> Inspect and see the width and height and bring it here as `fig.asp=891/483`.

This can result in horrible things, like here.


```r
diamonds %>%
  ggplot(aes(x = carat, fill = cut)) + 
  geom_density(color = NA) +
  facet_wrap(vars(cut), ncol = 1) +
  scale_x_log10() +
  ggthemes::theme_tufte() +
  theme(legend.position = "none") 
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-2-1.png" width="70%" />

So you should also probably need to adjust the fig.width to make the text size 
as you want. And as seen above, the image quality can also be compromised, so
you probably need to fiddle with `fig.retina` and/or `dev`.

## Changing `dev`

`dev="CairoPNG"` is one alternative that can help you out, still using PNG
files. You need to have either cairo `capabilities()` or the `Cairo` package. 

Alternatively, you can use `dev="svg"` for potentially even better quality. But
it is vector-based graphics, so it could lead to very big files (e.g. scatter
plots with millions of points).

### dev="CairoPNG"


```r
diamonds %>%
  ggplot(aes(x = carat, fill = cut)) + 
  geom_density(color = NA) +
  facet_wrap(vars(cut), ncol = 1) +
  scale_x_log10() +
  ggthemes::theme_tufte() +
  theme(legend.position = "none") 
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-3-1.png" width="70%" />


### dev="svg"


```r
diamonds %>%
  ggplot(aes(x = carat, fill = cut)) + 
  geom_density(color = NA) +
  facet_wrap(vars(cut), ncol = 1) +
  scale_x_log10() +
  ggthemes::theme_tufte() +
  theme(legend.position = "none") 
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-4-1.svg" width="70%" />


## fig.retina


### `fig.retina=1`


```r
diamonds %>%
  ggplot(aes(x = carat, fill = cut)) + 
  geom_density(color = NA) +
  facet_wrap(vars(cut), ncol = 1) +
  scale_x_log10() +
  ggthemes::theme_tufte() +
  theme(legend.position = "none") 
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-5-1.png" width="70%" />

### `fig.retina=2`


```r
diamonds %>%
  ggplot(aes(x = carat, fill = cut)) + 
  geom_density(color = NA) +
  facet_wrap(vars(cut), ncol = 1) +
  scale_x_log10() +
  ggthemes::theme_tufte() +
  theme(legend.position = "none") 
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-6-1.png" width="70%" />

### `fig.retina=3`


```r
diamonds %>%
  ggplot(aes(x = carat, fill = cut)) + 
  geom_density(color = NA) +
  facet_wrap(vars(cut), ncol = 1) +
  scale_x_log10() +
  ggthemes::theme_tufte() +
  theme(legend.position = "none") 
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-7-1.png" width="70%" />

### `fig.retina=4`


```r
diamonds %>%
  ggplot(aes(x = carat, fill = cut)) + 
  geom_density(color = NA) +
  facet_wrap(vars(cut), ncol = 1) +
  scale_x_log10() +
  ggthemes::theme_tufte() +
  theme(legend.position = "none") 
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-8-1.png" width="70%" />

### `fig.retina=5`


```r
diamonds %>%
  ggplot(aes(x = carat, fill = cut)) + 
  geom_density(color = NA) +
  facet_wrap(vars(cut), ncol = 1) +
  scale_x_log10() +
  ggthemes::theme_tufte() +
  theme(legend.position = "none") 
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-9-1.png" width="70%" />

