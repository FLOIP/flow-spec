# Context

The Flow Context contains the variables available for a Flow to read and write during execution. The context is populated by the engine before and during execution. The Context is referenced by expressions, for the purpose of evaluating branches, and displaying content to Contacts.

Top-level keys in the Context:

* `@contact`: Information about the contact the Flow is running with
* `@run`: Information about the current session
* `@parent`: In the case of nested flows, the parent Flow's Run (Synonym: @run.parent)
* `@child`: In the case of nested flows, the most recent child Flow's Run (Synonym: @run.child)
* `@results`: Results collected by blocks in the Flow (Synonym: @run.results)
* `@block`: Information collected by the current executing Block
* `@session`: Temporary session variables set and read by Blocks

## Contact

The Contact (`@contact`) contains information about the person the Flow is running with.

The contact's properties are exposed by name directly on the contact, as long as these names don't conflict with reserved names.

```
contact: {
  // phone number of the contact, in international E.164 format
  phone: "233501112222",
  // Other accounts the contact has, with service prefixes (such as social messaging accounts):
  urns: [
    "phone:233501112222",
    "twitter:@exampleaccount",
    "whatsapp:233501112222"
  ],
  // The contact's preferred mode of interaction (see Supported Modes)
  preferred_mode: "IVR",
  // The contact's preferred account to use
  preferred_urn: "phone:233501112222",
  // The contact's active account being used on this interaction
  active_urn: "phone:233501112222",
  // Time the contact was first seen/created on this system
  created_at: "2017-03-05 12:33:42",
  // Contact's timezone, in the format of the tzdata tz database. Null if unknown/not supported.
  timezone: "Africa/Accra",
  // Preferred language ID of the contact
  language: "eng",
  // Groups the contact is in (see below)
  groups: [...],
  // Properties of the contact (see below)
  properties: [...],
  // Example properties by name:
  birthday: "2012-04-05",
  gender: "female",
  ...
}
```

### Groups

The groups the contact is a member of:

```
contact: {
  groups: [
    {
      name: "Soybean Farmers",
      __value__: "Soybean Farmers",
      id: "1234-1234-1234-1234-1234"
    },
    {
      name: "Savings Group A",
      __value__: "Savings Group A",
      id: "1234-1234-1234-1234-1235"
    }
  ]
}
```

### Properties

The custom properties of the Contact. These can be set and read by Blocks. They are keyed by the property name.

```
contact: {
  properties: {
    birthday: {
      value: "2012-04-05",
      __value__: "2012-04-05",
      name: "birthday",
      type: "date",
      id: "1234-1234-1234-1234-1234"
    },
    gender: {
      value: "female",
      __value__: "female",
      name: "gender",
      type: "enum",
      id: "1234-1234-1234-1234-1235"
    }
  }
}
```

## Run

The Run object (`@run`) contains information about the flow session in progress.

```
run: {
  // Contains the language that the flow is currently running in. See [Language Objects and Identifiers](flows.md#language-objects-and-identifiers])
  language: {
    id: "eng-female"
    label: "English, Female Voice"
    variant: "female"
    iso_639_3: "eng"
    __value__: "eng-female"
  }

  // contains configuration details around localization of variables
  localization: {
    // Date format to present dates to users in (TODO)
    // Decimal format: whether numbers use commas or periods (TODO)
  }

  // Start time of the run
  entered_at: "2017-05-06 14:34:33",

  // End time of the run, if finished
  exited_at: "2017-05-06 16:31:12",

  // The flow that is running
  flow: {
    id: "1234-1234-1234-1234-1234",
    name: "Ice Cream Order Form",
    __value__: "Ice Cream Order Form",
  }

  // The mode the flow is running in. See Supported Modes in Flow Fundamentals.
  mode: "IVR",

  // A link to the results captured during this Run. (See @results)
  results: {...}

  // A link to the parent Flow's Run object (see below)
  parent: {...}

  // A link to the child Flow's Run object (see below)
  child: {...}
}
```

## Parent

In the case of nested Flows, the Parent object (`@parent` or `@run.parent`) is a link to the [Run](context.md#Run) of the outer Flow that launched this Flow run. This allows accessing results collected in the parent Flow, for example: `@parent.results.<block_name>.value`. More than one level up can be accessed, as in: `@parent.parent.results.<block_name>.value`.

## Child

In the case of nested Flows, the Child object (`@child` or `@run.child`) is a link to the [Run](context.md#Run) of the most recently executed child Flow. This allows accessing results collected within a child Flow, for example: `@child.results.<block_name>.value`.

## Results

The Results object (`@results`) holds values collected by blocks during the Run. These are keyed by the result name the block writes (`block.name`).

```
results: {
  // Example name (from block.value)
  favorite_ice_cream {
    // timestamp when the block was entered
    entered_at: "2022-03-15 14:03:03",
    // timestamp when the block was exited
    exited_at: "2022-03-15 14:03:35",
    // raw input response received from the contact, if applicable
    response: "choc",
    // parsed or standardized response. null if a valid response was not captured by the block.
    value: "chocolate",
    __value__: "chocolate",
    // The block that wrote this variable
    block: {
      id: "1234-1234-1234-1234-1234",
      name: "favorite_ice_cream",
      __value__: "favorite_ice_cream",
      label: "What's your favorite ice cream?"
    }
    // The exit that was selected coming out of this block
    exit: {
      name: "chocolate",
      __value__: "chocolate",
      id: "1234-1234-1234-1234-1234"
    }
  },
  ...
}
```

## Block

The Block object (`@block`) presents the currently executing block. It's often used in choice categorization tests and branching logic.

```
block: {
    // timestamp when the block was entered
    entered_at: "2022-03-15 14:03:03",
    // raw response received from the contact, if applicable. This could be text or an IVR digit.
    response: "choc",
}
```

## Session Variables

Session variables (`@session`) provide temporary storage for the duration of the Run that blocks can write and read. They are not retained as part of results beyond the Run. They are accessible and common to parent and child flows.

TODO: Consider. What would be the mechanism for how these are written?
