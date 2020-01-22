# Agriculture Processing Pipeline Documentation
This documentation covers technical information on setting up, configuring, and running the pipeline.
This includes information on creating new transformers that are not templates.

Additional information on our Data Science group can be found on our [University of Arizona site](https://datascience.cals.arizona.edu/) and on [OSF](https://osf.io/emq9s/).

Each template transformer has a `HOW_TO.md` document that provides details on how to use the that template.
Links to technical and HOW TO documentation are provided in the section below.
Note that not all transformers will have HOW TO documentation, only the template ones.

The term `transformer` refers to a algorithm that takes source data files and performs an action on the data.
These actions can be transforming the source data, or calculating one or more values from it.
Transformers can work with RGB, LAS, hyper-spectral, and other data types.

To assist in the development of transformers, `template transformer`s are provided with the intent of lightening the development load of implementing new algorithms.
Template transformer's repository names start with `template`.
Each template transformer has a `HOW_TO.md` document that provides details on how to use the that template.
Links to technical and HOW TO documentation are provided in the section below.
Note that not all transformers will have **HOW TO** documentation, only the template ones.

If you are finding problems with the documentation, or have requests or ideas on how to improve the documentation, please create an [issue](https://github.com/AgPipeline/computing-pipeline/issues/new/choose) so that we can work on it.

# Documentation links
<!-- Please provide links to the documents listed below -->
<!-- Use the repository name in the "Transformer name" column" -->

###Transformers 
Before reading about specific transformers, it is helpful to be familiar with the [conceptual basis](https://agpipeline.github.io/transformers/transformers) for these transformers.
Specifically [Environmental](https://agpipeline.github.io/transformers/environment) and [Algorithm](https://agpipeline.github.io/transformers/algorithm) concepts.

| Transformer name | Technical link | How To link | Comments |
| ---------------- | -------------- | ----------- | -------- |
| [template-rgb-plot](https://github.com/AgPipeline/template-rgb-plot) | [Technical](https://github.com/AgPipeline/AgPipeline.github.io/blob/master/transformers/template_rgb_plot.md) | [How To](https://github.com/AgPipeline/template-rgb-plot/blob/master/HOW_TO.md) | Code template for new Algorithms |
