# Layer 4: Smart Devices

Layer 4 contains the specifications for blocks that represent functionality specific to using a flow player on a smart device (e.g. smart phone, tablet).
Support for this layer should be implemented by all engines that target the `smart-device` channel. These blocks may make use of the [Expression Specification](../expressions.md) for generating output.

Namespace: `SmartDevices`

## Select One Block

Uses the [Layer 3 Select One Block](https://github.com/FLOIP/flow-spec/blob/s3/mobile-primitives/layers/layer3/blocks.md#select-one-block) with the additional fields below.

## Number in Range Block

(TODO: does this go at a higher level?)

- Type: `SmartDevices\Number`
- Number of exits: 1
- Supported channels: `smart-device`, `ivr`, `simple-text`, `rich-messaging`, `offline`

Key | Description
--- | ---
`prompt` (resource) | The question prompt that should be displayed to the contact, e.g. "What is your age? Enter a number greater than 8."
`range-min` (optional) | If included, the entered number must be greater than or equal to this value.
`range-max` (optional) | If included, the entered number must be less than or equal to this value.

## Text Block

(TODO: does this go at a higher level?)

- Type: `SmartDevices\Text`
- Number of exits: 1
- Supported channels: `smart-device`, `ivr`, `simple-text`, `rich-messaging`, `offline`

Key | Description
--- | ---
`prompt` (resource) | The question prompt that should be displayed to the contact, e.g. "How are you feeling?"


## GPS Block (to implement in MVP2)

- Type: `SmartDevices\GPS`
- Number of exits: 1
- Supported channels: `smart-device`

This block first requests the device's in-built GPS capability to return the current GPS location of the device. If the device is not GPS capable, or its GPS capability is disabled, the block presents the user with a map on which to choose their location.

TBD

## Photo Block (to implement in MVP2)

- Type: `SmartDevices\Photo`
- Number of exits: 1
- Supported channels: `smart-device`

This block first attempts to activate the camera on the device and prompts the user to take a picture. If this succeeds the picture is stored as a file upload with all available metadata. If this does not succeed it provides the user with the option

TBD
