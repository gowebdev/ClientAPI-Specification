Client API Specification (v1.2)
========================

Table of content
----------------

* [Abstract terms](#abstract-terms)
* [Authorisation and user management](#authorisation-and-user-management)
* [Channels](#channels)
* [Films](#films)
* [Radio](#radio)
* [Services](#services)
* [Logging](#logging)

Abstract terms
--------------

Api based on RESTful principles and used token-based authentification.

Common request headers:

Header | Example | Description
-------|---------|------------
Accept-Language | Accept-Language: uk,ru;q=0.8,en;q=0.6 | Used to define prefered language of response
If-Modified-Since | If-Modified-Since:Fri, 01 Jan 2010 00:00:00 GMT | The If-Modified-Since request-header field is used with a method to make it conditional: if the requested variant has not been modified since the time specified in this field, an entity will not be returned from the server; instead, a 304 (not modified) response will be returned without any message-body.
Origin | Origin: http://example.com | This header specifies that cross-domain Ajax request required

Common response headers:

Header | Example | Description
-------|---------|------------
Last-Modified | Last-Modified:Sun, 18 May 2014 09:05:54 GMT | The Last-Modified entity-header field indicates the date and time at which the origin server believes the variant was last modified.

HTTP status codes:

Code | Description
-----|------------
200 OK | Request successfull. If token send, then validation of token successfull
304 Not Modified | Response not modified from last request. Reply on request with If-Modified-Since header
400 Bad Request | Request params not specified of request method wrong
401 Unauthorized | Token not specified in headers
403 Forbidden | Token not found or expired
404 Not Found | API resource not found
406 Not Acceptable | Token was previously deleted because other device join to same service

In every response returned JSON document with standart structure:

Name | Type | Required | Description
-----|------|----------|------------
error | integer | yes | Status of request. If 0 - no error found, if greater - error occured.
errorMessage | string | no | Textual description of error. present only if "error" not equal 0
time| timestamp | yes | Unit timestamp of time on server 

Example:
```json
{
  "error": 1,
  "errorMessage": "Textual error description",
  "time": 1400404339
}
```

Authorisation and user management
---------------------------------

### Registration

### Authorisation

### User profile

Authorisation required. Profile of user may be obtained with authentication through email and password or throught passing auth token.
Authorisation required.
```
GET /1.2/user
```
Deprecated resource, removed in API v.1.3:
```
GET /1.2/users/profile
```
Request params, used only when auth througn email and password. If auth token passed in header, this params omitted:
Name|Required|Type|Default|Description
----|--------|----|-------|-----------
email|no|string||E-mail
password|no|string||Password
Response:
```json
{
    "error": 0,
    "balance": {
        "amount": 9.472,
        "currency": "EUR"
    },
    "profile": {
        "id": 36574,
        "email": "john.watson@example.com",
        "hash": "c7244869cbb730dcf13d0f5f94655b591da6a32f",
        "last_name": "Watson",
        "first_name": "John",
        "gender": "MALE",
        "birthday": "1852-12-12",
        "contract_number": "00036574",
        "status": "ACTIVE",
        "tester": 1
    },
    "baseServices": [
        {
            "id": 44170,
            "service_id": 9,
            "name": "Basic",
            "custom_name": "In kitchen",
            "cost": 0.033,
            "total_cost": 0.033,
            "status": "ACTIVE",
            "catchup": 1,
            "ad": 0
        },
        {
            "id": 45572,
            "service_id": 9,
            "name": "Advanced",
            "custom_name": "In hall",
            "cost": 0.033,
            "total_cost": 0.033,
            "status": "ACTIVE",
            "catchup": 1,
            "ad": 0
        }
    ],
    "time": 1400576569
}
```
### Logout

### Status

### Reset password



Channels
--------

### Channel list

### EPG and TVoD

### Favourite channels

Films
-----

### Genres

### Searching

### Suggested videos

Radio
-----

### Channel list

Services
--------
### Services list
Get list of all active services and related channels.
Authorisation not required.
```
GET /1.2/services
```


### Create new client's base service
Authorisation required.
```
PUT /1.2/services/clientBaseService
```
Name|Required|Type|Default|Description
----|--------|----|-------|-----------
service|yes|integer||New service id
custom_name|no|string||New custom name

### Update existed client's base service
Authorisation required.
```
POST /1.2/services/clientBaseService
```
Name|Required|Type|Default|Description
----|--------|----|-------|-----------
id|yes|integer||Id of client`s base service
service|yes|integer||New service id
custom_name|no|string||New custom name
status|no|ENUM('ACTIVE','SUSPENDED','BLOCKED','CLOSED')||New status

### Delete existed client's base service
Authorisation required.
```
DELETE /1.2/services/clientBaseService
```
Name|Required|Type|Default|Description
----|--------|----|-------|-----------
id|yes|integer||Id of client`s base service
### Additional client service

Logging
-------

### Register event
Authorisation required.
```
POST /1.2/logger/log
```
Name|Required|Type|Default|Description
----|--------|----|-------|-----------
time|no|Timestamp GMT|Current timestamp|Time when error occured. If ommited — current time used, else must be timestamp GMT
file|no|string||File in which error occured, if able to get
line|no|integer||Line in which error occured, if able to get
message|yes|string||Traces, messages in common format


