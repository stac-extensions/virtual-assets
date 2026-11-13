# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [v1.0.1] - 2026-11-13

### Changed

- Extension maturity is now Pilot (was Proposal) ([#2](https://github.com/stac-extensions/virtual-assets/issues/2))
- The JSON Schema now enforces the [mandatory role](README.md#mandatory-role) rule on assets:
  an asset with `vrt:hrefs` must have the `virtual` role, and an asset with the `virtual` role must have `vrt:hrefs`.
  The rule does not apply to Item Asset Definitions (`item_assets`).

### Fixed

- `href` in `vrt:hrefs` uses `anyOf` instead of `oneOf` in the JSON Schema,
  so validation does not depend on the `iri-reference` format implementation ([#5](https://github.com/stac-extensions/virtual-assets/issues/5))
- The JSON Schema did not validate the role rule, because the `if` condition had no `then` clause
- The type of `vrt:algorithm` in the README is `string`, as in the JSON Schema

## [v1.0.0] - 2023-11-09

Initial release of the Virtual Assets Extension.

[Unreleased]: <https://github.com/stac-extensions/virtual-assets/compare/v1.0.1...HEAD>
[v1.0.1]: <https://github.com/stac-extensions/virtual-assets/compare/v1.0.0...v1.0.1>
[v1.0.0]: <https://github.com/stac-extensions/virtual-assets/tree/v1.0.0>
