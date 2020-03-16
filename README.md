# Introduction

A container and data format for describing the _content and logic of digital interactions_, using the Flow Data paradigm. It provides for the open publication, exchange, and analysis of Flow-like content across supporting platforms.

Flows represent a collection of actions \("Blocks"\) and the decision-making logic that links Blocks together into a flowchart-like description of an interactive mobile service, business process, or anything else that can be modelled as programmatic flow-chart.

| Authors | Mark Boots \(VOTO Mobile\)  Peter Lubell-Doughtie \(Ona\)  Eduardo Jezierski \(InSTEDD\)  Gustavo Giráldez \(InSTEDD\)  Evan Wheeler \(UNICEF\) |
| :--- | :--- |
| Media Type | TODO: once registered: application/vnd.org.flowinterop.flows+json |
| Version | 1.0.0-rc.1 |
| Last updated | 2019-03-16 |
| Created | 2016-09-10 |
|  |  |

## Introduction

The purpose of the project is to enable useful interoperability between Flow-based platforms, and to incentivize the joining of future software tools into an interoperable ecosystem. We will accomplish this through a set of open specifications, and a set of open-source toolsets that reduce barriers to adoption. The initial focus is on tools for mobile/web engagement, although the most basic layer of the specification is general and can represent non-mobile business logic.

### What are Flows?

Flows are a modern paradign for describing the logic of digital information systems that interact with individuals, often for the purpose of \(a\) collecting data or \(b\) providing information through interactive requests. Some common examples of this are in mobile services using voice-based or SMS-based conversations over basic mobile phones. Flows follow the "flowchart" paradigm, consisting of actions \(nodes\) and connections between actions, which can incorporate decision-making logic.

### Who is working on this?

This is an initial collaboration between makers of Flow-like platforms and supporting tools: Nyaruka, InSTEDD, Ona, and VOTO Mobile. UNICEF and USAID are providing guidance and input. The project received initial funding from USAID. We intend that this project will outlast USAID funding and grow beyond the initial set of collaborating organisations. For more information on how to contribute, see [Project Charter and Governance Rules](https://github.com/floip/flow-specification/tree/7a09ac6d0cd28370fd159bce33d69f61c8eb4c30/charter.md).

## Specification - Table of Contents

1. [Language](./#language)
2. [Glossary and Definitions](glossary.md)
3. [Flow Fundamentals](flows.md)
   1. [Containers](flows.md#containers)
   2. [Flows](flows.md#flows)
   3. [Blocks](flows.md#blocks)
4. [Expressions](expressions.md)
5. [Layers in the Specification](layers/)
   1. [Layer 1: Core](layers/blocks.md)
   2. [Layer 2: Console IO](layers/blocks-1.md)
   3. [Layer 3: Mobile Primitives](layers/blocks-2.md)
   4. [Layer 4: Smart Devices](layers/blocks-3.md)

## Language

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in RFC 2119.

## Dates

All dates are RFC 3339 5.6 date-time, with offset-based timezone and space instead of T option. 

Example: **2017-06-30 15:35:27.134+00:00**

