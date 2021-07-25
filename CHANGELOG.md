# Changelog
This changelog documents material changes to the Flow Results specification. Versions of the specification adhere to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0-rc3] - 2021-07-30 (proposed)

### Changed
- SelectOneResponse and SelectManyResponse block types: Introduce a test-based system for mapping responses received via text, IVR, and other channels to choices in a standard way. Restructure choice definitions from a map of `[{choice-name: resources}]` to a structure that is easier to work with in Javascript. (#53)

### Added

- [API Specification](api-specification.md) for publishing, listing, and running Flows on external systems
- Add additional "test" functions for interoperability with RapidPro

## [1.0.0-rc2] - 2020-06-30

### Added

- Added `tags` to blocks for semantic categorization
- Positions (x,y) of Blocks on the canvas added in a standard part of the spec (ui_metadata instead of vendor_metadata)

### Changed

- `exit.tag` renamed to `exit.name` for consistency with Flows and Blocks
- Remove `exit.label` as there is no need for a translated resource of exit names (#55)
