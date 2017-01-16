# Glossary and Definitions

This list provides a consistent definition of terms used in the FLOIP Specification

Term      | Description                     | Synonyms (discouraged)
----      |---------------------------------|------------------------
Flow Definition | A document which fully describes the blocks and logical connections in a flow. A Flow Runner can execute a Flow Definition. | Tree, Flow Content
Run       | An instance of a Contact traversing through a Flow. | Flow execution, session
Block     | A block is an action within a Flow. Blocks can execute instantly, or take time to wait for contact input. Blocks have one or more Outputs, and Connections from their Outputs to the start of other blocks. | Step, Action, Ruleset
Contact   | An entity which can interact with a Flow. | Phonebook Entry, Subscriber
Context   | A dictionary of variables providing the necessary context for running a Flow with a Contact. This includes things like the details of the Contact, information on the active channel, etc. | 
Runner    | A system capable of executing a Flow Definition with a Contact and doing something useful with the results. | Engine
Connection  | A logical link between the Output of a Block and the start of another Block. Connections + Blocks fully specify a Flow Definition. | Noodle
Mode      | A channel for transporting messages / exchanging data between a Contact and  Flow Runner. Some examples modes include: Text (SMS, USSD), IVR, and Rich Media (Messenger, WhatsApp, Telegram, Twitter) | Channel Type, Content Type
Expression | A text expression that can be evaluated using the Context to substitute named variables at runtime. Expressions can be used to generate content for Contacts during a Run, or to evaluate conditional decisions in Blocks with multiple Outputs. | 
Output | A Block has one or more outputs, based on the possible results of logical decisions that can happen within the block. Each Output has a Connection to another Block, or represents the end of a Run. | Exit
Flow Log   | A detailed dataset describing what happened in a Flow Run. It includes the path taken through the Blocks, metadata on when each block was entered and exited, the responses given by the Contact, and the final state of the Run. | Session data, Call data, Block Interactions
Flow Results | A summary or subset of the Flow Log, describing only the Contact's responses that are relevant to the desired data collection goals. | Flow data, Responses
