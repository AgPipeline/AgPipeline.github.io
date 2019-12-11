# RGB Plot Transformer - Technical information
This document describes the goals of this Transformer Template, and how the repository is structured.
Also, information on the contents and intent of each of the executable and dependency files is provided (dependency files are files that upstream components are dependent upon).

## Goals
The fundamental goal of this repository is to make it as easy as possible for plant science researchers to implement their algorithms for incorporation into processing pipelines.
To that end, we provide a standard interface, and request minimal configuration on the part of the researcher/developer.

We also provide a testing mechanism that can be used to prove the algorithm implementation before incorporating it into a processing pipeline.
This testing mechanism can also be used to track down issues arising in an active processing pipeline.

## Files
This section provides technical information on the contents of the files found in the [Template RGB Plot repository](https://github.com/AgPipeline/template-rgb-plot).

This section is not intended to provide information on all of the files in the repository, or to replace or document what is implemented in the code.
There are files in the repository that conform to convention, such as '.gitignore' and are documented elsewhere.
It is expected that the contents of the files are sufficiently documented so that external documentation isn't needed; this is not to say that there isn't external documentation generated from the source files, only that it's not to be found here.

### algorithm_rgb.py <a name="algorithm_rgb" />
This is the main file for the algorithm implementation.
Its intent is to keep everything that is specific to the algorithm in one place; to serve as the main entry point.

However, for more complicated algorithms, it may be necessary to have additional files that support the algorithm.

Note that the default Dockerfile that is [generated](#generate) copies all `.py` files, but not any other file types.

## generate.py <a name="generate" />
This executable python script creates the Dockerfile based upon the contents of the [algorithm_rgb.py](#algorithm_rgb) file.
Additionally empty `requirements.txt` and `packages.txt` files are generated.
These last two can be used to install Python modules and system packages needed by the algorithm into a Docker image.

The raw Dockerfile contents are stored as a list of strings in this file.
These strings are modified as needed when the Dockerfile is written.
The decision to use this approach, instead of using template files, was made to reduce the footprint of the repository, and to keep the Docker related information in one spot.

The script, when run, first checks that expected top-level variables are defined in the algorithm_rgb.py file and that some of these variables are not empty.
Depending upon what the intent of a variable is, it's possible to get away with keeping them empty in the algorithm_rgb.py file.

Once the script has verified the environment, it create the empty files and creates a Dockerfile, all of which can be used to build a Docker image.

The script is not intended to run silently.
It will print which stage its on and any problems or concerns it comes across.

## testing.py <a name="testing" />
The intent behind this executable script it to allow the algorithm developer to test their implementation in a way that provides a high degree of certainty that it will succeed in a processing pipeline.
This script is not intended to replicate the environment of a processing pipeline. 

This script takes a list of folders and/or files, loads their content, calls into the algorithm, and writes out the name of each file and the calculated values, one per line in CSV format.
This output is preceded with a CSV header line.

When run with no parameters, the script prints out usage information.

When run with files specified, the arguments (file names) are checked and a check is made that needed variable are defined in the algorithm file.
Once the checks are passed, the CSV header is printed.
Each file is then loaded sequentially, and a call is made to the algorithm passing in the contents of the file.
The name of each file and its returned values are printed in CSV format after the algorithm returns.
