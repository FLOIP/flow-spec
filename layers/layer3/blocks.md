# Layer 3: Mobile Primitives

Layer 3 contains the specification for I/O blocks that represent discrete single actions, that have direct analogues across several channels used in mobile messaging (e.g. IVR, SMS, USSD).
Support for this layer should be implemented by all engines that target the `ivr`, `simple-text` (SMS and USSD), and `rich-messaging` channels. 
These blocks may make use of the [Expression Specification](../expressions.md) for generating output.  Higher levels may make use of embedded Layer 3 primitives to describe more advanced functionality.

Namespace: `MobilePrimitives`

## Message Block

- Type: `MobilePrimitives\Message`
- Number of exits: 1
- Supported channels: `ivr`, `text`, `rich_messaging`, `offline`

This block presents a single message to the contact. The form of the message can depend on the channel: e.g. a voice recording for the `ivr` channel, and text for the `text` channel.

### Block `config`

Key | Description
--- | ---
`message` (resource) | The content to be output. This is a localized resource; it supports parsing of expressions in rendering.
`message-audio` (resource) | For channels that play audio, the localized recordings to play.

### Example
```
TODO
```

## Select One Block

- Type: `ConsoleIO\SelectOne`
- Number of exits: 1, or dependent on number of choices

This block obtains the answer to a Multiple Choice question from the contact. The contact must choose a single choice from a set of choices.

### Block `config`

Key | Description
--- | ---
`prompt` (resource) | The question prompt that should be displayed to the contact, e.g. "What is your favorite kind of ice-cream? Reply 1 for chocolate, 2 for vanilla, and 3 for strawberry."
`prompt-audio` (resource, required for `ivr`) | For channels that play audio, the localized recordings to play.
`question-prompt` (resource, optional) | For instances when the question prompt should be separated from the presentation of choices, e.g. "What is your favorite kind of ice-cream?". If included, must also provide `choices-prompt` and omit `prompt`.
`choices-prompt` (resource, optional, required by `question-prompt`) | For instances when the question prompt should be separated from the presentation of choices, e.g. "Reply 1 for chocolate, 2 for vanilla, and 3 for strawberry."
`choices` (mapping of choice tags to choice resources) | This is a mapping of tags to localized names for choices, describing each choice in the multiple-choice set, e.g. `{"chocolate":[chocolate-resource], "vanilla":[vanilla-resource] , "strawberry":[strawberry-resource]}`.

This block can be configured to have a single exit, or a number of exits chosen based on the response given.

### Output behaviour

This block writes the tag of the selected choice to the output variable corresponding to the `name` of the block.

### Example
```
TODO
```
