# Layer 4: Smart Devices

Layer 4 contains the specifications for blocks that represent functionality specific to using a flow player on a smart device (e.g. smart phone, tablet).
Support for this layer should be implemented by all engines that target the `offline` channel. These blocks may make use of the [Expression Specification](../expressions.md) for generating output.

Namespace: `SmartDevices`

## Contents
- [Location Response Block](#location-response-block)
- [Photo Response Block](#photo-response-block)


## Location Response (GPS) Block

- Type: `SmartDevices\LocationResponse`
- Suggested number of exits: 1 + error exit
- Supported channels: `offline`

### Detailed Behaviour
This block first requests the device's in-built GPS capability to return the current GPS location of the device. If the device is not GPS capable, or its GPS capability is disabled, the block presents the user with a map on which to choose their location.

### Block `config`
TBD

## Photo Response Block

- Type: `SmartDevices\PhotoResponse`
- Suggested number of exits: 1 + error exit
- Supported channels: `offline`

### Detailed Behaviour
This block first attempts to activate the camera on the device and prompts the user to take a picture, or select a saved picture on the device. If this succeeds the picture is stored as a file upload with all available metadata.

### Block `config`
TBD
