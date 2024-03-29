# Layer 3: Mobile Primitives

Layer 3 contains the specification for I/O blocks that represent discrete single actions, that have direct analogues across several channels used in mobile messaging \(e.g. IVR, SMS, USSD\). Support for this layer should be implemented by all engines that target the `IVR`, `TEXT` \(SMS and USSD\), and `RICH_MESSAGING` modes. These blocks may make use of the [Expression Specification](expressions.md) for generating output. Higher levels may make use of embedded Layer 3 primitives to describe more advanced functionality.

_Namespace_: `MobilePrimitives`

## Contents

* [Message Block](2-mobile-primitives.md#message-block)
* [Select One Response \(Multiple Choice Question\) Block](2-mobile-primitives.md#select-one-response-multiple-choice-question-block)
* [Select Many Responses \(Multiple Choice Question\) Block](2-mobile-primitives.md#select-many-responses-multiple-choice-question-block)
* [Numeric Response Block](2-mobile-primitives.md#numeric-response-block)
* [Open Response \(Open-ended Question\) Block](2-mobile-primitives.md#open-response-block)

## Message Block

* Type: `MobilePrimitives.Message`
* Suggested number of exits: 1
* Supported modes: `IVR`, `TEXT`, `RICH_MESSAGING`, `OFFLINE`

This block presents a single message to the contact. The form of the message can depend on the mode: e.g. a voice recording for the `IVR` mode, and text for the `TEXT` mode.

### Block `config`

| Key | Description |
| :--- | :--- |
| `prompt` \(resource\) | The content to be output. This is a localized resource; it supports parsing of expressions in rendering. |

### Detailed behaviour by mode

* `SMS` \(SMS\): Sends `prompt` as an SMS to the contact.
* `USSD` \(USSD\): Displays `prompt` as the next USSD prompt to the user. \(Note on USSD session management: If there are following blocks in the flow, the user has an opportunity to reply with anything to proceed. If there are no following blocks, the contact is prompted to dismiss the session.\)
* `IVR`: Plays `prompt` to the contact.
* `RICH_MESSAGING`: Display `prompt` within the conversation with the contact. Optionally, platforms may attach the audio from the `prompt` resource \(if provided\) so that the contact can choose to play it.
* `OFFLINE`: Display `prompt` within the session with the contact.

### Output behaviour

None

### Example

```json
{
  "type": "MobilePrimitives.Message",
  "name": "welcome_message",
  "label": "Welcome Message",
  "exits": [...]
  "config": {
    "message": "42095857-6782-425d-809b-4226c4d53d4d"
  }
}
```

## Select One Response \(Multiple Choice Question\) Block

* Type: `MobilePrimitives.SelectOneResponse`
* Suggested number of exits: 1 + default exit (used in case of error or invalid input), or multiple exits based on choices
* Supported modes: `IVR`, `TEXT`, `RICH_MESSAGING`, `OFFLINE`

This block obtains the answer to a Multiple Choice question from the contact. The contact must choose a single choice from a set of choices.

This block can be configured to have a single exit, or a number of exits with possibilities based on the response given. The exit specification is as described in [Block `exits`](../flows.md#blocks).

### Block `config`

| Key | Description |
| :--- | :--- |
| `prompt` \(resource, optional\) | The question prompt that should be displayed to the contact, e.g. "What is your favorite kind of ice cream? Reply 1 for chocolate, 2 for vanilla, and 3 for strawberry." Either `prompt` or `question_prompt` should be provided. (For legacy compatibility, implementations may omit prompts completely when guidance has been given to the Contact in a previous block, but this is not recommended. In this case, the block will wait silently for a response.) |
| `question_prompt` \(resource, optional\) | For instances when the question prompt should be separated from the presentation of choices, e.g. "What is your favorite kind of ice cream?". If included, blocks must provide suitable resources within `choices` to present the choices on each supported mode. For the `IVR` mode, they must also provide `digit_prompts`. |
| `choices` \(array of choices\) | Set of choices to select from. See choices configuration below. |

#### Choices configuration
Each choice in `choices` has the following elements:
| Key | Description |
| :--- | :--- |
| `name` \(string\) | Key identifying this choice. This is what will be written into the block output (`block.value`) when a contact selects this choice, e.g. "chocolate" or "Somewhat Agree". |
| `ivr_test` \(object, optional\) | See below.|
| `text_tests` \(array of objects, optional\) | See below|
| `prompt` \(resource, optional\) | Resource used to present/display/announce this choice to contacts, appropriate for the language and mode. |

##### `ivr_test` object
This test applies to IVR flows.
| Key | Description |
| :--- | :--- |
| `test_expression` \(expression\) | The first choice with an expression that evaluates to a truthy value is the selected choice. Often this expression would examine the raw response from the contact, e.g. "block.response = 1". IVR responses are not expected to vary across languages, so this test applies to all. \). |

##### `text_tests` object
These tests apply to non-IVR flows. There may be multiple tests per choice: any matching test will indicate that the choice has been selected.
| Key | Description |
| :--- | :--- |
| `language` \(string, optional\) |[Language Identifier](flows.md#language-identifiers) for which this test applies. If omitted, the test will apply to all languages. Multiple tests may apply to the same language. |
| `test_expression` \(expression\) | The first choice with an expression that evaluates to a truthy value is the selected choice. Often this expression would examine the raw response from the contact, e.g. "block.response = 1". |

#### Channel-specific `config`:

| Key | Description |
| :--- | :--- |
| `IVR`: `digit_prompts` \(array of resources\) | An ordered set of audio prompts, with the same length as `choices`, with content such as "Press 1", "Press 2", "Press 3". This is required when using `question_prompt` to present choices individually.  |
| `IVR`: `randomize_choice_order` \(boolean, optional\) | Indicates that the choices should be presented in a random order to each Contact. (Used to minimize response order bias in surveying). Default false. Requires the use of `question_prompt`, `choices_prompt`, and `IVR.digit_prompts` to present choices individually. |

### Detailed behaviour by mode

* `TEXT` \(SMS\): Send an SMS with the prompt text, according to the prompt configuration in `config` above, and wait to capture a response. \(Lack a response after the flow's configured `timeout`, or an invalid response: proceed through the default exit.\)
* `TEXT` \(USSD\): Display a USSD menu prompt with the prompt text, according to the prompt configuration in `config` above, then wait to capture the menu response. \(Dismissal of the session, timeout, or invalid response: proceed through the default exit.\)
* `IVR`: Play the prompt, according to the prompt configuration in `config` above, then wait to capture the DTMF response.  \(Timeout or invalid response: proceed through the default exit.\)
  * When `question_prompt` is provided, `prompt` is ignored, and the prompt presented to the contact is generated as follows:
    * "What is your favorite kind of ice cream?" "For chocolate," "Press 1". "For vanilla", "Press 2".  "For strawberry," "Press 3".
    * `<question_prompt>`    `<choices["chocolate"]>` `<digit_prompts[0]>`    `<choices["vanilla"]>` `<digit_prompts[1]>`    `<choices["strawberry"]>` `<digit_prompts[2]>`
* `RICH_MESSAGING`: Display the prompt text according to the prompt configuration in `config` above. Platforms may wait to capture a text response, or display rich menu items for each choice and wait to capture a menu choice.  \(If displaying menu items, platforms should display only `question_prompt`.\) \(Timeout or invalid response: proceed through the default exit.\)
* `OFFLINE`: Display the prompt text according to `question_prompt`, and a menu of items for all `choices`. Wait to capture a menu selection.

### Output behaviour

This block writes the `name` of the selected choice to the output variable corresponding to the `name` of the block.

### Example

```json
{
  "type": "MobilePrimitives.SelectOneResponse",
  "name": "favorite_ice_cream",
  "label": "Favorite Ice Cream",
  "exits": [
    {
      "uuid": "95fd672c-92e9-4352-b761-7008b27cbe26",
      "test": "block.value = chocolate",
      "name": "chocolate",
    },
    {
      "uuid": "9fab760c-a680-4e40-83b7-9b3f8c66ccdb",
      "test": "block.value = vanilla",
      "name": "vanilla",
    },
    {
      "uuid": "d99d43ec-6f0a-42b4-97f9-aa1c50ddebe0",
      "test": "block.value = strawberry",
      "name": "strawberry",
    }
    {
      "uuid": "78012084-b811-4177-88ea-5de5d3eba57d",
      "default": true,
      "name": "Default",
    },
  ],
  "config": {
    "prompt": "42095857-6782-425d-809b-4226c4d53d4d",
    "choices": [
      {
        "name": "chocolate",
        "prompt": "b0f6d3ec-b9ec-4761-b280-6777d965deab",
        "ivr_test": {
          "test_expression": "block.response = '7'"
        }
        "text_tests": [
          {
            "test_expression": "block.response = '1'"
          },
          {
            "language": "eng",
            "test_expression": "block.response = 'chocolate'"
          },
          {
            "language": "fre",
            "test_expression": "block.response = 'chocolat'"
          },
        ]
      },
      {
        "name": "vanilla",
        "prompt": "b75fa302-8ff7-4f49-bf26-8f915e807222",
        "ivr_test": {
          "test_expression": "block.response = '8'"
        }
        "text_tests": [
          {
            "test_expression": "block.response = '2'"
          },
          {
            "language": "eng",
            "test_expression": "block.response = 'vanilla'"
          },
          {
            "language": "eng",
            "test_expression": "block.response = 'plain'"
          },
          {
            "language": "fre",
            "test_expression": "block.response = 'vanille'"
          },
        ]
      },
      {
        "name": "strawberry",
        "prompt": "22619b04-b06d-483e-af83-ee3ba9c8c867",
        "ivr_test": {
          "test_expression": "block.response = '9'"
        }
        "text_tests": [
          {
            "test_expression": "block.response = '3'"
          },
          {
            "language": "eng",
            "test_expression": "block.response = 'strawberry'"
          },
          {
            "language": "fre",
            "test_expression": "block.response = 'fraise'"
          },
        ]
      }
    ]
  }
}
[...]

"resources": [
  {
    uuid: "42095857-6782-425d-809b-4226c4d53d4d",
    values: [
      {
          language_id: "eng",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "What is your favorite kind of ice cream? Reply 1 for chocolate, 2 for vanilla, and 3 for strawberry."
      },
      {
          language_id: "fre",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "Quelle est votre sorte de crème glacée préférée ? Répondez 1 pour le chocolat, 2 pour la vanille et 3 pour la fraise."
      },
      {
          language_id: "eng",
          modes: ["IVR"],
          content_type: "AUDIO",
          mime_type: "audio/wav",
          value: "favorite_ice_cream_question.wav"
      },
      {
          language_id: "fre",
          modes: ["IVR"],
          content_type: "AUDIO",
          mime_type: "audio/wav",
          value: "question_creme_glacee_preferee.wav"
      },      
    ]
  },
  {
    uuid: "b0f6d3ec-b9ec-4761-b280-6777d965deab",
    values: [
      {
          language_id: "eng",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "Chocolate"
      },
      {
          language_id: "fre",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "Chocolat"
      }   
    ]
  },
  {
    uuid: "b75fa302-8ff7-4f49-bf26-8f915e807222",
    values: [
      {
          language_id: "eng",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "Vanilla"
      },
      {
          language_id: "fre",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "Vanille"
      }   
    ]
  },
  {
    uuid: "22619b04-b06d-483e-af83-ee3ba9c8c867",
    values: [
      {
          language_id: "eng",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "Strawberry"
      },
      {
          language_id: "fre",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "Fraise"
      }   
    ]
  },
  {
    uuid: "10a11345-9575-4e4a-bf61-0e04758626e7",
    values: [
      {
          language_id: "eng",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "Invalid"
      },
      {
          language_id: "fre",
          modes: ["SMS", "USSD"],
          content_type: "TEXT",
          mime_type: "text/plain",
          value: "Invalide"
      }   
    ]
  },
]
```

## Select Many Responses \(Multiple Choice Question\) Block

* Type: `MobilePrimitives.SelectManyResponses`
* Suggested number of exits: 1 + default exit (used in case of error or invalid input).
* Supported modes: `IVR`, `TEXT`, `RICH_MESSAGING`, `OFFLINE`

This block obtains the answer to a Multiple Choice question from the contact. The contact can select from zero to many options from a set of choices.

### Block `config`

| Key | Description |
| :--- | :--- |
| `prompt` \(resource, optional\) | The question prompt that should be displayed to the contact, e.g. "What kinds of ice cream do you like: chocolate, vanilla, strawberry? Select all that apply." Either `prompt` or `question_prompt` should be provided. (For legacy compatibility, implementations may omit prompts completely when guidance has been given to the Contact in a previous block, but this is not recommended. In this case, the block will wait silently for a response.) |
| `question_prompt` \(resource, optional\) | For instances when the question prompt should be separated from the presentation of choices, e.g. "What is your favorite kind of ice cream?". If included, blocks must provide suitable resources within `choices` to present the choices on each supported mode. For the `IVR` mode, they must also provide `digit_prompts`. |
| `choices` \(array of choices\) | Set of choices to select from. See choices configuration below. |
| `minimum_choices` \(integer, optional\) | The minimum number of choices the Contact must select to proceed. Default if not provided: 0. |
| `maximum_choices` \(integer, optional\) | The maximum number of choices the Contact can select. Default if not provided: unlimited \(ie: the total number of `choices`\). |

#### Choices configuration
Each choice in `choices` has the following elements:
| Key | Description |
| :--- | :--- |
| `name` \(string\) | Key identifying this choice. This is what will be written into the block output (`block.value`) when a contact selects this choice, e.g. "chocolate" or "Somewhat Agree". |
| `ivr_test` \(object, optional\) | See below.|
| `text_tests` \(array of objects, optional\) | See below|
| `prompt` \(resource\) | Resource used to present/display/announce this choice to contacts, appropriate for the language and mode. |

This block can be configured to have a single exit, or a number of exits with possibilities based on the response given. The exit specification is as described in [Block `exits`](../flows.md#blocks).

##### `ivr_test` object
This test applies to IVR flows.
| Key | Description |
| :--- | :--- |
| `test_expression` \(expression\) | The first choice with an expression that evaluates to a truthy value is the selected choice. Often this expression would examine the raw response from the contact, e.g. "block.response = 1". IVR responses are not expected to vary across languages, so this test applies to all. \). |

##### `text_tests` object
These tests apply to non-IVR flows. There may be multiple tests per choice: any matching test will indicate that the choice has been selected.
| Key | Description |
| :--- | :--- |
| `language` \(string, optional\) |[Language Identifier](flows.md#language-identifiers) for which this test applies. If omitted, the test will apply to all languages. Multiple tests may apply to the same language. |
| `test_expression` \(expression\) | The first choice with an expression that evaluates to a truthy value is the selected choice. Often this expression would examine the raw response from the contact, e.g. "block.response = 1". |

#### Channel-specific `config`:

| Key | Description |
| :--- | :--- |
| `IVR`: `digit_prompts` \(array of resources\) | An ordered set of audio prompts, with the same length as `choices`, with content such as "Press 1", "Press 2", "Press 3". This is required when using `question_prompt` to present choices individually.  |
| `IVR`: `randomize_choice_order` \(boolean, optional\) | Indicates that the choices should be presented in a random order to each Contact. (Used to minimize response order bias in surveying). Default false. Requires the use of `question_prompt`, `choices_prompt`, and `IVR.digit_prompts` to present choices individually. |

### Detailed behaviour by mode

* `TEXT` \(SMS\): Send an SMS with the prompt text, according to the prompt configuration in `config` above, and wait to capture multiple responses. \(Lack of the right number of responses after the flow's configured `timeout`, or an invalid response: proceed through the default exit.\)
* `TEXT` \(USSD\): Display a USSD menu prompt with the prompt text, according to the prompt configuration in `config` above, then wait to capture text describing multiple choices. \(Dismissal of the session, timeout, or invalid response: proceed through the default exit.\)
* `IVR`: Play the prompt, according to the prompt configuration in `config` above, then wait to capture multiple DTMF responses. Implementations may choose to optimize the user experience for additional guidance on answering multiple options.  \(Timeout or invalid response: proceed through the default exit.\)
  * When `question_prompt` is provided, `prompt` is ignored, and the prompt presented to the contact is generated as follows:
    * "What is your favorite kind of ice cream?" "For chocolate," "Press 1". "For vanilla", "Press 2".  "For strawberry," "Press 3".
    * `<question_prompt>`    `<choices["chocolate"]>` `<digit_prompts[0]>`    `<choices["vanilla"]>` `<digit_prompts[1]>`    `<choices["strawberry"]>` `<digit_prompts[2]>`
* `RICH_MESSAGING`: Display the prompt text according to the prompt configuration in `config` above. Platforms may wait to capture a text response, or display rich menu items for each choice and wait to capture a menu choice.  \(If displaying menu items, platforms should display only `question_prompt`.\) \(Timeout or invalid response: proceed through the default exit.\)
* `OFFLINE`: Display the prompt text according to `question_prompt`, and a menu of items for all `choices`. Wait to receive a menu confirmation.

### Output behaviour

This block writes an array of `name`s of the selected choices to the output variable corresponding to the `name` of the block.

### Example

```json
[...]
{
  "type": "MobilePrimitives.SelectManyResponses",
  "name": "ice_cream_order",
  "label": "Ice Cream Order",
  "exits": [
    {
      "uuid": "95fd672c-92e9-4352-b761-7008b27cbe26",
      "test": "block.value = 'chocolate'",
      "name": "chocolate",
    },
    {
      "uuid": "9fab760c-a680-4e40-83b7-9b3f8c66ccdb",
      "test": "block.value = 'vanilla'",
      "name": "vanilla",
    },
    {
      "uuid": "d99d43ec-6f0a-42b4-97f9-aa1c50ddebe0",
      "test": "block.value = 'strawberry'",
      "name": "strawberry",
    }
    {
      "uuid": "78012084-b811-4177-88ea-5de5d3eba57d",
      "default": true,
      "name": "Default",
    },
  ],
  "config": {
    "prompt": "42095857-6782-425d-809b-4226c4d53d4d",
    "minimum_choices": 1,
    "choices": [
      {
        "name": "chocolate",
        "prompt": "b0f6d3ec-b9ec-4761-b280-6777d965deab",
        "ivr_test": {
          "test_expression": "block.response = '7'"
        }
        "text_tests": [
          {
            "test_expression": "block.response = '1'"
          },
          {
            "language": "eng",
            "test_expression": "block.response = 'chocolate'"
          },
          {
            "language": "fre",
            "test_expression": "block.response = 'chocolat'"
          },
        ]
      },
      {
        "name": "vanilla",
        "prompt": "b75fa302-8ff7-4f49-bf26-8f915e807222",
        "ivr_test": {
          "test_expression": "block.response = '8'"
        }
        "text_tests": [
          {
            "test_expression": "block.response = '2'"
          },
          {
            "language": "eng",
            "test_expression": "block.response = 'vanilla'"
          },
          {
            "language": "eng",
            "test_expression": "block.response = 'plain'"
          },
          {
            "language": "fre",
            "test_expression": "block.response = 'vanille'"
          },
        ]
      },
      {
        "name": "strawberry",
        "prompt": "22619b04-b06d-483e-af83-ee3ba9c8c867",
        "ivr_test": {
          "test_expression": "block.response = '9'"
        }
        "text_tests": [
          {
            "test_expression": "block.response = '3'"
          },
          {
            "language": "eng",
            "test_expression": "block.response = 'strawberry'"
          },
          {
            "language": "fre",
            "test_expression": "block.response = 'fraise'"
          },
        ]
      }
    ]
  }
}
```

## Numeric Response Block

* Type: `MobilePrimitives.NumericResponse`
* Suggested number of exits: 1 + default exit (used in case of error or invalid input), or multiple based on ranges of interest
* Supported modes: `IVR`, `TEXT`, `RICH_MESSAGING`, `OFFLINE`

This block obtains a numeric response from the contact.

### Block `config`

| Key | Description |
| :--- | :--- |
| `prompt` \(resource, optional\) | The question prompt that should be displayed to the contact, e.g. "How old are you? Please reply with your age in years." (For legacy compatibility, implementations may omit the prompt when guidance has been given to the Contact in a previous block, but this is not recommended. In this case, the block will wait silently for a response.) |
| `validation_minimum` \(number, optional\) | The minimum value \(inclusive\) that will be accepted as a response to this block; responses less than this will result in a block value of `null`. |
| `validation_maximum` \(number, optional\) | The maximum value \(inclusive\) that will be accepted as a response to this block; responses greater than this will result in a block value of `null`. |

#### Channel-specific `config`:

| Key | Description |
| :--- | :--- |
| `IVR`: `max_digits` \(number\) | After receiving this many digits, do not wait for any more; accept the digits entered so far as the complete response. |

This block can be configured to have a single exit, or a number of exits with possibilities based on the range of the numeric response given. The exit specification is as described in [Block `exits`](../flows.md#blocks).

### Detailed behaviour by mode

* `TEXT` \(SMS\): Send an SMS with the prompt text, according to the prompt configuration in `config` above, and wait to capture a response. \(Lack a response after the flow's configured `timeout`, or an invalid response: proceed through the default exit.\)
* `TEXT` \(USSD\): Display a USSD menu prompt with the prompt text, according to the prompt configuration in `config` above, then wait to capture the menu response. \(Dismissal of the session, timeout, or invalid response: proceed through the default exit.\)
* `IVR`: Play the prompt, according to the prompt configuration in `config` above, then wait to capture the DTMF response.  \(Timeout or invalid response: proceed through the default exit.\)
* `RICH_MESSAGING`: Display the prompt text according to the prompt configuration in `config` above. Platforms may wait to capture a text response, or display a numeric entry widget and wait to capture a response. \(Timeout or invalid response: proceed through the default exit.\)
* `OFFLINE`: Display the prompt text according to the prompt configuration in `config` above, and display a numeric entry widget. Wait to capture a response.

### Output behaviour

This block writes the numeric value received to the output variable corresponding to the `name` of the block.

### Example

```json
[...]
{
  "type": "MobilePrimitives.NumericResponse",
  "name": "patient_age",
  "label": "How old are you?",
  "exits": [...],
  "config": {
    "prompt": "986a0f39-bfdf-4aa0-9fe2-28c90f422e1f",
    "validation_minimum": 0,
    "validation_maximum": 120,
    "IVR": {
      "max_digits": 3
    }
  }
}
```

## Open Response Block

* Type: `MobilePrimitives.OpenResponse`
* Suggested number of exits: 1 + default exit (used in case of error or invalid input), or multiple based on patterns of interest
* Supported modes: `IVR`, `TEXT`, `RICH_MESSAGING`, `OFFLINE`

This block obtains an open-ended response from the contact. Dependent on the mode, this is a TEXT response, audio recording, or other type of media recording \(e.g. video\).

This block can be configured to have a single exit, or a number of exits with possibilities based on patterns in the response given. The exit specification is as described in [Block `exits`](../flows.md#blocks).

### Block `config`

| Key | Description |
| :--- | :--- |
| `prompt` \(resource, optional\) | The question prompt that should be displayed to the contact, e.g. "Please leave us feedback on your experience at the Childrens Hospital." (For legacy compatibility, implementations may omit the prompt when guidance has been given to the Contact in a previous block, but this is not recommended. In this case, the block will wait silently for a response.) |

#### Channel-specific `config`:

| Key | Description |
| :--- | :--- |
| `IVR`: `max_duration_seconds` \(number\) | The maximum duration to record for, before proceeding to the next block. |
| `IVR`: `end_recording_digits` \(string, optional\) | A set of key-press digits that terminate an open-ended recording, e.g.: "1789#" |

### Detailed behaviour by mode

* `TEXT` \(SMS\): Send an SMS with the prompt text, according to the prompt configuration in `config` above, and wait to capture a response. \(Invalid/empty response: proceed through the default exit.\)
* `TEXT` \(USSD\): Display a USSD menu prompt with the prompt text, according to the prompt configuration in `config` above, then wait to capture the menu response. \(Invalid/empty response: proceed through the default exit.\)
* `IVR`: Play the prompt, according to the prompt configuration in `config` above, then record the user's response.  \(Timeout with nothing recorded: proceed through the default exit.\)
* `RICH_MESSAGING`: Display the prompt text according to the prompt configuration in `config` above, and wait to capture a text response or an upload \(audio, video\) from the contact. \(Invalid/empty response: proceed through the default exit.\)
* `OFFLINE`: Display the prompt text according to the prompt configuration in `config` above, and display a text entry widget. Wait to capture a response.

### Output behaviour

For `TEXT`, `OFFLINE`, and `RICH_MESSAGING` modes that capture a text response, this block writes the text received to the output variable corresponding to the `name` of the block. For responses captured as media \(`IVR`, `RICH_MESSAGING`\) the ID or URL of the recording is written.

### Example

```json
[...]
{
  "type": "MobilePrimitives.OpenResponse",
  "name": "OpenResponseFeedback",
  "label": "Patient Feedback",
  "semantic_label": "patient_feedback",
  "exits": [...],
  "config": {
    "prompt": "b969cd54-c894-4f5a-891e-f1b24e32982b",
    "IVR": {
      "max_duration_seconds": 120,
      "end_recording_digits": "1234567890#*"
    }
  }
}
```
