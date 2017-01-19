# Glossary and Definitions

This list provides a consistent definition of terms used in the FLOIP Specification

Term      | Description                     | Synonyms (discouraged)
----      |---------------------------------|------------------------
Block     | A block is an action within a Flow. Blocks can execute instantly, or take time to wait for contact input. Blocks have one or more Exits, and Connections from their Exits to the start of other blocks. | Step, Action, Ruleset
Channel   | The details of the connection a Flow is being Run across, to communicate with a Contact. This could include the details of an SMS provider and shortcode, a telephone number and telecom connection, or a Facebook bot.
Connection  | A logical link between the Exit of a Block and the start of another Block. Connections + Blocks fully specify a Flow Definition. | Noodle
Contact   | An entity (often, a person with a digital identity and device) which can interact with a Flow. Note: Basic flows with only primitive Blocks might not need to deal with Contacts. | Phonebook Entry, Subscriber
Context   | A dictionary of variables providing the necessary context for running a Flow with a Contact. A Context may have a Contact, and other variables such as the Channel. | 
Exit | A Block has one or more Exits, based on the possible results of logical decisions that can happen within the block. Each Exit has a Connection to another Block, or represents the end of a Run. | Exit
Expression | A text expression that can be evaluated using the Context to substitute named variables at runtime. Expressions can be used to generate content for Contacts during a Run, or to evaluate conditional decisions in Blocks with multiple Exits. | 
Flow Definition | A document which fully describes the blocks and logical connections in a flow. A Flow Runner can execute a Flow Definition. | Tree, Flow Content
Flow Log   | A detailed dataset describing what happened in a Flow Run. It includes the path taken through the Blocks, metadata on when each block was entered and exited, the responses given by the Contact, and the final state of the Run. | Session data, Call data, Block Interactions
Flow Results | A summary or subset of the Flow Log, describing only the Contact's responses that are relevant to the desired data collection goals. | Flow data, Responses
Mode      | The type of medium for transporting messages / exchanging data between a Contact and  Flow Runner. Some examples modes include: Text (SMS, USSD), IVR, and Rich Media (Messenger, WhatsApp, Telegram, Twitter). A mode is the generic type of the Channel. | Channel Type, Content Type
Run       | An instance of a Contact traversing through a Flow. | Flow execution, session
Runner    | A system capable of executing a Flow Definition with a Contact and doing something useful with the results. | Engine
