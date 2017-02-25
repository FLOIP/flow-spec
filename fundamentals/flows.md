# Flows

Flows represent a collection of actions ("Blocks") and the decision-making logic that links Blocks together into a flowchart-like description of an interactive mobile service, business process, or anything else you'd like to model with them.

## Format

Containers and Flow Definitions are a set of nested key-value elements formatted in [YAML, version 1.2](http://www.yaml.org/spec/1.2/spec.html).

## Container

A Container is a "package" document containing one or more Flow Definitions, useful for sharing multiple Flows in a single document. The required keys for a Container are:

Key | Description
--- | ---
`specification_version` (string)| The version of the Flow Spec that this package is compliant with.
`uuid` (64 bit integer TODO)| A globally unique identifier for this Container. FLOIP uses [TODO format of uuids]
`name` (string)| A human-readable name for the Container content.
`description` (string)| An extended human-readable description of the content.
`platform_metadata` (mapping)| A set of key-value elements that is not controlled by the Specification, but could be relevant to a specific Platform.
`flows` (list)| A list of the Flows within the Container (see below)
`resources` (list)| A list of the Resources needed for executing the Flows in the Container

### Example

```
TODO
```

## Flows

The required keys for a Flow are:

Key | Description
--- | ---
`uuid` (64 bit integer TODO)| A globally unique identifier for this Flow. [TODO: format of uuids]
`name` (string)| A human-readable name for the Flow content
`label` (string, optional)| An extended user-provided description for the flow.
`last_modified` (timestamp, UTC)| The time when this flow was last modified, in UTC, with microsecond precision: "2016-12-25 13:42:05.234598"
`interaction_timeout` (integer)| The number of seconds of inactivity after which Contact input for this flow is no longer accepted, and Runs in progress are terminated
`platform_metadata` (mapping)| A set of key-value elements that is not controlled by the Specification, but could be relevant to a specific Platform.
`supported_modes` (list)|A list of the supported Modes that the Flow has content suitable for. (See below)
`blocks` (list)| A list of the Blocks in the flow (see below).  The flow will start execution at the _first_ block in this list.

Supported modes include:
  - `text`: general text-based interactions
    - `sms`: content specific for SMS
    - `ussd`: content specific for USSD
  - `ivr`: content specific for interactive voice response
  - `rich_messaging`: content used for messaging platforms, for example:
    - `twitter`
    - `facebook_messenger`
    - `wechat`
    - `telegram`

### Example

```
TODO
```

## Blocks

The required keys for a Block are:

Key | Description
--- | ---
`uuid` (64 bit integer uuid TODO)| A globally unique identifier for this Block [TODO: format of uuids]
`name` (string, word-characters only) | A human-readable "variable name" for this block. This must be restricted to word characters so that it can be used as a variable name in expressions.
`label` (string, optional)| A human-readable free-form description for this Block.
`semantic_label` (string, optional)| A user-controlled field that can be used to label the meaning of the data collected by this block, e.g.: an ICD10 category name or other semantic classification system. ("ICD10::gender")
`type` (string)| A specific string designating the type or "subclass" of this Block. This must be one of the Block type names within the specification [TODO: layers of specification]
`config` (mapping)| Additional parameters that are specific to the type of the block. Details are provided within the Block documentation.
`exits`| a list of all the exits for the block. Exits must contain the required keys below, and can contain additional keys based on the Block type

Each exit must contain:

Key | Description
--- | ---
`uuid` (64 bit integer uuid TODO)| A globally unique identifier for this Exit
`label` (resource)| This is the human-readable name of the exit (as a translated resource), which might be presented to a contact.
`tag` (string, word characters only)| This is an identifier for the exit, suitable for use as a variable name in rolling up results (e.g.: "male"). It does not need to be unique within the block; multiple exits may be tagged the same. (Some authoring tools may choose to auto-generate the tag from the label's primary language, to avoid usability problems with these getting out of sync.)
`destination_block` (uuid)| This is the uuid of the Block this exit connects to.
`semantic_label` (string, optional)| A user-controlled field that can be used to label the meaning of the exit, e.g.: an ICD10 category name or other semantic classification system. ("ICD10::female")
`test` (expression, optional)| For blocks that evaluate conditions, this is an expression that determines whether this exit will be selected as the path out of the block. The first exit with an expression that evaluates to a "truthy" value will be chosen.
`config` (mapping)| This contains additional information required for each mode supported by the block. Details are provided within the Block documentation
  
### Example
  
```
TODO
```
  



