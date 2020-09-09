---
layout: post
title:  "Building a training set thanks to bounding-box annotations"
author:	Nicholas Esterer and Emmanuelle Gouillart
---

![Screenshot of bounding box app](/assets/bounding-box-app.png)

Image processing tasks such as object detection or segmentation are often
performed using deep learning. In order to train a neural network, one must first
build a training set, which requires annotating a large set of images.
Commercial tools and open-source tools (such as [VIA](http://www.robots.ox.ac.uk/~vgg/software/via/)
or [VoTT](https://github.com/microsoft/VoTT)) are available in order to annotate batches of images.
We have built a [Dash app demo for annotating batches of images](https://dash-gallery.plotly.host/dash-image-annotation/),
which you can adapt to integrate an image annotator into your image processing
workflow.

The app can be seen at [https://dash-gallery.plotly.host/dash-image-annotation/](https://dash-gallery.plotly.host/dash-image-annotation/),
and its source code is available at
[https://github.com/plotly/dash-sample-apps/tree/master/apps/dash-image-annotation](https://github.com/plotly/dash-sample-apps/tree/master/apps/dash-image-annotation).


## Drawing bounding boxes on images

Image data can be visualized in plotly figures using the `px.imshow` function
from `plotly.express` or the `go.Image` trace, as explained in the [image
visualization tutorial of the plotly documentation](https://plotly.com/python/imshow/). 
In order to draw rectangles to annotate an image, you must select a drawing
tool for the figure's `dragmode`, here `drawrect` for drawing a rectangle.

```python
import plotly.express as px
from skimage import data
img = data.chelsea()
fig = px.imshow(img)
fig.update_layout(
    dragmode='drawrect',
    newshape_line_color='cyan')
)
fig.show()
```

With the `newshape` attribute of the figure's layout, you can style the drawn
rectangles (line width, color, ...). 

![Screenshot of bounding box app](/assets/imshow-chelsea-bb.png)

If you want the rectangle icon to appear in the figure's modebar (this is useful
to reselect the rectangle-drawing tool after selecting the pan or zoom mode,
for example), you can pass a list of buttons to add, corresponding to the
different drawing tools. The whole list is given below but you can choose to select only
some of them.

```python
fig.show(config={'modeBarButtonsToAdd':['drawline',
                                        'drawopenpath',
                                        'drawclosedpath',
                                        'drawcircle',
                                        'drawrect',
                                        'eraseshape'
                                       ]})
```

## Annotations and Dash callbacks

Drawing a new annotation triggers [a `relayout` event of the plotly figure](https://dash.plotly.com/interactive-graphing),
which can be used as an `Input` to a Dash callback. Here, the callback stores the
geometry of the annotation in a `dcc.Store` object so that it persists when
the user switches to another image to annotate. A "previous" and "next" button
update the image in the figure so that it is easy to annotate whole
batches of images.

Conversely, when navigating back to an image previously annotated, the plotly
figure is the `Output` of a callback, and annotations retrieved from the `dcc.Store`
are simply added to the figure layout so that they are displayed to the user.



## Interactive data table

Annotations can be modified either in the figure or in the interactive data
table, which are all updated thanks to a specific callback, and a `dcc.Store`
component with the data of the annotations. For example if you draw an
annotation with the wrong label (say "car" instead of "tree"), the easiest way to
correct this is to use the "Type" dropdown in the table. This will update the
color of the annotation in the figure, and the `dcc.Store`. 

See the [tutorial on Dash data tables](https://dash.plotly.com/datatable) for more details about
interactive data tables in Dash. 

## What's next? 

We hope you enjoyed this app! Suggestions are welcome, and can be submitted via the [source
repo](https://github.com/plotly/dash-sample-apps/) or [on Twitter](https://twitter.com/EGouillart).


If you need to annotate images to build a training set, you can adapt this
open-source app
to your application. Thanks to the modularity of Dash applications, you can for
example:

* use labels corresponding to your applications (e.g. different kinds of cells
  in cell biology) 
* retrieve images from a URL such as an S3 bucket
* ask annotators to enter their name using an input field or a dropdown,
  since it's usually considered good practice to have several people annotating
  the same dataset, in order to reduce user bias. Since Dash is stateless, it
  is possible to have several possible annotating images at the same time,
  without any conflict.
* connect the image annotation task to the training of a neural net, etc.


You might also be interested in checking out the [Dash app on live model training](https://dash-gallery.plotly.host/dash-live-model-training/) 
(and its [source code](https://github.com/plotly/dash-sample-apps/tree/master/apps/dash-live-model-training)),
and the Dash app on [object detection using a pre-trained state-of-the-art
neural net](https://dash-gallery.plotly.host/dash-detr/) (and its [source code](https://github.com/plotly/dash-detr)).

Finally, you can also check out other related posts from this blog, for
example the post on [trainable image segmentation](https://eoss-image-processing.github.io/2020/06/24/trainable-segmentation.html).
