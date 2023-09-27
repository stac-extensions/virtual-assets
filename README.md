# Virtual Assets Extension Specification

- **Title:** Virtual Assets
- **Identifier:** <https://stac-extensions.github.io/virtual-assets/v1.0.0/schema.json>
- **Field Name Prefix:** virtual
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot

This document explains the Virtual Assets Extension to the
[SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
The **virtual assets** extensions is an extension for STAC Item and Collection that allows a virtual assets
to be composed from 1 or more [assets](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#asset-object)
or virtual assets themself. It describes cross references and repositioning. It can be extended by other extensions to define
algorithms applied as well as various kinds of metadata altered or added.

- Examples:
  - [Item example](examples/item-sentinel2.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [ ] Collections
- [ ] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name           | Type      | Description |
| -------------------- | ----------| ----------- |
| virtual:hrefs        | \[string] | **REQUIRED.** array of URIs to the assets object composing the virtual asset. Relative and absolute URI are both allowed. Each Uri **MUST** contain the [fragment component](https://www.ietf.org/rfc/rfc3986.html#section-3.5) to identify the asset key. The fragment only preceded by `#` char identify an asset of the current Item or Collection. Order is important as it describes the composition index (e.g. RGB composition with `red`, `green` and `blue` asset) |

### Asset cross referencing using `hrefs`

URIs defined in the href arrays may reference several location according to the type of URI notation used.
Here are the accepted URI types and their location resolution

- `#B04` : Asset with key `B04` in the current STAC Item.
- `./item-sentinel2.json#B04` : Asset with key `B04` in the STAC Item with the relative URL `./item-sentinel2.json`.
- `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B04` : Asset
with key `B04` in the STAC Item with the absolute URL `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json`.

## Mandatory role

Every asset defined with `virtual:hrefs` field must declare the `"virtual"` asset role in the [`roles` field](https://github.com/radiantearth/stac-spec/blob/master/best-practices.md#asset-roles) array.

## Asset `href` link field

When it is possible, the href link field of the virtual asset should be set to a resource
that can be used to fetch the virtual asset (e.g. composition service).
When not possible, the href link field should be set to the STAC Item or Collection that contains the virtual asset.

## Use Cases

### Compressed Archive

The [Collection example](examples/collection.json) describe an archive virtual asset composed of the 2 other assets of the collection.
An implementation for this collection could propose an additional download button that would package the assets in an archive
and return it to the user.

### Raster Composition

The [Raster composition](https://github.com/stac-extensions/composite#raster-composition-using-virtualassets) use case is described in the
[`composite`](https://github.com/stac-extensions/composite) extension with additional fields.

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
