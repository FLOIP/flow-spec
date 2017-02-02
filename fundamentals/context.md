# Flow Context

The Flow Context contains the variables available for a flow to read and write
during execution. The context is populated by the engine before and during
execution, different variables will be available depending on which flow
extensions are available.

## Configuration variables

Configuration variables live under the `@config` key in the context. They control
the execution of the flow in various ways, mostly related to localization but
also for flow control.

 * default_exit_block: UUID of the block that flow will go to if reaching
   an exit without a destination block
 * language: contains the language that the flow is currently running in
       code: eng / fra / spa
       name: English / French / Spanish
       __string__: English
       __default__: eng
 * timezone: contains the timezone that the flow is currently running in
       code: UTC+500
       name: US Pacific
       __string__: US Pacific
       __default__: UTC+500
 * localization: contains configuration details around localization of variables
       date_format: whether we are day or month first
       decimal_format: whether we use commas or periods

## Flow Steps

The `@steps` variable contains information about the path taken by the flow
during execution. Each block visited adds a single step upon entering the block.

TODO: Do steps include any information about messages sent or received?

```
[
  {
    "block": "1234-1234-1234-1234-1234",
    "arrived_on": "2017-01-17 14:14",
    "left_on": "2017-01-17 14:15",
    "flow_version": "2017-01-17 14:15" # to know what version of this flow processed this step?
  },
  {
    "block": "1234-1234-1234-1234-1234",
    "arrived_on": "2017-01-17 14:14",
    "left_on": "2017-01-17 14:15",
    "flow_version": "2017-01-17 14:15" # to know what version of this flow processed this step?
  }
]
```

## Flow variables

The `@flow` variable contains a mapping of variables populated by Case blocks
(or other blocks with names).

```
{
  "age": {
    "block": "1234-1234-1234-1234-1234",
    "created_by": "1234-1234-1234-1234-1234",
    "created_on": "2017-01-17 14:14",
    "value": 36,
    "input": "I am 36 years old",
    "exit": "1234-1234-1234-1234-1234",
    "exit_label": "Middle Aged",
    "mime_type": "text/plain", # optional, assumed to be text/plain
    __default__: 36
  },
  "gender": {
    "block": "1234-1234-1234-1234-1234",
    "created_by": "1234-1234-1234-1234-1234",
    "created_on": "2017-01-17 14:14",
    "value": "M",
    "input": "m",
    "exit": "1234-1234-1234-1234-1234",
    "exit_label": "Male",
    __default__: "M"
  },
  "voicemail": {
    "block": "1234-1234-1234-1234-1234",
    "created_by": "1234-1234-1234-1234-1234",
    "created_on": "2017-01-17 14:14",
    "value": "https://s3.aws/1234-1234-1234-1234.wav",
    "mime_type": "audio/wav"
    "input": "https://s3.aws/1234-1234-1234-1234.wav",
    "exit": "1234-1234-1234-1234-1234",
    "exit_label": "Long Message",
    __default__: "https://s3.aws/1234-1234-1234-1234.wav",
    "meta": {
      "duration": 2332 # ms?
    }
  },
  "mugshot": {
    "created_by": "1234-1234-1234-1234-1234",
    "created_on": "2017-01-17 14:14",
    "value": "https://s3.aws/1234-1234-1234-1234.jpeg",
    "mime_type": "image/jpeg"
    "input": "https://s3.aws/1234-1234-1234-1234.jpeg",
    "exit": "1234-1234-1234-1234-1234",
    "exit_label": "Valid Image",
    __default__: "https://s3.aws/1234-1234-1234-1234.jpeg",
    "meta": {
      "dimensions": "300x500"
    }
  }
}
```
