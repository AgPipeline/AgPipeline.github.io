# Transformers
This document provides information on the design of transformers as well as providing details on how we've implemented that design.

Each transformer is constructed by combining different conceptual Types, each which corresponds to distinct files and specific functional [signatures](https://developer.mozilla.org/en-US/docs/Glossary/Signature/Function). 
After the [Overview section](#overview) below, different Types are presented in their [own section](#type-details) starting with the Entry Point Type first.

This documentation emphasises Docker usage of the code with the expectation that the concepts can be transferred to non-Docker contexts.
In other words, while we make extensive use of Docker, it's not required.

## Overview <a name="overview" />
Each transformer is the combination of several conceptual Types that are implemented as specifically-named files containing specifically-named-and-parameterized functions (specific function [signatures](https://developer.mozilla.org/en-US/docs/Glossary/Signature/Function)).
The main motivations for this approach are:
1. To separate common code from mutable code as much as possible. For example, converting a `.bin` file to a georeferenced `.tif` image file doesn't appear to have much in common with creating an othomosaic of the georeferenced image files. However it's only the specifics of the data transformation that are different, not the flow of control and the runtime environment.
2. Reduce the cognitive overhead needed to produce a working transformer. By providing a consistent workflow and environment, developers can focus on the task at hand with minimal book keeping knowledge with a reduction of required tasks.
3. Provide stable transformers quickly. Using the same code for the workflow and the request environment means that mistakes are reduced and testing can be more robust. For example, developers of [Algorithms](#algorithm) can focus on their piece of a transformer knowing that workflow and environmental concerns are already taken care of.
4. Flexibility of runtime environments. By isolating the code that works with a specific runtime environment, the ability to "migrate" a transformer to a different environment is amplified; implement code for your environment, combine it with the entry point and algorithm code, and a new transformer is created for your runtime environment. Next you can combine the proven environment with any other algorithms you need to more easily create your workflow using existing pieces. 

The image below shows a graphical overview of how the transformers are organised conceptually and as repositories or libraries.
Each of the columns with a thicker border on the right can be where a transformer is considered complete.
(In the diagram below, these are the `Algorithm` and `Plot-level` columns.)

<img src="https://github.com/AgPipeline/AgPipeline.github.io/raw/24532912772b0969a2ae255911d415a59bb41126/resources/Transformer Hierarchy Diagram.png" width="800" />

As can be seen in the above image, the minimal set of Types needed for a complete transformer are Entry Point, Environmental, and Algorithm.
This can be extended to be more specific by adding Plot Level.
Additionally, if the runtime environment requires it, a Transformer can also have an Override Entry Point to perform initialization tasks.

The explanation of the left-most tabs on the above image are:
* Type - the generic type of code concept/repository each column represents
* Info - high level intent statement and one or more example repositories in the [AgPipeline GitHub Organization](https://github.com/AgPipeline)
* Details - contains information on expected file names, provided functions/methods, and file dependencies
* More - additional information that doesn't fit in the above rows

The top-most tabs represent a conceptual grouping of the columns they cover. The explanation for these tabs are:
* Optional Entry point - this Type is used when the environment a transformer is running in has requirements that can't be met with the code base implementing the Types. For example, establishing a connection and fetching metadata needed by downstream code
* One complete transformer - the minimal set of Types that needs to be implemented to fulfill the requirements for a transformer
* Science transformer - to facilitate the development of Scientific algorithms and their incorporation into workflows, it's desirable to have specialized transformer templates that are geared towards one set of data conditions; [plot-level RGB data](https://github.com/AgPipeline/template-rgb-plot) for example 

While in most cases code repositories can correspond to one of the columns in the diagram and build upon each other (moving left to right), there are special cases where a repository contains more than one of the shown columns.
This is typically due to special cases where the default workflow isn't sufficient for the task.
An example of this can be found in the [OpenDroneMap repository](https://github.com/AgPipeline/transformer-opendronemap).
This repository keeps the naming conventions and function signatures, but has its own implementation quirks. 

## Transformer environmental expectations
Transformers have a few runtime environment expectations that are important to note:
1. workspace (or scratch space): a disk location where files can be created, deleted, or modified as needed
2. request metadata: the expectation is that metadata of some sort is provided that defines the scientific environment that it's running in and information related to the current request
3. transformer metadata: for recurrent runs, transformer specific metadata returned from any previous requests
4. resulting metadata: transformer returned metadata specific to the current request will be correctly handled by the calling process (stored, moved, ignored, etc)
5. cleanup: the calling process will cleanup the environment used by the transformer based upon its knowledge of the request and the results returned by the transformer (cleaning up the workspace, for example)

## Type Details <a name="type-details" />
This section provides additional information on each of the Types in the diagram shown in the [Overview](#overview) section above.
Each of the Type subsections provided here has information on its intent along with other concepts related to that Type.

### Entry Point <a name="entry-point" />
As the name implies, this is considered the entry point to a transformer.
The purpose of the entry point is to provide a common flow of control that each transformer can utilize, and a basic set of command line parameters common to all derived transformers.

While the entry point defines the flow of control through a transformer, it doesn't have an expectation for the data the [Environmental](#environment) or [Algorithm](#algorithm) Types receive.
For example, the default `Entry Point` in the AgPipeline [base-docker-support](https://github.com/AgPipeline/base-docker-support) implementation uses a dict to allow each implementation of the `Environment` Type to define its set of parameters to pass to the `Algorithm` implementation.
The contents of this dict is defined by the Environment and Algorithm code, and not by the Entry Point code.

At times it may be necessary to create a custom entry point for transformers and the AgPipeline [base-docker-support](https://github.com/AgPipeline/base-docker-support) implementation is designed to handle that case.
The AgPipeline implementation is not only designed to be flexible but to provide a consistent experience for developers, maintainers, and anyone else.
Please refer to the documentation (TBD) for detailed information on the implementation of this Type.

The implementation of this Type can then be built into a library or Docker image which can subsequently be used by other Types.

Note that in some cases a preamble needs to happen before the main work can be done.
For example, configuring and waiting on a message queue.
Refer to the [Override Entry Point](#override) documentation for more information on preambles.

### Environmental <a name="environmemt" />
The purpose of the Environmental Type is to interpret the context that a transformer is running in and to provide a consistent interface into that context on a per-request basis.
This is accomplished logistically by encapsulating the context in a class and passing an instance of that class to the [Algorithm](#algorithm) Type.

What this means in practical terms is that transformers for different environments can use the same [Entry Point](#entry-point) and [Algorithm](#algorithm) code.
In the AgPipeline code base, this is accomplished by creating different implementations of the Transformer class.
For example, the common_image folder in the [ua-gantry-environment](https://github.com/uacic/ua-gantry-environment/tree/master/common-image) repository, or the base_transformer_class folder in the [drone-pipeline-environment](https://github.com/AgPipeline/drone-pipeline-environment/tree/master/base-transformer-class) repository.

The [drone-pipeline-environment](https://github.com/AgPipeline/drone-pipeline-environment/tree/master/base-transformer-class) implementation of the Transformer class uses the metadata and working space specified on the command line to obtain its request environment, initialize its instance, provide a standard set of data for the Algorithm, and specifies a standard set of Algorithm parameters.

This frees the [Algorithm](#algorithm) code from having any specialized knowledge of its execution environment. 

Please refer to the [Environmental documentation](https://agpipeline.github.io/transformers/environment) for detailed information on an implementation of this Type.

### Algorithm <a name="algorithm" />
Now that the workflow is defined by the [Entry Point](#entry-point) Type and the environment is standardized by the [Environmental](#environment) Type, the algorithm can focus on processing the data it receives.

Algorithms can be everything from [scrubbing and standardizing metadata](https://github.com/uacic/transformer-cleanmetadata) to [calculating canopy cover](https://github.com/AgPipeline/transformer-canopycover) on a plot-level basis.
When combined with the workflow implemented by the [Entry Point](#entry-point) Type and a specific [Environmental](#environment) Type, one has a complete Transformer.

Continuing with the theme of making the creation of transformers as simple as possible, this Type can be specialized to work on a very specific set of data.
For the AgPipeline organization, this means [plot-level](plot-level) Algorithms for RGB, Lidar, and other data products.
Each of the implementations for these plot-level datum is represented by a distinct Algorithm Type that handles the details of looking for and loading data, understanding the common environment, and other tasks (such as knowing the plot name).
The writer of Scientific algorithms that work on the plot level only needs to process the actual data and produce the results.

Please refer to the [Algorithm documentation](https://agpipeline.github.io/transformers/algorithm) for detailed information on this Type.

### Plot-level <a name="plot-level" />
This type significantly reduces the overhead associated with creating a transformer.
By providing information on what the transformer does and some scientific context, along with the analysis results, a complete Transformer can be created for different environments with minimal work.

The intent here is to remove as much context from the implementation of the analysis as possible, by providing it elsewhere through the Types listed above.
This will allow the analysis developer to focus on implementing their piece of the transformer as narrowly as possible, massively reducing the specialized knowledge necessary for developing a full transformer.

By providing a simple solution to writing analysis code and providing a common means of testing, a solution can be developed with confidence that the solution has met the minimum requirements and should work as expected.

Please refer to the [plot-level RGB documentation](https://agpipeline.github.io/transformers/template_rgb_plot) for detailed information on this Type as implemented for RGB data.

### Override Entry Point <a name="override" />
This Type is for the situations where the existing flow of control and Type (Environmental, Algorithm, etc.) implementations are sufficient, but can't provide the *completely* correct environment the transformer needs to be running in.
For example, needing to configure and listen on a message queue before running the rest of the transformer.

There aren't any specifics behind this Type and the implementation details are up to the developers.

What is expected in the Docker environment is that the `ENTRYPOINT` for the image(s) will be changed to point to a new location.
One way to create the Docker image might be to use a finished image as the basis for your images, and use the Dockerfile to copy your code to the new image wile also specify the new entry point.
