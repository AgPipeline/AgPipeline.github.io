# Algorithm
This is the name given to the portion of a transformer that provides the analysis and/or transformation of data.

Please read the [Transformers](https://agpipeline.github.io/transformers/transformers) overview documentation for some additional context.

In this document we will be providing an conceptual overview of the Algorithm code structure and its implementation framework for the UA makeflow solution.

## Overview
The use of the term Algorithm appears to be nebulous in that implementations can range from processing raw sensor data to performing time based analysis of derived data.
The unifying principle for Algorithm implementation is the transformation of data.
For example, the Algorithm implementations for RGB processing in the AgPipeline includes masking soil from the images, clipping images to plot boundaries, and calculating canopy cover from the plot-clipped image data.

By standardizing a minimal set of required file names and function signatures with return values, Algorithm implementations can easily be used in different environments.
Outside of satisfying these requirements, an Algorithm implementation is free to become as complicated as necessary to do its work.

As implemented in the AgPipeline makeflow environment, Algorithm provide the following hooks:
1. Add parameters for command line processing (optional)
2. A check to determine if the runtime environment is suitable for processing a request (optional)
3. Process the data, saving files to the workspace, and return a result

At a minimum, an instance of the environment, provided by the Environment class, is passed in when checking the runtime environment and when processing data.
For the AgPipeline [Environment class implementation](https://github.com/AgPipeline/agpypeline/blob/master/agpypeline/environment.py), additional data is also provided in the form of curated metadata and transformer metadata.

## AgPipeline Implementation
Algorithm code is expected to be derived from the [Algorithm](https://github.com/AgPipeline/agpypeline/blob/master/agpypeline/algorithm.py) class defined in the [AgPypeline library](https://github.com/AgPipeline).
The one mandatory function ([perform_process](#perform_process) is to be defined by the derived class, otherwise an exception is raised.

There is one mandatory function that needs to be defined, and several other optional functions, for an Algorithm implementation.
Outside of these pre-defined [function signatures](https://developer.mozilla.org/en-US/docs/Glossary/Signature/Function) Algorithm implementations are free to add whatever functions, files, data, API calls, etc. that's necessary for their processing.

**Mandatory** functions: <a name="perform_process" />
- perform_process(self, environment: Environment, check_md: dict, transformer_md: dict, full_md: list): implements the algorithm

**Optional** functions:
- add_parameters(parser: argparse.ArgumentParser): allows an Algorithm to specify additional command line parameters, or otherwise manipulate the command line parameter parser
- check_continue(self, environment: Environment, check_md: dict, transformer_md: dict, full_md: list): gives the Algorithm code a chance to determine if the environment is sufficient for processing; all the correct files available, for example

Parameter descriptions:
- parser: an instance of [argparse.ArgumentParser](https://docs.python.org/3/library/argparse.html) used to parse command line parameters
- environment: an instance of the Environment class
- check_md: request specific metadata as defined by the Environment class implementation
- transformer_md: previously returned Transformer-specific metadata (if available)
- full_md: a list of all loaded metadata for this request
