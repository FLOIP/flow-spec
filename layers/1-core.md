# Layer 1: Core

Layer 1 contains the specification for basic logical blocks that can be used without reference to any mobile channels. These blocks provide basic flow control for Flows.  
They make use of the [Expression Specification](https://github.com/floip/flow-specification/tree/7a09ac6d0cd28370fd159bce33d69f61c8eb4c30/layers/expressions.md) for generating content and evaluating decisions.

_Namespace_: `Core`

## Contents

* [Log Block](1-core.md#log-block)
* [Case Block](1-core.md#case-block)
* [Run Another Flow Block](1-core.md#run-another-flow-block)
* [Output Block](1-core.md#output-block)
* [Set Contact Property Block](1-core.md#set-contact-property)
* [Set Group Membership Block](1-core.md#set-group-membership)

## Log Block

* Type: `Core.Log`
* Suggested number of exits: 1
* Supported channels: all

This block appends a low-level message to the Context's `log`.

### Block `config`

| Key | Description |
| :--- | :--- |
| `message` \(string or resource\) | The content to be output. This can be a string or a localized resource; it supports parsing of expressions in rendering |

### Detailed Behaviour

The Context for a Flow shall have a `log` key, which preserves a mapping of timestamps and log messages for debugging. These logs must be maintained for the duration of the Run, and may be maintained for a longer period. The Log Block provides one way for a Flow to write to this log. On executing this block, the platform will append a key/value pair to the `log` object within the Context as below, and then proceed to the next block.

e.g.,

```text
"2017-03-05T12:30:42.123+00:00":"This is a sample log message."
```

### Example

```text
{
  "type": "Core.Log",
  "name": "test_log_block",
  "label": "Test Log Block",
  "exits": [
    {
      "uuid": "572f6e0d-6fd7-42f2-b5d4-fdcce49a1a12",
      "name": "Default",
      "default": true,
      "config": {}
    }
  ],
  "config": {
    "message": "bdd02d17-2baa-428e-8159-8d075b571d2d"
  }
}
```

## Case Block

* Type: `Core.Case`
* Suggested number of exits: variable
* Supported channels: all

This block evaluates a list of expressions, one for each exit, and terminates through the first exit where the corresponding expression evaluates to a "truthy" result.

### Block `config`

| Key | Description |
| :--- | :--- |
| none |  |

Keys for each `exit` are:

| Key | Description |
| :--- | :--- |
| `test` | The expression to be evaluated. If the expression evalutes to a "truthy" value, the block will terminate through this exit. |
| `default` \(boolean\) | If this key is present and true, the exit is treated as the flow-through default in a case evaluation. The block will terminate through this exit if no test expressions in other exits evaluate true. |

Each exit must specify one of either `test` or `default`. Each Case block must have exactly one `default` exit. Conventionally the `default` exit is listed last in the list.

### Detailed Behaviour

This block will sequentially evaluate the `test` expressions in each exit \(passing over any `default` exit\), in order. If the `test` expression evaluates to a truthy value using the Context and [Expressions](../expressions.md) framework, flow proceeds through the corresponding exit \(and no further exits are evaluated\). If no `test` expressions are found truthy, the flow proceeds through the `default` exit.

Truthy values include all values that are not `0`, `false`, `null`, or `undefined`.

### Example

```text
{
  "type": "Core.Case",
  "name": "patient_age_decision",
  "label": "Patient Age Decision",
  "exits": [
    {
      "uuid": "4c0dd2c8-a08f-45f7-9bf6-82bbff3fa968",
      "name": "under_18",
      "test": "contact.patient_age < 18",
      "destination_block": "338d216f-996c-4c6a-a1f5-fa2d1abe67a3"
    },
    {
      "uuid": "8968deb6-c4f3-4163-b3fc-d518bea14332",
      "name": "over_18",
      "test": "contact.patient_age >= 18",
      "destination_block": "7e0cded0-4bb2-49d7-8001-8eedd9d14f3b"
    },
    {
      "uuid": "4cfbab2d-132e-4583-8974-85424bff2424",
      "name": "default",
      "default": true,
    }
  ]
},
```

## Run Another Flow Block

* Type: `Core.RunFlow`
* Suggested number of exits: 1 + default exit (used in case of error or invalid input)
* Supported channels: all

This block starts and runs another Flow, and returns execution to the current Flow when finished.

### Block `config`

| Key | Description |
| :--- | :--- |
| `flow_id` \(uuid\) | The UUID of the other Flow to run. |

### Detailed behaviour

_Entry:_

On entry to this block, control proceeds into the other Flow given by `flow_id`. The Context for the outer flow is saved and stored within the new inner Flow's Context under the `parentFlowContext` key.

_Exit:_

Multiple levels of nested Flows shall be supported. When an inner Flow terminates, this block resumes execution in the outer Flow. The Context for the inner flow is saved and stored under the `childFlowContext` key, and flow proceeds through the next block. If an exception exit is triggerred within an inner flow causing the inner flow to terminate, flow proceeds through the error exit.

### Example

```text
    [...]
    "type": "Core.RunFlow",
    "name": "RunAnotherFlow",
    "label": "Another Flow",
    "config": {
      "flow_id": "ea5d7659-16cd-4e9a-86dc-28398cb41aed"
    },
    "exits": [...]
```

## Output Block

* Type: `Core.Output`
* Suggested number of exits: 1
* Supported channels: all

This block provides a connection to the Flow Results specification, by storing a named Output variable.

### Block `config`

| Key | Description |
| :--- | :--- |
| `value` \(expression\) | The expression that will be evaluated and written to the Results. |

### Detailed Behaviour

Not all block interactions and low-level logs are important to users; most users are concerned with a subset of results that have specific meaning -- the "Flow Results". \(See [Flow Results specification](https://github.com/FLOIP/flow-results/blob/master/specification.md).\) Any block type, as part of its specified runtime behaviour, may write to the Flow Results. The Output Block is a low-level block that does just simply one thing: write a named variable corresponding to the `name` of the block to the Flow Results, determined by the `value` expression.

### Example

```text
TODO
```

## Set Contact Property

* Type: `Core.SetContactProperty`
* Suggested number of exits: 1
* Supported channels: all

A common use-case for platforms that run flows on Contacts is to modify the Contact's properties based on the interactions within a flow. To simplify this common use-case, all blocks have a [standard capability to specify how a contact property should be updated](../flows.md#setting-contact-properties). The SetContactProperty block is a simple block that performs only this single operation, and is consistent with the standard block specification.

### Block `config`

| Key | Description |
| :--- | :--- |
| `set_contact_property` \(object, required\) | See below |

#### `set_contact_property` object

| Key | Description |
| :--- | :--- |
| `property_key` \(string\) | The attribute of the Contact that will be set \(or updated if it already has a value set\). |
| `property_value` \(expression\) | The expression that will be evaluated and stored. |

### Detailed Behaviour

The property update shall be evaluated and stored immediately prior to following the exit node out of the block.

The `property_key` is a string attribute within the context of the Contact, and is not further restricted by this specification. For complete block interoperability across vendors, vendors would need to agree on the format and identity of `property_key`. \(For instance, `property_key` could be "gender" , or it could be a UUID referenced to an external taxonomy service.\)

### Example

```text
{
    "type": "Core.SetContactProperty",
    "name": "test_contact_property",
    "label": "Test Contact Property",
    "config": {
      "set_contact_property": {
        "property_key": "gender",
        "property_value": "male"
      }
    },
    "exits": [
      {
        "uuid": "15e38cd1-0ed1-49ce-93d3-96b9e33a965a",
        "name": "Default",
        "default": true,
      }
    ]
}
```

## Set Group Membership

* Type: `Core.SetGroupMembership`
* Suggested number of exits: 1
* Supported channels: all

Another common operation for platforms that run flows on Contacts is to organize Contacts into groups based on the results of flow interactions. \(Examples include managing sign-ups for opt-in campaigns, organizing Contacts into demographic groups, etc.\)

### Block `config`

| Key | Description |
| :--- | :--- |
| `group_key` \(string\) | An identifier for the group that membership will be set within. |
| `group_name` \(string, optional\) | A human-readable label in addition to the `group_key`, in cases where the `group_name` needs to be displayed to the Contact. |
| `is_member` \(expression, boolean\) | Determines the membership state: falsy to remove the contact from the group, truthy to add, and null for no change to the existing membership. |

### Detailed Behaviour

The Contact's membership in the group shall be set according to the `is_member` expression, immediately before following the exit node out of the block.

The `group_key` is a string and is not further restricted by the spec. For complete block interoperability across vendors, vendors would need to agree on the format and identity of `group_key`.

### Example

```text
{
    "type": "Core.SetGroupMembership",
    "name": "ContactGMBlock",
    "label": "Test Group Membership",
    "config": {
      "group_key": "7294",
      "group_name": "Healthcare workers",
      "is_member": true,
    },
    "exits": [
      {
        "uuid": "c43106ba-be75-4a86-8da4-837de8348a22",
        "name": "Default",
        "default": true,
      }
    ]
  }
```

