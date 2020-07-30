# Environmental
This is the name given to the portion of a transformer that standardizes the runtime environment.

Please read the [Transformers](https://agpipeline.github.io/transformers/transformers) overview documentation for some additional context.

In this document we will be providing an conceptual overview of the Environmental code structure and then uses the implementation of the UA makeflow solution as a practical example.

## Overview
As mentioned above, the intention of an Environment Type is to provide an standardized runtime environment.
This allows the algorithms to become simplified since they don't need to bother with the details of what their actual environment is.

As implemented in the AgPipeline makeflow environment, the flow of control through a transformer can support both a messaging system that triggers the transformer, or running directly from a command line.
To this end, the Environment Transformer provides hooks to perform the following tasks:
1. Add parameters for command line processing (optional)
2. Perform downloads of any data that's to be made available to the algorithm (optional)
3. Provide a well defined set of parameters to the algorithm

Given the context-sensitive nature of an Environment Type, the handlers for these tasks are implemented with a class named 'Environment' in a file named [environment.py](https://github.com/AgPipeline/agpypeline/blob/master/agpypeline/environment.py) in the [AgPypeline library](https://github.com/AgPipeline/agpypeline) for easy separation.

### Environment class
As mentioned above, this class resides in a file named `environment.py` and is referenced by the Entry Point and Algorithm code (as a class instance) of a Transformer.
The [Entry Point]() provides the execution framework of a transformer while the [Algorithm](https://agpipeline.github.io/transformers/algorithm) does the work.
Both of these rely on the Environment class to provide a suitable and well defined runtime context.

With the AgPipeline, the Entry Point implementation expects the Environment class to provide a (mandatory) *get_transformer_params()* function that arranges function parameters for algorithms, and to provide some optional functions (implementation dependent) for defining command line parameters and downloading data.
What this means is that the Environment class needs to provide the data Algorithm code expects in the format expected.
Additionally, since Algorithms receive an instance of the Environment class as a parameter, they also expect any additional, necessary, functions to be implemented by the class.
To simplify the development of Environment classes, the AgPipeline defines a standard interface for all Algorithms; satisfying these interface requirements means that an implementation of a single Environment class can be used by any of these Algorithms.

## AgPipeline Implementation
The AgPipeline is descended from the [TERRA REF](https://github.com/terraref) project as described in the [Organization Info](https://github.com/AgPipeline/Organization-info) repository and elsewhere.

**Note**: please refer the the [implementation of the Environment class](https://github.com/AgPipeline/agpypeline/blob/master/agpypeline/environment.py) for an up-to-date information of what the class provides.

To handle the majority of the Algorithm implementations used to process the data, the following variables, properties, and functions are defined in the AgPipeline's implementation of the Environment class (see above Note).

*variables*:<a name="transformer_env_variables" />
- args: command line arguments for the current request (see [argparse.ArgumentParser.parse_args()](https://docs.python.org/3/library/argparse.html))

*functions*:<a name="transformer_env_functions" />
- generate_transformer_md(): returns a dictionary containing Transformer attribution fields (name, description, version, author, repository based upon Configuration information)

Other functions: to support the workflow, the following functions are implemented by the [Transformer class](https://github.com/AgPipeline/drone-pipeline-environment/blob/master/base-transformer-class/transformer_class.py) and should **not** be used elsewhere.

- add_parameters(self, parser: argparse.ArgumentParser): adds parameters and other information for command line processing
- get_transformer_params(self, args: argparse.Namespace, metadata: dict): prepares parameters which are passed on to Algorithm code
