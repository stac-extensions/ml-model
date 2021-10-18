# ML Model Extension Specification

- **Title:** ML Model
- **Identifier:** <https://stac-extensions.github.io/ml-model/v1.0.0/schema.json>
- **Field Name Prefix:** ml-model
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @duckontheweb

This document explains the ML Model Extension to the [SpatioTemporal Asset
Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification. More details to come...

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Scope & Vision

The goal of the STAC ML Model Extension is to provide a way of cataloging machine learning (ML) models that operate on Earth observation (EO) data
described as a STAC catalog. The metadata related to machine learning models and their related artifacts (e.g. training data, performance  metrics,
etc.) can be extremely broad. This extension limits its scope to ML model metadata that aids in the discoverability and usability/reusability of
these models for the following types of use-cases:

- **Adoption of Models in Analytic Pipelines**

    Individuals and organizations hoping to use ML model predictions into their own analytic pipelines need a way of discovering models that will
    work for a given geographic area, application domain, and type of input data and that produce a specific kind of output (object detection v.
    classification). Consider the example of a global non-profit organization that wants to use ML to track deforestation. A data engineer from this
    organization might be interested in discovering segmentation models that accurately produce land cover classes over parts of South America using
    Sentinel 2 imagery. The STAC ML Model Extension aims to support this use-case by describing metadata related to the recommended area over which
    the model may be used, a description of the model architecture and type of input data it requires, and links to containerized model images or
    model files that can be used to run the model to generate inferences.

- **Re-training of Existing Models in New Contexts**

    The process of training ML models on Earth observation data can be extremely time-consuming and costly due to the volume of data required.
    Providing tools that ease the discovery of existing models and training data will make ML models more accessible by reducing this training
    burden. Suppose the non-profit from the previous example found a model that generated the kind of predictions they were interested in, but was
    not applicable to their region of interest. Rather than creating a new model from scratch, the organization might be interested in using transfer
    learning to re-train the existing model on data from their area of interest. In this case, they would need enough information about the training
    environment and model architecture to reproduce the model weights and continue training the model using new data. The STAC ML Model Extensions
    aims to support this use-case by providing links to serialized versions of the model (e.g. a PyTorch checkpoint file) as well as enough detail
    about the training environment that a data scientist could reasonably implement transfer learning using new data.

- **Reproducibility of ML Experiments**

    The ability to reproduce published ML experiments is crucial for verifying and building upon previous ML research. Increasingly, individuals and
    institutions are making an effort to publish code and examples along with academic publications to enable this kind of reproducibility. However,
    the quality and usability of this code and related documentation can vary widely and there are currently no standards that ensure that a new
    researcher could reproduce a given set of published results from the documentation. The STAC ML Model Extension aims to address this issue by
    providing a detailed description of the training data and environment used in a ML model experiment. 

## Item Properties

| Field Name                 | Type                      | Description |
| -------------------------- | ------------------------- | ----------- |
| ml-model:learning_approach | string                    | **REQUIRED**. The learning approach used to train the model. It is STRONGLY RECOMMENDED that you use one of the values described below, but other values are allowed. |
| ml-model:prediction_type   | string                    | **REQUIRED.** The type of prediction that the model makes. It is STRONGLY RECOMMENDED that you use one of the values described below, but other values are allowed.   |
| ml-model:architecture      | string                    | **REQUIRED.** Identifies the architecture employed by the model (e.g. RCNN, U-Net, etc.). This may be any string identifier, but publishers are encouraged to use well-known identifiers whenever possible. |

### Additional Field Information

#### ml-model:learning_approach

Describes the learning approach used to train the model. It is STRONGLY RECOMMENDED that you use one of the 
following values, but other values are allowed.

- `"supervised"`
- `"unsupervised"`
- `"semi-supervised"`
- `"reinforcement-learning"`

#### ml-model:prediction_type

Describes the type of predictions made by the model. It is STRONGLY RECOMMENDED that you use one of the 
following values, but other values are allowed. Note that not all Prediction Type values are valid
for a given [Learning Approach].

- `"object-detection"`
- `"classification"`
- `"segmentation"`
- `"regression"`

## Interpretation of STAC Fields

The semantics of ML model metadata can sometimes differ significantly from the use-cases for which STAC was originally intended (Earth observation data). We feel that the benefits of structuring this metadata as a STAC Extensions outweigh the possible downsides, but it does require us to be specific about how certain STAC fields should be interpreted. The following definitions clarify the meaning of core fields from the STAC spec; for any fields not specifically defined here, please refer to the core STAC spec.

### Spatiotemporal Fields

| Field Name     | Type                      | Description                                                                                      |
|----------------|---------------------------|--------------------------------------------------------------------------------------------------|
| geometry       | [GeoJSON Geometry Object] | The geographic area over which the model was trained.                                            |
| start_datetime | string                    | The first or start date and time for the images that the model was trained on.                   |
| end_datetime   | string                    | The last or end date and time for the images that the model was trained on.                      |
| datetime       | string                    | *In general this should not be used, since a date range will almost always be more appropriate.* |

### Licensing

All licensing fields and links should refer to licensing for the *model itself* and not training data or other artifacts associated with the model.
See the [STAC Licensing] section for details on those fields.

### Other Common Metadata

All other fields defined in the [STAC Common Metadata] documentation should be interpreted as referring to the imagery used to train the model.

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type                | Description |
| ------------------- | ----------- |
| TBD                 | More detail to come... |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

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

[Learning Approach]: <#ml-modellearning_approach>
[GeoJSON Geometry Object]: <https://tools.ietf.org/html/rfc7946#section-3.1>
[STAC Date and Time Range]: <https://github.com/radiantearth/stac-spec/blob/master/item-spec/common-metadata.md#date-and-time-range>
[STAC Licensing]: <https://github.com/radiantearth/stac-spec/blob/master/item-spec/common-metadata.md#licensing>
[STAC Common Metadata]: <https://github.com/radiantearth/stac-spec/blob/master/item-spec/common-metadata.md>