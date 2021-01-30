# Layer 1: Core

Layer 1 contains the specification for basic logical blocks that can be used without reference to any mobile channels. These blocks provide basic flow control for Flows.  
They make use of the [Expression Specification](https://github.com/floip/flow-specification/tree/7a09ac6d0cd28370fd159bce33d69f61c8eb4c30/layers/expressions.md) for generating content and evaluating decisions.

_Namespace_: `Core`

## Contents

* [Log Block](blocks.md#log-block)
* [Case Block](blocks.md#case-block)
* [Run Another Flow Block](blocks.md#run-another-flow-block)
* [Output Block](blocks.md#output-block)

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

```text

```

e.g.,

```text
"2017-03-05T12:30:42.123+00:00":"This is a sample log message."
```

### Example

```text
TODO
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
TODO
```

## Run Another Flow Block

* Type: `Core.RunFlow`
* Suggested number of exits: 1 + error exit
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
TODO
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

