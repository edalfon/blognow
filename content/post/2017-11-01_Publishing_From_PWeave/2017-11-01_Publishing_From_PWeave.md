---
title: Publishing blog posts from PWeave
author: edalfon
date: '2017-11-01'
categories:
  - Python
tags:
  - PWeave
image: /post/2017-11-01_Publishing_From_PWeave/2017-11-01_Publishing_From_PWeave_figure1_2.png
---

This blog started mostly as an archive for R-related things as I was starting
to use R as my main tool / language for data analysis. For a while, I've been
also using Python. Looking for a literate programming alterative in Python
I found [Pweave](http://mpastell.com/pweave). So I'd like to also use this blog
as an archive for Python-related things, starting with how to use PWeave to
write blog posts. Yeah, I guess I can keep using RMarkdown to write
Python-related blog posts, but I find Python support still limited and
therefore I prefer to write the posts in the main editor I use to write python
code (currently Atom).

# PWeave

Using PWeave you can write using several formats and then compile the notebook
into, also, several formats (e.g. markdown, html). It seems it was inspired by
Sweave, which was the most common literate programming approach for R users
before RmarkDown came to the scene.

I would use either the pmd format (markdown + python code chuncks) or simply
python scripts, compiled into a notebook using PWeave (similar to knitr's spin).

# Blogdown integration

If you are not using a different container for images, in `blogdown` they are
located in the `static/post` directory that is then published as `/post` by
netlify. So tipically an image is referenced from html or markdown
files as `/post/post_dir/figure-html/x.png`

PWeave on the other hand by default generates images in the `./images`
directory.

So, to get it to work, you can change the figures directory, for example,
using the [command line args](http://mpastell.com/pweave/script.html).

Assuming the post .pmd source file is located in the standard directory
`content/post` and considering the imagens need to be in
the `static/post` directory, let's change it to that. That way, the
figures will be generated in the correct location and you just need to add the
files and push the changes to the repository for netlify to process them.

However, you need to change the generated markdown file, to
amend the location of the figures. Changing it from the relative path
(`../../post`) to the absolute path (`/post`), since that will be the location
in netlify. Alternatively, realizing that netlify arranges the file by
year/month/post_title/index.html you could try to fix the reference also to
a relative path. But that seems like a silly option in comparison, and
error-prone in case the date is not available or who knows.

So, then you would have to do something like:

- cd to the directory where the blog source is located (`BLOG_PATH/content/post`)
- `pweave -f markdown 2017-11-01_Publishing_From_PWeave.pmd --figure-directory=/post/2017-11-01_Publishing_From_PWeave`
- `sed -i 's+++g' ./2017-11-01_Publishing_From_PWeave.md`

I guess this is pretty trivial to configure using, for example,
[atom-shell-commands](https://atom.io/packages/atom-shell-commands). Although
I am not sure if it is possible to run both commands above with only one
atom-shell-command. And it seems you cannot avoid using two commands, because
Pweave will do what you tell him in the `--figure-directory`, but that would be
either a relative path (in which case would be relative to the directory where
the output file is being generated) or an absolute path. And none of those
options will ever match the directory structure within netlify (`/post/post_dir`).

# PWeave supported features

## Matplotlib plots

It supports `matplotlib` plots.


```python
# Import the necessary libraries
import matplotlib.pyplot as plt
import pandas as pd

# Initialize Figure and Axes object
fig, ax = plt.subplots()

# Load in data
tips = pd.read_csv("https://raw.githubusercontent.com/mwaskom/seaborn-data/master/tips.csv")

# Create violinplot
ax.violinplot(tips["total_bill"], vert=False)

# Show the plot
plt.show()
```

![](/post/2017-11-01_Publishing_From_PWeave/2017-11-01_Publishing_From_PWeave_figure1_1.png)\





```python
# And this from the official matplotlib examples
# https://matplotlib.org/3.1.1/tutorials/introductory/sample_plots.html

import matplotlib.pyplot as plt
import numpy as np

np.random.seed(19680801)
data = np.random.randn(2, 100)

fig, axs = plt.subplots(2, 2, figsize=(5, 5))
axs[0, 0].hist(data[0])
axs[1, 0].scatter(data[0], data[1])
axs[0, 1].plot(data[0], data[1])
axs[1, 1].hist2d(data[0], data[1])

plt.show()
```

![](/post/2017-11-01_Publishing_From_PWeave/2017-11-01_Publishing_From_PWeave_figure1_2.png)\


## Mathjax

The easy way ...

this is how blogdown does it. Wrap it up in a 'math inline' css class.
it becomes somewhat verbose, but it works.
otherwise you would have to mess with Hugo, including the proper
libraries and so on in the header.html or footer.html or something like
that, ..., or even worse, not generating a markdown file but an html
file, which PWeave does for you, but then you would have to deal
with getting the yaml headers rigth in the html file, and getting rid of
the html headers jibber-jabber, so that dates and summary are properly
displayed in the blog entries page

<p><span class="math inline">\(y=x^2\)</span></p>

[//]: # ($w(n) = \alpha - \beta\cos\frac{2\pi n}{N-1}$, where $\alpha=0.54$ and $\beta=0.46$)

<p><span class="math inline">\(w(n) = \alpha - \beta\cos\frac{2\pi n}{N-1}\)</span>, where <p><span class="math inline">\(\alpha=0.54\)</span> and <span class="math inline">\(\beta=0.46\)</span></p>

## Interactive plots via bokeh

Sort of native support. Pweave has a couple of functions to help you out here.
They include the proper headers in the html file and include Bokeh's output
properly unescaped in the generated file.

Just a warning, it works if you pweave into an html file. That makes sense,
but if you want to pweave it into a markdown file, the `pweave.bokeh.show`
function will not do the trick. That's because they use
`IPython.display.display_html` (see [here](https://github.com/mpastell/Pweave/blob/master/pweave/bokeh/__init__.py)).
It seems at some point they were experimenting in providing support also
for markdown (`display_markdown` was imported but its usage commented out).
So, if you need to include Bokeh plots in markdown generated files (like in my
case, files that later will be converted into html by other tool -Hugo via
netlify in my case-, otherwise, perhaps it makes no sense at all), then you
have to work a little bit more. Here's the example:



```python
# Original example taken from http://mpastell.com/pweave/bokeh.html
# This works smoothly if you pweave into html (md2html)
from bokeh.plotting import figure
from pweave.bokeh import output_pweave, show

# but if you are generating a markdown file, this line does not work
#output_pweave()

# So let's just tweak the original function to make it work for markdown
# https://github.com/mpastell/Pweave/blob/master/pweave/bokeh/__init__.py

# For it to work, all the output should be within a single container (div)
from IPython.display import display_markdown
display_markdown('<div>', raw=True)

# Then just do the same from output_pweave(), but with display_markdown
from bokeh.resources import CDN
out = CDN.render_css()
out += CDN.render_js()
from IPython.display import display_markdown
display_markdown(out, raw=True)

x = [1, 2, 3, 4, 5]
y = [6, 7, 2, 4, 5]
p = figure(title="simple line example", x_axis_label='x', y_axis_label='y')
p.line(x, y, legend="Temp.", line_width=2)

# but if you are generating a markdown file, this line does not work either
#show(p)

# So let's just tweak the original function to make it work for markdown
# https://github.com/mpastell/Pweave/blob/master/pweave/bokeh/__init__.py
from bokeh.embed import components
from IPython.display import display_markdown
script, div = components(p)
out = script
out+= div
display_markdown(out, raw=True)

# Do not forget to close the div
display_markdown('</div>', raw=True)
```

```
---------------------------------------------------------------------------ModuleNotFoundError
Traceback (most recent call last)<ipython-input-1-2f8bd52361da> in
<module>
      1 # Original example taken from
http://mpastell.com/pweave/bokeh.html
      2 # This works smoothly if you pweave into html (md2html)
----> 3 from bokeh.plotting import figure
      4 from pweave.bokeh import output_pweave, show
      5
ModuleNotFoundError: No module named 'bokeh'
```



So, yeah, it works. But it is somewhat cumbersome. It would be better that
Pweave would detect the target file and fix the output accordingly. Maybe a PR?

## Interactive plots via plotly

For Plotly, as fas as I know, there is no native support. You can nonetheless
try to include Plotly as follows.

First, take advantage of the `plotly.offline.plot` function, to generate
the HTML for the plot, making sure you use the `output_type="div"` and not the
default standalone HTML file. Also, you would need to let it `include_plotlyjs`.
Either by `include_plotlyjs=True`, which creates a fully standalone plot that
you can use offline, because it includes all the Plotly script (the downside is
that the file size would be > 3MB), or by setting `include_plotlyjs="cdn"`,
which includes in the div a link to the plotly script (but then to view the
file you need to be online).

Then, you need to use `display_html` from `IPython.display` with `raw=True` to
include the code in the output file. Otherwise, Pweave would escape it and
you would lose all the html markup. Ok, here's a weird thing: if you pweave the
pmd to html, `display_html` works, but if you want to generate a markdown file
it seems you need to use `display_markdown` (weird in the sense that I do not
like it).

And I think that's it. Here's the example.


```python
# Basic plotly example from the official web page
import plotly
import plotly.graph_objects as go
fig = go.Figure(
    data=[go.Bar(y=[2, 1, 3])], layout_title_text="A Figure Displaying Itself"
)

plotly_div = plotly.offline.plot(
    fig,
    include_plotlyjs="cdn",
    show_link=False,
    output_type='div'
)

#from IPython.display import display_html
#display_html(plotly_div, raw=True)
from IPython.display import display_markdown
display_markdown(plotly_div, raw=True)
```



<div>
        
                <script type="text/javascript">window.PlotlyConfig = {MathJaxConfig: 'local'};</script>
        <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>    
            <div id="e4b8f7f8-0d4d-4b6f-ade0-83d2d49cda61" class="plotly-graph-div" style="height:100%; width:100%;"></div>
            <script type="text/javascript">
                
                    window.PLOTLYENV=window.PLOTLYENV || {};
                    
                if (document.getElementById("e4b8f7f8-0d4d-4b6f-ade0-83d2d49cda61")) {
                    Plotly.newPlot(
                        'e4b8f7f8-0d4d-4b6f-ade0-83d2d49cda61',
                        [{"type": "bar", "y": [2, 1, 3]}],
                        {"template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "A Figure Displaying Itself"}},
                        {"responsive": true}
                    )
                };
                
            </script>
        </div>


## Other html widgets ???


# Other alternatives for literate programming in python

Check [this](https://gist.github.com/mrtns/da998d5fde666d6da26807e1f246246e) out

## knitpy

[knitpy](https://github.com/jankatins/knitpy)

A port of knitr, but ...
- it seems there is no active development and it still lack many features
- no support in atom (syntax, etc.)

## Python handout

Python [handout](https://github.com/danijar/handout)

- It's like knitr's spin, that compiles python scripts into markdown
- I like that it uses """ for the text, which makes it clearer than using normal
comments (like pweave, that also supports compiling python scripts)
- I don't like that you sort of need to expllicitly import and use `handout`
within the document.

## Codebraid

[codebraid](https://github.com/gpoore/codebraid) looks like full-featured

- uses pandoc
- supports also multiple languages, python, julia, javascript, R

## more?

https://pypi.org/project/antiweb/0.2.2/

# Session Info


```python
from sinfo import sinfo
sinfo_html = sinfo(html=True)
display_markdown(sinfo_html.data, raw=True)
```


<details>
<summary>Click to view module versions</summary>
<pre>
-----
matplotlib  3.2.1
numpy       1.18.1
pandas      1.0.3
plotly      4.6.0
sinfo       0.3.1
-----
</pre>
<details>
<summary>Click to view dependency modules</summary>
<pre>
attr                19.3.0
backcall            0.1.0
colorama            0.4.3
concurrent          NA
cycler              0.10.0
cython_runtime      NA
dateutil            2.8.1
decorator           4.4.2
defusedxml          0.6.0
encodings           NA
entrypoints         0.3
genericpath         NA
importlib_metadata  1.6.0
ipykernel           5.2.0
ipython_genutils    0.2.0
ipywidgets          7.5.1
jedi                0.15.2
jinja2              2.11.1
jsonschema          3.2.0
kiwisolver          1.2.0
markupsafe          1.1.1
matplotlib          3.2.1
mistune             0.8.4
mpl_toolkits        NA
nbconvert           5.6.1
nbformat            5.0.4
nt                  NA
ntpath              NA
nturl2path          NA
numpy               1.18.1
opcode              NA
pandas              1.0.3
pandocfilters       NA
parso               0.6.2
pickleshare         0.7.5
plotly              4.6.0
posixpath           NA
prompt_toolkit      3.0.5
pvectorc            NA
pweave              0.30.3
pydoc_data          NA
pyexpat             NA
pygments            2.6.1
pyparsing           2.4.7
pyrsistent          NA
pytz                2019.3
retrying            NA
sinfo               0.3.1
six                 1.14.0
sre_compile         NA
sre_constants       NA
sre_parse           NA
testpath            0.4.4
tornado             6.0.4
traitlets           4.3.3
wcwidth             NA
zipp                NA
zmq                 19.0.0
</pre>
</details> <!-- seems like this ends pre, so might as well be explicit -->
<pre>
-----
IPython             7.13.0
jupyter_client      6.1.2
jupyter_core        4.6.3
notebook            6.0.3
-----
Python 3.7.6 | packaged by conda-forge | (default, Mar 23 2020, 22:22:21) [MSC v.1916 64 bit (AMD64)]
Windows-10-10.0.18362-SP0
4 logical CPU cores, Intel64 Family 6 Model 142 Stepping 9, GenuineIntel
-----
Session information updated at 2020-04-13 23:56
</pre>
</details>
