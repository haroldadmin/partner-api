# Partner API

> **WARNING**
> This is just the first draft of the API and subject to change during development.

## Scope
The scope of this API is to allow upcover's insurance partners to:

1. Get a quick quote
2. *Redirect* users to get a full quote and purchase cover from a co-branded experience
3. Fetch insurance/cover status for any user

## Partner API vs Widgets
upcover plans to offer two sperate yet complementary ways to help partners offer insurance to their customers. These are:

1. Partner API
2. Widgets

This document refers to the first of these approaces i.e. the Partner API. The widgets on the other hand will offer a no frills, easy to integrate co-branded solutions that can be added to any flow. Partners may combine these experiences so that users can be shown quick quotes using a partner API without the need for user to answer any basic questions and then redirect the users to either a co-branded site or an embedded widget to complete the full quote and buying experience.

## OpenAPI Schema Files
Please, refer to OpenAPI schema files (`.yaml` and/or `.json`) for detailed reference of the API. This document only provides a general overview and flow of the APIs.

## Environments
The endpoints below are tentative, however upcover will still offer three seperate environments for partner APIs.

> **Development**
> `https://devapi.upcover.com`

> **Staging**
> `https://stageapi.upcover.com`

> **Production**
> `https://prodapi.upcover.com`


## Responses

In general, Partner APIs may return any of the following HTTP status codes on API requests:

Code | Description 
--- | ---
200 | Successful Operation
400 | Invalid Request
403 | Authentication Failure
500 | Unknown Error

All API responses follow and consistent response wrapper. 

#### Error Response
> `{code: number, message: string, success: boolean = false}`
#### Success Response
Response objects described in any APIs will be passed in the *data* field of the success response.

> `{code: number = 200, message: string, success: boolean = true, data: <payload>}`

## Insurance Products
Currently, the following insurance products are supported, with more coming soon.

- HEALTH_PROFESSIONAL
- MANAGEMENT_LIABILITY
- CYBER_LIABILITY

## Authentication
> **POST**
> `/api/v1/token`

upcover requires a valid JWT token for access to any/all of its APIs. This API can be used to *login* to upcover and get an access token. 

> **Request OpenAPI Schema**
> `{"type":"object","properties":{"username":{"type":"string"},"password":{"type":"string"},"authType":{"type":"string","enum":\["USERNAME\_PASSWORD"\]}}}`

> **Response OpenAPI Schema**
> `{"type":"object","properties":{"accessToken":{"type":"string"},"expiresInSeconds":{"type":"integer","format":"int64"}}}`

## Occupations
> **GET**
> `/api/v1/occupations`

This API can be used to either list all occupations or search for occupations given a keyword. The results can be filtered by providing an optional insuranceProduct filter as well.

**OpenAPI Schema**
```
      {
       "parameters": \[
          {
            "name": "search",
            "in": "query",
            "required": false,
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "productFilter",
            "in": "query",
            "required": false,
            "schema": {
              "type": "string",
              "enum": \[
                "HEALTH\_PROFESSIONAL",
                "MANAGEMENT\_LIABILITY",
                "CYBER\_LIABILITY"
              \]
            }
          }
        \]
        }
