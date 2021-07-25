# API Usage

The Flow Specification is designed to be useful in both file-based and API-based data exchange. File-based usage is adequate for testing, archiving and transferring flows manually between systems. However, in many production situations, interoperable systems are likely to exchange flows and request actions on flows via API methods. When implementations provide access to Flows and Flow operations via an API, the following standardized endpoints, parameters, and envelope formats shall be used.

## API Authentication

Two methods of authentication are supported for clients accessing Flow Results APIs. All methods should use HTTPS for security.

### Token-based authentication

Implementations must support token-based authentication, via the "Authorization" HTTP header, using the Token method. An example of a complete authorization header is:

```text
Authorization: Token 0b79bab50daca910b000d4f1a2b675d604257e42
```

Implementations can determine the format of tokens. The issuance, expiry, and exchange of tokens is left outside the scope of the Flow Results specification.

### HTTPS Basic Auth

Providing additional support for HTTPS Basic Auth is optional, but recommended.

## API Request and Response format

The Flow API adheres to the [JSON API](http://jsonapi.org/format/) specification, version 1.0, an open standard for how a client should request that resources be fetched or modified, and how a server should respond to those requests, via JSON. All API requests and responses defined below reflect the JSON API norms for query parameters and pagination. Adopting JSON API allows API clients to make use of standard [JSON API libraries](http://jsonapi.org/implementations/).

### Pagination Parameters

Endpoints that paginate responses must do so consistent with the JSON API specification. The standard pagination query parameters are:
* `page[size]`: The requested number of responses per pagination page
* `page[afterCursor]`: The cursor to requests responses after this id, when paginating forward
* `page[beforeCursor]`: The cursor to request responses prior to this id, when paginating in reverse.

Endpoints that paginate shall provide a `links` section with `self`, `next`, and `previous` as appropriate.

## API Endpoints

### Base URL

Endpoints are defined relative to a base URL chosen by the implementing system, i.e.:

_Base URL_: [https://my.example-flow-server.com/api/v1](https://my.example-flow-server.com/api/v1)

### Containers and Flows

#### Publish a Container:

This endpoint is used to publish a Container of Flows to an external system. The Container should contain all related Flows that depend on each other (e.g.: accessed via RunAnotherFlow blocks, etc.). Flows in the container with the same `uuid` that already exist on the system will be updated (subject to the `update_mode` parameter below).

The receiving system should consider how it will manage resource media referenced within the Container (i.e.: whether it makes local copies of the resource media or depends on external availability, etc.)

_URL_: POST \[Base URL\]/flow-spec/containers

_Query parameters_:
* `update_mode` (Optional): One of the following values that determine when existing Flows are updated with the submitted definition. (`most_recent` is the default.) If an update would be rejected due to existing Flows under `most_recent` or `never`, the system must respond with `409 Conflict`.
  * `always`: Always update existing Flows with the same `uuid`s.
  * `most_recent`: Update existing Flows when the `last_modified` value is more recent than the system has already.
  * `never`: Never update existing Flows if they already exist on the system.

_Request body_: The request body shall specify the `type` of `containers`. It shall contain, within the `attributes` parameter, the JSON structure of the Container.

The `uuid` of the Container and each Flow can be omitted \(or null\) if the client wants the receiving system to assign a new UUID for these. If a `uuid` is provided for Flows, the client must ensure that the UUID conforms to the [JSON API specification](http://jsonapi.org/format/#crud-creating-client-ids) requirements for client-generated IDs:

* A server MAY accept a client-generated ID along with a request to create a resource. An ID MUST be specified with an id key, the value of which MUST be a universally unique identifier. The client SHOULD use a properly generated and formatted UUID as described in RFC 4122 \[RFC4122\]. A server MUST return 403 Forbidden in response to an unsupported request to create a resource with a client-generated ID.

Receiving systems need not maintain and track Container `uuid`s; a Container is used only for encapsulation of Flows during data exchange and can be temporary. It is the `uuid` of individual Flows that determines Flow identity. An individual Flow can be submitted and/or updated as part of one or more different Containers.

_Request example:_

```text
POST [Base URL]/flow-spec/containers HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{  
  "data":{  
    "type":"containers",
    "attributes": {
      "specification_version": "1.0.0",
      "uuid": null,
      "name": "Museum of Interop Reception",
      "description": "Welcome and main menu for the Museum of Interoperability",
      "flows": [
        {
          "uuid": null,
          "name": "Museum of Interop Reception",
          "label": "Welcome and main menu for the Museum of Interoperability",
          "last_modified": "2021-03-14 16:11:43.326Z",
          ...
        }
      ]
    }
  }
}
```

_Response_:

The response from the server must adhere to the [JSON API specification](http://jsonapi.org/format/#crud-creating-responses) for POST responses. When Client-Generated IDs are provided, systems should make use of the `204 No Content` response mechanism to report acceptance to clients without sending back long documents.

_Response example:_

```text
HTTP/1.1 201 Created
Location: https://my.example-flow-server.com/api/v1/flow-spec/containers/0c364ee1-0305-42ad-9fc9-2ec5a80c55fa
Content-Type: application/vnd.api+json

{  
  "data":{  
    "type":"containers",
    "id":"0c364ee1-0305-42ad-9fc9-2ec5a80c55fa",
    "attributes":{
      "specification_version": "1.0.0",
      "uuid": "0c364ee1-0305-42ad-9fc9-2ec5a80c55fa",
      "name": "Museum of Interop Reception",
      "description": "Welcome and main menu for the Museum of Interoperability",
      "flows": [
        {
          "uuid": "b15be41c-d29b-41fb-b981-26b2ebe8a6ff",
          "name": "Museum of Interop Reception",
          "label": "Welcome and main menu for the Museum of Interoperability",
          "last_modified": "2021-03-14 16:11:43.326Z",
          ...
        }
      ],
      "resources": [
        ...
      ]
    },
    "links":{  
        "self":"https://my.example-flow-server.com/api/v1/containers/0c364ee1-0305-42ad-9fc9-2ec5a80c55fa"
    }
  }
}
```

#### List all Flows

This endpoint is used to request a list of the Flows available on a system for an authorized user. 

(Note that Flows are submitted within Containers for efficiency when sharing resources, and to bundle dependent Flows together. However, it is still possible to examine the individual list of Flows.)

_URL_: GET \[Base URL\]/flow-spec/flows

_Query parameters_: [Pagination parameters](api-specification.md#pagination-parameters)

_Request body_: None

_Request example:_

```text
GET [Base URL]/flow-spec/flows HTTP/1.1
Accept: application/vnd.api+json
```

_Response body_: The response from the server must adhere to the [JSON API specification](http://jsonapi.org/format/#fetching-resources-responses) for fetching a collection of resources. The `data` array includes the list of flows. Implementations may decide what summary information they want to publish in the `attributes` for each package.

_Response example:_

```text
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "links": {
    "self": "https://my.example-flow-server.com/api/v1/flow-spec/flows?page=3&size=100",
    "next": null,
    "previous": "https://my.example-flow-server.com/api/v1/flow-spec/flows?page=2&size=100"
  },
  "data": [
    {
      "type": "flows",
      "id": "b15be41c-d29b-41fb-b981-26b2ebe8a6ff",
      "attributes": {
          "uuid": "b15be41c-d29b-41fb-b981-26b2ebe8a6ff"
          "name": "Museum of Interop Reception",
          "label": "Welcome and main menu for the Museum of Interoperability",
          "last_modified": "2021-03-14 16:11:43.326Z",
      }
    },
    {
      "type": "flows",
      "id": "09455c2b-f294-47c5-afcb-6561d7a736c6",
      "attributes": {
          "uuid": "09455c2b-f294-47c5-afcb-6561d7a736c6"
          "name": "Concerning Dangers from Dissensions Between the States",
          "label": "Concerning Dangers from Dissensions Between the States",
          "last_modified": "2021-03-14 16:11:43.326Z",
      }
    },
    {
      "type": "flows",
      "id": "37823bb4-7803-4f7d-b5a0-2f19fd75dbeb",
      "attributes": {
          "uuid": "37823bb4-7803-4f7d-b5a0-2f19fd75dbeb"
          "name": "The Utility of the Union In Respect to Revenue",
          "label": "The Utility of the Union In Respect to Revenue",
          "last_modified": "2021-03-14 16:11:43.326Z",
      }
    }
  ]
}
```
#### Get Flow Content

This endpoint is used to request the JSON definition of a single Flow stored on the system.

_URL_: GET \[Base URL\]/flow-spec/flows/\[id\]

_Query parameters_: None

_Request body_: None

_Request example:_

```text
GET [Base URL]/flow-spec/flows/b15be41c-d29b-41fb-b981-26b2ebe8a6ff HTTP/1.1
Accept: application/vnd.api+json
```

_Response body_: The response from the server must adhere to the [JSON API specification](http://jsonapi.org/format/#fetching-resources-responses) for fetching a resource.

_Response example:_

```text
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{  
  "links":{  
    "self":"https://my.example-flow-server.com/api/v1/flow-spec/flows/b15be41c-d29b-41fb-b981-26b2ebe8a6ff"
  },
  "data":{  
    "type":"flows",
    "id":"b15be41c-d29b-41fb-b981-26b2ebe8a6ff",
    "attributes":{
      "uuid": "b15be41c-d29b-41fb-b981-26b2ebe8a6ff",
      "name": "Museum of Interop Reception",
      "label": "Welcome and main menu for the Museum of Interoperability",
      "last_modified": "2021-03-14 16:11:43.326Z",
      ...
    }
  }
}
```

#### Get a Container with Flows

This endpoint is used to request a Container with a set of Flows and all dependent resources.

Systems may generate a temporary UUID for the Container.

_URL_: GET \[Base URL\]/flow-spec/containers

_Query parameters_: None

_Request body_:
* `with_flows` (array): An array of Flow UUIDs to request to be assembled, along with dependent resources, into a Container.

_Request example:_

```text
GET [Base URL]/flow-spec/containers HTTP/1.1
Accept: application/vnd.api+json

{  
  "data":{  
    "type":"containers",
    "attributes": {
      "with_flows": ["b15be41c-d29b-41fb-b981-26b2ebe8a6ff", "37823bb4-7803-4f7d-b5a0-2f19fd75dbeb"]
    }
  }
}
```

_Response body_: A Container assembled with the requested Flows and all required Resources.

_Response example:_

```text
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{  
  "data":{  
    "type":"containers",
    "id":"0c364ee1-0305-42ad-9fc9-2ec5a80c55fa",
    "attributes":{
      "specification_version": "1.0.0",
      "uuid": "0c364ee1-0305-42ad-9fc9-2ec5a80c55fa",
      "name": "Museum of Interop Reception",
      "description": "Welcome and main menu for the Museum of Interoperability",
      "flows": [
        {
          "uuid": "b15be41c-d29b-41fb-b981-26b2ebe8a6ff",
          "name": "Museum of Interop Reception",
          "label": "Welcome and main menu for the Museum of Interoperability",
          "last_modified": "2021-03-14 16:11:43.326Z",
          ...
        },
        {
          "uuid": "37823bb4-7803-4f7d-b5a0-2f19fd75dbeb"
          "name": "The Utility of the Union In Respect to Revenue",
          "label": "The Utility of the Union In Respect to Revenue",
          "last_modified": "2021-03-14 16:11:43.326Z",
        }
      ],
      "resources": [
        ...
      ]
    },
    "links":{  
        "self":"https://my.example-flow-server.com/api/v1/containers/0c364ee1-0305-42ad-9fc9-2ec5a80c55fa"
    }
  }
}
```

### Triggering Flow Runs

#### Trigger a Flow Run against one or more Contacts

This endpoint is used to ask a system to run (launch) an outbound Flow against one or more Contacts. 

Synchronization of Contact information between systems is a relevant (and difficult) problem to solve generally, especially when two-way synchronization and conflict resolution are required. The Flow Specification therefore leaves contact synchronization outside the scope of this API spec. However, there is a frequent requirement to ensure that systems running Flows at the request of another system have an up-to-date copy of Contact properties/parameters as of the start of a flow. This endpoint therefore includes a basic ability to transmit the current state of Contacts to populate context at the start of the Flow. Additional information can be transferred within `vendor_metadata`.

_URL_: POST \[Base URL\]/flow-spec/run_requests

_Query parameters_: None

_Request body_: The request body shall specify the `type` of `run_requests`. It shall contain, within the `attributes` parameter, the following information:

| Key | Description |
| :--- | :--- |
| `flow` \(uuid\) | UUID of the Flow to initiate on Contacts.
| `contacts` \(array of contacts, optional\) | An array of contacts that should receive the Flow. (See example below.) Contacts must have a `urn` to reach them at. (URNs for telephone numbers may omit the "tel:" prefix.) Contacts optionally have a set of current `properties`, a `preferred_mode`, and a `preferred_language`. (Systems may impose limits for the number of contacts supported in one run request.)  |
| `groups` \(array of group identifiers, optional\) | For systems that have a synchronized contact database, an array of group identifiers. Flows will be initiated to all contacts in these groups. Only one of `contacts` or `groups` shall be provided. (Systems may impose limits for the number of groups supported in one run request.) |
| `default_mode` \(string\) | The default mode to run the Flow for all contacts, unless overriden by `preferred_mode`. |
| `default_language` \(string\) | The default language (as a Flow Language Identifier) to run the Flow for all contacts, unless overriden by `preferred_language`. |
| `delay_until` \(date-time, optional\) | Indicates the system must delay the start of flow runs until the specified date and time (GMT). |
| `vendor_metadata` \(object, optional\) | Contains additional options on Flow runs that are relevant to specific vendor systems. Vendors must use namespaces within `vendor_metadata` to avoid collisions. Suggested namespaces use reverse domain name notation, with periods replaced by underscores (e.g.: A vendor with domain https://my.example-flow-server.com would place entries within `vendor_metadata.com_example-flow-server_my`). |

_Request example:_

```text
POST [Base URL]/flow-spec/run_requests HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{  
  "data":{  
    "type":"run_requests",
    "id": "8e144477-930e-4fa4-8b72-31aa6ccf74f3",
    "attributes": {
      "flow": "b15be41c-d29b-41fb-b981-26b2ebe8a6ff",
      "default_mode": "SMS",
      "default_language": "eng-american",
      "contacts": [
        {
          "id": <optional>,
          "urn": "+15552029099",  // Required
          "properties": [         // Optional
            {"key": "first_name", "value": "Alexander"},
            {"key": "last_name", "value": "Hamilton"},
            {"key": "age", "value": 264}
          ],
          "groups": [
            {"key": "founding_fathers", "group_name": "Founding Fathers"},
            {"key": "americans", "group_name": "Americans"},
            {"key": "federalists", "group_name" "Federalist Party Members"}
          ]
          "preferred_mode": "IVR",  // Optional. Mode must exist within the Flow
          "preferred_language": "eng-american"  // Optional
        },
        {
          "id": <optional>,
          "urn": "+15552021011",  // Required
          "properties": [         // Optional
            {"key": "first_name", "value": "Gilbert du Motier, Marquis de"},
            {"key": "last_name", "value": "Lafayette"},
            {"key": "age", "value": 264}
          ],
          "groups": [
            {"key": "frenchmen", "group_name": "Frenchmen"},
            {"key": "notables", "group_name": "Assembly of Notables"}
          ]
          "preferred_mode": "SMS",  // Optional. Mode must exist within the Flow
          "preferred_language": "fra" // Optional
        }
      ],
      "delay_until": "2021-03-19 09:05:00"
    }
  }
}
```

_Response body_: The response from the system must adhere to the [JSON API specification](http://jsonapi.org/format/#crud-creating-responses) for POST responses. When a Client-Generated ID is provided, systems should make use of the `204 No Content` response mechanism to report acceptance to clients without sending back long documents.

_Response example:_

```text
HTTP/1.1 204 No Content
Content-Type: application/vnd.api+json
```

#### Get Run Requests

This endpoint lists all run requests

_URL_: GET \[Base URL\]/flow-spec/run_requests

_Query parameters_: 

* `filter[flow]`: Only include run requests for the Flow given by the specified `uuid`.
* `filter[start-timestamp]`: Only show run requests that were posted after this timestamp \(exclusive\). \(This is a timestamp in the format of RFC 3339, section 5.6, `date-time`.\)
* `filter[end-timestamp]`: Only show run requests that were recorded before and on this timestamp \(inclusive\). \(This is a timestamp in the format of RFC 3339, section 5.6, `date-time`.\)
* [Pagination parameters](api-specification.md#pagination-parameters)

_Request body_: None

_Request example:_

```text
GET [Base URL]/flow-spec/run_requests HTTP/1.1
Accept: application/vnd.api+json
```

_Response body_: The response from the server must adhere to the [JSON API specification](http://jsonapi.org/format/#fetching-resources-responses) for fetching a resource. The `type` of the resource must be `run_requests`.

The `status` of `run_requests` shall be one of:
* `SCHEDULED`: The request is delayed and scheduled for the future according to the `delay_until` parameter.
* `IN_PROGRESS`: The system is currently executing Flow runs for this request.
* `COMPLETED`: The system has finished executing Flow runs for this request.

Lists of `contacts` and their initial properties may be omitted from the response.

_Response example:_

```text
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{  
  "data": [
    { 
      "type":"run_requests",
      "id": "8e144477-930e-4fa4-8b72-31aa6ccf74f3"
      "attributes": {
        "flow": "b15be41c-d29b-41fb-b981-26b2ebe8a6ff",
        "created_at": "2021-03-19 08:03:14"
        "default_mode": "SMS",
        "default_language": "eng-american",
        "delay_until": "2021-03-19 09:05:00",
        "status": "COMPLETED"
      }
    },
    { 
      "type":"run_requests",
      "attributes": {
        "flow": "37823bb4-7803-4f7d-b5a0-2f19fd75dbeb",
        "created_at": "2021-03-21 07:14:14"
        "default_mode": "IVR",
        "default_language": "eng-american",
        "delay_until": "2021-03-22 09:05:00",
        "status": "SCHEDULED"
      }
    }
  ]
}
```

#### Get Run Request

This endpoint lists one run request.

_URL_: GET \[Base URL\]/flow-spec/run_requests/\[id\]

_Query parameters_: None

_Request body_: None

_Request example:_

```text
GET [Base URL]/flow-spec/run_requests/"8e144477-930e-4fa4-8b72-31aa6ccf74f3" HTTP/1.1
Accept: application/vnd.api+json
```

_Response body_: The response from the server must adhere to the [JSON API specification](http://jsonapi.org/format/#fetching-resources-responses) for fetching a resource. The `type` of the resource must be `run_requests`.

The `status` of `run_requests` shall be one of:
* `SCHEDULED`: The request is delayed and scheduled for the future according to the `delay_until` parameter.
* `IN_PROGRESS`: The system is currently executing Flow runs for this request.
* `COMPLETED`: The system has finished executing Flow runs for this request.

Lists of `contacts` and their initial properties may be omitted from the response.

_Response example:_

```text
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{  
  "data":{  
    "type":"run_requests",
    "id": "8e144477-930e-4fa4-8b72-31aa6ccf74f3",
    "attributes": {
      "flow": "b15be41c-d29b-41fb-b981-26b2ebe8a6ff",
      "default_mode": "SMS",
      "default_language": "eng-american",
      "delay_until": "2021-03-19 09:05:00",
      "status": "COMPLETED"
    }
  }
}
```

