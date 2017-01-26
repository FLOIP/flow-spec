# Layer 1: Core Primitives

Layer 1 contains the specification for basic logical blocks that can be used without reference to any mobile channels. 
These blocks provide basic flow control for Flows.  
They make use of the [Expression Specification](../expressions.md) for generating content and evaluating decisions.

TODO: Do we want to have layer namespacing on block types? Block names could be:
> - `If`
> - `IfBlock`
> - `Primitive\If`
> What do we want?

## Log Block

- Type: `Log`
- Number of exits: 1

This block appends a low-level message to the context's log.  Required keys in the block's `config` mapping are:

Key | Description
--- | ---
`message` (string or resource) | The content to be output. This can be a string or a localized resource; it supports parsing of expressions in rendering

### Details

It is expected that the context for a Flow will have a `log` key, which preserves a mapping of timestamps and log messages for debugging. These logs must be maintained for the duration of the Run, and may be maintained for a longer period.  The Log Block provides a way of writing to this log.

### Example
```
TODO
```

## Case Block

- Type: `Case`
- Number of exits: variable

This block evaluates a list of expressions, one for each exit, and terminates through the first exit where the corresponding expression evaluates to a "truthy" result.

Required keys in the block's `config` mapping are:

Key | Description
--- | ---
none |

Required keys for each exit are:

Key | Description
--- | ---
`test` | The expression to be evaluated. If the expression evalutes to a "truthy" value, the block will terminate through this exit.
`default`| If this key is present, the exit is treated as the flow-through default in a case evaluation. The block will terminate through this exit if no test expressions in other exits evaluate true.

Each exit must specify one of either `test` or `default`. Each Case block must have exactly one `default` exit. Conventionally the `default` exit is listed last in the list.

### Example
```
TODO
```

## Run Another Flow Block

- Type: `RunFlow`
- Number of exits: 1 [TODO: Should there be multiple exits, e.g.: if the sub-flow encounters an exception?]

This block starts and runs another Flow, and returns execution to the current Flow when finished.

Required keys in the block's `config` mapping are:

Key | Description
--- | ---
`flow_id` (uuid)| The UUID of the other Flow to run.


### Example
```
TODO
```

## Output Block

- Type: `Output`
- Number of exits: 1

This block provides a connection to the Flow Results specification, by storing a named Output variable.

Required keys in the block's `config` mapping are:

Key | Description
--- | ---
`name` (string) | The variable name in the Flow Results output to write.
`value` (expression)| The expression that will be evaluated and written to the Results. [TODO: Can this be a localized resource?]

### Details

Not all block interactions and low-level logs are important to users; most users are concerned with a subset of results that have specific meaning -- the "Flow Results".  [TODO: Flow Results specification]  Any block type, as part of its specified runtime behaviour, may write to the Flow Results.  The Output Block is a low-level block that does just this one thing: write a named value to the Flow Results.

### Example
```
TODO
```


