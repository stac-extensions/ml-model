# Migration Guide: ML Model Extension to MLM Extension

<!-- lint disable no-undefined-references -->

>[!IMPORTANT]
> For specific field migration details from [ML-Model](README.md) to [Machine Learning Model (MLM)][mlm] please refer
> to the [MLM Migration Document](https://github.com/stac-extensions/mlm/blob/main/docs/legacy/ml-model.md).

<!-- lint enable no-undefined-references -->

## Context

The ML Model Extension was started at Radiant Earth on October 4th, 2021.
It was possibly the first STAC extension dedicated to describing machine learning models.
The extension incorporated inputs from 9 different organizations and was used to describe models
in Radiant Earth's MLHub API. The announcement of this extension and its use in Radiant Earth's MLHub
is described [here](https://medium.com/radiant-earth-insights/geospatial-models-now-available-in-radiant-mlhub-a41eb795d7d7).
Radiant Earth's MLHub API and Python SDK are now [deprecated](https://mlhub.earth/?gad_source=1&gclid=CjwKCAjwk8e1BhALEiwAc8MHiBZ1JcpErgQXlna7FsB3dd-mlPpMF-jpLQJolBgtYLDOeH2k-cxxLRoCEqQQAvD_BwE).
In order to support other current users of the ML Model extension, this document lays out a migration path to convert
metadata to the [Machine Learning Model Extension (MLM)][mlm].

## Shared Goals

Both the ML Model Extension and the [Machine Learning Model (MLM)][mlm] extension aim to provide a standard way to
catalog machine learning (ML) models that work with, but are not limited to, Earth observation (EO) data.

Their main goals are:

1. **Search and Discovery**: Helping users find and use ML models.
2. **Describing Inference and Training Requirements**: Making it easier to run these models by describing input requirements and outputs.
3. **Reproducibility**: Providing runtime information and links to assets so that model inference is reproducible.

## Schema Changes

### ML Model Extension
- **Scope**: Item, Collection
- **Field Name Prefix**: `ml-model`
- **Key Sections**:
  - Item Properties
  - Asset Objects
  - Inference/Training Runtimes
  - Relation Types
  - Interpretation of STAC Fields

### MLM Extension
- **Scope**: Collection, Item, Asset, Links
- **Field Name Prefix**: `mlm`
- **Key Sections**:
  - Item Properties and Collection Fields
  - Asset Objects
  - Relation Types
  - Model Input/Output Objects
  - Best Practices

### Notable Differences

- The MLM Extension covers more details at both the Item and Asset levels, making it easier to describe and use model metadata.
- The MLM Extension covers more runtime requirements using distinct asset roles.
- The MLM extension has better integration with the STAC Extensions and Python ecosystem.

## Getting Help

If you have any questions about a migration, feel free to contact the maintainers by opening a discussion or issue 
on the [MLM repository][mlm].

If you see a feature missing in the MLM, feel free to open an issue describing your feature request.

[mlm]: https://github.com/stac-extensions/mlm
