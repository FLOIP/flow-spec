# Changelog

This changelog documents material changes to the Flow Results specification. Versions of the specification adhere to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## \[Unreleased]

### Deprecated

- `Console.IO` and `ConsoleUI.Print` had theoretical use-cases that did not materialize. They have been removed.
- Given that the `Console` namespace no longer has blocks, it has been removed.

### Added
* Webhook block type within the Core namespace (`Core.Webhook`), for asynchronous or synchronous requests to external endpoints during Flow runs. [(#24)](https://github.com/FLOIP/flow-spec/issues/24)

### Changed
#### Resources
- Resources have been re-located from the container to the flow to which they apply. Including resources with the flows they belong to is more straightforward for implementers. If a resource needs to be used in multiple places the same URL can be used, but referenced from multiple resources.

#### Response Blocks (`OpenResponse`, `Numeric`, `SelectOne`, `SelectMany`)
- Clarify that prompts are optional (but recommended). This enables legacy conversions where prompts have been sent to the Contact in a previous Message block, and it's not possible to merge the Message block with the Response block. In the case where no prompt exists, the block will wait silently for a response.

#### `MobilePrimitives.OpenResponse`
- `TEXT.max_response_characters` removed. Behaviour for this feature is unclear. Would the message from a user be rejected if it was too long? Would it be truncated? Since most channels have character inherent character limits anyways, suggest to remove this from the spec and allow vendors to configure a limit with vendor metadata if needed.
- `IVR.end_recording_digits` added. These will allow a caller to end a recording, instead of waiting for timeout.

#### `MobilePrimitives.SelectOne` and `MobilePrimitives.SelectMany`
- `IVR.randomize_choice_order` added. This field works in conjunction with pre-existing `question_prompt`, `choices.prompt` and `IVR.digit_prompts` to present choices in random order to reduce response bias.
- Clarification: prompt is optional when question_prompt is provided. On IVR, A prompt doesn’t need to be provided for each choice unless question_prompt is provided.
- Choice tests refactored. IVR and Text test cases are now separate, with text tests defined per-language. 

#### `MobilePrimitives.Numeric`
- Clarification on what the block’s behaviour is when an invalid input is given.
“responses less than this will proceed through the default exit” change to “responses less than this will result in a block value of null.”

#### `Core.SetGroupMembership`
- Allow a list of groups, to add to or remove a Contact from multiple groups at once.
- Add a new action "clear", which removes a Contact from all existing groups without needing to specify them.


## \[1.0.0-rc3] - 2021-07-30

### Changed

* SelectOneResponse and SelectManyResponse block types: Introduce a test-based system for mapping responses received via text, IVR, and other channels to choices in a standard way. Restructure choice definitions from a map of `[{choice-name: resources}]` to a structure that is easier to work with in Javascript. [(#53)](https://github.com/FLOIP/flow-spec/issues/53)
* Remove `config` on exits, as this seems to be unused and unecessary. Replace with `vendor_metadata` on exits. [(#57)](https://github.com/FLOIP/flow-spec/issues/57)

### Added

* [API Specification](https://github.com/FLOIP/flow-spec/tree/680a4430b683a1d01f13bb4d8515e8f69ca5b016/api-specification.md) for publishing, listing, and running Flows on external systems. [(#58)](https://github.com/FLOIP/flow-spec/issues/58)
* Add additional "test" functions for interoperability with RapidPro

## \[1.0.0-rc2] - 2020-06-30

### Added

* Added `tags` to blocks for semantic categorization
* Positions (x,y) of Blocks on the canvas added in a standard part of the spec (ui\_metadata instead of vendor\_metadata)

### Changed

* `exit.tag` renamed to `exit.name` for consistency with Flows and Blocks
* Remove `exit.label` as there is no need for a translated resource of exit names [(#55)](https://github.com/FLOIP/flow-spec/issues/55)
