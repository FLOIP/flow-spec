# Layer 2: Console IO Primitives

Layer 2 contains the specification for I/O blocks that can be used in console environments for reading and writing data; often to provide support for automated testing.
These blocks provide standard-input / standard-ouput operations, equivalent to printf() and scanf(). Support for this layer is optional; many engines will not need this.
They make use of the [Expression Specification](../expressions.md) for generating output.

Namespace: `ConsoleIO`

## Print Block

- Type: `ConsoleIO\Print`
- Number of exits: 1

This block prints a message to standard output, by evaluating an expression.

Key | Description
--- | ---
`message` (string or resource) | The content to be output. This can be a string or a localized resource; it supports parsing of expressions in rendering

### Example
```
TODO
```

## Read Block

- Type: `ConsoleIO\Read`
- Number of exits: 2

This block reads a line from standard input, and stores it in one or more context variables.

Key | Description
--- | ---

`format_string` (string) | This is a ["scanf"](http://www.cplusplus.com/reference/cstdio/scanf/)-compatible format string, where any %-characters will be read into context variables
`destination_variables` (list) | This is a list of strings, containing the variable names in the context where the results will be stored. The number of variable names must match the number of %-characters in `format_string`.

This block has 2 exits: the first is chosen when all variables are read successfully. The second is chosen when there is a read error.

### Example
```
TODO
```
