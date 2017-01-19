# Layer 1: Core Primitives

Layer 1 contains the specification for basic logical blocks that can be used without reference to any mobile channels. 
These blocks provide basic flow control for Flows.  
They make use of the [Expression Specification](../expressions.md) for generating content and evaluating decisions.

TODO: Do we want to have layer namespacing on block types? Block names could be:
> - `If`
> - `IfBlock`
> - `Primitive\If`
> What do we want?

## Message Block

- Type: `Message`
- Number of exits: 1

This block presents a textual message to the entity running the flow. Required keys in the block's `config` mapping are:

Key | Description
--- | ---
`message` (string or resource) | The content to be output. This can be a string or a localized resource; it supports parsing of expressions in rendering

### Example
```
TODO
```

## If Block

- Type: `If`
- Number of exits: 2

This block evaluates a single Expression, exiting through the first exit if the expression evaluates to a "truthy" result, otherwise exiting through the second exit.

Required keys in the block's `config` mapping are:

Key | Description
--- | ---
`test` (string) | The expression to be evaluated. If the expression evaluates to a 'falsy' result (0, false, or null), the block will terminate through the second exit. All other results are considered 'truthy' and will terminate through the first exit.

## Case Block

- Type: `Case`
- Number of exits: variable

This block evaluates a list of expressions, one for each exit, and terminates through the first exit where the corresponding expression evaluates to a "truthy" result.

Required keys in the block's `config` mapping are:

Key | Description
--- | ---
none |

Required keys for each exit's `config` mapping are:

Key | Description
--- | ---
`test` | The expression to be evaluated. If the expression evalutes to a "truthy" value, the block will terminate through this exit.

### Example
```
TODO
```

## Composite If Block

- Type: `CompositeIf`
- Number of exits: 2

This block evaluates a set of expressions, and chooses the first exit if either 'at least' or 'all' the expressions evaluate to a truthy result. It can operate in an OR mode ("at least one to be true") or AND mode ("all to be true"), depending on the configuration.

Required keys in the block's `config` mapping are:

Key | Description
--- | ---
`operation` (enum: `and` or `or`)| In `and` mode, all expressions must be truthy for the block to choose the first exit. In `or` mode, at least one expression must be truthy to choose the first exit.
`tests`| A list of expressions to evaluate 

Required keys for each exit's `config` mapping are:

Key | Description
--- | ---
none | 


### Example
```
TODO
```


