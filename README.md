JSON API for Terrans
--------------------

## Motivation
Therea are a lot of REST API implementations. And it is always takes a lot of time to decide what to use. For example, should we use unix time or ISO8601? How foreign keys should be handled? How to return errors? etc.

This document just describes practices that works good in production. REST API should be intuitive and understandable without reading tons of docs. Several years ago we started using jsonapi.org but it is rather unstable and every time when I visit jsonapi.org I understand that everything written before is not compatible with newer version.

Moreover, jsonapi.org becomes too complex. We call it jsonapi for Zergs. In our company we prefer to use jsonapi for Terrans :).  This document describes it. Everything is written in document is just recomendation no "MUST", use common sense for edge cases.


## General rules

1. All routes should be prefixed with /api/v1.  
2. Resource names in plural. For example, "users", "orders", "posts" etc.
3. Field names in camelCase
4. Date and time in ISO8601 UTC. For example, "2015-04-02T14:20Z". Dates without time look like "2015-04-02".
5. Enumarable alwais in upper case. For example, userRole: "ADMIN", "MODERATOR", "CUSTOMER"
6. Every return object contains "id" field. Value of "id" is always represented as string. (to avoid problems with numeric types representations)
7. All links to related objects are located in "links" section. Every link has collection name ("type" field) and object identifier ("id").  IMPORTANT: links should not conflict with attributes name of base object.


## Common routes. Example for posts

There is a collection "/api/v1/posts" and there are posts in the collection. Each post os idintified by "/api/v1/posts/:id". Think about routes as identifiers, HTTP methods - are available actions.

- **GET /api/v1/posts** - get posts collection. Can return only subset of fields for each post.
- **POST /api/v1/posts**  - add post to collection. Returns newly created object.
- **GET /api/v1/posts/31** - get post with id=31. May return more fields in object than it was in list call. For example, it can be inneficient to return large payload in list. 
- **PUT /api/v1/posts/31** - replace(add) post with id=31
- **PATСH /api/v1/posts/31** - update some fields in post with id=31
- **DELETE /api/v1/posts/31** - delete post with id=31

### Create post
#### Request

POST /api/v1/posts

```javascript
{
    "data": {
        "title": "Babel vs Traceur",
        "text": "What is better..."
    }
}
```

#### Response

HTTP status - 201 Created
```javascript
{
    "status": 1,
    "data": {
        "id": "2312",
        "title": "Babel vs Traceur",
        "text": "What is better..."

        "links": {
            "comments": []
        },

        "createdAt": "2015-04-02T14:20Z",
        "updatedAt": "2015-04-02T14:20Z", 
    }
}
```


### List posts

#### Request
GET /api/v1/posts

#### Response
HTTP status - 200 Ok
```javascript
{
    "status": 1,
    "data": [{
        "id": "2312",
        "title": "Babel vs Traceur",
        "text": "What is better..."

        "links": {
            "comments": [
                {"type": "comments", "id": 23},
                {"type": "comments", "id": 24}
            ]
        },

        "createdAt": "2015-04-02T14:20Z",
        "updatedAt": "2015-04-02T14:20Z", 
    }]
}
```


### Get on post

#### Request
GET /api/v1/posts/2312

#### Response
HTTP status - 200 Ok
```javascript
{
    "status": 1,
    "data": {
        "id": "2312",
        "title": "Babel vs Traceur",
        "text": "What is better..."

        "links": {
            "comments": [
                {"type": "comments", "id": 23},
                {"type": "comments", "id": 24}
            ]
        },

        "createdAt": "2015-04-02T14:20Z",
        "updatedAt": "2015-04-02T14:20Z", 
    }
}
```


### Update post

#### Request
PATCH /api/v1/posts/2312

```javascript
{
    "data": {
        "title": "Babel vs Traceur!",
    }
}
```

#### Response
HTTP status - 200 Ok
```javascript
{
    "status": 1,
    "data": {
        "id": "2312",
        "title": "Babel vs Traceur!",
        "text": "What is better...",

        "links": {
            "comments": [
                {"type": "comments", "id": 23},
                {"type": "comments", "id": 24}
            ]
        },

        "createdAt": "2015-04-02T14:20Z",
        "updatedAt": "2015-04-02T14:20Z", 
    }
}
```


### Delete post

#### Request
DELETE /api/v1/posts/2312

#### Response
HTTP status - 200 Ok
```javascript
{
    "status": 1,
}
```

### More complex example: order creation
POST /api/v1/orders

#### Request
HTTP status - 200 Ok
```javascript
{
    "data": {
        "name": "Order electronics",
        "deliveryDate": "2015-04-20",

        "links": {
            "customer": {"type": "customers", "id": "212"},
            "payments": [
                {"type": "payments", "id": "34"},
                {"type": "payments", "id": "35"}
            ],
        },

        "orderedProducts": [
            { 
                "quantity": 20, 
                "links": {
                    "product":{ "type": "products", "id": "123" } 
                } 
            },{ 
                "quantity": 21, 
                "links": {
                    "product":{ "type": "products", "id": "124" } 
                } 
            }
        ]
    }
}
```


#### Response
HTTP status - 200 Ok
```javascript
{
    "status": 1,
    "data": {
        "id": "2312",
        "name": "Заказ техники",
        "deliveryDate": "2015-04-20",

        "status": "PENDING",

        "links": {
            "customer": {"type": "customers", "id": "212"},
            "payments": [
                {"type": "payments", "id": "34"},
                {"type": "payments", "id": "35"}
            ],
        },

        "orderedProducts": [
            { 
                "quantity": 20, 
                "links": {
                    "product":{ "type": "products", "id": "123" } 
                } 
            },{ 
                "quantity": 21, 
                "links": {
                    "product":{ "type": "products", "id": "124" } 
                } 
            }
        ],

        "createdAt": "2015-04-02T14:20Z",
        "updatedAt": "2015-04-02T14:20Z", 
    }
}
```



### Load realated data

Use "include" parameter to load related data. The parameter is optional and can support not all relations but only those that makes sense to support. 

Related data should be placed in "linked" section. 

#### Request
GET /api/v1/posts?include=comments

#### Response
HTTP status - 200 Ok
```javascript
{
    "status": 1,
    "data": [{
        "id": "2312",
        "title": "Babel vs Traceur",
        "text": "What is better..."

        "links": {
            "comments": [
                {"type": "comments", "id": "23"},
                {"type": "comments", "id": "24"}
            ]
        },

        "createdAt": "2015-04-02T14:20Z",
        "updatedAt": "2015-04-02T14:20Z", 
    }],

    "linked": {
        "comments": [
            {
                "id": 23, 
                "text", "пиши есчо",
                "links": {
                    "author": {"type": "users", "id": "13" }
                }
            },
            {
                "id": 24,
                "text": "отличный пост",
                "links": {
                    "author": {"type": "users", "id": "13" }
                }
            }
        ]
    }
} 
```

You can pass nested names to include

GET /api/v1/posts?include=comments,comments.author

### Filtering and sorting

GET /api/v1/posts?sort=title,-text&limit=20&offset=100&filter={}

Filters are often wery complex, therefoere it is ok path filter as json object.

### Errors

When error occurs server should return error object with:

1. code
2. message
3. fields with json pointers to fields with errors. Values are error codes.

#### Response
HTTP status - 200 Ok
```javascript
{
    "status": 0,
    "error": {
        "code": "FORMAT_ERROR",                 // Error code
        "message": "Check data format",         // Message for developer
        "fields": {
            "title": "REQUIRED",                // Error code for field "title" 
            "text": "REQUIRED",                 // Error code for field "text"                   
        }
    }
} 
```


### Authentication example


### Registation example


