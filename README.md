# Template Extension Specification

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

* `"supervised"`
* `"unsupervised"`
* `"semi-supervised"`
* `"reinforcement-learning"`

#### ml-model:prediction_type

Describes the type of predictions made by the model. It is STRONGLY RECOMMENDED that you use one of the 
following values, but other values are allowed. Note that not all Prediction Type values are valid
for a given [Learning Approach].

* `"object-detection"`
* `"classification"`
* `"segmentation"`
* `"regression"`

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
