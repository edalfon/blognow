---
title: visNetwork in blogdown?
author: ''
date: '2021-11-17'
slug: visnetwork-in-blogdown
categories: []
tags: []
description: ~
image: ~
math: ~
license: ~
hidden: no
comments: yes
---

<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<link href="{{< blogdown/postref >}}index_files/vis/vis-network.min.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/vis/vis-network.min.js"></script>
<script src="{{< blogdown/postref >}}index_files/visNetwork-binding/visNetwork.js"></script>
<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<link href="{{< blogdown/postref >}}index_files/vis/vis-network.min.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/vis/vis-network.min.js"></script>
<script src="{{< blogdown/postref >}}index_files/visNetwork-binding/visNetwork.js"></script>
<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<link href="{{< blogdown/postref >}}index_files/vis/vis-network.min.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/vis/vis-network.min.js"></script>
<script src="{{< blogdown/postref >}}index_files/visNetwork-binding/visNetwork.js"></script>

Ok, it works just like that

``` r
library(visNetwork)
nodes <- data.frame(id = 1:4)
edges <- data.frame(from = c(2,4,3,3), to = c(1,2,4,2))

visNetwork(nodes, edges, width = "100%") |> 
  visNodes(shape = "square", 
           color = list(background = "lightblue", 
                        border = "darkblue",
                        highlight = "yellow"),
           shadow = list(enabled = TRUE, size = 10)) |> 
  visLayout(randomSeed = 12) # to have always the same network     
```

<div id="htmlwidget-1" style="width:100%;height:480px;" class="visNetwork html-widget "></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"nodes":{"id":[1,2,3,4]},"edges":{"from":[2,4,3,3],"to":[1,2,4,2]},"nodesToDataframe":true,"edgesToDataframe":true,"options":{"width":"100%","height":"100%","nodes":{"shape":"square","color":{"background":"lightblue","border":"darkblue","highlight":"yellow"},"shadow":{"enabled":true,"size":10}},"manipulation":{"enabled":false},"layout":{"randomSeed":12}},"groups":null,"width":"100%","height":null,"idselection":{"enabled":false},"byselection":{"enabled":false},"main":null,"submain":null,"footer":null,"background":"rgba(0, 0, 0, 0)"},"evals":[],"jsHooks":[]}</script>

But remember, to add several htmlwidgets from a single chunk (e.g. in a loop) you need to use `knitr::knit_print`

``` r
library(visNetwork)
nodes <- data.frame(id = 1:4)
edges <- data.frame(from = c(2,4,3,3), to = c(1,2,4,2))

vn1 <- visNetwork(nodes, edges, width = "100%") 

vn2 <- visNetwork(nodes, edges, width = "100%") |> 
  visNodes(shape = "square", 
           color = list(background = "lightblue", 
                        border = "darkblue",
                        highlight = "yellow"),
           shadow = list(enabled = TRUE, size = 10)) |> 
  visLayout(randomSeed = 12) # to have always the same network    

cat("\n\nFirst \n\n")
```

    ## 
    ## 
    ## First

``` r
knitr::knit_print(vn1)
```

<div id="htmlwidget-2" style="width:100%;height:500px;" class="visNetwork html-widget "></div>
<script type="application/json" data-for="htmlwidget-2">{"x":{"nodes":{"id":[1,2,3,4]},"edges":{"from":[2,4,3,3],"to":[1,2,4,2]},"nodesToDataframe":true,"edgesToDataframe":true,"options":{"width":"100%","height":"100%","nodes":{"shape":"dot"},"manipulation":{"enabled":false}},"groups":null,"width":"100%","height":null,"idselection":{"enabled":false},"byselection":{"enabled":false},"main":null,"submain":null,"footer":null,"background":"rgba(0, 0, 0, 0)"},"evals":[],"jsHooks":[]}</script>

``` r
cat("\n\nSecond \n\n")
```

    ## 
    ## 
    ## Second

``` r
knitr::knit_print(vn2)
```

<div id="htmlwidget-3" style="width:100%;height:500px;" class="visNetwork html-widget "></div>
<script type="application/json" data-for="htmlwidget-3">{"x":{"nodes":{"id":[1,2,3,4]},"edges":{"from":[2,4,3,3],"to":[1,2,4,2]},"nodesToDataframe":true,"edgesToDataframe":true,"options":{"width":"100%","height":"100%","nodes":{"shape":"square","color":{"background":"lightblue","border":"darkblue","highlight":"yellow"},"shadow":{"enabled":true,"size":10}},"manipulation":{"enabled":false},"layout":{"randomSeed":12}},"groups":null,"width":"100%","height":null,"idselection":{"enabled":false},"byselection":{"enabled":false},"main":null,"submain":null,"footer":null,"background":"rgba(0, 0, 0, 0)"},"evals":[],"jsHooks":[]}</script>
