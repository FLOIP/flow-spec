# Layer 2: Console IO

Layer 2 contains the specification for I/O blocks that can be used in console environments for reading and writing data; often to provide support for automated testing. These blocks provide standard-input / standard-ouput operations, equivalent to printf\(\) and scanf\(\). Support for this layer is optional; many engines will not need this. They make use of the [Expression Specification](https://github.com/floip/flow-specification/tree/7a09ac6d0cd28370fd159bce33d69f61c8eb4c30/layers/expressions.md) for generating output.

Namespace: `ConsoleIO`

## Print Block

* Type: `ConsoleIO.Print`
* Number of exits: 1

This block prints a message to standard output, by evaluating an expression.

| Key | Description |
| :--- | :--- |
| `message` \(string or resource\) | The content to be output. This can be a string or a localized resource; it supports parsing of expressions in rendering |

### Example

```text
TODO
```

## Read Block

* Type: `ConsoleIO.Read`
* Suggested Number of exits: 2

This block reads a line from standard input, and stores it in one or more context variables.

| Key | Description |
| :--- | :--- |
| `format_string` \(string\) | This is a ["scanf"](http://www.cplusplus.com/reference/cstdio/scanf/)-compatible format string, where any %-characters will be read into context variables. |
| `destination_variables` \(list\) | This is a list of strings, containing the variable names in the context where the results will be stored. The number of variable names must match the number of %-characters in `format_string`. |

The block writes an output value of TRUE or FALSE to the block name for its result; TRUE if the read was successful.

This block typically has 2 exits: the first is chosen when variables are read successfully. The second (default exit) is used when there is a read error.

### Example

```text
TODO
```

