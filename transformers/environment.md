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

Given the context-sensitive nature of an Environment Type, the handlers for these tasks are implemented with a class named 'Transformer' in a file named `transformer_class.py` for easy separation.
Once implemented, this allows for the easy customization of run-time environments for any Algorithm Transformer; just copy the `transformer_class.py` file, and any supporting files, over an existing transformer and it's ready for a new environment.

### Transformer class
As mentioned above, this class resides in a file named `transformer_class.py` and is referenced by the Entry Point and Algorithm code of a Transformer.
The [Entry Point]() provides the execution framework of a transformer while the [Algorithm](https://agpipeline.github.io/transformers/algorithm) does the work.
Both of these rely on the Transformer class to provide a suitable and well defined runtime context.

With the AgPipeline, the Entry Point implementation expects the Transformer class to provide a (mandatory) *get_transformer_params()* function that arranges function parameters for algorithms, and to provide some optional functions (optionality is implementation dependent) for defining command line parameters and downloading data.
What this means is that the Transformer class needs to provide the data Algorithm code expects in the format expected.
Additionally, since Algorithms receive an instance of the Transformer class as a parameter, they also expect any additional, necessary, functions to be implemented by the class.
To simplify the development of Transformer classes, the AgPipeline defines a standard interface for all Algorithms; satisfying these interface requirements means that an implementation of a single Transformer class can be used by any of these Algorithms.

## AgPipeline Implementation
The AgPipeline is descended from the [TERRA REF](https://github.com/terraref) project as described in the [Organization Info](https://github.com/AgPipeline/Organization-info) repository and elsewhere.

**Note**: please refer the the [implementation of the Transformer class](https://github.com/AgPipeline/drone-pipeline-environment/blob/master/base-transformer-class/transformer_class.py) for an up-to-date information of what the class provides.

To handle the majority of the Algorithm implementations used to process the data, the following variables, properties, and functions are defined in the AgPipeline's implementation of the [Transformer class](https://github.com/AgPipeline/drone-pipeline-environment/blob/master/base-transformer-class/transformer_class.py) (please refer to the class' implementation as the authoritative source of available variables, properties, and functions).

*variables*:<a name="transformer_env_variables" />
- sensor: the name of the sensor associated with the request (sensor as defined by [TERRA REF project](https://github.com/terraref/terrautils/blob/112d7b6032a677ebcc52868c41bd607e9b0af845/terrautils/sensors.py#L58))
- args: command line arguments for the current request (see [argparse.ArgumentParser.parse_args()](https://docs.python.org/3/library/argparse.html))

*properties*:<a name="transformer_env_properties" />
- default_epsg: this returns the EPSG code for the gantry system data and other data (such as plot boundaries) as an integer
- sensor_name: this returns the value of the [sensor variable](#transformer_env_variables) in the class instance as a string

*functions*:<a name="transformer_env_functions" />
- get_image_file_epsg(path): returns the file's EPSG code as a string
- get_image_file_geobounds(path): returns the file's geographic boundaries as a list of X and Y (check the documentation for exact order of returned values)
- generate_transformer_md(): returns a dictionary containing Transformer attribution fields (name, description, version, author, repository)

Other functions: to support the workflow, the following functions are implemented by the [Transformer class](https://github.com/AgPipeline/drone-pipeline-environment/blob/master/base-transformer-class/transformer_class.py) and should **not** be used elsewhere.

- add_parameters(self, parser: argparse.ArgumentParser): adds parameters and other information for command line processing
- get_transformer_params(self, args: argparse.Namespace, metadata: dict): prepares parameters which are passed on to Algorithm code
