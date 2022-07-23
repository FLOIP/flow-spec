# Layer 4: Smart Devices

Layer 4 contains the specifications for blocks that represent functionality specific to using a flow player on a smart device \(e.g. smart phone, tablet\).  
Support for this layer should be implemented by all engines that target the `OFFLINE` channel. These blocks may make use of the [Expression Specification](../expressions.md) for generating output.

Namespace: `SmartDevices`

## Contents

* [Location Response \(GPS\) Block](3-smart-devices.md#location-response-gps-block)
* [Photo Response Block](3-smart-devices.md#photo-response-block)

## Location Response \(GPS\) Block

* Type: `SmartDevices.LocationResponse`
* Suggested number of exits: 1 + default exit (used in case of error or invalid input)
* Supported channels: `OFFLINE`, `RICH_MESSAGING`

This block allows a device user to capture a location on a map, e.g. using a GPS device, or by manual selection.

### Block `config`

| Key | Description |
| :--- | :--- |
| `prompt` \(resource\) | The prompt to be presented to contacts to ask for the location. This is a localized resource; it supports parsing of expressions in rendering. |
| `accuracy_threshold_meters` \(number, optional, default 5.0\) | The GPS resolution that a device should wait for and achieve before capturing this location. |
| `accuracy_timeout_seconds` \(number, optiona, default 120\) | The timeout in seconds that is a maximum that the device should wait for the required accuracy. |

### Detailed Behaviour

* `OFFLINE`: This block first queries the device's in-built GPS capability to return the current GPS location of the device. If the device is not GPS capable, the block presents the user with a map and waits for them to choose their location.  If the device is GPS capable but location services are disabled, the device may prompt the user to enable location services. This block then waits to receive a location via manual selection or from the location service that meets the accuracy threshold.  \(No selection provided or unable to capture the required accuracy: proceed through the default exit.\)
* `RICH_MESSAGING`: This block prompts the Contact for permission to access their location from their device, or choose a location manually, and waits for a compatible response. \(No selection provided or unable to capture the required accuracy: proceed through the default exit.\)

### Output behaviour

This block writes the location coordinates to the output variable corresponding to the `name` of the block. The format of the coordinates is an array of 4 floating point numbers: latitude, longitude, elevation \(in meters\), and accuracy \(in meters\):

```json
[lat, long, elevation, accuracy]
```

e.g.

```json
[52.0835780,-106.6104880, 501.23, 0.5]
```

### Example

```json
TODO
```

## Photo Response Block

* Type: `SmartDevices.PhotoResponse`
* Suggested number of exits: 1 + default exit (used in case of error or invalid input)
* Supported channels: `OFFLINE`

### Block `config`

| Key | Description |
| :--- | :--- |
| `prompt` \(resource\) | The prompt to be presented to contacts to ask for the image. This is a localized resource; it supports parsing of expressions in rendering. |

### Detailed Behaviour

* `OFFLINE`, `RICH_MESSAGING`: This block first prompts the user to activate the camera on the device and waits for the user to take a picture, or select a saved picture on the device. \(No picture selected or captured: proceed through the default exit.\)

### Output behaviour

This block writes the ID of the captured media to the output variable corresponding to the `name` of the block.

