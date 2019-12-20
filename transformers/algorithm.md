# Algorithm
This is the name given to the portion of a transformer that provides the analysis and/or transformation of data.

Please read the [Transformers](https://github.com/AgPipeline/AgPipeline.github.io/blob/master/transformers/transformers.md) overview documentation for some additional context.

In this document we will be providing an conceptual overview of the Algorithm code structure and then uses the implementation of the XYZ solution as a practical example.

## Overview
The use of the term Algorithm appears to be nebulous in that implementations can range from processing raw sensor data to performing time based analysis of derived data.
The unifying principle for Algorithm implementation is the manipulation of data.
For example, the Algorithm implementations for RGB processing in the AgPipeline includes converting the *.bin* files to a georeferenced *.tif* file, masking soil from the TIFFs, clipping images to plot boundaries, and calculating canopy cover from the plot-clipped TIFF data.

By standardizing a minimal set of required file names and function signatures with return values, Algorithm implementations can more easily be used in different environments.
Outside of satisfying these requirements, an Algorithm implementation is free to become as complicated as necessary to do its work.

As implemented in the AgPipeline makeflow environment, Algorithm provide the following hooks:
1. Add parameters for command line processing (optional)
2. A check to determine if the runtime environment is suitable for processing a request (optional)
3. Process the data and returning a result

At a minimum, an instance of the environment, provided by the Transformer class, is passed in when checking the runtime environment and when processing data.
For the [AgPipeline Transformers](https://github.com/AgPipeline/ua-gantry-environment/blob/master/common-image/transformer_class.py), additional data is also provided in the form of curated metadata, transformer metadata, and gantry metadata.

## AgPipeline Overview
There is one mandatory function that needs to be defined, and several other optional functions, for an Algorithm implementation.
Outside of these pre-defined function signatures (signatures are: function name, their parameters, and return values) Algorithm implementations are free to add whatever functions, files, data, API calls, etc. that's necessary for their processing.

*functions*:<a name="transformer_algo_functions" />
- 
