# Entry Point

This is the name given to the portion of a transformer that provides the flow of control through a Transformer and can provide some core common functionality.

Please read the [Transformers](https://agpipeline.github.io/transformers/transformers) overview documentation for some additional context.

In this document we will be providing an conceptual overview of the Entry Point code structure and then use the implementation of the UA makeflow solution as a practical example.

## Overview

The main intention of an Entry Point Type is to provide the flow of control through a Transformer.
By providing a common flow of execution through all Transformers the task of implementing a complete Transformer is simplified.
Developers can focus on [Environment](https://agpipeline.github.io/transformers/environment) and/or [Algorithm](https://agpipeline.github.io/transformers/algorithm) specifics without worrying how to hook them together.

The Entry Point Type can also provide some common, core functionality that is to be available for all Transformers.
This core functionality can be directly implemented in the Entry Point code; such as requiring at least one file to be specific a command line.

The flow of control through Entry Point is not expected to be immutable.
The flow of control can be altered by definitions, function declarations, and other means.

## AgPipeline Implementation

In AgPipeline the file `entrypoint.py` file implements the Entry Point Type.

First an instance of the Environment is created (an instance of `transformer_class`).
Next command line parameters are configured and the command line parameters themselves are parsed.
After this, logging is configured and any metadata specified on the command line is loaded.
The setup of the Transformer is then completed by making calls to the Environment and Algorithm implementations.
Once setup is complete the Algorithm is run to process the data.
Finally, the results from the above steps are formatted and made available, and control is returned to the caller of entrypoint.py.
