# Virtual Assets Extension Specification

- **Title:** Virtual Assets
- **Identifier:** <https://stac-extensions.github.io/virtual-assets/v1.0.0/schema.json>
- **Field Name Prefix:** vrt
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot

This document explains the Virtual Assets Extension to the
[SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

The **virtual assets** is an extension for STAC that allows a virtual STAC asset to be composed from other
STAC [assets](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#asset-object) with repositioning,
and algorithms potentially applied as well as various kinds of metadata altered or added.

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

| Field Name         | Type                                                | Description                                                                                                                                                                               |
| ------------------ | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| vrt:hrefs          | \[[ref object](#object-referencing-using-vrthrefs)] | **REQUIRED**. Array of objects referencing the objects composing the virtual asset. Order is important as it describes the composition index (e.g. RGB composition with `red`, `green` and `blue` asset |
| vrt:algorithm      | \[string]                                           | Algorithm identifier to apply to the virtual asset to compose                                                                                                                             |
| vrt:algorithm_opts | object                                              | any object representing the options for the algorithm                                                                                                                                     |

### Object referencing using `vrt:hrefs`

| Field Name | Type   | Description                                                                                                                                                                                                                                                                                                                    |
| ---------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| key        | string | Identifier of the part in the virtuasl asset. Used in other fields to reference the part.                                                                                                                                                                                                                                      |
| href       | string | Relative and absolute URI pointing to the STAC item object using [JSON pointers](https://www.rfc-editor.org/rfc/rfc6901). Each Uri **MUST** contain the [fragment component](https://www.ietf.org/rfc/rfc3986.html#section-3.5) to identify the object in the stac object. The fragment only preceded by `#` char identify an object of the current Item or Collection. ) |

The [fragment component](https://datatracker.ietf.org/doc/html/rfc3986#section-3.5) of the URI (after the `#`) is used
to identify the object composing the virtual asset.
The pointed object can be any object of the STAC Item or Collection but would most likely be an asset or a band of an asset.
URIs defined in the href arrays may reference several location according to the type of URI notation used.
Here are the accepted URI types and their location resolution

- `#/assets/B04` : Asset with key `B04` in the current STAC Item.
- `#/assets/data/bands/0` : Asset with key `data` and band index 0.
- `./item-sentinel2.json#/assets/B04` : Asset with key `B04` in the STAC Item with the relative URL `./item-sentinel2.json`.
- `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#/assets/B04` : Asset
with key `B04` in the STAC Item with the absolute URL `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json`.

> \[!IMPORTANT]
> the `bands` pointer under an asset would be valid only for STAC core version >= 1.1 but it can be interpreted as the band index
> in the asset for version < 1.1 or in the items where it is not present.

## Positioning

The positioning of the source objects is defined by their position in the `vrt:hrefs` array.
Typically, in the case of the composition of a RGB image, the first pointer would be the red band,
the second the green band and the third the blue band.

```json
"vrt:hrefs": [
  { "key": "red", "href": "#/assets/B04" },
  { "key": "green", "href": "#/assets/B03" },
  { "key": "blue", "href": "#/assets/B02" }
]
```

## Mandatory role

Every asset defined with `vrt:hrefs` field **MUST** declare the `"virtual"` asset role
in the [`roles` field](https://github.com/radiantearth/stac-spec/blob/master/best-practices.md#asset-roles) array.

## Other Asset fields

### `href` field

The `href` link field in the [asset object](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#asset-object)
MUST be set to the self link of the asset.
This is the combination of the item `self` link and the asset json pointer in the fragment.

`https://raw.githubusercontent.com/stac-extensions/virtual-assets/main/examples/item-sentinel2.json#/assets/virtual`

### `type` field

The `type` field in the [asset object](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#asset-object)
MUST be set to the expected format of the virtual asset.
Typically, a raster virtual asset composed by several geotiff bands would have the type `image/tiff; application=geotiff`.

## Use Cases

### Simple RGB raster composition

A very simple case would be the composition of a RGB natural color image of a
[Sentinel-2 item](https://github.com/stac-extensions/raster/blob/main/examples/item-sentinel2.json).

```json
"assets": {
  "trc": {
    "roles": [ "data", "virtual" ],
    "title": "Sentinel-2 Natural Color",
    "type": "image/tiff; application=geotiff",
    "href": "https://raw.githubusercontent.com/stac-extensions/virtual-assets/main/examples/item-sentinel2.json",
    "vrt:hrefs": [ 
      { "key": "red", "href": "https://raw.githubusercontent.com/stac-extensions/virtual-assets/main/examples/item-sentinel2.json#/assets/B04" },
      { "key": "green", "href": "https://raw.githubusercontent.com/stac-extensions/virtual-assets/main/examples/item-sentinel2.json#/assets/B03" },
      { "key": "blue", "href": "https://raw.githubusercontent.com/stac-extensions/virtual-assets/main/examples/item-sentinel2.json#/assets/B02" }
    ]
  }
}
```

### Spectral index use case: NDVI

The following example describes a virtual asset `NDVI` with the Normalized Difference Vegetation Index computed from 2 other bands.
The `vrt:algorithm` field is specifying an algorithm name to apply to generate the virtual asset : `band_arithmetic` that
process band pixels values according to the `expression` defined in the `vrt:algorithm_opts` object.
Data value is also rescaled in the range `[-1,1]` using the `rescale` option.

```json
"assets":{
  "ndvi": 
  {
    "title": "Normalized Difference Vegetation Index",
    "roles": [ "virtual", "data", "index" ],
    "type": "image/tiff; application=geotiff",
    "vrt:hrefs": [
      { "key": "B04", "href": "#/assets/B04"}, 
      { "key": "B05", "href": "#/assets/B05"}],
    "vrt:algorithm": "band_arithmetic",
    "vrt:algorithm_opts": {
      "expression": "(B05–B04)/(B05+B04)",
      "rescale": [[-1,1]]
    }
  }
}
```

### File Archive

The [Collection example](examples/collection.json) describe an archive virtual asset composed of the 2 other assets of the collection.
An implementation for this collection could propose an additional download button that would package the assets in an archive
and return it to the user.

```json
"assets": {
  "archive": {
    "title": "Archive",
    "roles": [ "virtual", "data", "archive" ],
    "type": "application/zip",
    "vrt:hrefs": [
      { "key": "B04", "href": "#/assets/B04" },
      { "key": "B05", "href": "#/assets/B05" }
    ]
  }
}
```

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
