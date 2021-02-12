---
jupyter:
  jupytext:
    notebook_metadata_filter: all
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.3.0
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
  language_info:
    codemirror_mode:
      name: ipython
      version: 3
    file_extension: .py
    mimetype: text/x-python
    name: python
    nbconvert_exporter: python
    pygments_lexer: ipython3
    version: 3.7.3
---

# Image segmentation with Dash and scikit-image

Image segmentation consists in labeling pixels corresponding to different image regions. There exists a large number of image segmentation workflows, depending on how pixels of different labels can be differentiated: based on image intensity values, prominent object boundaries, spatial proximity of pixels, textures, etc. The scikit-image library has a large number of functions for image segmentation (see for example the [gallery examples for segmentation with scikit-image](https://scikit-image.org/docs/dev/auto_examples/index.html#segmentation-of-objects)). In this tutorial, we focus on use cases where user interaction is required, either for fast parameter selection, or for selecting specific parts of an image (as seeds, training set, etc.).


## Image thresholding

```python
import plotly.express as px
import plotly.graph_objects as go
from jupyter_dash import JupyterDash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State# Load Data
from skimage import data

img = data.camera()

# Build App
app = JupyterDash(__name__)
app.layout = html.Div([
    html.H3("Drag the slider to threshold the image"),
    dcc.Graph(id='graph', figure=px.imshow(img, binary_string=True)),
    dcc.Slider(id='slider', value=128, updatemode='drag')
])# Define callback to update graph

@app.callback(
    Output('graph', 'figure'),
    [Input("slider", "value")]
)
def update_figure(val):
    fig = px.imshow(img, binary_string=True)
    fig.add_trace(go.Contour(z=img, showscale=False,
                         contours=dict(start=0, end=val, size=val, coloring='lines'),
                         line_width=2))
    return fig

app.run_server(mode='inline', port=8053)
```

```python
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from jupyter_dash import JupyterDash
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State# Load Data
from skimage import data, segmentation

img = data.camera()
fig_img = px.imshow(img, binary_string=True)
fig_hist = px.histogram(img.ravel())
fig_hist.update_layout(dragmode='select')

# Build App
app = JupyterDash(__name__)
app.layout = html.Div([
    html.H3("Select a range of values in histogram figure"),
     html.Div([
         dcc.Graph(id='image', figure=fig_img),
    ], style={'width': '60%', 'display': 'inline-block', 'padding': '0 0'}),
    
    html.Div([
        dcc.Graph(id='histogram', figure=fig_hist),
    ], style={'width': '40%', 'display': 'inline-block', 'padding': '0 0'}),
])


@app.callback(
    Output('image', 'figure'),
    [Input("histogram", "selectedData"),
     ], 
    prevent_initial_call=True
)
def update_figure(select_data):
    vmin, vmax = select_data['range']['x']
    mask = np.logical_and(img > vmin, img < vmax)
    return px.imshow(mask, binary_string=True)


app.run_server(mode='inline', port=8061)
```

```python
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from jupyter_dash import JupyterDash
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State# Load Data
from skimage import data, filters

img = data.microaneurysms()

# Build App
app = JupyterDash(__name__)
app.layout = html.Div([
    html.H3("Hysteresis thresholding"),
     html.Div([
         dcc.Graph(id='graph', figure=px.imshow(255 - img)),
    ], style={'width': '49%', 'display': 'inline-block', 'padding': '0 0'}),
    
    html.Div([
        dcc.Graph(id='segmentation', figure=px.imshow(np.zeros_like(img), binary_string=True)),
    ], style={'width': '49%', 'display': 'inline-block', 'padding': '0 0'}),
    html.H5("Thresholds", id='title'),
    dcc.Slider(id="slider-low", min=0, max=255, value=180),
    dcc.Slider(id="slider-high", min=0, max=255, value=190),
])

@app.callback(
    Output('segmentation', 'figure'),
    [Input("slider-low", "value"),
     Input("slider-high", "value")],
)
def update_figure(low, high):
    
    mask = filters.apply_hysteresis_threshold(255 -img, low, high)
    fig = px.imshow(mask, binary_string=True)
    return fig

@app.callback(
    Output('title', 'children'),
    [Input("slider-low", "value"),
     Input("slider-high", "value")],
)
def update_title(low, high):
    return f"Threshold: low = {low}, high = {high}"

app.run_server(mode='inline', port=8055)
```

```python
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from jupyter_dash import JupyterDash
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State# Load Data
from skimage import data, segmentation

img = data.camera()

# Build App
app = JupyterDash(__name__)
app.layout = html.Div([
    html.H3("Flood Fill Demo: click on camera man"),
     html.Div([
         dcc.Graph(id='graph', figure=px.imshow(img, binary_string=True)),
    ], style={'width': '49%', 'display': 'inline-block', 'padding': '0 0'}),
    
    html.Div([
        dcc.Graph(id='segmentation', figure=px.imshow(np.zeros_like(img), binary_string=True)),
    ], style={'width': '49%', 'display': 'inline-block', 'padding': '0 0'}),
    html.H5("Tolerance", id='tolerance-title'),
    dcc.Slider(id="slider", min=0, max=50, value=5),
])

@app.callback(
    Output('segmentation', 'figure'),
    [Input("graph", "clickData"),
     Input("slider", "value")],
    prevent_initial_call=True
)
def update_figure(point, val):
    if point is None:
        return dash.no_update
    row, col = point['points'][0]['y'], point['points'][0]['x']
    mask = segmentation.flood(img, (row, col), tolerance=val)
    fig = px.imshow(mask, binary_string=True)
    return fig

@app.callback(
    Output('tolerance-title', 'children'),
    [Input("slider", "value")],
)
def update_title(val):
    return f"Tolerance: {val}"

app.run_server(mode='inline', port=8054)
```

```python
import plotly.express as px
import plotly.graph_objects as go
from skimage import data, filters, measure, segmentation, morphology
from scipy import ndimage

img = data.coins()
threshold = filters.threshold_otsu(img)
mask = img > threshold
mask = morphology.remove_small_objects(mask, 50)
mask = morphology.remove_small_holes(mask, 50)
mask = ndimage.binary_fill_holes(mask)
labels = measure.label(mask)

fig = px.imshow(img, binary_string=True)
fig.update_traces(hoverinfo='skip')

props = measure.regionprops(labels, img)
properties = ['area', 'eccentricity', 'perimeter', 'mean_intensity']
for index in range(1, labels.max()):
    label = props[index].label
    contour = measure.find_contours(labels == label, 0.5)[0]
    y, x = contour.T
    hoverinfo = ''
    for prop_name in properties:
        hoverinfo += f'<b>{prop_name}: {getattr(props[index], prop_name):.2f}</b><br>'
    fig.add_trace(go.Scatter(
        x=x, y=y, name=label,
        mode='lines', fill='toself', showlegend=False, 
        hovertemplate=hoverinfo, hoveron='points+fills'))
fig.show()
```

```python
import plotly.express as px
import plotly.graph_objects as go
import plotly
from skimage import data, filters, measure, segmentation, morphology, color
from skimage import img_as_ubyte
from scipy import ndimage

img = data.coins()
threshold = filters.threshold_otsu(img)
mask = img > threshold
mask = morphology.remove_small_objects(mask, 50)
mask = morphology.remove_small_holes(mask, 50)
mask = ndimage.binary_fill_holes(mask)
labels = measure.label(mask)

fig = px.imshow(img, binary_string=True)
fig.update_traces(hoverinfo='skip')

props = measure.regionprops(labels, img)
properties = ['area', 'eccentricity', 'perimeter', 'mean_intensity']
colorscale = plotly.colors.qualitative.Alphabet
n_colors = len(colorscale)

for index in range(1, labels.max()):
    label = props[index].label
    bbox = props[index].bbox
    hoverinfo = ''
    for prop_name in properties:
        hoverinfo += f'<b>{prop_name}</b>: {getattr(props[index], prop_name):.2f}<br>'
    label_img = props[index].image.astype(np.uint8)
    color_label_img = np.dstack((label_img,) * 4)
    color = np.concatenate((plotly.colors.hex_to_rgb(colorscale[index % n_colors]), [100])).astype(np.uint8)
    color_label_img *= color 
    aux_fig = px.imshow(color_label_img)
    aux_fig.update_traces(name=index, x0=bbox[1], y0=bbox[0], hovertemplate=hoverinfo)
    fig.add_trace(aux_fig.data[0])
fig.show()
```

```python
import numpy as np
from jupyter_dash import JupyterDash
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State
import plotly.express as px
import plotly.graph_objects as go
from skimage import io, segmentation, filters, draw, morphology
from scipy import ndimage

def path_to_indices(path):
    """From SVG path to numpy array of coordinates, each row being a (row, col) point
    """
    indices_str = [el.replace('M', '').replace(
        'Z', '').split(',') for el in path.split('L')]
    return np.rint(np.array(indices_str, dtype=float)).astype(np.int)

def path_to_mask(path):
    cols, rows = path_to_indices(path).T
    line_cols, line_rows = [], []
    for i in range(len(cols))[:-1]:
        rr, cc = draw.line(rows[i], cols[i], rows[i + 1], cols[i + 1])
        line_rows += list(rr)
        line_cols += list(cc)
    return line_rows, line_cols

img = io.imread(
    'https://upload.wikimedia.org/wikipedia/commons/e/e4/Mitochondria%2C_mammalian_lung_-_TEM_%282%29.jpg',
    as_gray=True)
edges = ndimage.gaussian_gradient_magnitude(img, 5)
line_width = 13


fig = px.imshow(img, binary_string=True)
fig.update_layout(
    dragmode='drawopenpath',
    newshape=dict(line_color='cyan', line_width=line_width)
)

app = JupyterDash(__name__)
app.layout = html.Div([
    html.H1("JupyterDash Demo"),
    dcc.Graph(id='graph', figure=fig, 
              config={'modeBarButtonsToAdd':[
                                        'drawopenpath',
                                        'eraseshape'
                                       ]}),
])# Define callback to update graph

@app.callback(
    Output('graph', 'figure'),
    [Input("graph", "relayoutData")],
    prevent_initial_call=True
)
def update_figure(relayout_data):
    print(relayout_data)
    if 'shapes' in relayout_data and len(relayout_data['shapes']) >= 2:
        mask = np.zeros(img.shape[:2], dtype=np.uint8)
        for i, shape in enumerate(relayout_data['shapes']):
            rows, cols = path_to_mask(shape['path'])
            mask[rows, cols] = i + 1
        mask = morphology.dilation(mask, np.ones((line_width, line_width)))
        output = segmentation.watershed(edges, mask)
        fig = px.imshow(img, binary_string=True)
        n_contours = len(relayout_data['shapes']) - 1
        fig.add_trace(go.Contour(z=output, showscale=False, opacity=0.3,
                         contours=dict(start=0, end=n_contours, size=n_contours, #coloring='lines'
                                      ),
                         line_width=2))
        fig.update_layout(
            shapes=relayout_data['shapes'],
            dragmode='drawopenpath',
            newshape=dict(line_color='cyan', line_width=line_width)
        )
        return fig
    else:        
        return dash.no_update

app.run_server(mode='inline', port=8080)
```

```python
from skimage import edges

```

```python
import plotly.express as px
import plotly.graph_objects as go
from skimage import data, filters, measure, segmentation, morphology
from scipy import ndimage

img = data.coins()
threshold = filters.threshold_otsu(img)
mask = img > threshold
mask = morphology.remove_small_objects(mask, 50)
mask = morphology.remove_small_holes(mask, 50)
mask = ndimage.binary_fill_holes(mask)
labels = measure.label(mask)

fig = px.imshow(img, binary_string=True)
fig.update_traces(hoverinfo='skip')
fig.add_trace(go.Contour(
    z=labels,
    contours=dict(start=0, end=labels.max() + 1, size=1,
                          coloring='lines'),
            line=dict(width=1),
            showscale=False,
            #colorscale=custom_viridis,
            #opacity=opacity,

))

fig.show()
```

```python

```
