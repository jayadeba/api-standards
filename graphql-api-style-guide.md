
## Overview

As described at `https://graphql.org/ `, GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. GraphQL provides a complete and understandable description of the data in your API, gives clients the power to ask for exactly what they need and nothing more, makes it easier to evolve APIs over time, and enables powerful developer tools.

The rules of the language are governed by the [RFC Specification](https://spec.graphql.org/June2018/). It is the goal of this document to outline the set of rules, standards, and conventions that apply to the design of GraphQL APIs at PayPal.

It is important to remember that your API is created for and consumed by human beings, so your first priority should always be on enabling other developers to do their jobs effectively.


# Table Of Contents
* [Interpreting the Guidelines](#interpretation)
	* [Terms Used](#terms)
		* [Entity](#entity)
		* [Service](#service)
		* [Entity Lifecycle API Operations](#lifecycle)
		* [Service API Operations](#service-op)
* [Naming Conventions](##naming-conventions)
	* [GraphQL URLs](#urls)
	* [Fields](#fields)
	* [Queries](#queries)
	* [Mutations](#mutations)
	* [Input Objects](#input)
	* [Responses](#response)

* [Directives](#graphql-directives)
  * [authPolicy](#authpolicy)
  * [dataClassification](#data-classification)
  * [visibility](#visibility)
  * [labels](#labels)
  * [slo](#slo)
  * [stringLength](#stringlength)
  * [pattern](#pattern)
  * [listLength](#listlength)
  * [range](#range)
  * [requestHeader](#request-header)
  * [responseHeader](#response-header)
 
* [Error Handling](#error-handling)
	* [GraphQL Error Schema](#schema)
	* [Error Class](#error-class)
	* [Input Validation Errors](#input-errors)


<h1 id="interpretation">Interpreting The Guidelines</h1>
<h2 id="terms">Terms Used</h2>

The following terms and concepts from domain-driven design (DDD) are used throughout this document.

<h4 id="entity">Entity</h4> 
An object primarily identified by its identity and has a thread or continuity. All entities have a lifecycle, from creation through archival or deletion.

<h4 id="service">Service</h4>
A service describes an activity or action, stands on its own in the domain, and is solely identified by the action/activity it implements. Services usually operate on many entities to complete an activity and don't have states of their own. Services implement procedural or controller style actions. **Note**: The term "Service" here MUST not be confused with a Micro-service.

<h4 id="lifecycle">Entity Lifecycle API Operations</h4>
The API lifecycle operations that operate on an entity, from creation through deletion or archival. Example of entity lifecycle operations are: Create, Read, Update, Delete, and other entity state change operatiions.

<h4 id="service-op">Service API Operations</h4>
The API operation that expose a service (action or activity) to its clients.

<h1 id="naming-conventions">GraphQL API Naming conventions </h1>

<h2 id="urls">GraphQL URLs</h2>

All GraphQL API requests are routed through the URL `https://apihost/graphql. The requests are then federated to each domain's GraphQL server based on where it is hosted. 

<h2 id="fields">Fields</h2>

* All field names SHOULD use the following conventions; if the field name consists of only one word, express/spell the word in all lower case letters (e.g. amount). If it contains more than one word, capitalize the first letter of each subsequent word (e.g. `checkingAccountNumber`).
* Entries (values) of an enumeration SHOULD be composed of only upper-case alphanumeric characters and the underscore character (`_`).
* Array names SHOULD be in plurals.
* Acronyms SHOULD be avoided (e.g. use `fooSpecificBalance` instead of `FSB`). The exception is the popular industry acronym for a concept (example: cvv, csc etc.), if it exists, SHOULD be used. 
* Collection sounding suffixes in field names or query parameters SHOULD be avoided, e.g. use `errors` in lieu of `errorList`.
* Names SHOULD NOT use the words, such as `encrypted`, `decrypted`, `encoded`, `info`, `additional`, `auxiliary`, `supplementary`, or any name that describes the underlying implementaton (e.g. Names like `hashedId` `aliasHmac` etc. SHOULD be avoided).
* All names (fields, query and path) SHOULD be in American english, e.g. favor vs favour, authorized vs authorised.
* Generally accepted conventions for names SHOULD be preferred over other names for fields or query parameters, e.g. `id` SHOULD be used to describe an identifier everywhere; long names like identifier SHOULD be avoided; in case of a conflict, an appropriate prefix or suffix SHOULD be used, such as `merchantId`, `partnerId` etc.
* Field or query parameter names that represent a RFC3339 full-date SHOULD end with the suffix `Date`, e.g. `createDate`, `updateDate`, `deleteDate`, `dueDate`.
* Field or query parameter names that represent a RFC3339 date-time or full-time SHOULD end with the suffix `Time`, e.g. `createTime`, `updateTime`, `deleteTime`, `dueTime`.
* All GraphQL type names SHOULD be in CamelCase, with the first letter of each word in uppercase (e.g. **LanguageCode**).


<h2 id="queries">Queries</h2>

* All GraphQL queries on entity collections MUST use plural nouns as names. Examples are, `orders`, `invoices`, and `disputes`
* GraphQL queries that explicitly lookup an entity by its primary key SHOULD use a singular noun as names. Examples are, `order`, `invoice`, `dispute`. 
* For queries that define generic searches, the prefix `search` MUST be used. Example: `searchContent`

<h2 id="mutations">Mutations</h2>


For all [Entity Lifecycle Mutations](#lifecycle), the following conventions MUST be used:

* For create operation on an enity, the prefix `Create` MUST be used.
* For update operation on an enity, the prefix `Update` MUST be used.
* For all deletion operation on an enity, the prefix `Delete` MUST be used.
* The entity name MUST follow the prefix `Create`, `Update`, or `DELETE`. For example, `createOrder`, `updateOrder`, `DeleteOrder`.

For all [Service mutations](#service-op) (mutations that implement an `action` or `activity` instead of a `thing`), a verb MUST be used to describe the activity or action. For example, `notifyUser`, `evaluateCreditEligibility`.

Lastly, when naming mutations, wherever possible, names that describe the intent of the operations SHOULD be preferred over generic CRUD style names. For example, a name `setUserProfile` is preferred over `updateUser`, for making the user profile changes.


<h2 id="input">Input Objects</h2>

* All input objects MUST use `Input` as the suffix.

<h2 id="response">Responses</h2>

* All entity lifecycle mutations MUST have the response object named after the Entity name. For example, CreateOrder/UpdateOrder/DeleteOrder mutation MUST have the response object named as `Order`.
* All service mutations MUST have the response object named after the service result, in the form of a noun. For example, the service API operation, `createCreditEligibility` SHOULD have the response object named as `CreditEligibilityEvaluation`.
* Queries that operate on an Entity Collections, the response object MUST have the suffix `Connection`. For example, a GraphQL query `orders` would name `OrderConnection` as the response object (**Note**: Queries on an entity collection MUST implement pagination. Please check GraphQL API patterns for details on pagination). 


<h1 id="graphql-directives">GraphQL Directives </h1>


<h2 id="authpolicy">authPolicy</h2>

The `authPolicy` directive SHOULD be used to describe all authorization requirements for queries, mutatons and their request/respons objects (as applicable). 

**Example**:


~~~
type Query {

  orders(...): [Order]
    @authPolicy(
      scopes: ["FOO_SCOPE"]
    )
    
}
~~~


<h2 id="data-classification">dataClassification</h2>


The `dataClassification` directive SHOULD be used to describe the data classification of request and response elements.

**Example**:

~~~

type User {

    email: String! @dataClassification(dataClass: "Foo",dataCategory:"Bar")
    
}

~~~


<h2 id="visibility">visibility</h2>


The directive `visibility` MUST be used to describe the visibility of a query, mutation, or a request/response element. The values can be one of the following: EXTERNAL, PARTNER, INTERNAL


**For type field elements:**

~~~

type User {

    email: String! @visibility(extent: "PARTNER")
    
}

~~~

**For Operations (Queries/Mutations):**

~~~
type Query {

  assets(): [Asset] @visibility(extent: "PARTNER")
  
}

~~~


<h2 id="labels">labels</h2>


The directive `labels` is used for the purpose of grouping several related queries or mutations. A label name is usually the bounded context (aka namespace) names in the domain. The value of the directive is also used for the purpose of discovery


**For Operations (Queries/Mutations):**

~~~

type mutation {

    createUser(input: CreateUserInput): User @labels(value: ["identity"])
   
    addBank(input: addBankInput): Bank @labels(value: ["wallet"])
  

}

~~~

<h2 id="slo">slo</h2>


The directive `slo` SHOULD be used to describe the service level objectives of a GraphQL operation. Given below is the example usage;


**For Operations (Queries/Mutations):**

~~~

type mutation {

    createUser(input: CreateUserInput): User 
        @slo(responseTime95thPercentile: 100, errorRate: 0.001)

}

~~~

<h2 id="stringlength">stringLength</h2>


The directive `stringLength` is used to describe lenth validation requirements of a string. Given below is the example usage;


~~~

type User {

  firstName: String! @stringLength(min: 1, max: 255)

  
}

~~~

<h2 id="pattern">pattern</h2>


The directive `pattern` is used to describe format validation requirements of a string. Given below is the example usage;



~~~

type User {

  userId: String! @pattern(regexp: "^[0-9a-zA-Z]*$")

  
}

~~~

<h2 id="listlength">listLength</h2>


The directive `listLength` is used to describe the length validation requirements of an array. Given below is the example usage;



~~~

type User {

  aliases: [String!] @listLength(min: 1, max: 5)

  
}

~~~

<h2 id="range">range</h2>


The directive `range` is used to describe the range validation requirements of a number. Given below is the example usage;



~~~

type User {

  total_items: Int! @range(min: 1, max: 32767)

  
}

~~~


<h2 id="request-header">requestHeader</h2>

The directive `requestHeader` is used to describe the request headers of a GraphQL Query/Mutation. The directive SHOULD be repeated for each request header. 


**For Operations (Queries/Mutations):**

~~~

type mutation {

    createCardArt(input: CardArtInput): CardArt
        @requestHeader(name: "Idempotency-Key", 
            description: "The request idempotency key.", required: true)

}

~~~

<h2 id="response-header">responseHeader</h2>


The directive `responseHeader` is used to describe the response headers of a GraphQL Query/Mutation. The directive SHOULD be repeated for each response header.

**For Operations (Queries/Mutations):**

~~~

type mutation {

    createCardArt(input: CardArtInput): CardArt
        @responseHeader(name: "Foo-Id", 
            description: "The response correlationId.", required: true)

}

~~~



<h1 id="error-handling">Error Handling</h1>


<h3 id="schema">GraphQL Error Schema</h3>

All the GraphQL errors SHOULD be propagated via the `extensions` element of the errors array. Given below is the schema:

* **`extensions`**: Must have a map as its value. This entry is reserved for implementors to add additional information to errors, and there are no additional restrictions on its contents.
    * **`name`**: A human-readable name for the error. The `name` describes the class of an error. The name is a `CAP_SNAKE_CASE` fixed string described in the section [Error Class](#error-class).
    * `details`: An array that contains detailed information as to how the error occurred.
      * `issue`: An application level error code.`issue` SHOULD be a `CAP_SNAKE_CASE` string.
      * `description`: A human-readable explanation, describing an `issue`. The `description` MAY change over the lifetime of an API, so clients **SHOULD NOT** depend on this value remaining constant.

**Example**:

~~~
Status: 200 OK
{
  "errors": [
    {
      "message": "The requested action could not be performed, semantically incorrect, or failed business validation.",
      "path": [
        "sender"
      ],
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "extensions": {
        "name": "UNPROCESSABLE_ENTITY",
        "details": [
          {
            "issue": "SENDER_HAS_NEGATIVE_BALANCE",
            "description": "Cannot process the payment as the sender has negative balance."
          }
        ]
      }
    }
  ],
  "extensions" : {
      "FooId":"123456789"
  }
}
~~~

<h3 id="error-class">Error Class</h3>

The following listed error classes SHOULD be used in all GraphQL error responses.

|Error Class | Description |
|:----------|:-------------|
|`INVALID_REQUEST` | Request is not well-formed, syntactically incorrect, or violates schema.| 
|`AUTHENTICATION_FAILURE` | Authentication failed due to missing Authorization header, or invalid authentication credentials.|
|`NOT_AUTHORIZED ` | Authorization failed due to insufficient permissions.|
|`UNPROCESSABLE_ENTITY` | The requested action could not be performed, semantically incorrect, or failed business validation.|
|`INTERNAL_ERROR` | An internal server error has occurred.   |

<h3 id="input-errors">Input Validation Errors</h3>

In validating requests, APIs MAY further describe fine grained validation errors using a sub category of error codes using the `issue` field of `details` array in the error. Given below are some of the suggested fine grained error codes. 

| **Issue value** | **Description** |
|:----------|:------------|
| `MISSING_REQUIRED_PARAMETER` | A required field or parameter is missing. |
| `INVALID_STRING_MIN_LENGTH` | The value of a field is too short. |
| `INVALID_STRING_MAX_LENGTH` | The value of a field is too long. |
| `INVALID_STRING_LENGTH` | The value of a field is either too short or too long. |
| `INVALID_PARAMETER_SYNTAX` | The value of a field does not conform to the expected format. |
| `INVALID_INTEGER_MIN_VALUE` | The integer value of a field is too small. |
| `INVALID_INTEGER_MAX_VALUE` | The integer value of a field is too large. |
| `INVALID_PARAMETER_VALUE` | The value of a field is invalid. |
| `INVALID_ARRAY_MIN_ITEMS` | The number of items in an array parameter is too small. |
| `INVALID_ARRAY_MAX_ITEMS` | The number of items in an array parameter is too large. |
| `INVALID_ARRAY_LENGTH` | The number of items in an array parameter is too small or too large. |
| `INVALID_PATCH_PATH` | The value of `path` in a HTTP PATCH request item is invalid. |
| `CANNOT_REUSE_IDEMPOTENCY_KEY` | The idempotency key cannot be reused for a different request payload. |
| `READ_ONLY` | The field in the request body is read only so it can't be modified. |
| `RESOURCE_CONFLICT` | The server has detected a conflict while processing this request. |

