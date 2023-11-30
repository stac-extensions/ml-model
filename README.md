# ML Model Extension Specification

- **Title:** ML Model
- **Identifier:** <https://stac-extensions.github.io/ml-model/v1.0.0/schema.json>
- **Field Name Prefix:** ml-model
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @rbavery

This document explains the ML Model Extension to the [SpatioTemporal Asset
Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

- Examples:
  - [Item example](examples/dummy/item.json): Shows the basic usage of the extension in a STAC Item
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Scope & Vision

The goal of the STAC ML Model Extension is to provide a way of cataloging machine learning (ML) models that operate on Earth observation (EO) data
described as a STAC catalog. The metadata related to machine learning models and their related artifacts (e.g. training data, performance  metrics, inference requirements
etc.) can be extremely broad. This extension limits its scope to ML model metadata that aids in the discoverability and usability/reusability of
these models for inference (model prediction). The spec also places emphasis on providing model card information for each model, so that users understand the performance and limitations of the model. Therefore, the main use case for this extension is as follows:

- **Adoption of Models in Analytic Pipelines**

    Individuals and organizations hoping to use ML model predictions into their own analytic pipelines need a way of discovering models that will
    work for a given geographic area, application domain, and type of input data and that produce a specific kind of output (object detection v.
    classification). Consider the example of a global non-profit organization that wants to use ML to track deforestation. A data engineer from this
    organization might be interested in discovering segmentation models that accurately produce land cover classes over parts of South America using
    Sentinel 2 imagery. The STAC ML Model Extension aims to support this use-case by describing metadata related to the recommended area over which
    the model may be used, a description of the model architecture and type of input data it requires, and links to model files that can be used to run the model to generate inferences.

## Item Properties

| Field Name                 | Type                      | Description |
| -------------------------- | ------------------------- | ----------- |
| ml-model:type              | string                    | **REQUIRED.** This MUST always be the constant `"ml-model"`. This purpose of this field is to provide a convenient way to filter ML Model Items in a STAC API. |
| ml-model:learning_approach | string                    | **REQUIRED.** The learning approach used to train the model. It is STRONGLY RECOMMENDED that you use one of the values [described below](#ml-modellearning_approach), but other values are allowed. |
| ml-model:prediction_type   | string                    | **REQUIRED.** The type of prediction that the model makes. It is STRONGLY RECOMMENDED that you use one of the values [described below](#ml-modelprediction_type), but other values are allowed.   |
| ml-model:architecture      | string                    | **REQUIRED.** Identifies the architecture employed by the model (e.g. RCNN, U-Net, etc.). This may be any string identifier, but publishers are encouraged to use well-known identifiers whenever possible. |
| ml-model:training-processor-type | string              | The type of processor used during training. Must be one of `"cpu"` or `"gpu"`. |
| ml-model:training-os       | string                    | Identifies the operating system on which the model was trained. See the [description below](#ml-modeltraining-os) for recommended values. |

### Additional Field Information

#### ml-model:learning_approach

Describes the learning approach used to train the model. It is STRONGLY RECOMMENDED that you use one of the
following values, but other values are allowed.

- `"supervised"`
- `"unsupervised"`
- `"semi-supervised"`
- `"self-supervised"`
- `"reinforcement-learning"`

#### ml-model:prediction_type

Describes the type of predictions made by the model. It is STRONGLY RECOMMENDED that you use one of the
following values, but other values are allowed. Note that not all Prediction Type values are valid
for a given [Learning Approach](#ml-modellearning_approach).

- `"object-detection"`
- `"classification"`
- `"segmentation"`
- `"regression"`
- `"multi-modal"`

### ml-model:training-architecture

It is STRONGLY RECOMMENDED that one of the following architecture identifiers, used by Docker and Golang:(https://go.dev/doc/install/source#environment). For example:

- `linux-amd64` - Linux x86-64
- `windows-amd64` - Windows x86-64
- `darwin-arm64` - MacOs Silicon
- `darwin-amd64` - MacOs x86-64

## Asset Objects

### Roles

| Role Name                  | Description |
| -------------------------- | ----------- |
| ml-model:inference-runtime | Represents a file containing instructions for running a containerized version of the model to generate inferences. See the [Inference/Training Runtimes](#inferencetraining-runtimes) section below for details on related fields. |
| ml-model:training-runtime  | Represents a file containing instructions for running a container to train the model. See the [Inference/Training Runtimes](#inferencetraining-runtimes) section below for details on related fields. |
| ml-model:checkpoint        | Represents a PyTorch checkpoint file that can used to load the model (see [official PyTorch documentation for details](https://pytorch.org/tutorials/beginner/saving_loading_models.html)) |

### Inference/Training Runtimes

Assets with the `ml-model:inference-runtime` or `ml-model:training-runtime` role represents files containing instructions for running a containerized
version of the model to either generate inferences or train the model, respectively. Currently, only [Compose
files](https://github.com/compose-spec/compose-spec/blob/master/spec.md#compose-file) are supported, but support is planned for other formats,
including [Common Workflow Language (CWL)](https://www.commonwl.org/) and [Workflow Description Language (WDL)](https://openwdl.org/).

The `"type"` field should be used to indicate the format of this asset. Assets in the Compose format should have a `"type"` value of
`"text/x-yaml; application=compose"`.

While the Compose file defines nearly all of the parameters required to run the containerized model image, we still need a way to define which host
directory containing input data should be mounted to the container and to which host directory the output predictions should be written. The Compose
file MUST define volume mounts for input and output data using the Compose
[Interpolation syntax](https://github.com/compose-spec/compose-spec/blob/master/spec.md#interpolation). The input data volume MUST be defined by an
`INPUT_DATA` variable and the output data volume MUST be defined by an `OUTPUT_DATA` variable.

For example, the following Compose file snippet would mount the host input directory to `/var/data/input` in the container and would mount the host
output data directory to `/var/data/output` in the host container. In this contrived example, the script to run the model takes 2 arguments: the
location to the input data directory and the location to the output data directory.

```yaml
services:
  ...
  model_runtime:
    ...
    volumes:
      - "${INPUT_DATA}:/var/data/input"
      - "${OUTPUT_DATA}:/var/data/output"
    command: "run-model.sh /var/data/input /var/data/output"
```

A user would then set the `INPUT_DATA` and `OUTPUT_DATA` environment variables when running the model. An example using `docker-compose` might look
like the following:

```console
$ INPUT_DATA=/local/path/to/model/inputs; \
  OUTPUT_DATA=/local/path/to/model/outputs; \
  docker-compose up -f path/to/inference-runtime.yml
```

It is RECOMMENDED that model publishers use the Asset `description` field to describe any other requirements or constraints for running the model
container.

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type                         | Description |
| ---------------------------- | ----------- |
| ml-model:inferencing-image   | Links with this relation type refer to Docker images that may be used to generate inferences using the model. The `href` value for links of this type should contain a fully-qualified URI for the image as would be required for a command like `docker pull`. These URIs should be of the form `<registry_domain>/<user_or_organization_name>/<image_name>:<tag>`. Links with this relation type should have a `"type"` value of `"docker-image"` to indicate a Docker image. |
| ml-model:training-image   | Links with this relation type refer to Docker images that may be used to train the model. The `href` value for links of this type should contain a fully-qualified URI for the image as would be required for a command like `docker pull`. These URIs should be of the form `<registry_domain>/<user_or_organization_name>/<image_name>:<tag>`. Links with this relation type should have a `"type"` value of `"docker-image"` to indicate a Docker image. |
| ml-model:train-data          | Links with this relation type refer to datasets used to train the model. It is STRONGLY RECOMMENDED that these links refer to a STAC Collection implementing the [Label Extension](https://github.com/stac-extensions/label) |
| ml-model:test-data           | Links with this relation type refer to datasets used to test the model during training. It is STRONGLY RECOMMENDED that these links refer to a STAC Collection implementing the [Label Extension](https://github.com/stac-extensions/label). |

## Interpretation of STAC Fields

The semantics of ML model metadata can sometimes differ significantly from the use-cases for which STAC was originally intended (Earth observation
data). We feel that the benefits of structuring this metadata as a STAC Extensions outweigh the possible downsides, but it does require us to be
specific about how certain STAC fields should be interpreted. The following definitions clarify the meaning of core fields from the STAC spec; for
any fields not specifically defined here, please refer to the core STAC spec.

### Spatiotemporal Fields

| Field Name     | Type                      | Description                                                                                  |
|----------------|---------------------------|----------------------------------------------------------------------------------------------|
| geometry       | [GeoJSON Geometry Object](https://tools.ietf.org/html/rfc7946#section-3.1) | The geographic area over which the model may be used. Note that this may be the same as the area over which the model was trained, but could also represent additional areas where model performance has been tested or where the model publisher believes it will perform well based on similarities to the training environment.     |
| start_datetime | string                    | The first or start date and time for images that should be used to generate inferences using the model. |
| end_datetime   | string                    | The last or end date and time for images that should be used to generate inferences using the model. To represent an open interval (e.g. imagery from 2021-01-01T00:00:00Z or later), use the maximum value (`"9999-12-31T23:59:59Z"`) for `end_datetime`. |
| datetime       | string                    | *This should always be `null`, since a date range (using `start_datetime` and `end_datetime`) will almost always be more appropriate.* |

All other fields defined in the [STAC Common Metadata](https://github.com/radiantearth/stac-spec/blob/master/item-spec/common-metadata.md)
documentation should be interpreted as referring to imagery that may be used for running the model to generate inferences.

## Usage with Other STAC Extensions

It is RECOMMENDED that following STAC Extensions be used in conjunction with the ML Model STAC Extension to fully describe geospatial ML models:

- [Scientific Citation Extension](https://github.com/stac-extensions/scientific): This extension should be used to describe how the model should
  cited in publications, as well as to reference any existing publications associated with the model.

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Contributing Examples & Tutorials

All community members are encouraged to contributes their own examples of cataloged ML models. If you have a model that you have cataloged using this
extension, please open a PR to include it in the `examples` directory. Here are some guidelines for contributing example catalogs:

- New examples should go in their own sub-directory under the `examples` directory (e.g. `examples/my-new-model`)
- All links and assets referenced in the catalog must be publicly available
- Include any supplementary files (model checkpoint files, etc.) that are not served publicly in the same directory as the model catalog and
  reference these using relative links.
- If possible, please include a Collection (even if it only contains a single Item) with any relevant summaries.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid.
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
