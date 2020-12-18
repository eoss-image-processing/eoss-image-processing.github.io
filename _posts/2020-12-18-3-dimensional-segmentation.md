---
layout: post
title:  "Annotating 3D imaging data with Dash, scikit-image, and superpixels"
author:	Nicholas Esterer, Emmanuelle Gouillart, and Sebastian Urchs
---
![short animation showing some functions of a superpixel app](/assets/superpixel_app.gif)

A common way to display three-dimensional, volumetric medical imaging data is to take two-dimensional slices along the main axes of the image and display them side by side. In a dash app, we can give the user the ability to interactively control the position of these slices and thus move freely through the volume. This is a good way to quickly and intuitively explore a three-dimensional volume. However, if we want to annotate regions that cross multiple slices, this can be laborious or even impractical, because the annotations will have to be manually repeated in each slice. 

To facilitate such an annotation task, we can segment our volumetric image into superpixels. Superpixels are sets of neighbouring pixels (or voxels in a volume) in a three-dimensional neighbourhood that share similar features - for example image intensity. Annotating superpixels instead of individual pixels makes it much easier to propagate our annotation labels across all three dimensions of the volume. 

Here we have built a dash app that shows a  brain with a brain tumour, obtained by [Magnetic Resonance Imaging](http://www.scholarpedia.org/article/MRI) from the [Brain Tumor Segmentation (BraTS)](http://braintumorsegmentation.org/) Challenge. We first decompose the volumetric image into superpixels with [scikit-image](https://scikit-image.org/), and then use two linked plotly Graphs to display interactive, two-dimensional slices of the volume. You can [explore this app](https://dash-gallery.plotly.host/dash-3d-image-partitioning/) in the [dash app gallery](https://dash-gallery.plotly.host/Portal/) and find [the source code](https://github.com/plotly/dash-sample-apps/tree/master/apps/dash-3d-image-partitioning) in the [dash-sample-apps repository on github](https://github.com/plotly/dash-sample-apps).


## Interactively display volumetric medical imaging data

We load the brain image as a numpy array and slice along the first and second volume axis to create two lists of two-dimensional array slices. We then use the `array_to_data_url` utility function from [Dash canvas](https://dash.plotly.com/canvas) to convert the two lists of 2D numpy array slices into two lists of image data strings. Finally, we store these pre-computed lists in a [Store element](https://dash.plotly.com/dash-core-components/store) inside the app layout. This allows us to store this information in the browser so we don't have to pass them back and forth between the users machine and the server every time we need to access them.

To display the image slices, we create empty plotly Figure objects for each axis (called "Top" and "Side" in the app), set the [`dragmode`](https://plotly.com/python/reference/layout/#layout-dragmode) to `"drawopenpath"` to allow users to draw annotations on the image, and finally add an initial slice to display as background images with [the `add_layout_image` method](https://plotly.com/python/images/). Under each figure we also add a slider element so the user can interactively move through the available image slices. 

When a slider is moved to a new value, this triggers a clientside callback that will access the lists of image slices from the `Store` element, pick the image slice at the index selected by the slider-value, and then update the corresponding figures `layout.Image` parameter with it (i.e. update the currently visible background image). Because the necessary data is stored in a `Store` element and we are using a [`clientside_callback`](https://dash.plotly.com/clientside-callbacks), no data needs to be passed between the user's machine and the server to select a new slice. 

The new dash-slicer Dash component packages the rather complex clientside callback functions into a simple to use component that allows you to achieve the same efficiency with only a few lines of code. Check it out here: https://dash.plotly.com/slicer.


## Identifying and annotating superpixels

We identify superpixels by applying [k-means clustering](https://scikit-image.org/docs/dev/api/skimage.segmentation.html#skimage.segmentation.slic) on our brain image volume to find regions  that share similar intensity values. We also set a minimum intensity threshold, to avoid superpixels spilling out into the background of the image. Finally we [detect the boundaries](https://scikit-image.org/docs/dev/api/skimage.segmentation.html#skimage.segmentation.find_boundaries) between superpixels, and then slice and store the array of detected boundaries in the same way as the brain image volume in a `Store` element. This allows us to display the superpixel boundaries on top of the brain image slices. 

By default the app starts with the `"Show Segmentation"` button clicked and so we draw the corresponding segmentation slice on top of the brain image slice by appending it to the `layout.Image` property list. 

To store the user generated segmentation information, we create a third, empty volume, and also slice and store it in a `Store` element. When the user draws a path across the currently visible image slice, this fires the [`relayoutData` event](https://dash.plotly.com/dash-core-components/graph) and we listen to and extract the drawn path information with callbacks. 

We can now identify all superpixels that the drawn paths touch and use them as a mask to paint ones inside the "generated segmentation" volume. If the `"Show Segmentation"` button is pressed, we add the sliced "generated segmentation" volume as a third background image to the `layout.Image` property, so we can display all three volume slices on top of each other.

## Switching between a 2D and 3D representation of the segmentation

Once the volume has been annotated by the user, it can be helpful to look at the "generated segmentation" as a 3D rendering. For this purpose, the app layout contains a third, 3D Graph object inside a `Div` that is initially hidden with `style={"display": "none"}`. Because the two 2D Graphs are also located inside a `Div`, we can switch between the 2D and 3D view by switching  the `display` CSS attributes of their parent `Div` elements between `"none"` and `""` in another clientside callback that listens to the number of clicks on a button.

## Downloading the annotated data

Once the image has been annotated, it can be interesting for the user to be able to download the generated segmentation. The data are already stored in the user's browser, but to make them downloadable without sending them back to the server, we need to use some tricks. Once the download button is clicked, the "generated segmentation" is converted from a list of slices back to a numpy array, then to the [`NIfTI` file format](https://nifti.nimh.nih.gov/) (commonly used in medical imaging), and the `NIfTI` object is encoded into a base64 string that is stored inside a `Store` element.

```python
def save_found_slices(fstc_slices):
    fstc_slices = fstc_slices[0]
    fstc_ndarray = slice_image_list_to_ndarray(fstc_slices)
    # if the tensor is all zero (no partitions found) return None
    if np.all(fstc_ndarray == 0):
        return None
    # from https://gist.github.com/arokem/423d915e157b659d37f4aded2747d2b3
    fstc_nii = nib.Nifti1Image(skimage.img_as_ubyte(fstc_ndarray), affine=None)
    fstcbytes = io.BytesIO()
    file_map = fstc_nii.make_file_map({"image": fstcbytes, "header": fstcbytes})
    fstc_nii.to_file_map(file_map)
    fstcb64 = base64.b64encode(fstcbytes.getvalue()).decode()
    return fstcb64
```

A clientside callback fires when the base64 string is written to the `Store`, decodes the string into a [`Blob`](https://developer.mozilla.org/en-US/docs/Web/API/Blob) and writes to the `href` property of a hidden `Div` as a "download link". The final step is another clientside callback that fires when the `href` of the hidden `Div` is updated and simulates a "click" by the user on this link. 

```javascript
app.clientside_callback(
    """
function (href) {
    if (href != "") {
        let download_a=document.getElementById("download-link");
        download_a.click();
    }
    return '';
}
""",
    Output("dummy", "children"),
    [Input("download-link", "href")],
)
```

This opens the download interface of the users browser where the blob can be saved to disk.

## What's next

We hope you like this app for volumetric image annotation with superpixels. If you have suggestions, you can submit them via the [dash-sample-apps repository](https://github.com/plotly/dash-sample-apps) or on [Twitter](https://twitter.com/EGouillart). If you have support questions, the best place for them is the Plotly [community forum](https://community.plotly.com/). 

If you want to annotate and explore 3D volumetric image data, you can use this app as a starting 
point and adapt it for your own needs. You might also want to check out the other [image processing apps in the dash app gallery](https://dash-gallery.plotly.host/Portal/?search=[Image%20Processing]). Some additional resources:

- the new [`Dash-Slicer` component](https://github.com/plotly/dash-slicer) that makes it [very easy to build interactive 3D image slicers](https://dash.plotly.com/slicer) in Dash in only a few lines of code
- the new [functionality for image annotation in Dash](https://dash.plotly.com/annotations) and the corresponding [Blogpost](https://eoss-image-processing.github.io/2020/05/06/shape-drawing.html)