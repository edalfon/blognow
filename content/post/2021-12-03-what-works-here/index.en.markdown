---
title: What works here?
author: E!
date: '2021-12-03'
slug: what-works-here
categories:
tags:
  - blogdown
description: still need to bring older posts here!
image: https://i.imgur.com/lsjJif9h.png
math: yes
license: ~
hidden: no
comments: yes
---

<script src="{{< blogdown/postref >}}index.en_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/plotly-binding/plotly.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/typedarray/typedarray.min.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/jquery/jquery.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/crosstalk/css/crosstalk.min.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/crosstalk/js/crosstalk.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/plotly-htmlwidgets-css/plotly-htmlwidgets.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/plotly-main/plotly-latest.min.js"></script>

# Details - Summary

<details>
<summary>
Click Here!
</summary>
<p>
And hidden details will show
</p>
</details>

They can also contain code

<details>
<summary>
Click Here!
</summary>
<p>

``` r
toydf <- data.frame(foo = 1:10, bar = 20:29)
base::summary(toydf)
```

    ##       foo             bar       
    ##  Min.   : 1.00   Min.   :20.00  
    ##  1st Qu.: 3.25   1st Qu.:22.25  
    ##  Median : 5.50   Median :24.50  
    ##  Mean   : 5.50   Mean   :24.50  
    ##  3rd Qu.: 7.75   3rd Qu.:26.75  
    ##  Max.   :10.00   Max.   :29.00

</p>
</details>

They can also contain plots

<details>
<summary>
Click Here!
</summary>
<p>

``` r
hist(mtcars$mpg)
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-2-1.png" width="480" />
</p>
</details>

# Tabs

There does not seem to be native support, as in normal rmarkdown fo example.

https://github.com/rstudio/blogdown/issues/69
https://github.com/yihui/hugo-prose/issues/25

But [completely css tabs](https://kyusuf.com/post/completely-css-tabs/)
seems to work.

<style>
  * {
  box-sizing: border-box;
}

body {
  padding: 10px;
  background: #f2f2f2;
}

.tabs {
  display: flex;
  flex-wrap: wrap;
  max-width: 700px;
  background: #efefef;
  box-shadow: 0 48px 80px -32px rgba(0,0,0,0.3);
}

.input {
  position: absolute;
  opacity: 0;
}

.label {
  width: 100%;
  padding: 20px 30px;
  background: #e5e5e5;
  cursor: pointer;
  font-weight: bold;
  font-size: 18px;
  color: #7f7f7f;
  transition: background 0.1s, color 0.1s;
}

.label:hover {
  background: #d8d8d8;
}

.label:active {
  background: #ccc;
}

.input:focus + .label {
  box-shadow: inset 0px 0px 0px 3px #2aa1c0;
  z-index: 1;
}

.input:checked + .label {
  background: #fff;
  color: #000;
}

@media (min-width: 600px) {
  .label {
    width: auto;
  }
}

.panel {
  display: none;
  padding: 20px 30px 30px;
  background: #fff;
}

@media (min-width: 600px) {
  .panel {
    order: 99;
  }
}

.input:checked + .label + .panel {
  display: block;
}


</style>

<div class="tabs">

<input name="tabs" type="radio" id="tab-1" checked="checked" class="input"/>
<label for="tab-1" class="label">Orange</label>

<div class="panel">

    <h1>Orange</h1>
    <p>The orange (specifically, the sweet orange) is the fruit of the citrus species Citrus × sinensis in the family Rutaceae</p>
    <p>The fruit of the Citrus × sinensis is considered a sweet orange, whereas the fruit of the Citrus × aurantium is considered a bitter orange. The sweet orange reproduces asexually (apomixis through nucellar embryony); varieties of sweet orange arise through mutations.</p>

</div>

<input name="tabs" type="radio" id="tab-2" class="input"/>
<label for="tab-2" class="label">Tangerine</label>

<div class="panel">

    <h1>Tangerine</h1>
    <p>The tangerine (Citrus tangerina) is an orange-colored citrus fruit that is closely related to, or possibly a type of, mandarin orange (Citrus reticulata).</p>
    <p>The name was first used for fruit coming from Tangier, Morocco, described as a mandarin variety. Under the Tanaka classification system, Citrus tangerina is considered a separate species.</p>

</div>

<input name="tabs" type="radio" id="tab-3" class="input"/>
<label for="tab-3" class="label">Clemantine</label>

<div class="panel">

    <h1>Clemantine</h1>
    <p>A clementine (Citrus ×clementina) is a hybrid between a mandarin orange and a sweet orange, so named in 1902. The exterior is a deep orange colour with a smooth, glossy appearance. Clementines can be separated into 7 to 14 segments. Similarly to tangerines, they tend to be easy to peel.</p>

</div>

</div>

Good!, verbose as hell, but, it works.

<div class="tabs">

<input name="tabs" type="radio" id="tab-10" checked="checked" class="input"/>
<label for="tab-10" class="label">Code</label>

<div class="panel">

``` r
toydf <- data.frame(foo = 1:10, bar = 20:29)
base::summary(toydf)
```

    ##       foo             bar       
    ##  Min.   : 1.00   Min.   :20.00  
    ##  1st Qu.: 3.25   1st Qu.:22.25  
    ##  Median : 5.50   Median :24.50  
    ##  Mean   : 5.50   Mean   :24.50  
    ##  3rd Qu.: 7.75   3rd Qu.:26.75  
    ##  Max.   :10.00   Max.   :29.00

</div>

<input name="tabs" type="radio" id="tab-20" class="input"/>
<label for="tab-20" class="label">Plot</label>

<div class="panel">

``` r
hist(mtcars$mpg)
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-4-1.png" width="480" />

</div>

<input name="tabs" type="radio" id="tab-30" class="input"/>
<label for="tab-30" class="label">Iteractive</label>

<div class="panel">

``` r
# Example from plotly page https://plotly.com/r/bubble-charts/
library(plotly)

data <- read.csv("https://raw.githubusercontent.com/plotly/datasets/master/school_earnings.csv")

data$State <- as.factor(c(
  'Massachusetts', 'California', 'Massachusetts', 'Pennsylvania', 'New Jersey', 
  'Illinois', 'Washington DC', 'Massachusetts', 'Connecticut', 'New York', 
  'North Carolina', 'New Hampshire', 'New York', 'Indiana', 'New York', 
  'Michigan', 'Rhode Island', 'California', 'Georgia', 'California', 
  'California'
))

fig <- plot_ly(
  data, x = ~Women, y = ~Men, text = ~School, type = 'scatter', 
  mode = 'markers', size = ~Gap, color = ~State, colors = 'Paired', 
  marker = list(opacity = 0.5, sizemode = 'diameter')
)
fig <- fig %>% layout(title = 'Gender Gap in Earnings per University',
         xaxis = list(showgrid = FALSE),
         yaxis = list(showgrid = FALSE),
         showlegend = FALSE)

fig
```

<div class="plotly html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-1" style="width:480px;height:384px;"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"visdat":{"1c5231735fab":["function () ","plotlyVisDat"]},"cur_data":"1c5231735fab","attrs":{"1c5231735fab":{"x":{},"y":{},"text":{},"mode":"markers","marker":{"opacity":0.5,"sizemode":"diameter"},"color":{},"size":{},"colors":"Paired","alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"type":"scatter"}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"title":"Gender Gap in Earnings per University","xaxis":{"domain":[0,1],"automargin":true,"showgrid":false,"title":"Women"},"yaxis":{"domain":[0,1],"automargin":true,"showgrid":false,"title":"Men"},"showlegend":false,"hovermode":"closest"},"source":"A","config":{"modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"data":[{"x":[96,71,64,72],"y":[151,88,78,81],"text":["Stanford","Berkeley","UCLA","SoCal"],"mode":"markers","marker":{"color":"rgba(166,206,227,1)","size":[94.4897959183673,24.6938775510204,19.1836734693878,10],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(166,206,227,1)"}},"type":"scatter","name":"California","textfont":{"color":"rgba(166,206,227,1)","size":[94.4897959183673,24.6938775510204,19.1836734693878,10]},"error_y":{"color":"rgba(166,206,227,1)","width":[]},"error_x":{"color":"rgba(166,206,227,1)","width":[]},"line":{"color":"rgba(166,206,227,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[79],"y":[114],"text":"Yale","mode":"markers","marker":{"color":"rgba(62,133,187,1)","size":[57.7551020408163],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(62,133,187,1)"}},"type":"scatter","name":"Connecticut","textfont":{"color":"rgba(62,133,187,1)","size":57.7551020408163},"error_y":{"color":"rgba(62,133,187,1)","width":57.7551020408163},"error_x":{"color":"rgba(62,133,187,1)","width":57.7551020408163},"line":{"color":"rgba(62,133,187,1)","width":57.7551020408163},"xaxis":"x","yaxis":"y","frame":null},{"x":[68],"y":[82],"text":"Emory","mode":"markers","marker":{"color":"rgba(147,190,154,1)","size":[19.1836734693878],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(147,190,154,1)"}},"type":"scatter","name":"Georgia","textfont":{"color":"rgba(147,190,154,1)","size":19.1836734693878},"error_y":{"color":"rgba(147,190,154,1)","width":19.1836734693878},"error_x":{"color":"rgba(147,190,154,1)","width":19.1836734693878},"line":{"color":"rgba(147,190,154,1)","width":19.1836734693878},"xaxis":"x","yaxis":"y","frame":null},{"x":[78],"y":[118],"text":"Chicago","mode":"markers","marker":{"color":"rgba(115,189,88,1)","size":[66.9387755102041],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(115,189,88,1)"}},"type":"scatter","name":"Illinois","textfont":{"color":"rgba(115,189,88,1)","size":66.9387755102041},"error_y":{"color":"rgba(115,189,88,1)","width":66.9387755102041},"error_x":{"color":"rgba(115,189,88,1)","width":66.9387755102041},"line":{"color":"rgba(115,189,88,1)","width":66.9387755102041},"xaxis":"x","yaxis":"y","frame":null},{"x":[73],"y":[100],"text":"Notre Dame","mode":"markers","marker":{"color":"rgba(144,163,89,1)","size":[43.0612244897959],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(144,163,89,1)"}},"type":"scatter","name":"Indiana","textfont":{"color":"rgba(144,163,89,1)","size":43.0612244897959},"error_y":{"color":"rgba(144,163,89,1)","width":43.0612244897959},"error_x":{"color":"rgba(144,163,89,1)","width":43.0612244897959},"line":{"color":"rgba(144,163,89,1)","width":43.0612244897959},"xaxis":"x","yaxis":"y","frame":null},{"x":[94,112,76],"y":[152,165,112],"text":["MIT","Harvard","Tufts"],"mode":"markers","marker":{"color":"rgba(248,131,123,1)","size":[100,90.8163265306122,59.5918367346939],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(248,131,123,1)"}},"type":"scatter","name":"Massachusetts","textfont":{"color":"rgba(248,131,123,1)","size":[100,90.8163265306122,59.5918367346939]},"error_y":{"color":"rgba(248,131,123,1)","width":[]},"error_x":{"color":"rgba(248,131,123,1)","width":[]},"line":{"color":"rgba(248,131,123,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[62],"y":[84],"text":"Michigan","mode":"markers","marker":{"color":"rgba(230,50,34,1)","size":[33.8775510204082],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(230,50,34,1)"}},"type":"scatter","name":"Michigan","textfont":{"color":"rgba(230,50,34,1)","size":33.8775510204082},"error_y":{"color":"rgba(230,50,34,1)","width":33.8775510204082},"error_x":{"color":"rgba(230,50,34,1)","width":33.8775510204082},"line":{"color":"rgba(230,50,34,1)","width":33.8775510204082},"xaxis":"x","yaxis":"y","frame":null},{"x":[84],"y":[114],"text":"Dartmouth","mode":"markers","marker":{"color":"rgba(252,181,104,1)","size":[48.5714285714286],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(252,181,104,1)"}},"type":"scatter","name":"New Hampshire","textfont":{"color":"rgba(252,181,104,1)","size":48.5714285714286},"error_y":{"color":"rgba(252,181,104,1)","width":48.5714285714286},"error_x":{"color":"rgba(252,181,104,1)","width":48.5714285714286},"line":{"color":"rgba(252,181,104,1)","width":48.5714285714286},"xaxis":"x","yaxis":"y","frame":null},{"x":[90],"y":[137],"text":"Princeton","mode":"markers","marker":{"color":"rgba(255,142,39,1)","size":[79.7959183673469],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(255,142,39,1)"}},"type":"scatter","name":"New Jersey","textfont":{"color":"rgba(255,142,39,1)","size":79.7959183673469},"error_y":{"color":"rgba(255,142,39,1)","width":79.7959183673469},"error_x":{"color":"rgba(255,142,39,1)","width":79.7959183673469},"line":{"color":"rgba(255,142,39,1)","width":79.7959183673469},"xaxis":"x","yaxis":"y","frame":null},{"x":[86,67,80],"y":[119,94,107],"text":["Columbia","NYU","Cornell"],"mode":"markers","marker":{"color":"rgba(233,159,143,1)","size":[54.0816326530612,43.0612244897959,43.0612244897959],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(233,159,143,1)"}},"type":"scatter","name":"New York","textfont":{"color":"rgba(233,159,143,1)","size":[54.0816326530612,43.0612244897959,43.0612244897959]},"error_y":{"color":"rgba(233,159,143,1)","width":[]},"error_x":{"color":"rgba(233,159,143,1)","width":[]},"line":{"color":"rgba(233,159,143,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[93],"y":[124],"text":"Duke","mode":"markers","marker":{"color":"rgba(158,123,186,1)","size":[50.4081632653061],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(158,123,186,1)"}},"type":"scatter","name":"North Carolina","textfont":{"color":"rgba(158,123,186,1)","size":50.4081632653061},"error_y":{"color":"rgba(158,123,186,1)","width":50.4081632653061},"error_x":{"color":"rgba(158,123,186,1)","width":50.4081632653061},"line":{"color":"rgba(158,123,186,1)","width":50.4081632653061},"xaxis":"x","yaxis":"y","frame":null},{"x":[92],"y":[141],"text":"U.Penn","mode":"markers","marker":{"color":"rgba(158,118,158,1)","size":[83.469387755102],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(158,118,158,1)"}},"type":"scatter","name":"Pennsylvania","textfont":{"color":"rgba(158,118,158,1)","size":83.469387755102},"error_y":{"color":"rgba(158,118,158,1)","width":83.469387755102},"error_x":{"color":"rgba(158,118,158,1)","width":83.469387755102},"line":{"color":"rgba(158,118,158,1)","width":83.469387755102},"xaxis":"x","yaxis":"y","frame":null},{"x":[72],"y":[92],"text":"Brown","mode":"markers","marker":{"color":"rgba(245,229,134,1)","size":[30.2040816326531],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(245,229,134,1)"}},"type":"scatter","name":"Rhode Island","textfont":{"color":"rgba(245,229,134,1)","size":30.2040816326531},"error_y":{"color":"rgba(245,229,134,1)","width":30.2040816326531},"error_x":{"color":"rgba(245,229,134,1)","width":30.2040816326531},"line":{"color":"rgba(245,229,134,1)","width":30.2040816326531},"xaxis":"x","yaxis":"y","frame":null},{"x":[94],"y":[131],"text":"Georgetown","mode":"markers","marker":{"color":"rgba(177,89,40,1)","size":[61.4285714285714],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(177,89,40,1)"}},"type":"scatter","name":"Washington DC","textfont":{"color":"rgba(177,89,40,1)","size":61.4285714285714},"error_y":{"color":"rgba(177,89,40,1)","width":61.4285714285714},"error_x":{"color":"rgba(177,89,40,1)","width":61.4285714285714},"line":{"color":"rgba(177,89,40,1)","width":61.4285714285714},"xaxis":"x","yaxis":"y","frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>

</div>

</div>

Far from perfect, but it works.

# Options

Remember to set this options, taken from the blogdown slides

> Hugo is fast. You are likely to be satisfied by the speed of the live preview via Serve Site (or blogdown::serve_site()).

> But live preview can be (a lot) faster. It may just take 10 milliseconds to re-render the site, if you install the processx package, and set an option in R:

    options(blogdown.generator.server = TRUE)

> By default, blogdown renders the full site to disk on changes and reload it in your browser, which may take one or two seconds. If you set this option, blogdown will take advantage of Hugo’s own server (via the command hugo server), which is much faster.

    options(
      blogdown.generator.server = TRUE,
      blogdown.hugo.server = c('-D', '-F', '--navigateToChanged')
    )
