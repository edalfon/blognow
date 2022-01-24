---
title: Dusting off an old blog
author: E!
date: '2021-12-01'
slug: dusting-off-an-old-blog
categories:
  - R
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
<link href="{{< blogdown/postref >}}index.en_files/vis/vis-network.min.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/vis/vis-network.min.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/visNetwork-binding/visNetwork.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/htmlwidgets/htmlwidgets.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/datatables-css/datatables-crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/datatables-binding/datatables.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/jquery/jquery-3.6.0.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/dt-core/css/jquery.dataTables.min.css" rel="stylesheet" />
<link href="{{< blogdown/postref >}}index.en_files/dt-core/css/jquery.dataTables.extra.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/dt-core/js/jquery.dataTables.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/crosstalk/css/crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/crosstalk/js/crosstalk.min.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/plotly-binding/plotly.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/typedarray/typedarray.min.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/jquery/jquery.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/crosstalk/css/crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/crosstalk/js/crosstalk.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/plotly-htmlwidgets-css/plotly-htmlwidgets.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/plotly-main/plotly-latest.min.js"></script>

Let’s use `{blogdown}` to dust off an old blog, where I used to take notes on
data analysis things (using SQL mainly on Postgres, R, Python, in that order).

**TODO**: bring old posts

Here just to try the blog setup and remember key things to tweak:

# Theme

Let’s use [Hugo Theme Stack https://github.com/CaiJimmy/hugo-theme-stack/](https://github.com/CaiJimmy/hugo-theme-stack/)
(arguments [here](https://restlessdata.com.au/p/updating-a-blog-theme/) were convincing )

I won’t spend much time tweaking this, so it will work/look as per the defaults.
Just a few changes are:

-   menu.main.params.newTab: False
-   hugo-theme-stack/assets/scss/variables.scss:
    -   –main-top-padding: 25px; instead of 50
    -   –section-separation: 20px; instead of 40
    -   –card-border-radius: 50px; instead of 10
-   hugo-theme-stack/assets/scss/partials/article.scss
    -   .article-image set default height: 100px; instead of 150, and 150 and 150
        for md and xl instead of 200 and 250
-   hugo-theme-stack/assets/scss/partials/layout/article.scss
    -   .article-header -> .article-image, set max-height: 30vh; instead of 50
        object-fit: contain; instead of cover
-   tableOfContents -> startLevel: 1

# Make sure plots are rendered

``` r
hist(mtcars$mpg, main = "Histogram")
```

<img src="https://i.imgur.com/esLppi3.png" width="480" />

And of course throw a ggplot in there too!

``` r
library(ggplot2)

ggplot(diamonds, aes(x = cut, fill = clarity)) +
  geom_bar(position = "dodge") +
  theme_classic() +
  theme(legend.position = "none")
```

<img src="https://i.imgur.com/PpfM35W.png" width="480" />

# Try a couple of `{htmlwidgets}`

For this to work, you gotta set `unsafe: true` in the `config.yaml`.

``` yaml
markup:
    goldmark:
        renderer:
            ## Set to true if you have HTML content inside Markdown
            unsafe: true
```

After doing this, it works!
(you may not need to do this if using `.Rmd` that directly renders to `html`
using pandoc)

## `{visNetwork}`

``` r
# An example from visNetwork
# https://datastorm-open.github.io/visNetwork/options.html
library(visNetwork)
nb <- 10
nodes <- data.frame(id = 1:nb, label = paste("Label", 1:nb),
 group = sample(LETTERS[1:3], nb, replace = TRUE), value = 1:nb,
 title = paste0("<p>", 1:nb,"<br>Tooltip !</p>"), stringsAsFactors = FALSE)

edges <- data.frame(from = c(8,2,7,6,1,8,9,4,6,2),
 to = c(3,7,2,7,9,1,5,3,2,9),
 value = rnorm(nb, 10), label = paste("Edge", 1:nb),
 title = paste0("<p>", 1:nb,"<br>Edge Tooltip !</p>"))

visNetwork(nodes, edges, height = "500px", width = "100%") %>% 
  visOptions(highlightNearest = TRUE, nodesIdSelection = TRUE) |> 
  visLayout(randomSeed = 123)
```

<div id="htmlwidget-1" style="width:100%;height:500px;" class="visNetwork html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"nodes":{"id":[1,2,3,4,5,6,7,8,9,10],"label":["Label 1","Label 2","Label 3","Label 4","Label 5","Label 6","Label 7","Label 8","Label 9","Label 10"],"group":["C","B","C","A","B","B","A","A","A","C"],"value":[1,2,3,4,5,6,7,8,9,10],"title":["<p>1<br>Tooltip !<\/p>","<p>2<br>Tooltip !<\/p>","<p>3<br>Tooltip !<\/p>","<p>4<br>Tooltip !<\/p>","<p>5<br>Tooltip !<\/p>","<p>6<br>Tooltip !<\/p>","<p>7<br>Tooltip !<\/p>","<p>8<br>Tooltip !<\/p>","<p>9<br>Tooltip !<\/p>","<p>10<br>Tooltip !<\/p>"]},"edges":{"from":[8,2,7,6,1,8,9,4,6,2],"to":[3,7,2,7,9,1,5,3,2,9],"value":[11.3576287728075,10.7921192649282,11.4786546453925,9.7849055910817,10.7139406494203,10.0636842666134,9.10598221259996,9.22295796058431,10.3655683939263,9.27683652589545],"label":["Edge 1","Edge 2","Edge 3","Edge 4","Edge 5","Edge 6","Edge 7","Edge 8","Edge 9","Edge 10"],"title":["<p>1<br>Edge Tooltip !<\/p>","<p>2<br>Edge Tooltip !<\/p>","<p>3<br>Edge Tooltip !<\/p>","<p>4<br>Edge Tooltip !<\/p>","<p>5<br>Edge Tooltip !<\/p>","<p>6<br>Edge Tooltip !<\/p>","<p>7<br>Edge Tooltip !<\/p>","<p>8<br>Edge Tooltip !<\/p>","<p>9<br>Edge Tooltip !<\/p>","<p>10<br>Edge Tooltip !<\/p>"]},"nodesToDataframe":true,"edgesToDataframe":true,"options":{"width":"100%","height":"100%","nodes":{"shape":"dot"},"manipulation":{"enabled":false},"layout":{"randomSeed":123}},"groups":["C","B","A"],"width":"100%","height":"500px","idselection":{"enabled":true,"style":"width: 150px; height: 26px","useLabels":true,"main":"Select by id"},"byselection":{"enabled":false,"style":"width: 150px; height: 26px","multiple":false,"hideColor":"rgba(200,200,200,0.5)","highlight":false},"main":null,"submain":null,"footer":null,"background":"rgba(0, 0, 0, 0)","highlight":{"enabled":true,"hoverNearest":false,"degree":1,"algorithm":"all","hideColor":"rgba(200,200,200,0.5)","labelOnly":true},"collapse":{"enabled":false,"fit":false,"resetHighlight":true,"clusterOptions":null,"keepCoord":true,"labelSuffix":"(cluster)"}},"evals":[],"jsHooks":[]}</script>

## `{DT}`

**TODO**: DT does not seem to be working

``` r
mtcars |> 
  DT::datatable(
    escape = FALSE,
    class = "display cell-border responsive nowrap compact",
    options = list(
      dom = 't',
      columnDefs = list(list(visible=FALSE, targets=c(2)))
    ),
    fillContainer = FALSE
  ) 
```

<div id="htmlwidget-1" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"filter":"none","vertical":false,"fillContainer":false,"data":[["Mazda RX4","Mazda RX4 Wag","Datsun 710","Hornet 4 Drive","Hornet Sportabout","Valiant","Duster 360","Merc 240D","Merc 230","Merc 280","Merc 280C","Merc 450SE","Merc 450SL","Merc 450SLC","Cadillac Fleetwood","Lincoln Continental","Chrysler Imperial","Fiat 128","Honda Civic","Toyota Corolla","Toyota Corona","Dodge Challenger","AMC Javelin","Camaro Z28","Pontiac Firebird","Fiat X1-9","Porsche 914-2","Lotus Europa","Ford Pantera L","Ferrari Dino","Maserati Bora","Volvo 142E"],[21,21,22.8,21.4,18.7,18.1,14.3,24.4,22.8,19.2,17.8,16.4,17.3,15.2,10.4,10.4,14.7,32.4,30.4,33.9,21.5,15.5,15.2,13.3,19.2,27.3,26,30.4,15.8,19.7,15,21.4],[6,6,4,6,8,6,8,4,4,6,6,8,8,8,8,8,8,4,4,4,4,8,8,8,8,4,4,4,8,6,8,4],[160,160,108,258,360,225,360,146.7,140.8,167.6,167.6,275.8,275.8,275.8,472,460,440,78.7,75.7,71.1,120.1,318,304,350,400,79,120.3,95.1,351,145,301,121],[110,110,93,110,175,105,245,62,95,123,123,180,180,180,205,215,230,66,52,65,97,150,150,245,175,66,91,113,264,175,335,109],[3.9,3.9,3.85,3.08,3.15,2.76,3.21,3.69,3.92,3.92,3.92,3.07,3.07,3.07,2.93,3,3.23,4.08,4.93,4.22,3.7,2.76,3.15,3.73,3.08,4.08,4.43,3.77,4.22,3.62,3.54,4.11],[2.62,2.875,2.32,3.215,3.44,3.46,3.57,3.19,3.15,3.44,3.44,4.07,3.73,3.78,5.25,5.424,5.345,2.2,1.615,1.835,2.465,3.52,3.435,3.84,3.845,1.935,2.14,1.513,3.17,2.77,3.57,2.78],[16.46,17.02,18.61,19.44,17.02,20.22,15.84,20,22.9,18.3,18.9,17.4,17.6,18,17.98,17.82,17.42,19.47,18.52,19.9,20.01,16.87,17.3,15.41,17.05,18.9,16.7,16.9,14.5,15.5,14.6,18.6],[0,0,1,1,0,1,0,1,1,1,1,0,0,0,0,0,0,1,1,1,1,0,0,0,0,1,0,1,0,0,0,1],[1,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,0,0,0,0,0,1,1,1,1,1,1,1],[4,4,4,3,3,3,3,4,4,4,4,3,3,3,3,3,3,4,4,4,3,3,3,3,3,4,5,5,5,5,5,4],[4,4,1,1,2,1,4,2,2,4,4,3,3,3,4,4,4,1,2,1,1,2,2,4,2,1,2,2,4,6,8,2]],"container":"<table class=\"display cell-border responsive nowrap compact\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>mpg<\/th>\n      <th>cyl<\/th>\n      <th>disp<\/th>\n      <th>hp<\/th>\n      <th>drat<\/th>\n      <th>wt<\/th>\n      <th>qsec<\/th>\n      <th>vs<\/th>\n      <th>am<\/th>\n      <th>gear<\/th>\n      <th>carb<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"dom":"t","columnDefs":[{"visible":false,"targets":2},{"className":"dt-right","targets":[1,2,3,4,5,6,7,8,9,10,11]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>

Let’s see later what is going on. (Clash with other widgets in this page?)

## `{plotly}`

``` r
# Example from plotly page
# https://plotly.com/r/bubble-charts/
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

<div id="htmlwidget-2" style="width:480px;height:384px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-2">{"x":{"visdat":{"23bc168368eb":["function () ","plotlyVisDat"]},"cur_data":"23bc168368eb","attrs":{"23bc168368eb":{"x":{},"y":{},"text":{},"mode":"markers","marker":{"opacity":0.5,"sizemode":"diameter"},"color":{},"size":{},"colors":"Paired","alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"type":"scatter"}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"title":"Gender Gap in Earnings per University","xaxis":{"domain":[0,1],"automargin":true,"showgrid":false,"title":"Women"},"yaxis":{"domain":[0,1],"automargin":true,"showgrid":false,"title":"Men"},"showlegend":false,"hovermode":"closest"},"source":"A","config":{"modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"data":[{"x":[96,71,64,72],"y":[151,88,78,81],"text":["Stanford","Berkeley","UCLA","SoCal"],"mode":"markers","marker":{"color":"rgba(166,206,227,1)","size":[94.4897959183673,24.6938775510204,19.1836734693878,10],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(166,206,227,1)"}},"type":"scatter","name":"California","textfont":{"color":"rgba(166,206,227,1)","size":[94.4897959183673,24.6938775510204,19.1836734693878,10]},"error_y":{"color":"rgba(166,206,227,1)","width":[]},"error_x":{"color":"rgba(166,206,227,1)","width":[]},"line":{"color":"rgba(166,206,227,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[79],"y":[114],"text":"Yale","mode":"markers","marker":{"color":"rgba(62,133,187,1)","size":[57.7551020408163],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(62,133,187,1)"}},"type":"scatter","name":"Connecticut","textfont":{"color":"rgba(62,133,187,1)","size":57.7551020408163},"error_y":{"color":"rgba(62,133,187,1)","width":57.7551020408163},"error_x":{"color":"rgba(62,133,187,1)","width":57.7551020408163},"line":{"color":"rgba(62,133,187,1)","width":57.7551020408163},"xaxis":"x","yaxis":"y","frame":null},{"x":[68],"y":[82],"text":"Emory","mode":"markers","marker":{"color":"rgba(147,190,154,1)","size":[19.1836734693878],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(147,190,154,1)"}},"type":"scatter","name":"Georgia","textfont":{"color":"rgba(147,190,154,1)","size":19.1836734693878},"error_y":{"color":"rgba(147,190,154,1)","width":19.1836734693878},"error_x":{"color":"rgba(147,190,154,1)","width":19.1836734693878},"line":{"color":"rgba(147,190,154,1)","width":19.1836734693878},"xaxis":"x","yaxis":"y","frame":null},{"x":[78],"y":[118],"text":"Chicago","mode":"markers","marker":{"color":"rgba(115,189,88,1)","size":[66.9387755102041],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(115,189,88,1)"}},"type":"scatter","name":"Illinois","textfont":{"color":"rgba(115,189,88,1)","size":66.9387755102041},"error_y":{"color":"rgba(115,189,88,1)","width":66.9387755102041},"error_x":{"color":"rgba(115,189,88,1)","width":66.9387755102041},"line":{"color":"rgba(115,189,88,1)","width":66.9387755102041},"xaxis":"x","yaxis":"y","frame":null},{"x":[73],"y":[100],"text":"Notre Dame","mode":"markers","marker":{"color":"rgba(144,163,89,1)","size":[43.0612244897959],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(144,163,89,1)"}},"type":"scatter","name":"Indiana","textfont":{"color":"rgba(144,163,89,1)","size":43.0612244897959},"error_y":{"color":"rgba(144,163,89,1)","width":43.0612244897959},"error_x":{"color":"rgba(144,163,89,1)","width":43.0612244897959},"line":{"color":"rgba(144,163,89,1)","width":43.0612244897959},"xaxis":"x","yaxis":"y","frame":null},{"x":[94,112,76],"y":[152,165,112],"text":["MIT","Harvard","Tufts"],"mode":"markers","marker":{"color":"rgba(248,131,123,1)","size":[100,90.8163265306122,59.5918367346939],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(248,131,123,1)"}},"type":"scatter","name":"Massachusetts","textfont":{"color":"rgba(248,131,123,1)","size":[100,90.8163265306122,59.5918367346939]},"error_y":{"color":"rgba(248,131,123,1)","width":[]},"error_x":{"color":"rgba(248,131,123,1)","width":[]},"line":{"color":"rgba(248,131,123,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[62],"y":[84],"text":"Michigan","mode":"markers","marker":{"color":"rgba(230,50,34,1)","size":[33.8775510204082],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(230,50,34,1)"}},"type":"scatter","name":"Michigan","textfont":{"color":"rgba(230,50,34,1)","size":33.8775510204082},"error_y":{"color":"rgba(230,50,34,1)","width":33.8775510204082},"error_x":{"color":"rgba(230,50,34,1)","width":33.8775510204082},"line":{"color":"rgba(230,50,34,1)","width":33.8775510204082},"xaxis":"x","yaxis":"y","frame":null},{"x":[84],"y":[114],"text":"Dartmouth","mode":"markers","marker":{"color":"rgba(252,181,104,1)","size":[48.5714285714286],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(252,181,104,1)"}},"type":"scatter","name":"New Hampshire","textfont":{"color":"rgba(252,181,104,1)","size":48.5714285714286},"error_y":{"color":"rgba(252,181,104,1)","width":48.5714285714286},"error_x":{"color":"rgba(252,181,104,1)","width":48.5714285714286},"line":{"color":"rgba(252,181,104,1)","width":48.5714285714286},"xaxis":"x","yaxis":"y","frame":null},{"x":[90],"y":[137],"text":"Princeton","mode":"markers","marker":{"color":"rgba(255,142,39,1)","size":[79.7959183673469],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(255,142,39,1)"}},"type":"scatter","name":"New Jersey","textfont":{"color":"rgba(255,142,39,1)","size":79.7959183673469},"error_y":{"color":"rgba(255,142,39,1)","width":79.7959183673469},"error_x":{"color":"rgba(255,142,39,1)","width":79.7959183673469},"line":{"color":"rgba(255,142,39,1)","width":79.7959183673469},"xaxis":"x","yaxis":"y","frame":null},{"x":[86,67,80],"y":[119,94,107],"text":["Columbia","NYU","Cornell"],"mode":"markers","marker":{"color":"rgba(233,159,143,1)","size":[54.0816326530612,43.0612244897959,43.0612244897959],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(233,159,143,1)"}},"type":"scatter","name":"New York","textfont":{"color":"rgba(233,159,143,1)","size":[54.0816326530612,43.0612244897959,43.0612244897959]},"error_y":{"color":"rgba(233,159,143,1)","width":[]},"error_x":{"color":"rgba(233,159,143,1)","width":[]},"line":{"color":"rgba(233,159,143,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[93],"y":[124],"text":"Duke","mode":"markers","marker":{"color":"rgba(158,123,186,1)","size":[50.4081632653061],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(158,123,186,1)"}},"type":"scatter","name":"North Carolina","textfont":{"color":"rgba(158,123,186,1)","size":50.4081632653061},"error_y":{"color":"rgba(158,123,186,1)","width":50.4081632653061},"error_x":{"color":"rgba(158,123,186,1)","width":50.4081632653061},"line":{"color":"rgba(158,123,186,1)","width":50.4081632653061},"xaxis":"x","yaxis":"y","frame":null},{"x":[92],"y":[141],"text":"U.Penn","mode":"markers","marker":{"color":"rgba(158,118,158,1)","size":[83.469387755102],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(158,118,158,1)"}},"type":"scatter","name":"Pennsylvania","textfont":{"color":"rgba(158,118,158,1)","size":83.469387755102},"error_y":{"color":"rgba(158,118,158,1)","width":83.469387755102},"error_x":{"color":"rgba(158,118,158,1)","width":83.469387755102},"line":{"color":"rgba(158,118,158,1)","width":83.469387755102},"xaxis":"x","yaxis":"y","frame":null},{"x":[72],"y":[92],"text":"Brown","mode":"markers","marker":{"color":"rgba(245,229,134,1)","size":[30.2040816326531],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(245,229,134,1)"}},"type":"scatter","name":"Rhode Island","textfont":{"color":"rgba(245,229,134,1)","size":30.2040816326531},"error_y":{"color":"rgba(245,229,134,1)","width":30.2040816326531},"error_x":{"color":"rgba(245,229,134,1)","width":30.2040816326531},"line":{"color":"rgba(245,229,134,1)","width":30.2040816326531},"xaxis":"x","yaxis":"y","frame":null},{"x":[94],"y":[131],"text":"Georgetown","mode":"markers","marker":{"color":"rgba(177,89,40,1)","size":[61.4285714285714],"sizemode":"diameter","opacity":0.5,"line":{"color":"rgba(177,89,40,1)"}},"type":"scatter","name":"Washington DC","textfont":{"color":"rgba(177,89,40,1)","size":61.4285714285714},"error_y":{"color":"rgba(177,89,40,1)","width":61.4285714285714},"error_x":{"color":"rgba(177,89,40,1)","width":61.4285714285714},"line":{"color":"rgba(177,89,40,1)","width":61.4285714285714},"xaxis":"x","yaxis":"y","frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>

# Let’s check Math is working

And that it works inline \$ x^2 \$ like this

And in blocks like this

$$
log(p/1-p)
$$

(You need to enable `math: yes` in the yaml header)
