# Flow Fundamentals

Flows represent a collection of actions \("Blocks"\) and the decision-making logic that links Blocks together into a flowchart-like description of an interactive mobile service, business process, or anything else that can be modeled as programmatic flow-chart.

## Contents

* [Format](flows.md#format)
* [Data Formats](flows.md#format)
  * [Resources](flows.md#resources)
  * [Language Objects and Identifiers](flows.md#language-objects-and-identifiers)
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

A Resource describes a collection of localized strings or media resources, used when content needs to be presented to Contacts in multiple languages. Resources have the following logical structure:

```text
Resource {
  uuid: string,
  values: ResourceValue[]
}

ResourceValue {
  language_id: string (Language Identifier),
  content_type: SupportedContentType,
  mime_type: string,
  modes: SupportedMode[],
  value: string,
}

SupportedContentType {
  TEXT = 'TEXT',
  AUDIO = 'AUDIO',
  IMAGE = 'IMAGE',
  VIDEO = 'VIDEO',
}

SupportedMode {
  TEXT = 'TEXT',
  SMS = 'SMS',
  USSD = 'USSD',
  IVR = 'IVR',
  RICH_MESSAGING = 'RICH_MESSAGING',
  OFFLINE = 'OFFLINE',
}
```

for example,

```text
{
   uuid: "c2dbafbd-e9bd-408f-aabc-25cf67040002",
   values: [
      {
         language_id: "eng",
         modes: ["SMS", "USSD"],
         content_type: "TEXT",
         mime_type: "text/plain",
         value: "Howdy! You've reached the Museum of Interoperability!"
      },
      {
         language_id: "eng",
         modes: ["RICH_MESSAGING"],
         content_type: "TEXT",
         mime_type: "text/plain",
         value: "Howdy! You've reached the Museum of Interoperability! This is a long message for you because we've gone beyond the limitations for 180 characters. I'm your guide, Florian. I hope you're excited for this two hour tour through the history of interoperable data systems."
      },
      {
         language_id: "eng",
         modes: ["RICH_MESSAGING"],
         content_type: "IMAGE",
         mime_type: "image/png",
         value: "https://your-server-somewhere.flowinteroperability.org/example-image.png"
      }
   ]
}
```

A resource may provide one or many values within it. This provides flexibility to use the same text or media for several modes, or to specify unique media or text in each mode. 

The `mime_type` field should be provided for all values; when provided, this must be an IANA media type \(RFC 6838\).

The `language_id` must correspond to a [Language Identifier](flows.md#language-identifiers) described in the Flow's languages.

### Language Objects and Identifiers

Flows describe a list of languages they have content for. Flows use the ISO 639-3 language coding to describe nearly all spoken and written languages. They also support multiple languages with the same ISO 639-3 code \(referred to as language _variants_\), to support use-cases where organizations might have distinct sets of content for the same language. A common example of variants could be "English - India" and "English - East Africa", or "English - Male voice" and "English - Female voice". 

Language objects must have the following keys:

| Key | Description |
| :--- | :--- |
| `id` \(string\) | Language Identifier, described below, e.g. "`eng-female`" |
| `label` \(string, optional\) | Human-readable description for this language and variant. |
| `iso_639_3` \(string\) | [ISO 639-3 code](https://iso639-3.sil.org/code_tables/639/data) for the language. This is a 3-letter string, e.g. "`eng`".  "`mis`" is the ISO 639-3 code for languages not yet included in ISO 639-3. |
| `variant` \(string, optional\) | Where multiple languages/content sets are used with the same ISO 639-3 code, `variant` describes the specialization, e.g. "`east_africa`". |
| `bcp_47` \(string, optional\) | The [BCP 47 ](https://tools.ietf.org/html/bcp47)locale code for this language, e.g. "`en-GB`". These codes are often useful in conjunction with speech synthesis and speech recognition tools.  |

#### Language Identifiers

The _recommended_ structure for Language Identifier strings is:

* &lt;`iso_639_3`&gt;-&lt;`variant`&gt;, where the variant is not null.
* &lt;`iso_639_3`&gt;, when the variant is null.

#### Language Example

```text
languages: [
   {
      id: "eng-male",
      label: "English - Male Voice",
      iso_639_3: "eng",
      variant: "male",
      bcp_47: "en-US"
   },
   {
      id: "eng-female",
      label: "English - Female Voice",
      iso_639_3: "eng",
      variant: "female",
      bcp_47: "en-US"
   },
   {
      id: "mis-mysecretlanguage",
      label: "My secret code language",
      iso_639_3: "mis",
      variant: "mysecretlanguage",
      bcp_47: null
   },
   {
      id: "fre",
      label: "Francais",
      iso_639_3: "fre",
      variant: null,
      bcp_47: fr-FR
   }
]

```

### UUID Format

The term "uuid" refers to a universally unique identifier. Implementations may use any UUID version from [RFC4122](https://tools.ietf.org/html/rfc4122) or [Version 6 IDs](https://bradleypeabody.github.io/uuidv6/). Platforms are recommended to use version 6 IDs for performance and compatibility. The hyphenated string representation of the UUID must be used within JSON documents, for instance:

```text
"uuid":"2b375764-9fcc-11e7-abc4-cec278b6b50a"
```

### Vendor Metadata

To enable extensions of the Specification, entities support the `vendor_metadata` optional field. This is intended for parameters that are not controlled by the Specification, but could be relevant to a specific vendor/platform/implementation. Vendors must use namespaces within `vendor_metadata` to avoid collisions. Suggested namespaces use reverse domain name notation, with periods replaced by underscores (e.g.: A vendor with domain https://my.example-flow-server.com would place entries within `vendor_metadata.com_example-flow-server_my`).

## Top-level Specification Elements

### Containers

A Container is a "package" document containing one or more Flow Definitions, useful for sharing multiple Flows in a single document. The required keys for a Container object are:

| Key | Description |
| :--- | :--- |
| `specification_version` \(string\) | The version of the Flow Spec that this package is compliant with, e.g. `1.0.0-rc1` |
| `uuid` \(uuid\) | A globally unique identifier for this Container. \(See [UUID Format](flows.md#uuid-format).\) |
| `name` \(string\) | A human-readable name for the Container content. |
| `description` \(string\) | An extended human-readable description of the content. |
| `vendor_metadata` \(object, optional\) | A set of key-value elements that is not controlled by the Specification, but could be relevant to a specific vendor/platform/implementation. |
| `flows` \(array\) | A list of the Flows within the Container \(see below\) |
| `resources` \(object\) | A set of the Resources needed for executing the Flows in the Container, keyed by resource uuid. |

#### Example

```text
{
   "specification_version": "1.0.0-rc1",
   "uuid": "95f29456-8a33-4d26-b22a-00b0b169056c",
   "name": "Summary Case Report Test",
   "description": "Summary Case Report Test",
   "vendor_metadata": {},
   "flows": [...],
   "resources": [...]
}
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
| `vendor_metadata` \(object, optional\) | A set of key-value elements that is not controlled by the Specification, but could be relevant to a specific vendor/platform/implementation. |
| `supported_modes` \(array\) | A list of the supported Modes that the Flow has content suitable for. \(See below\) |
| `first_block_id` \(uuid\) | The ID of the block in `blocks` that is at the beginning of the flow. |
| `exit_block_id` \(uuid, optional\) | If provided, the ID of the block in`blocks`that will be jumped to if there is an error or deliberate exit condition during Flow Run. If not provided, the Flow Run will end immediately. |
| `languages` \(array\) | A list of the languages that the Flow has suitable content for. See language object specification below. |
| `blocks` \(array\) | A list of the Blocks in the flow \(see below\).  The flow will start execution at the _first_ block in this list. |

#### Modes

Possible modes for `supported_modes` are:

* `TEXT`: general text-based interactions. This includes SMS and USSD channels, which may have distinct behaviour while sharing the same content.
* `SMS`: content specific for SMS
* `USSD`: content specific for USSD
* `IVR`: content specific for interactive voice response
* `RICH_MESSAGING`: content used for data channels that support multimedia including text, audio, images, and video, such as social network chatbots \(Facebook Messenger, WhatsApp, Twitter, Telegram, etc.\)
* `OFFLINE`: content used for mobile apps designed to run offline without a data connection.

#### Flow Example

```text
{
   "uuid": "06c912aa-0d36-4d9c-b144-0cd3a38e8293",
   "name": "Summary Case Report Test",
   "label": "Summary Case Report Test for mHero Applications",
   "last_modified": "2021-02-05 18:55:59.229Z",
   "interaction_timeout": 172800,
   "first_block_id": "08509fdf-2cd2-43a8-9627-011254d30aa7",
   "supported_modes": [
      "SMS",
      "USSD",
      "RICH_MESSAGING"
   ],
   "languages": [
      {
         "id": "eng",
         "label": "English",
         "iso_639_3": "eng",
         "variant": null,
         "bcp_47": "en-UK"
      }
   ],
   "vendor_metadata": {},
   "blocks": [...]
}
```

### Blocks

The required keys for a Block are:

| Key | Description |
| :--- | :--- |
| `uuid` \(uuid\) | A globally unique identifier for this Block \(See [UUID Format](flows.md#uuid-format).\) |
| `name` \(string, word-characters only\) | A human-readable "variable name" for this block. This must be restricted to word characters so that it can be used as a variable name in expressions. When blocks write results output, they write to a variable corresponding the `name` of the block. |
| `label` \(string, optional\) | A human-readable free-form description for this Block. |
| `semantic_label` \(string, optional\) | A user-controlled field that can be used to code the meaning of the data collected by this block in a standard taxonomy or coding system, e.g.: a FHIR ValueSet, an industry-specific coding system like SNOMED CT, or an organization's internal taxonomy service. \(e.g. "SNOMEDCT::Gender finding"\) |
| `tags` \(array of strings, optional\) | an arbitrary list of strings for categorization of the block's content, meaning, etc. This has a similar purpose to `semantic_label`, but the assumption is that many related blocks might have the same tags. |
| `vendor_metadata` \(object, optional\) | A set of key-value records that is not controlled by the Specification, but could be relevant to a specific vendor/platform/implementation. |
| `ui_metadata` \(object\) | A set of key-value records describing information about how blocks are displayed on a UI/flowchart editor: |
| `ui_metadata.canvas_coordinates.x`, `ui_metadata.canvas_coordinates.y` \(numbers\) | Coordinates indicating location of this block on the Flow Builder's canvas. Origin (0, 0) is top-left. |
| `type` \(string\) | A specific string designating the type or "subclass" of this Block. This must be one of the Block type names within the specification, such as `Core.RunFlow` or `MobilePrimitives.Message`. |
| `config` \(object\) | Additional parameters that are specific to the type of the block. Details are provided within the Block documentation. |
| `exits` \(array\) | a list of all the exits for the block. Exits must contain the required keys below, and can contain additional keys based on the Block type |


#### Exit node specification

Each exit node in `exits` must contain:

| Key | Description |
| :--- | :--- |
| `uuid` \(uuid\) | A globally unique identifier for this Exit |
| `name` \(string\) | This is an identifier for the exit, suitable for use in rolling up results \(e.g.: "male"\),  and to display on flowchart canvases. Expressions can reference the name of the exit taken out of another block via `@(flow.block_name.exit)`. |
| `destination_block` \(uuid\) | This is the uuid of the Block this exit connects to. It can be null if the exit does not connect to a block \(if it is the final block\). |
| `semantic_label` \(string, optional\) | A user-controlled field that can be used to code the meaning of the data collected by this exit in a standard taxonomy or coding system, e.g.: a FHIR ValueSet, an industry-specific coding system like SNOMED CT, or an organization's internal taxonomy service. \(e.g. "SNOMEDCT::Feminine Gender"\) |
| `test` \(expression, optional\) | For blocks that evaluate conditions, this is an expression that determines whether this exit will be selected as the path out of the block. The first exit with an expression that evaluates to a "truthy" value will be chosen. |
| `default` \(boolean, optional\) | If this key is present and true, the exit is treated as the flow-through default in a case evaluation. The block will terminate through this exit if no test expressions in other exits evaluate true..  |
| `config` \(object\) | This contains additional information required for each mode supported by the block. Details are provided within the Block documentation |

Each exit must specify one of either `test` or `default`. Each block must have exactly one `default` exit. The `default` exit is listed last in the list.

#### Setting Contact Properties

A common use-case for platforms that run flows on Contacts is to modify the Contact's properties based on the interactions within a flow. To simplify this common use-case, all blocks have a standard capability to specify how a contact property should be updated. This update shall happen immediately prior to following the exit node out of the block. This is specified via the optional `set_contact_property` object within the Block `config`:

```text
config {
   ...,
   set_contact_property: {
      property_key: <string>
      property_value: <Expression>
   }
}
```

The `property_key` is a string attribute within the context of the Contact, and is not further restricted by this specification. For complete block interoperability across vendors, vendors would need to agree on the format and identity of `property_key`. \(e.g. `property_key` could be "gender" , or it could be a UUID referenced to an external taxonomy service.\)

#### Block Example

```text
{
   "uuid": "0a0da5ab-66bc-4cd5-94c1-1e97f1875ee0",
   "vendor_metadata": {},
   "type": "MobilePrimitives.OpenResponse",
   "name": "patient_id",
   "label": "Patient ID",
   "semantic_label": "VMO::Person::Patient::ID",
   "exits": [
      {
         "uuid": "3b5c4568-6f8e-43d8-9c23-e0f444c9fd26",
         "name": "Default",
         "default": true,
         "config": {},
         "destination_block": "80a8a17e-2438-454b-b2f0-4b64fd06d700"
      }
   ],
   "config": {
      "prompt": "d99f9833-ace6-4f7f-957e-0a516e3dbb47"
      "set_contact_property": {
         "property_key": "gender"
         "property_value": "block.value"
      }
   }
}
```

