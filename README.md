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
301 | Redirect
400 | Invalid Request
403 | Authentication Failure
500 | Unknown Error

All API responses follow consistent response wrapper and structure. 

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

**Request OpenAPI Schema**
```javascript
{
   "type":"object",
   "properties":{
      "username":{
         "type":"string"
      },
      "password":{
         "type":"string"
      },
      "authType":{
         "type":"string",
         "enum":[
            "USERNAME_PASSWORD"
         ]
      }
   }
}
```

**Response OpenAPI Schema**
```javascript
{
   "type":"object",
   "properties":{
      "accessToken":{
         "type":"string"
      },
      "expiresInSeconds":{
         "type":"integer",
         "format":"int64"
      }
   }
}
```

Any subsequent request to other API must contain Bearer token in the http `Authorization` header

```javascript
{
   "securitySchemes":{
      "bearerToken":{
         "type":"http",
         "scheme":"bearer",
         "bearerFormat":"JWT"
      }
   }
}
```
## Occupations
> **GET**
> `/api/v1/occupations`

This API can be used to either list all occupations or search for occupations given a keyword. The results can be filtered by providing an optional insuranceProduct filter as well.

**Request OpenAPI Schema**
```javascript
{
   "parameters":[
      {
         "name":"search",
         "in":"query",
         "required":false,
         "schema":{
            "type":"string"
         }
      },
      {
         "name":"productFilter",
         "in":"query",
         "required":false,
         "schema":{
            "type":"string",
            "enum":[
               "HEALTH_PROFESSIONAL",
               "MANAGEMENT_LIABILITY",
               "CYBER_LIABILITY"
            ]
         }
      }
   ]
}
```
**Response OpenAPI Schema**

```javascript
{
   "type":"object",
   "properties":{
      "data":{
         "type":"array",
         "items":{
            "type":"object",
            "properties":{
               "name":{
                  "type":"string"
               },
               "occupationId":{
                  "type":"integer",
                  "format":"int64"
               },
               "supportedProducts":{
                  "type":"array",
                  "items":{
                     "type":"string",
                     "enum":[
                        "HEALTH_PROFESSIONAL",
                        "MANAGEMENT_LIABILITY",
                        "CYBER_LIABILITY"
                     ]
                  }
               }
            }
         }
      },
      "code":{
         "type":"integer",
         "format":"int32"
      },
      "message":{
         "type":"string"
      },
      "success":{
         "type":"boolean",
         "default":true
      }
   }
}
```

## Limits
For some insurance products individual limits are required e.g. HEALTH_PROFESSIONAL requires that `piPreferredLimit` and `pandPLPreferredLimit` are specified seperately. For all other products only `preferredLimit` is required.

## Excess
Currently, the insurance excess can not be changed and is set at *500 AUD* in the API. We might add the ability to change this in the future.

## Quick Quote
> **POST**
> `/api/v1/quickQuote`

There are only few fields that are required to get a quick quote, yet the request allows the partners to input as much information as possible as that will help speed up the full quote process.

>**Quote ID**
>Quote ID which is a `udid` based string ID is returned in case of a successful quick quote. The same id is later used against full quote and can be used to fetch quote status etc as well.

**Request OpenAPI Schema**

```javascript
{
   "required":[
      "clientInformation",
      "occupationId",
      "product"
   ],
   "type":"object",
   "properties":{
      "product":{
         "type":"string",
         "enum":[
            "HEALTH_PROFESSIONAL",
            "MANAGEMENT_LIABILITY",
            "CYBER_LIABILITY"
         ]
      },
      "occupationId":{
         "type":"integer",
         "format":"int64"
      },
      "averageRevenue":{
         "minimum":0,
         "type":"integer",
         "format":"int64",
         "default":100000
      },
      "preferredLimit":{
         "maximum":20000000,
         "minimum":500000,
         "type":"integer",
         "format":"int64",
         "default":10000000
      },
      "piPreferredLimit":{
         "maximum":20000000,
         "minimum":500000,
         "type":"integer",
         "format":"int64",
         "default":1000000
      },
      "clientInformation":{
         "$ref":"#/components/schemas/ClientInformation"
      },
      "pandPLPreferredLimit":{
         "type":"integer",
         "format":"int64"
      }
   }
}
```

**ClientInformation Schema**

```javascript
{
   "required":[
      "emailAddress",
      "firstName",
      "lastName"
   ],
   "type":"object",
   "properties":{
      "firstName":{
         "type":"string"
      },
      "lastName":{
         "type":"string"
      },
      "abn":{
         "maxLength":11,
         "minLength":11,
         "type":"string"
      },
      "entityType":{
         "type":"string",
         "default":"SOLE_TRADER",
         "enum":[
            "ASSOCIATION",
            "COOPERATIVE",
            "EMPLOYEE",
            "PARTNERSHIP",
            "PRIVATE_COMPANY",
            "PUBLIC_COMPANY",
            "SOLE_TRADER"
         ]
      },
      "companyName":{
         "type":"string"
      },
      "mobileNumber":{
         "type":"string"
      },
      "landlineNumber":{
         "type":"string"
      },
      "addressLine1":{
         "type":"string"
      },
      "addressLine2":{
         "type":"string"
      },
      "suburb":{
         "type":"string"
      },
      "state":{
         "type":"string",
         "default":"NSW",
         "enum":[
            "ACT",
            "NSW",
            "NT",
            "QLD",
            "SA",
            "TAS",
            "VIC"
         ]
      },
      "postalCode":{
         "maxLength":4,
         "minLength":3,
         "type":"string"
      },
      "emailAddress":{
         "type":"string",
         "format":"email"
      }
   }
}
```

**Response OpenAPI Schema**

```javascript
{
   "type":"object",
   "properties":{
      "quoteId":{
         "type":"string",
         "format":"uuid"
      },
      "product":{
         "type":"string",
         "enum":[
            "HEALTH_PROFESSIONAL",
            "MANAGEMENT_LIABILITY",
            "CYBER_LIABILITY"
         ]
      },
      "annualPrice":{
         "type":"number",
         "format":"double"
      },
      "preferredLimit":{
         "type":"integer",
         "format":"int64"
      },
      "piPreferredLimit":{
         "type":"integer",
         "format":"int64"
      },
      "pandPLPreferredLimit":{
         "type":"integer",
         "format":"int64"
      }
   }
}
```


## Full Quote Link

> **GET**
> `/api/v1/linkFullQuote`

This API can be used to either automatically redirect the user to co-branded full quote page where user can answer the declaration questions and pay to bind the policy OR use the link returned from this API to load (`src`) a dynamic/re-sizeable embeddable widget so the user never has to leave the partner's site to complete the experience.

**Request OpenAPI Schema**

```javascript
{
   "parameters":[
      {
         "name":"quoteId",
         "in":"query",
         "required":true,
         "schema":{
            "type":"string",
            "format":"uuid"
         }
      },
      {
         "name":"redirect",
         "in":"query",
         "required":false,
         "schema":{
            "type":"boolean",
            "default":true
         }
      }
   ]
}
```

**Response OpenAPI Schema**

Either returns the url OR redirects w/ 301 header.

```javascript
{
   "type":"string"
}
```

## Quote Status
> **GET**
> `/api/v1/quoteStatus`

This API can be used to check status of any quote.

**Request OpenAPI Schema**

```javascript
{
   "parameters":[
      {
         "name":"quoteId",
         "in":"query",
         "required":true,
         "schema":{
            "type":"string",
            "format":"uuid"
         }
      }
   ]
}
```

**Response OpenAPI Schema**

In case the quote was binded the status will reflect `PURCHASED` status along with more details being added in the `purchasedPolicy` field.

```javascript
{
   "type":"object",
   "properties":{
      "status":{
         "type":"string",
         "default":"PURCHASED",
         "enum":[
            "QUICK_QUOTE",
            "FULL_QUOTE",
            "PURCHASED",
            "CANCELLED"
         ]
      },
      "purchasedPolicy":{
         "type":"object",
         "properties":{
            "quoteId":{
               "type":"string",
               "format":"uuid"
            },
            "product":{
               "type":"string",
               "enum":[
                  "HEALTH_PROFESSIONAL",
                  "MANAGEMENT_LIABILITY",
                  "CYBER_LIABILITY"
               ]
            },
            "occupationId":{
               "type":"integer",
               "format":"int64"
            },
            "inceptionDate":{
               "type":"string",
               "description":"ISO format date YYYY-MM-DD"
            },
            "purchaseDate":{
               "type":"string",
               "description":"ISO format date YYYY-MM-DD"
            },
            "expiryDate":{
               "type":"string",
               "description":"ISO format date YYYY-MM-DD"
            },
            "preferredLimit":{
               "type":"integer",
               "format":"int64"
            },
            "piPreferredLimit":{
               "type":"integer",
               "format":"int64"
            },
            "annualPrice":{
               "type":"number",
               "format":"double"
            },
            "pandPLPreferredLimit":{
               "type":"integer",
               "format":"int64"
            }
         }
      }
   }
}
```

