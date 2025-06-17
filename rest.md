---
title: REST Style Guide
---
## Introduction

Successful APIs feel like they were built and designed by a single team. Uniformly applying a look & feel across services eases adoption, reduces friction and promotes a great integration experience for developers.

This guide has been crafted to help teams build consistently and to answer the common questions encountered during API development. Every REST API that we build - public or not - must adhere to these standards.

## Overview

### REST preferred
We prefer services designed using a **Representational State Transfer (REST)** architectural style. We consider a RESTful API to adhere to level 2 of the [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html).

### Design the API first
We embrace [API first](https://swagger.io/resources/articles/adopting-an-api-first-approach/) as a core architectural principle, where APIs must be defined using [OpenAPI](https://swagger.io/specification/) as a specification language before implementation is coded. 

### APIs are products
We should give the same care and attention to designing APIs as we would a product. Consider APIs as a **standalone platform capability** that Handshaik can offer rather than just a back end for a specific internal product.

* **Prioritise developer experience** by designing APIs that shield integrators from unnecessary complexity and don't push the heavy lifting to the client
* **Understand the consumer** and build with their use cases in mind. Be an advocate for their needs.
* **Make time for great documentation** that as a developer, you'd hope to receive from another platform. 
* **Be agnostic of deployment** by designing/documenting APIs at the same quality standard regardless of audience. Assume your API will go to the general public.

## Conventions

Each guideline is uniquely numbered to save time and promote easy referencing in conversation, pull request and documentation. The number holds no significance for rule importance or weighting. When a guideline is removed, the numbering of subsequent guidelines remains unchanged.

The requirement level keywords "**must**", "**must not**", "**required**", "**shall**", "**shall not**", "**should**", "**should not**", "**recommended**", "**may**", and "**optional**" used in this document (case insensitive) are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt). 

## Sources

The following resources have been as a reference point in the creation of this document. All are recommended reading as fantastic examples of comprehensive style guides. As we evolve this document, it's helpful to see what great looks like!

* [Zalando's API Style Guide](https://opensource.zalando.com/restful-api-guidelines)
* [Haufe's API Style Guide](https://haufe-lexware.gitbooks.io/haufe-api-styleguide/content/introduction/introduction.html)

## Contributing

Please consult our guidelines on [contribution](). The arbiters of this document are:

* Gabriel Bauernfreund
* Yaara Cohen

## Rules: REST Basics

### RS100 - HTTP methods **must** use verbs in a consistent manner
Our platform uses [verbs aligned with REST](https://restfulapi.net/http-methods/) to indicate the intent of a request. At present, usage of the following verbs is acceptable:

| Method | Action |
|----|----|
| GET | Retrieve a resource (or collection of resources)| 
| POST | Create a new resource| 
| PUT | Update an existing resource by storing every field defined by the `PUT` request body, regardless of whether it has been populated (overwrite) | 
| PATCH | Update an existing resource by storing only the fields defined by the `PATCH` request body that have been populated by the API consumer (partial update) |
| DELETE | Delete an existing resource | 
| OPTIONS | Inspect the available HTTP methods of a given endpoint (rarely implemented)|
| HEAD | Semantically the same as `GET` but just retrieves headers, no body (rarely implemented)|

### RS101 - Resource names **must** be plural nouns

Our URI's should adhere to [REST best practice](https://cloud.google.com/blog/products/api-management/restful-api-design-nouns-are-good-verbs-are-bad) and structure URI referring to resources that are _things_ (nouns) instead of referring to *actions* (verbs). 

Acceptable:

```
GET /items
GET /items/12345
POST /items
```

Unacceptable:

```
GET /allItems
GET /itemById
POST /createItem
```

### RS102 - URIs **should** use nesting for related resources

Where possible, we should prefer nested resource URIs since they allow us to convey that one resource belongs to another. By giving the appearance of a hierarchical relationship, we make our APIs self documenting and improve developer experience. 

As a general rule when designing URI structure, consider the likely access patterns of integrators. 

If a resource makes no sense unless accessed via it's parent entity:
``` 
GET /suppliers/{supplier_id}/inventoryItems

```

Otherwise:
```
GET /inventoryItems?supplier_id=123456
```

###  RS103 - URIs **should not** be overly nested

To keep URIs concise, we should not present nested resources hierarchies beyond a hierarchy depth of **three**. Many levels of hierarchy add to complexity of consumption and some popular web browsers do not support URLs of more than 2000 characters.

Desirable: 

```
GET /lessons/{lesson_id}
GET /lessons?school_id={school_id}&course_id={course_id}
GET /lessons?school_id={school_id}&course_id={course_id}&module_id={module_id}


```
Undesirable:

```
GET /schools/{school_id}/courses/{course_id}/modules/{module_id}/lessons
GET /schools/{school_id}/courses/{course_id}/modules/{module_id}/lessons/{lesson_id}

```
###  RS104 - URIs **must not** require trailing slashes

Request URIs must not expect a trailing slash or include them in links provided to clients. A trailing slash adds no semantic value and may cause confusion.

Acceptable:
```
GET /lessons
GET /lessons/{lesson_id}
```

Unacceptable:
```
GET /lessons/
GET /lessons/{lesson_id}/
```

While our APIs must not depend on trailing slashes, they need to be robust and able to function where trailing slashes are provided by clients. Consider [Postel's law] (https://devopedia.org/postel-s-law) here: *"Be  conservative in what you send, be liberal in what you accept."*

###  RS105 - URI path parameters **must** be named correctly

When a HTTP method operates on a specific resource a unique identifier will usually be required in the URI. We should name those id fields prefixed with the entity type's name to allow for clear structure when nesting resources.

Acceptable:
```
GET /modules/{module_id}
GET /modules/{module_id}/lesons/{lesson_id}

```

Uacceptable:
```
GET /modules/{id}
GET /modules/{id}/lessons/{id2}

```

###  RS106 - URI resource / endpoint names **must not** include 'api'

A resource name must not contain 'api' as a prefix in or a part of the path. The consumer already understands that an endpoint is an API so this is therefore superfluous. 

Acceptable:
```
GET /lessons/{lesson_id}

```

Uacceptable:
```
GET /lessonsApi/{lesson_id}
GET /api/lessons/{lesson_id}
GET /lessons-api/{lesson_id}

```

###  RS107 - URI path parameters **must** use snake_case

Applying a single stylistic preference across all APIs makes them appear more consistent and professional. There is no single de facto format to adopt here. We have embraced snake_case as it more closely aligns to the current code.

Acceptable:
``` json
GET /lesson/{lesson_id}
```

Unacceptable:
``` json
GET /lessons/{lessonId}
GET /lessons/{LESSONID}
GET /lessons/{lesson-id}
```

###  RS108 - Headers **must not** be prefixed by 'X-'

Historically, creators of REST APIs have often distinguished between standardised and unstandardised parameters by prefixing the names of unstandardized parameters with the string "X-" .  In practice, this is an awkward convention which is entirely unnecessary. For further reading, please review [RFC6648](https://www.rfc-editor.org/rfc/rfc6648).

###  RS109 - Headers **must** use Hyphenated-Pascal-Case notation

**Hyphenated-Pascal-Case** is the defacto standard for HTTP header names. Header names are case-insensitive which makes camel casing inappropriate. [RFC2822](https://www.rfc-editor.org/rfc/rfc2822#section-2.2) defines that a header name must be composed of printable US-ASCII characters, except colon. 

Acceptable:
``` json
My-Header-Name
```

Unacceptable:
``` json
myHeaderName
my_header_name
MYHEADERNAME
my-header-name
```
### RS110 - Autocomplete URI **must** be named correctly

Endpoint must explicitly include term ``autocomplete`` to clearly reflect its function.

``` json
GET /lessons/autocomplete 
GET /modules/autocomplete
``` 

## Rules: Documentation

### RS200 - Every API **must** provide a OpenAPI v3.0 specification (or above)
We use the [OpenAPI specification](https://swagger.io/specification/) as standard to define APIs. API designers are required to provide the API specification using a single self-contained **OpenAPI 3.x** file. 

### RS201 - API specifications **should** contain top level meta information:

To allow APIs to be aggregated and published in a consistent manner, we require the following metadata to be added to the OpenAPI document:

* `#/info/title` as (unique) identifying, functional descriptive name of the API
* `#/info/version` to distinguish API specifications versions following semantic rules
* `#/info/description` containing a proper description of the API
* `#/info/contact/{name,url,email}` containing the responsible team

###  RS202 - HTTP methods **must** be properly described

The **OpenAPI specification** provides the ability to associate two different textual descriptions to API methods. 

* `summary` is for a brief overview of the method. **Should** be at least 8 characters
* `description` is for a more detailed explanation of the the method. **Should** be at least 15 characters and include details of none obvious behaviour and alternative terminologies to better resonate the documentation to target audience.

Example:

``` yaml
summary: Create an employee
description: 
  Create a new staff member at a specific branch. If supplier portal is enabled,
  then this will trigger the onboarding flow for the supplier user
parameters:
  - name: supplierId
    in: path
    required: true
    schema:
      type: string
```


### RS203 - HTTP methods **must** document request and response payloads

The OpenAPI document must include the [schema definition](https://swagger.io/docs/specification/data-models/) for the types accepted and returned from every HTTP method. Developers need to be able to understand what can possibly be returned from a HTTP method to be able to write predictable, quality integrations.

Each status code that can returned should be documented and the schema referenced:

``` yaml
responses:
  '200':
    description: Success
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/ReadSupplierModel'
  '400':
    description: Bad Request
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/ProblemDetails'
  '404':
    description: Not Found
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/ProblemDetails'
```

###  RS204 - HTTP methods **must** document all query string parameters

Every query string parameter accepted by a HTTP method **must** include a textual description describing it's behaviour. 

Example:
``` yaml
paths:
  /contacts:
    get:
      parameters:
        - in: query
          name: status
          description: Only return contacts where the `status` field matches value(s) provided
          schema:
            type: string
            enum: [active, inactive]
```

### RS205 - HTTP methods **should** adhere to a specific documentation format for query strings

Most method parameters are query strings that perform filter operations. Behaviour can be concisely documented using the **"Only return X where Y"** format. 

To better guide developer expectation on filtering, we **should** try to reference the relevant field name that a filter operates upon:

Format examples:

* Only return contacts where the \`status` field matches value(s) provided
* Only return contacts where the \`name` field starts with the value provided
* Only return contacts where the \`type` field matches one or more of the value(s) provided

Note that since most OpenAPI document rendering tools support markdown, you can make that relationship even more clear by surrounding the field name with upticks (`).

###  RS206 - Payload schema fields **must** clearly document their purpose

Every field presented in request or response payloads needs a **textual description of it's purpose**. We should always try to make field names as self-explaining as possible, but it is not always sufficient. 

To avoid misuse and ambiguity, we **must** populate the associated `description` field to provide a better explanation of the data's purpose and the format in which it is presented. To ensure the content is useful, the description field should be **at least 15 characters in length.**

Model property with description:

``` yaml
email:
  type: string
  description: The employee email address
  nullable: true,
```

###  RS207 - Payload schema fields **should** provide an example value

Every field presented in request or response payloads needs a example value to be included. The `example` field further enhances developer experience by contextualising the data being present. Try to use example data that is valid for the given field type and that resembles the real-world.

Adding an example:
``` yaml
email:
  type: string
  description: The employee email address
  nullable: true
  example: craig.bryan@screwfix.com
```

### RS208 - Payload fields **must** document enumeration options

Our OpenAPI documents **must** provide the ability to surface the available textual options for enumerations used in request parameters or model fields:

``` yaml
paths:
  /items:
    get:
      parameters:
        - in: query
          name: sort
          description: Sort order
          schema:
            type: string
            enum: [ascending, descending]
```
Textual values mean payloads are human readable and therefore easier to troubleshoot. It also makes accidental breaking changes to APIs less likely (eg. inserting a new option at enumeration position 1 changes the meaning of subsequent values).

###  RS209 - HTTP methods **should** include standard 500 response

Every API we build has the possibility for unhandled error, corresponding to a `500 Internal Error`. As such, we should indicate that a 500 status response could be returned from every endpoint in it's documentation as standard.

``` json
responses:
  '500':
    description: Server Error
    content:
      application/problem+json:
        schema:
          $ref: '#/components/schemas/ProblemDetails'
```

###  RS210 - Payload field documentation **must not** be automatically generated

We should prioritise developer experience by ensuring the documentation we present has been carefully considered. While tools like Cursor/CoPilot can document fields automatically, this will typically result in poor quality content that provides no more insight about the meaning of a field than the tool can surmise from its name. 

The following patterns are a clear indication that field documentatation has been auto-generated and is likely of poor quality:

- **Full stop suffix**: full stops are typically inserted by documentation generators and look clunky in OpenAPI renderers.
- **Gets prefix**: a 'Gets' prefix is a clear indication of auto-generation, needlessly leaking out the implementation of the underlying class into the OpenAPI specification.

## Rules: Requests

###  RS300 - GET methods **must not** accept a request body

Parameters to `GET` methods **must** be passed by query string. If a `GET` request URI [becomes excessively long](https://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.2.1) and a body is required, consider using a `POST` verb.

### RS301 - GET methods **must not** have side effects

Our platform needs to behave predictably and adhere to REST standards. `GET` HTTP methods need only be concerned with retrieving data and **must not** additionally create or update data whilst doing so.

### RS302 - GET methods **must** support pagination when returning collections

Any endpoints that provide a collection response need to be able to divide results into smaller chunks (pages). 

### RS303 - GET methods **should** prefer offset pagination when returning collections

Paging methodology depends on the data and underlying storage technology, but where possible, prefer offset over tokenised pagination for simplicity.

Request parameters for offset pagination **must** include:
```
pageSize
```
The maximum number of records to return per page
``` 
pageNumber 
```
The number of pages to skip before fetching records

Example:
```
GET /v1/items?pageSize=100&pageNumber=3
```

###  RS304 - GET methods **must** envelope their result sets when returning collections

When implementing paging, the collection of models contained in the response payload must be nested in a pagination response wrapper, rather than just a naked array:

Example:

``` json
{
  "items": [
    {
      "id": 5514123,
      "name": "Dave Brown",
      "email": "dave.brown@example.com"
    }
  ],
  "pageNumber": 1,
  "pageSize": 10,
  "pageItemCount": 1
}
```
As in the above example, metadata about the page should be included alongside the results themselves:

* pageNumber - the page number requested by the user or applied as a default, where 1 represents the first page
* pageSize - the number of items per page requested by the user or applied as a default
* pageItemCount - the number of items returned in this response

### RS305 - GET methods **must** prefer query string parameters in [exploded form](https://swagger.io/docs/specification/serialization/) 

Request parameters that are array types should be populated by accepting multiple copies of the same query string, rather than a delimited list. This is the standard way that most model binding works and is supported by more REST SDKs than alternative methods.

This approach:

```
GET /tasks?status=started&status=complete
```
Over these:

```
GET /tasks?status=started,complete
GET /tasks?status=started|complete
```

### RS306 - DELETE requests **should** use soft deletes

Where the underlying data storage technology/schema supports, prefer to [soft delete](https://www.brentozar.com/archive/2020/02/what-are-soft-deletes-and-how-are-they-implemented/) resources rather than destroy them.   

###  RS307 - PUT, PATCH, POST and DELETE requests **should** not require query strings

If a request changes the state of a resource, then data should be passed using the request body, rather than passing parameters as query strings. 

### RS308 - GET methods for collections **must** use sort parameters consistently

For collection resources that need to support sorting, we should standardise our sort parameters. This ensures developers can easily locate them and can implicitly understand how they work. Provide the following query string parameters:

* sortBy - an enumeration that presents the available sort orders for the method
* direction - an enumeration with two options (ascending, descending)

Usage:
```
GET /tasks?sort_by=created_at&direction=descending
```

### RS309 - GET methods **should** prefer simple query strings

Where a collection based resource needs to provide filtering, we should prefer a set of straightforward, single purpose query string parameters. While allowing parameters to embed a filter operation (similar to [OData](https://www.odata.org/)) is powerful and can reduce the total number of query string parameters, it adds massive complexity when building/consuming/documenting APIs and vastly increases the testing surface area.

Prefer:
```
GET /tasks?intParamFrom=5
```

To:
```
GET /tasks?intParam=gt:5
```

###  RS310 - Query string parameters **must** be optional 

Since query string parameters are not a fixed part of any URI, we can align our APIs with developer expectation and not not generally require them to be provided as part of a request. Consider applying default values to query strings rather than enforcing their presence. 

Prefer this:
``` json
{
  "parameters": [
    {
      "in": "query",
      "name": "limit",
      "schema": {
        "type": "integer",
        "default": 2
      },
      "required": false,
      "description": "The number of items to return"
    }
  ]
}
```

Over this:
``` json
{
  "parameters": [
    {
      "in": "query",
      "name": "limit",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "The number of items to return"
    }
  ]
}
```

###  RS311 - Query string parameters **must** be snake_case

Applying a single stylistic preference across all APIs makes them appear more consistent and professional. There is no single de facto format to adopt here. We have embraced `snake_case` as it more closely aligns to the current code 
* Words are lowercase
* Words are separated by underscores (_)
* No spaces or other punctuation is used
```
Example: my_variable_name
```

Acceptable:
``` json
"parameters": [
  {
    "name": "sort_by",
    "in": "query",
    "description": "The field to sort items in the page by",
    "schema": {
      "$ref": "#/components/schemas/NoteTypeSortOptions"
    }
  }
]
```

Unacceptable:
``` json
"parameters": [
  {
    "name": "SortBy",
    "in": "query",
    "description": "The field to sort items in the page by",
    "schema": {
      "$ref": "#/components/schemas/NoteTypeSortOptions"
    }
  }
]
```

### RS312 - Autocomplete **must** use term parameter

 To ensure simplicity and standardization across autocomplete functionalities, each autocomplete endpoint must accept a universal query parameter named ```term```. This parameter will contain the user's input string, serving as the basis for generating relevant autocomplete suggestions.

``` yaml
parameters:
  - in: query
    name: term
    required: true
    schema:
      type: string
``` 

## Rules: Responses

###  RS400 - HTTP responses **must** return an acceptable status code

Our platform should return standard, REST-aligned [HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) to indicate the outcome of a request. 

* Status codes in the `2xx` range: the request was fulfilled 
* Status codes in the `4xx` range: the request was not fulfilled due to an badly formed or limitation applied upon the request
* Status codes in the `5xx` range: the request was not fulfilled due to an unhandled platform fault

The following status codes are acceptable responses from our platform:

| Code | Description |
|----|----|
| 200 OK| Request has been fulfilled. Typically used to indicate a successful response to a GET request. |
| 201 Created| Request has been fulfilled and a new resource has been created. Typically used to indicate a successful response to a POST request. |
| 202 Accepted| Request has been received but not yet acted upon. Typically used when triggering an asynchronous process where success cannot yet be determined. | 
| 204 No Content | Request has been fulfilled but there is no need to send back a payload. Typically used to indicate a successful response to a PUT/PATCH/DELETE request.|
| 400 Bad Request | Request was malformed or not understood by the server. Used to indicate failed where a more specific code does not fit (eg. where **documented** business logic prevents a request from being executed) |
| 401 Unauthorized | Request credentials incorrect or not present. Indicates that the consumer's request is not sufficiently authentication to interact with endpoint. |
| 403 Forbidden | Request did not provide credentials with sufficient scope to be fulfilled. Indicates that the consumer does not have the required permissions to interact with this endpoint. |
| 404 Not Found | Requested resource was not found. Indicates that the consumer attempted to fetch or update a resource that could not be located by the id **provided in the URI**. |
| 422 Unprocessable Entity | Request was not accepted because a validation error has occurred. Used to indicate a problem with the provided payload for POST/PUT/PATCH requests, where the data provided falls outside of expected ranges (eg. payload containing `startDate` occuring after `endDate`)|
| 429 Too Many Requests | Request was not accepted because the application has exceeded the rate limit. This indicates that the consumer has exceeded the amount of requests that their quota allows.  |
| 500 Internal Error | Request triggered an unexpected error. This indicates an error with the API that needs attention from Handshaik Developers.|

### RS401 - POST responses **must** include a location header

After creating a new resource, the response **must** include a `Location` header indicating the relative URI of where the newly created resource can be obtained from:

Response example:
``` json
HTTP/1.1 201 Created
...
Location: /users/12345
Content-Type: application/json
...
```
### RS402 - POST responses **may** include a body

For the majority of use cases, it is unnecessary for `POST` requests to return a representation of the resource that was just created. If the consumer wants a fresh copy of the data they already hold, they can choose to obtain from the URI provided in the `Location` header. 

There is usually no need for a `POST` endpoint to fulfill both functions so it is preferable to not provide one, though this guideline recognises that it not always possible.

###  RS403 - PUT/PATCH/DELETE responses **must not** return a response body when status code is 204

The `204 No Content` status code indicates that no content will be sent to the consumer. Our APIs should therefore not include a response body for responses using this status code.

## Rules: Payload Format

###  RS500 - APIs **must** support JSON as payload data interchange format

All top-level data structures provided as body payload for HTTP requests and responses should be JSON complying to [RFC 7403](https://www.rfc-editor.org/rfc/rfc7493). This applies to both collections and single resource instances.

Requests and responses must indicate this by passing a `Content-Type` header set to `application/json`.

###  RS501 - APIs **must** use snake_case for request / response payload schema

Applying a single stylistic preference across all APIs makes them appear more consistent and professional. There is no single de facto format to adopt here. We have embraced snake_case as it more closely aligns to the current code. Note that for schema attributes, we accept that field names may occasionally need to contain abbreviations for brevity.

Acceptable:
```
{
   "snake_case_field" : 12345,
   "maxRPM" : 10000
}
```
Unacceptable:
```
{
   "PascalCaseField" : 12345,
   "kebab-case-field" : 12345,
   "camelCaseField" : 12345
}
```

### RS502 - APIs **must** use standard formats for date and times

All timestamps **must** be represented in Coordinated Universal Time (UTC) and formatted using [ISO 8601](https://www.w3.org/TR/NOTE-datetime). While the standard allows for time zone offsets to be presented, we consider localisation to be a user interface concern so our APIs **must** always present times as UTC without offset.

Acceptable date only:
``` json
{
  "sent_at": "2022-01-09",
}
```

Acceptable date & time:
``` json
{
  "sent_at": "2022-01-09T13:32:03.6759Z",
}
```
### RS503 - Payloads **should not** be overly structured

We should carefully consider the API consumer before arbitrarily grouping data. Where possible, data should be flattened in the JSON representation, unless a `map` or `array` data type is required.

While having reusable payload models for common structures can help to enforce payload standards across a service, there are trade-offs:

* Every integrator needs to build around the complexity of hierarchical data structures. Since the same API is likely to consumed many times, we should [prioritise the consumers developer experience](#prioritize-developer-experience) over our own.
* Sharing payload models between endpoints makes it difficult to evolve each method independently. This can result in accidental introduction of change into HTTP methods.

Flattened:
```
{
  "company": "Handshaik",
  "website": "https://www.Handshaik.com/",
  "street": "1 High Street,
  "town": "Leeds",
  "postalCode": "LS1 1AAU",
  "country": "England"
}
```
Unnecessary structure
```
{
  "company": "Handshaik",
  "website": "https://www.Handshaik.com/",
  "address": {
    "street": "1 High Street",
    "town": "Leeds",
    "postalCode": "LS1 1AA"
    "country": "England"
  }
}
```

###  RS504 - Boolean fields **must** be consistently named

Boolean payload fields must be prefixed with `is` to provide an indication of their purpose & data type at a glance.

Example:
```
{
   "is_flagged" : true
}
```

###  RS505 - Date & Timestamp fields **must** be consistently named

Payload fields that present a date or a date & time must be post fixed with `at` to provide an indication of their purpose & data type at a glance.

Example:
```
{
   "created_at" : "2022-01-09T13:32:03.6759Z"
}
```

## Rules: Errors

###  RS600 - Error responses **must** be presented in a standard format

All services should provide a unified standard for actionable feedback to integrators for `4xx` and `5xx` code range responses.

[RFC 7807](https://tools.ietf.org/html/rfc7807) defines a **Problem JSON object** using the media type `application/problem+json` to provide an extensible human and machine readable failure information beyond the HTTP response status code;. Every endpoint must be capable of returning a **Problem JSON** on client usage errors (4xx status codes) as well as server side processing errors (5xx status codes).

```
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json

{
   "type": "https://example.com/probs/out-of-credit",
   "title": "You do not have enough credit.",
   "detail": "Your current balance is 30, but that costs 50.",
   "instance": "/account/12345/msgs/abc",
   "balance": 30,
   "accounts": ["/account/12345", "/account/67890"]
}
```
* `type`: a URI reference that uniquely identifies the problem type
* `title`: a short readable error statement
* `status`: the original HTTP status code from the origin server
* `detail`: a longer text explanation of the issue
* `instance`: a URI that points to the resource that has issues



### RS601 - Error responses **must** not leak internals

If a REST service encounters an internal error that triggers a `500` response code, we should ensure that the `ProblemDetails` JSON object does not contain internal information about the error, such as a stack trace.

Stack traces can leak the internal workings of our services which is not only unhelpful to third party developers, but is also a information leaking concern.

## Rules: Versioning

###  RS700 - API specifications **must** use URI based versioning

The URI should include `/vN` with the major version (N) as a prefix. URL-based versioning is preferred for simplicity vs a header-based solution. Start with v1.

Examples:
```
https://api.Handshaik.com/v1/suppliers
https://api.Handshaik.com/v2/contacts
```
### RS701 - API version **must** only be incremented on breaking changes

We consider the following to be examples of breaking changes that would warrant an increment in version number:

* Renaming or removing an existing field or endpoint
* Changing the URL structure of an existing endpoint
* Changing the filters that an existing endpoint provides
* Changing error response codes or messages that an endpoint provides
* Modifying or adding a new validation to an existing resource
* Changing the data type of an existing field
* Requiring a parameter that wasn't previously required

### &cross;RS702 - API specification **should** include deprecation metadata

If a HTTP method, parameter, payload or field should be deprecated, then the OpenAPI specification **must** be updated to [present this information](https://swagger.io/docs/specification/paths-and-operations/) to developers.

## Rules: Security

###  RS800 - Public APIs **must** exclusively use HTTPS

We should ensure that we only publish APIs using secure encrypted channels. This protocol keeps communications secure so that malicious parties can't observe what data is being sent.

## Rules: Monitoring

### RS900 - APIs **should** include a health check path (/healthcheck)
As a simple solution for pull-based monitoring, every REST API we build should include a health check endpoint. Located at `/healthcheck` , the check should validate the status of the service by assessing availability of dependencies, database connections, endpoint connections, and resources.
