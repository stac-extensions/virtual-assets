# Virtual Assets Extension Specification

- **Title:** Virtual Assets
- **Identifier:** <https://stac-extensions.github.io/virtual-assets/v1.0.0/schema.json>
- **Field Name Prefix:** virtual
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot

This document explains the Virtual Assets Extension to the
[SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
The virtual assets extensions is an extension for STAC Item and Collection that allows a virtual assets to be composed
from *physical* assets (actual remote files) or virtual assets. It describes repositioning,
and algorithms potentially applied as well as various kinds of metadata altered or added.

- Examples:
  - [Item example](examples/item-sentinel2.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Item fields

| Field Name           | Type                      | Description |
| -------------------- | ------------------------- | ----------- |
| virtual:assets       | [Virtual Asset Object](#virtual-asset-object) | **REQUIRED**. Dictionary of virtual asset objects that can be composed, each with a unique key. |

The `virtual:assets` field is at the root level of the item as per the `assets` field.

### Virtual Asset Object

the virtual asset object is an extension of the core
[asset object](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#asset-object).
It inherits of all its fields (`href`, `title`, `description`, `type`, `roles`) except  that is replaced by `hrefs`.

| Field Name  | Type      | Description |
| ----------- | --------- | ----------- |
| hrefs       | \[string]    | **REQUIRED.** array of URIs to the assets object composing the virtual asset. Relative and absolute URI are both allowed. Each Uri **MUST** contain the [fragment component](https://www.ietf.org/rfc/rfc3986.html#section-3.5) to identify the asset key. The fragement only preceded by `#` char identify an asset of the current Item or Collection. |
| title       | string    | The displayed title for clients and users. |
| description | string    | A description of the Asset providing additional details, such as how it was processed or created. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| type        | string    | [Media type](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#asset-media-type) of the asset. See the [common media types](https://github.com/radiantearth/stac-spec/blob/master/best-practices.md#common-media-types-in-stac) in the best practice doc for commonly used asset types. |
| roles       | \[string] | The [semantic roles](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#asset-roles) of the asset, similar to the use of `rel` in links. |

#### Asset cross referencing using href

URIs defined in the href arrays may reference several location according to the type of URI notation used. 
Here are the accepted URI types and their location resolution

- `#B04` : Asset with key `B04` in the current STAC Item.
- `./item-sentinel2.json#B04` : Asset with key `B04` in the STAC Item with the relative URL `./item-sentinel2.json`.
- `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B04` : Asset
with key `B04` in the STAC Item with the absolute URL `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json`.

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
