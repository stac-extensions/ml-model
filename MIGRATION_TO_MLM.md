# Migration Guide: ML Model Extension to MLM Extension

## Context

The ML Model Extension was started at Radiant Earth on October 4th, 2021. It was possibly the first STAC extension dedicated to describing machine learning models. The extension incorporated inputs from 9 different organizations and was used to describe models in Radiant Earth's MLHub API. The announcement of this extension and its use in Radiant Earth's MLHub is described [here](https://medium.com/radiant-earth-insights/geospatial-models-now-available-in-radiant-mlhub-a41eb795d7d7). Radiant Earth's MLHub API and Python SDK are now [deprecated](https://mlhub.earth/?gad_source=1&gclid=CjwKCAjwk8e1BhALEiwAc8MHiBZ1JcpErgQXlna7FsB3dd-mlPpMF-jpLQJolBgtYLDOeH2k-cxxLRoCEqQQAvD_BwE). In order to support other current users of the ML Model extension, this document lays out a migration path to convert metadata to the Machine Learning Model Extension (MLM).

## Shared Goals

Both the ML Model Extension and the Machine Learning Model (MLM) Extension aim to provide a standard way to catalog machine learning (ML) models that work with, but are not limited to, Earth observation (EO) data. Their main goals are:

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

Notable differences:

- The MLM Extension covers more details at both the Item and Asset levels, making it easier to describe and use model metadata.
- The MLM Extension covers Runtime requirements within the [Container Asset](https://github.com/crim-ca/mlm-extension?tab=readme-ov-file#container-asset), while the ML Model Extension records [similar information](./README.md#inferencetraining-runtimes) in the `ml-model:inference-runtime` or `ml-model:training-runtime` asset roles.
- The MLM extension has a corresponding Python library, [`stac-model`](https://pypi.org/project/stac-model/) which can be used to create and validate MLM metadata. An example of the library in action is [here](https://github.com/crim-ca/mlm-extension/blob/main/stac_model/examples.py#L14). The ML Model extension does not support this and requires the JSON to be written manually by interpreting the JSON Schema or existing examples.
- MLM is easier to maintain and enhance in a fast moving ML ecosystem thanks to it's use of pydantic models, while still being compatible with pystac for extension and STAc core validation.

## Changes in Field Names

### Item Properties

| ML Model Extension                 | MLM Extension      | Notes                                                                                                                                                                                                                                                                                                                                                     |
| ---------------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ml-model:type`                    | N/A                | No direct equivalent, it is implied by the `mlm` prefix in MLM fields and directly specified by the schema identifier.                                                                                                                                                                                                                                    |
| `ml-model:learning_approach`       | `mlm:tasks`        | Removed in favor of specifying specific `mlm:tasks`.                                                                                                                                                                                                                                                                                                      |
| `ml-model:prediction_type`         | `mlm:tasks`        | `mlm:tasks` provides a more comprehensive enum of prediction types.                                                                                                                                                                                                                                                                                       |
| `ml-model:architecture`            | `mlm:architecture` | The MLM provides specific guidance on using *Papers With Code - Computer Vision* identifiers for model architectures. No guidance is provided in ML Model.                                                                                                                                                                                                  |
| `ml-model:training-processor-type` | `mlm:accelerator`  | MLM defines more choices for accelerators in an enum and specifies that this is the accelerator for inference. ML Model only accepts `cpu` or `gpu` but this isn't sufficient today where we have models optimized for different CPU architectures, CUDA GPUs, Intel GPUs, AMD GPUs, Mac Silicon, and TPUs. |
| `ml-model:training-os`             | N/A                | This field is no longer recommended in the MLM for training or inference; instead, users can specify an optional `mlm:training-runtime` asset.                                                                                                                                                                                                            |


### New Fields in MLM

| Field Name                       | Description                                                                                                             |
|----------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| **`mlm:name`**                   | A required name for the model.                                                                                          |
| **`mlm:framework`**              | The framework used to train the model.                                                                                  |
| **`mlm:framework_version`**      | The version of the framework. Useful in case a container runtime asset is not specified or if the consumer of the MLM wants to run the model outside of a container. |
| **`mlm:memory_size`**            | The in-memory size of the model.                                                                                        |
| **`mlm:total_parameters`**       | Total number of model parameters.                                                                                       |
| **`mlm:pretrained`**             | Indicates if the model is derived from a pretrained model.                                                              |
| **`mlm:pretrained_source`**      | Source of the pretrained model by name or URL if it is less well known.                                                 |
| **`mlm:batch_size_suggestion`**  | Suggested batch size for the given accelerator.                                                                         |
| **`mlm:accelerator_constrained`**| Indicates if the model requires a specific accelerator.                                                                 |
| **`mlm:accelerator_summary`**    | Description of the accelerator. This might contain details on the exact accelerator version (TPUv4 vs TPUv5) and their configuration. |
| **`mlm:accelerator_count`**      | Minimum number of accelerator instances required.                                                                       |
| **`mlm:input`**                  | Describes the model's input shape, dtype, and normalization and resize transformations.                                 |
| **`mlm:output`**                 | Describes the model's output shape and dtype.                                                                           |
| **`mlm:hyperparameters`**        | Additional hyperparameters relevant to the model.                                                                       |

### Asset Objects

| ML Model Extension Role      | MLM Extension Role      | Notes                                                                                              |
| ---------------------------- | ----------------------- | -------------------------------------------------------------------------------------------------- |
| `ml-model:inference-runtime` | `mlm:inference-runtime` | Direct conversion; same role and function.                                                         |
| `ml-model:training-runtime`  | `mlm:training-runtime`  | Direct conversion; same role and function.                                                         |
| `ml-model:checkpoint`        | `mlm:checkpoint`        | Direct conversion; same role and function.                                                         |
| N/A                          | `mlm:model`             | New required role for model assets in MLM. This represents the asset that is loaded for inference. |
| N/A                          | `mlm:source_code`       | Recommended for providing source code details.                                                     |
| N/A                          | `mlm:container`         | Recommended for containerized environments.                                                        |
| N/A                          | `mlm:training`          | Recommended for training pipelines.                                                                |
| N/A                          | `mlm:inference`         | Recommended for inference pipelines.                                                               |


The MLM is focused on search, discovery descriptions, and reproducibility of inference. Nevertheless, the MLM provides a recommended asset role for `mlm:training-runtime` and asset `mlm:training`, which can point to a container URL that has the training runtime requirements. The ML Model extension specifies a field for `ml-model:training-runtime` but like `mlm:training` it only contains the default STAC Asset fields and additional fields specified by the Container Asset. Training requirements typically differ from inference requirements so therefore we recommend that fields and assets for reproducing model training or fine-tuning models be contained in a separate STAC extension.

## Getting Help

If you have any questions about a migration, feel free to contact the maintainers by opening a discussion or issue on the [MLM repository](https://github.com/crim-ca/mlm-extension).

If you see a feature missing in the MLM, feel free to open an issue describing your feature request.