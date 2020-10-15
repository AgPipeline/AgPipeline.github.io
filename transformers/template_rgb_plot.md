# RGB Plot Transformer - Technical information

This document contain information on what the goals of this Transformer are and how the repository is structured.
Also, information on the contents and intent of each of the executable and dependency files is provided (dependency files are files that upstream components are dependent upon).

## Goals

The fundamental goal of this repository is to make it as easy as possible for plant science researchers to implement their algorithms for incorporation into processing pipelines.
To that end, we provide a standard interface, and request minimal configuration on the part of the researcher/developer.

We also provide a testing mechanism that can be used to prove the algorithm implementation before incorporating it into a processing pipeline.
This testing mechanism can also be used to track down issues arising in an active processing pipeline.

## Files

This section provides technical information on the contents of the file found in the [repository](https://github.com/AgPipeline/template-rgb-plot).

This section is not intended to provide information on all the files in the repository, or to replace or document what is implemented in the code.
There are files in the repository that conform to convention, such as '.gitignore' and are documented elsewhere.
It is expected that the contents of the files are sufficiently documented so that external documentation isn't needed; this is not to say that there isn't external documentation generated from the source files, only that it's not to be found here.

### algorithm_rgb.py <a name="algorithm_rgb" />

This is the main file for the algorithm implementation.
Its intent is to keep everything that is specific to the algorithm in one place; to serve as the main entry point.

However, for more complicated algorithms, it may be necessary to have additional files that support the algorithm.
This is anticipated and acceptable.

Note that the default Dockerfile that is [generated](#generate) copies all `.py` files, but not any other file types.
The generated Dockerfile can be modified as needed to copy, or otherwise change, the resulting docker image.
Note that re-generating the Dockerfile will cause any changes to be overwritten.

### calculate() function

The _calculate()_ function gets called upon to calculate the values from a loaded image.
This function may get called multiple times; once for each image loaded.
It receives one parameter which is a [numpy.ndarray](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html) containing the loaded image.

#### Alpha Channel <a name="alpha_channel" />

Depending upon the image, there may be one or more alpha channels associated with the image defining transparency.
There are many purposes for an alpha channel, including indicating which pixels are fully transparent and not important.
This section will focus on the pixels that have an associated alpha channel value of 0 (zero) indicating they are fully transparent.

**Plot Image with Alpha Channel** \
![alpha_channel](https://user-images.githubusercontent.com/45463434/94734883-b8d80880-031e-11eb-87ec-5509cc2665d1.png)

**Plot Image with Alpha Channel Removed** \
![no_alpha_channel](https://user-images.githubusercontent.com/45463434/94734957-db6a2180-031e-11eb-9d88-647fbf40a86a.png)

As can be seen in the images above, there are many pixels that are hidden by the alpha channel in the first image which are visible in the second image.
When calculating values based upon the plot boundaries it's important to **not** include pixels that are masked out (via the alpha channel) since they are considered not relevant, or [NoData](https://desktop.arcgis.com/en/arcmap/10.3/manage-data/raster-and-images/nodata-in-raster-datasets.htm).

Also note that there are black portions in the first image that represents areas where _soil has been masked out_, but are still within the plot boundaries.
While it may appear that disregarding any pixels that are black might be a good approach to ignoring masked areas, it's not that simple; the pixels behind masked areas can be any color.
The examples above use the color black for the pixels behind the mask only by chance (well, not really by chance, but it could have been any color).

Fortunately `numpy` has the ability to work with masked arrays using [numpy.ma](https://numpy.org/doc/stable/reference/maskedarray.html).
The following code shows how to create a masked array of the red channel for an image and then calculate the average red value for all visible pixels:
```python
import numpy as np
# This example assumes an RGBA image
# Convert alpha channel to numpy.ma acceptable format
alpha_mask = np.where(img_array[:, :, 3] == 0, 1, 0)
# Convert the red channel of the image to an numpy.ma array which masks out pixels we don't want
channel_masked = np.ma.array(img_array[:, :, 0], mask=alpha_mask)
# Get the average red value of the pixels for all unmasked pixels
avg_red = np.ma.average(channel_masked)
```
It's possible to create a masked array using the other channels (green and blue) as well as with the entire image.

Once you have a masked array, you can use the numpy masked array functions in `numpy.ma` to perform calculations on the non-masked portions of the image while ignoring the masked pixels.

A more manual approach in determining if a pixel is masked or not is to check the pixel's location in the alpha channel's for a value of 0 (zero).
Any value other than zero means that the pixel is visible, however faintly.
If an alpha channel value of zero is found, skip that pixel in any calculations.
For example:
```python
# Get the channels we want
red_channel = img_array[:, :, 0]
alpha_channel = img_array[:, :, 3]
# Initialize variables
total_red_pixels = 0
sum_red_pixels = 0
# Loop through the image pixels
for x in range(img_array.size[0]):
    for y in range(img_array.size[1]):
        # Check that the pixel is not masked out
        if alpha_channel[x][y] != 0:
            # Accumulate totals
            total_red_pixels += 1
            sum_red_pixels = red_channel[x][y]
# Get the average red value of the pixels for all unmasked pixels
avg_red = sum_red_pixels / total_red_pixels
```

## generate.py <a name="generate" />

This executable python script creates the Dockerfile based upon the contents of the [algorithm_rgb.py](#algorithm_rgb) file.
Additionally empty `requirements.txt` and `packages.txt` files are generated.
These last two can be used to install Python modules and system packages needed by the algorithm into a Docker image.

The raw Dockerfile contents are stored as a list of strings in this file.
These strings are modified as needed when writing Dockerfile.
The decision to use this approach, instead of using template files, was made to reduce the footprint of the repository, and to keep the Docker related information in one spot.

The script, when run, first checks that expected top-level variables are defined in the algorithm_rgb.py file and that some of these variables are not empty.
Depending upon what the intent of a variable is, it's possible to get away with keeping them empty in the algorithm_rgb.py file.

Once the script has verified the environment, it create the empty files (described above) and creates a Dockerfile, all of which can be used to build a Docker image.
The generated files can be modified as needed to install packages or otherwise change the resulting docker image.

**Note:** re-running this script will cause any changes to be overwritten.

The script is not intended to run silently.
It will print which stage its on and any problems or concerns it comes across.

## testing.py <a name="testing" />

The intent behind this executable script is to test the algorithm implementation in a way that provides a high degree of certainty that it will succeed in a processing pipeline.
This script is not intended to replicate the environment of a processing pipeline. 

This script takes a list of folders and/or files, loads their content, calls into the algorithm, and writes out the name of each file and the calculated values, one per line in CSV format.
This output is preceded with a CSV header line.

When run with no parameters, the script prints out usage information.

When run with files specified on the command line, the file names are checked that they exist, and another check is made to ensure that the variable are defined in the algorithm file.
Once the checks are passed, the CSV header is printed.
Each file is then loaded sequentially, and a call is made to the algorithm passing in the contents of that file.
The name of each file and all returned values are printed in CSV format after the algorithm returns.
