---
author:
  name: "Nicole Hoffman"
date: 2019-03-14
linktitle: Kilauea 2018 earthquakes - Visualization with Bokeh
type:
- post
- posts
title: Kilauea 2018 earthquakes - Visualization with Bokeh
weight: 10
series:
- seismology

tags: ["earthquakes"]

---

The [2018 eruption of Kilauea volcano in Hawaii](https://en.wikipedia.org/wiki/2018_lower_Puna_eruption) was accompanied by thousands of small earthquakes over the course of the eruption. This seismicity exhibited an interesting pattern: cycles of increasing earthquake frequency, each ending with a larger magnitude explosive event. Over the course of the 2+ month eruption, there were over 50 of these cycles.

The [Bokeh](https://bokeh.pydata.org/en/latest/) plotting library provides a lot of great tools to look at a dataset like this, where it might be useful to zoom in on certain cycles or look only at a particular date. Here, we will start with a simple plot, using only circles and line segments.


```python
# Plotting
import bokeh
from bokeh.plotting import figure, output_file, show
from bokeh.io import output_notebook
from bokeh.models import Range1d

# Data analysis
import pandas as pd
from datetime import datetime
```

#### USGS earthquake data

The earthquake event data was downloaded from the [USGS Earthquake Catalog](https://earthquake.usgs.gov/earthquakes/search/). Before we could use it for this plotting exercise, it needed to be cleaned up a bit. After combining the separate files, dropping uneeded columns, and sorting by increasing date, the data is ready to be loaded in.


```python
# Read in the data for the earthquakes
events = pd.read_csv('/home/nicole/earthquakes/kilauea/data/all_events_M15.csv')
events.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DTtime</th>
      <th>depth</th>
      <th>mag</th>
      <th>magType</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-04-28 11:29:50.700</td>
      <td>0.27</td>
      <td>2.08</td>
      <td>ml</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018-04-28 23:09:53.430</td>
      <td>0.27</td>
      <td>2.12</td>
      <td>ml</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018-04-29 05:46:18.950</td>
      <td>0.23</td>
      <td>1.90</td>
      <td>ml</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-04-29 09:33:02.850</td>
      <td>1.39</td>
      <td>1.95</td>
      <td>ml</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018-05-01 06:34:32.670</td>
      <td>1.81</td>
      <td>1.78</td>
      <td>md</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Read in the data for the explosions
explode = pd.read_csv('/home/nicole/earthquakes/kilauea/data/all_explode_M4.csv')
explode.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DTtime</th>
      <th>depth</th>
      <th>mag</th>
      <th>magType</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-05-17 04:15:30.350</td>
      <td>0.01</td>
      <td>5.0</td>
      <td>mw</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018-05-17 14:04:10.700</td>
      <td>0.01</td>
      <td>5.0</td>
      <td>mw</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018-05-19 09:58:33.210</td>
      <td>0.01</td>
      <td>5.1</td>
      <td>mw</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-05-20 01:58:13.320</td>
      <td>0.01</td>
      <td>4.9</td>
      <td>mw</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018-05-20 21:50:07.310</td>
      <td>0.01</td>
      <td>4.9</td>
      <td>mw</td>
    </tr>
  </tbody>
</table>
</div>



#### Converting to a datetime object

The "DTtime" column is not currently in a datetime format; plotting will be easier if we convert the column to a `numpy.datetime64` object.


```python
# Convert "DTtime" to a datetime object

print("Time column format: ", events['DTtime'].dtype)
events['DTtime'] = pd.to_datetime(events['DTtime'])
explode['DTtime'] = pd.to_datetime(explode['DTtime'])
print("Time column format (converted): ", events['DTtime'].dtype)
```

    Time column format:  object
    Time column format (converted):  datetime64[ns]


### Make the plot!

We are ready to make our Bokeh plot. The earthquake will be plotted as circles, with time on the x-axis and earthquake magnitude on the y-axis. To better mark the time of the explosive event, they will be plotted as vertical lines with a height equal to the magnitude.

To display inline a Jupyter notebook (which I am using), we use the `output_notebook()` function.


```python
# Display plot in the notebook
output_notebook()
```



<div class="bk-root">
	<a href="https://bokeh.pydata.org" target="_blank" class="bk-logo bk-logo-small bk-logo-notebook"></a>
    <span id="1001">Loading BokehJS ...</span>
</div>





```python
# Make vertical line coordinates for each explosion

# x0, x1 = time of explosion
# y0 = 1.5, y1 = magnitude of explosion

ex_time = explode['DTtime']
ystart= [1.8] * len(ex_time)
yend = explode['mag']
```


```python
# Make the plot
output_file('kilauea2018_cycles.html', title='Kilauea Summit Earthquakes')

p = figure(plot_width=1000, plot_height=600,
           x_axis_type='datetime',
           x_axis_label = 'Date',
           y_axis_label = 'Magnitude',
           y_range = Range1d(1, 7),
           title = 'Kilauea summit earthquakes')

p.segment(x0=ex_time,y0=ystart,x1=ex_time,y1=yend,
         line_width=1, color='gray', alpha=0.4)

p.circle(x=events['DTtime'], y=events['mag'],
         size=1.5, color='darkblue', alpha=0.3,
        legend='earthquakes (Ml or Md)')

p.asterisk(x=explode['DTtime'], y=explode['mag'],
          size=10, color='gray', alpha=0.8,
          legend='summit collapse event (Mw equivalent)')

p.legend.location = "top_left"
p.legend.orientation = "horizontal"
p.legend.background_fill_alpha = 0

# To view the plot inline, uncomment the following line
#show(p)
```

In order to be able to interact with the figure we created above, we need to save the html of the file and then place that code in the static directory of the hugo site. I created a directory for my Bokeh plots and subdirectories for different topics.

First, import the necessary methods from the Bokeh library and create the html for the plot. Then write out the html to a file, close the file, and move it into the appropriate directory. When the Hugo site is built, finding the image will be as easy as clicking on the link below.


```python
from bokeh.embed import file_html
from bokeh.resources import CDN

html = file_html(p, CDN, "my plot")

kilauea2018=open("kilauea2018.html",mode="w",encoding="utf-8")
kilauea2018.write(html)
kilauea2018.close()
```

#### Interactive plot!

Check out the [interactive image](https://www.nicolew.xyz/bokeh/kilauea2018/kilauea2018.html)!
