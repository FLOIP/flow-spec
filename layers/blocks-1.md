# Layer 2: Console IO

Layer 2 contains the specification for I/O blocks that can be used in console environments for reading and writing data; often to provide support for automated testing. These blocks provide standard-input / standard-ouput operations, equivalent to printf\(\) and scanf\(\). Support for this layer is optional; many engines will not need this. They make use of the [Expression Specification](../expressions.md) for generating output.

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
[...]
"type": "ConsoleIO.Print",
"name": "PrintMessage",
"label": "Print Message",
"semantic_label": "print_message",
"exits": [
[...]
"value": "Thanks @(WORD(contact.name, 1)) for your response!"
[...]
```

## Read Block

* Type: `ConsoleIO.Read`
* Number of exits: 2

This block reads a line from standard input, and stores it in one or more context variables.

| Key | Description |
| :--- | :--- |
| `format_string` \(string\) | This is a ["scanf"](http://www.cplusplus.com/reference/cstdio/scanf/)-compatible format string, where any %-characters will be read into context variables. |
| `destination_variables` \(list\) | This is a list of strings, containing the variable names in the context where the results will be stored. The number of variable names must match the number of %-characters in `format_string`. |

This block has 2 exits: the first is chosen when all variables are read successfully. The second is chosen when there is a read error.

### Example

```text
[...]
"type": "ConsoleIO.Read",
          "name": "ReadBlock",
          "label": "Read Block Test",
          "semantic_label": "read_block",
          "config": {
            "format_string": "{\n  char str [80];\n  int i;\n\n  printf (\"Enter your family name: \");\n  scanf (\"%79s\",str);  \n  printf (\"Enter your age: \");\n  scanf (\"%d\",&i);\n  printf (\"Mr. %s , %d years old.\\n\",str,i);\n    \n  return 0;\n}",
            "destination_variables": []
          },
          "exits": [
            {
              "uuid": "a252e382-d51b-4f56-bd0a-61409aabf1fc",
              "tag": "Default",
              "label": "Default",
              "default": true,
              "config": {}
            },
            {
              "uuid": "99ad7e39-4bc1-4a1f-9c07-d444db931188",
              "tag": "Error",
              "label": "Error",
              "config": {}
            } [...]
```

