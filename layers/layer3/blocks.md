# Layer 3: Mobile Primitives

Layer 3 contains the specification for I/O blocks that represent discrete single actions, that have direct analogues across several channels used in mobile messaging (e.g. IVR, SMS, USSD).
Support for this layer should be implemented by all engines that target the `ivr`, `simple-text` (SMS and USSD), and `rich-messaging` channels. 
These blocks may make use of the [Expression Specification](../expressions.md) for generating output.  Higher levels may make use of embedded Layer 3 primitives to describe more advanced functionality.

*Namespace*: `MobilePrimitives`

## Contents
- [Message Block]
- [Select One Response (Multiple Choice Question) Block]
- [Numeric Response Block]
- [Open Response (Open-ended Question) Block]


## Message Block

- Type: `MobilePrimitives\Message`
- Suggested number of exits: 1
- Supported channels: `ivr`, `text`, `rich_messaging`, `offline`

This block presents a single message to the contact. The form of the message can depend on the channel: e.g. a voice recording for the `ivr` channel, and text for the `text` channel.

### Block `config`

Key | Description
--- | ---
`message` (resource) | The content to be output. This is a localized resource; it supports parsing of expressions in rendering.
`message-audio` (resource) | For channels that play audio, the localized recordings to play.

### Detailed behaviour

- `text` (SMS): Sends `message` as an SMS to the contact.
- `text` (USSD): Displays `message` as the next USSD prompt to the user. (Note on USSD session management: If there are following blocks in the flow, the user has an opportunity to reply with anything to proceed. If there are no following blocks, the contact is prompted to dismiss the session.)
- `ivr`: Plays `message-audio` to the contact.
- `rich_messaging`: Display `message` within the conversation with the contact. Optionally, platforms may attach the `message-prompt` (if provided) as an audio attachment that the contact can choose to play.
- `offline`: Display `message` within the session with the contact.

### Output behaviour
None (TODO: Should the length of message listened be reported in variables, or only be part of the detailed flow interaction logs?)

### Example
```
TODO
```

## Select One Response (Multiple Choice Question) Block

- Type: `MobilePrimitives\SelectOneResponse`
- Suggested number of exits: 1 + error exit, or multiple based on choices
- Supported channels: `ivr`, `text`, `rich_messaging`, `offline`

This block obtains the answer to a Multiple Choice question from the contact. The contact must choose a single choice from a set of choices.

### Block `config`

Key | Description
--- | ---
`prompt` (resource) | The question prompt that should be displayed to the contact, e.g. "What is your favorite kind of ice-cream? Reply 1 for chocolate, 2 for vanilla, and 3 for strawberry."
`prompt-audio` (resource, required for `ivr`) | For channels that play audio, the localized recordings to play.
`question-prompt` (resource, optional) | For instances when the question prompt should be separated from the presentation of choices, e.g. "What is your favorite kind of ice-cream?". If included, must also provide `choices-prompt` and omit `prompt`.
`choices-prompt` (resource, optional, required by `question-prompt`) | For instances when the question prompt should be separated from the presentation of choices, e.g. "Reply 1 for chocolate, 2 for vanilla, and 3 for strawberry."
`choices` (mapping of choice tags to choice resources) | This is a mapping of tags to localized names for choices, describing each choice in the multiple-choice set, e.g. `{"chocolate":[chocolate-resource], "vanilla":[vanilla-resource] , "strawberry":[strawberry-resource]}`.

This block can be configured to have a single exit, or a number of exits with possibilities based on the response given. The exit specification is as described in [Block `exits`](https://github.com/FLOIP/flow-spec/blob/s3/mobile-primitives/fundamentals/flows.md#blocks). 

### Detailed behaviour

- `text` (SMS): Send an SMS with the prompt text, according to the prompt configuration in `config` above, and wait to capture a response. (Lack a response after the flow's configured `timeout`, or an invalid response: proceed through the error exit.)
- `text` (USSD): Display a USSD menu prompt with the prompt text, according to the prompt configuration in `config` above, then wait to capture the menu response. (Dismissal of the session, timeout, or invalid response: proceed through the error exit.)
- `ivr`: Play the audio prompt, acccording to the prompt configuration in `config` above, then wait to capture the DTMF reponse.  (Hangup, timeout, or invalid response: proceed through the error exit.)
- `rich_messaging`: Display the prompt text according to the prompt configuration in `config` above. Platforms may wait to capture a text response, or display rich menu items for each choice and wait to capture a menu choice.  (If displaying menu items, platforms should display only `question_prompt`.) (Timeout or invalid response: proceed through the error exit.)
- offline: Display the prompt text according to `question_prompt`, and a menu of items for all `choices`. Wait to capture a menu selection.

### Output behaviour

This block writes the tag of the selected choice to the output variable corresponding to the `name` of the block.

### Example
```
TODO
```

## Numeric Response Block

- Type: `MobilePrimitives\NumericResponse`
- Suggested number of exits: 1 + error exit, or multiple based on ranges of interest
- Supported channels: `ivr`, `text`, `rich_messaging`, `offline`

This block obtains a numeric response from the contact.

### Block `config`

Key | Description
--- | ---
`prompt` (resource) | The question prompt that should be displayed to the contact, e.g. "How old are you? Please reply with your age in years."
`prompt-audio` (resource, required for `ivr`) | For channels that play audio, the localized recordings to play.
`validation-minimum` (number) | The minimum value (inclusive) that will be accepted as a response to this block; responses less than this will proceed through the error exit.
`validation-maximum` (number) | The maximum value (inclusive) that will be accepted as a response to this block; responses greater than this will proceed through the error exit.

#### Channel-specific `config`:
Key | Description
--- | ---
`ivr`: `max-digits` (number) | After receiving this many digits, do not wait for any more; accept the digits entered so far as the complete response.

This block can be configured to have a single exit, or a number of exits with possibilities based on the range of the numeric response given. The exit specification is as described in [Block `exits`](https://github.com/FLOIP/flow-spec/blob/s3/mobile-primitives/fundamentals/flows.md#blocks). 

### Detailed behaviour

- `text` (SMS): Send an SMS with the prompt text, according to the prompt configuration in `config` above, and wait to capture a response. (Lack a response after the flow's configured `timeout`, or an invalid response: proceed through the error exit.)
- `text` (USSD): Display a USSD menu prompt with the prompt text, according to the prompt configuration in `config` above, then wait to capture the menu response. (Dismissal of the session, timeout, or invalid response: proceed through the error exit.)
- `ivr`: Play the audio prompt, acccording to the prompt configuration in `config` above, then wait to capture the DTMF reponse.  (Hangup, timeout, or invalid response: proceed through the error exit.)
- `rich_messaging`: Display the prompt text according to the prompt configuration in `config` above. Platforms may wait to capture a text response, or display a numeric entry widget and wait to capture a response. (Timeout or invalid response: proceed through the error exit.)
- `offline`: Display the prompt text according to the prompt configuration in `config` above, and display a numeric entry widget. Wait to capture a response.

### Output behaviour

This block writes the numeric value received to the output variable corresponding to the `name` of the block.

### Example
```
TODO
```


## Open Response (Open-ended Question) Block

- Type: `MobilePrimitives\OpenResponse`
- Suggested number of exits: 1 + error exit, or multiple based on patterns of interest
- Supported channels: `ivr`, `text`, `rich_messaging`, `offline`

This block obtains an open-ended response from the contact. Dependent on the channel, this is a text response, audio recording, or other type of media recording (e.g. video).

### Block `config`

Key | Description
--- | ---
`prompt` (resource) | The question prompt that should be displayed to the contact, e.g. "How old are you? Please reply with your age in years."
`prompt-audio` (resource, required for `ivr`) | For channels that play audio, the localized recordings to play.

#### Channel-specific `config`:
Key | Description
--- | ---
`ivr`: `max-duration-seconds` (number) | The maximum duration to record for, before proceeding to the next block.
`text`: `max-response-characters` (number, optional) | The maximum number of characters to prompt for and accept. (If not provided, no limit.)

This block can be configured to have a single exit, or a number of exits with possibilities based on patterns in the response given. The exit specification is as described in [Block `exits`](https://github.com/FLOIP/flow-spec/blob/s3/mobile-primitives/fundamentals/flows.md#blocks). 

### Detailed behaviour

- `text` (SMS): Send an SMS with the prompt text, according to the prompt configuration in `config` above, and wait to capture a response. (Lack a response after the flow's configured `timeout`: proceed through the error exit.)
- `text` (USSD): Display a USSD menu prompt with the prompt text, according to the prompt configuration in `config` above, then wait to capture the menu response. (Dismissal of the session or timeout: proceed through the error exit.)
- `ivr`: Play the audio prompt, acccording to the prompt configuration in `config` above, then wait to capture the DTMF reponse.  (Hangup or timeout with nothing recorded: proceed through the error exit.)
- `rich_messaging`: Display the prompt text according to the prompt configuration in `config` above, and wait to capture a text response or an upload (audio, video) from the contact. (Timeout: proceed through the error exit.)
- `offline`: Display the prompt text according to the prompt configuration in `config` above, and display a text entry widget. Wait to capture a response.

### Output behaviour

For `text`, `offline`, and `rich_messaging` channels that capture a text response, this block writes the text received to the output variable corresponding to the `name` of the block. For responses captured as media (`ivr`, `rich_messaging`) the ID of the recording is written. (For more information on standards for IDs of recordings, see [Captured Media Recording IDs](TODO).)

TODO: Do we want to capture the length of the recording other than in the detailed flow interaction log?

### Example
```
TODO
```
