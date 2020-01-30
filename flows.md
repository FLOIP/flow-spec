# Flow Fundamentals

Flows represent a collection of actions \("Blocks"\) and the decision-making logic that links Blocks together into a flowchart-like description of an interactive mobile service, business process, or anything else that can be modelled as programmatic flow-chart.

## Contents

* [Format](flows.md#format)
* [Data Formats](flows.md#format)
  * [Resources](flows.md#resources)
  * [UUID Format](flows.md#uuid-format)
* [Top-level Specification Elements](flows.md#top-level-specification-elements)
  * [Containers](flows.md#containers)
  * [Flows](flows.md#flows)
  * [Blocks](flows.md#blocks)

## Format

Containers and Flow Definitions are a set of nested key-value objects formatted in [JSON, version ECMA-404](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf).

## Data Formats

Throughout the specification, the terms "string", "object", and "array" refer to the corresponding entities in the JSON specification. An "expression" is a string conforming to the [Expressions Specification](https://github.com/floip/flow-specification/tree/7a09ac6d0cd28370fd159bce33d69f61c8eb4c30/expressions.md).

### Resources

A Resource describes a collection of localized strings or media resources, used when content needs to be presented to Contacts in multiple languages. Resources have the following following structure:

```text
  "text":{
    [iso-639-3 language code]:[localized text],
    [iso-639-3 language code]:[localized text],
    ...
    [iso-639-3 language code]:[localized text],
  },
  "audio":{
    [iso-639-3 language code]:[media ID],
    [iso-639-3 language code]:[media ID],
    ...
    [iso-639-3 language code]:[media ID],
  }
}
```

for instance,

```text
"6681c54c-9fcd-11e7-abc4-cec278b6b50a":{
  "text":{
    "eng":"How are you?",
    "aka":"Wo ho te sɛn?",
    "dag":"A mal’ alaafee?"
  },
  "audio":{
    "eng":"5718eb16.wav",
    "aka":"5f6e4694.wav",
    "dag":"68bd67f2.wav"
  }
}
```

[ISO 639-3](http://www-01.sil.org/iso639-3/) codes are used for language identifiers, to support all current and historically spoken human languages. A resource may include only `text`, only `audio`, or both, depending on the place within the specification where it is used. For example, resources providing localized expressions need only provide `text`.

### UUID Format

The term "uuid" refers to a universally unique identifier. Implementations may use any UUID version from [RFC4122](https://tools.ietf.org/html/rfc4122) or [Version 6 IDs](https://bradleypeabody.github.io/uuidv6/). Platforms are recommended to use version 6 IDs for performance and compatibility. The hyphenated string representation of the UUID must be used within JSON documents, for instance:

```text
"uuid":"2b375764-9fcc-11e7-abc4-cec278b6b50a"
```

### Media IDs

TODO

## Top-level Specification Elements

### Containers

A Container is a "package" document containing one or more Flow Definitions, useful for sharing multiple Flows in a single document. The required keys for a Container object are:

| Key | Description |
| :--- | :--- |
| `specification_version` \(string\) | The version of the Flow Spec that this package is compliant with. |
| `uuid` \(64 bit integer TODO\) | A globally unique identifier for this Container. FLOIP uses \[TODO format of uuids\] |
| `name` \(string\) | A human-readable name for the Container content. |
| `description` \(string\) | An extended human-readable description of the content. |
| `platform_metadata` \(object\) | A set of key-value elements that is not controlled by the Specification, but could be relevant to a specific Platform. |
| `flows` \(array\) | A list of the Flows within the Container \(see below\) |
| `resources` \(object\) | A set of the Resources needed for executing the Flows in the Container, keyed by resource uuid. |

#### Example

```text
TODO
```

### Flows

A Flow represents a set of Blocks and their direct connections. The required keys for a Flow are:

| Key | Description |
| :--- | :--- |
| `uuid` \(uuid\) | A globally unique identifier for this Flow. \(See [UUID Format](flows.md#uuid-format).\) |
| `name` \(string\) | A human-readable name for the Flow content |
| `label` \(string, optional\) | An extended user-provided description for the flow. |
| `last_modified` \(timestamp, UTC\) | The time when this flow was last modified, in UTC, with microsecond precision: "2016-12-25 13:42:05.234598" |
| `interaction_timeout` \(integer\) | The number of seconds of inactivity after which Contact input for this flow is no longer accepted, and Runs in progress are terminated |
| `platform_metadata` \(object\) | A set of key-value elements that is not controlled by the Specification, but could be relevant to a specific Platform. |
| `supported_modes` \(array\) | A list of the supported Modes that the Flow has content suitable for. \(See below\) |
| `languages` \(array\) | A list of the languages that the Flow has suitable content for, expressed as ISO 639-3 codes. The first language in the list is considered the default/primary language. Example: `["eng","aka","dag"]` |
| `blocks` \(array\) | A list of the Blocks in the flow \(see below\).  The flow will start execution at the _first_ block in this list. |

Supported modes include:

* `text`: general text-based interactions. This includes SMS and USSD channels, which may have distinct behaviour while sharing the same content.
  * `sms`: content specific for SMS
  * `ussd`: content specific for USSD
* `ivr`: content specific for interactive voice response
* `rich_messaging`: content used for messaging platforms, for example:
  * `twitter`
  * `facebook_messenger`
  * `wechat`
  * `telegram`

#### Example

```text
TODO
```

### Blocks

The required keys for a Block are:

| Key | Description |
| :--- | :--- |
| `uuid` \(uuid\) | A globally unique identifier for this Block \(See [UUID Format](flows.md#uuid-format).\) |
| `name` \(string, word-characters only\) | A human-readable "variable name" for this block. This must be restricted to word characters so that it can be used as a variable name in expressions. When blocks write results output, they write to a variable corresponding the `name` of the block. |
| `label` \(string, optional\) | A human-readable free-form description for this Block. |
| `semantic_label` \(string, optional\) | A user-controlled field that can be used to label the meaning of the data collected by this block, e.g.: an ICD10 category name or other semantic classification system. \("ICD10::gender"\) |
| `type` \(string\) | A specific string designating the type or "subclass" of this Block. This must be one of the Block type names within the specification, such as `Core\RunFlow` or `MobilePrimitives\Message`. |
| `config` \(object\) | Additional parameters that are specific to the type of the block. Details are provided within the Block documentation. |
| `exits` \(array\) | a list of all the exits for the block. Exits must contain the required keys below, and can contain additional keys based on the Block type |

Each exit must contain:

| Key | Description |
| :--- | :--- |
| `uuid` \(uuid\) | A globally unique identifier for this Exit |
| `label` \(resource\) | This is the human-readable name of the exit \(as a translated resource\), which might be presented to a contact. |
| `tag` \(string, word characters only\) | This is an identifier for the exit, suitable for use as a variable name in rolling up results \(e.g.: "male"\). It does not need to be unique within the block; multiple exits may be tagged the same. \(Some authoring tools may choose to auto-generate the tag from the label's primary language, to avoid usability problems with these getting out of sync.\) |
| `destination_block` \(uuid\) | This is the uuid of the Block this exit connects to. It can be null if the exit does not connect to a block \(if it is the final block\). |
| `semantic_label` \(string, optional\) | A user-controlled field that can be used to label the meaning of the exit, e.g.: an ICD10 category name or other semantic classification system. \("ICD10::female"\) |
| `test` \(expression, optional\) | For blocks that evaluate conditions, this is an expression that determines whether this exit will be selected as the path out of the block. The first exit with an expression that evaluates to a "truthy" value will be chosen. |
| `config` \(object\) | This contains additional information required for each mode supported by the block. Details are provided within the Block documentation |

#### Example

```text
TODO
```

