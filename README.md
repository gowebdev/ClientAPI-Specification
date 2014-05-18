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

### Base client service
Managing of client's base service.
Authorisation required.
#### Create new client's base service
```
PUT /1.2/services/clientBaseService
```
Name|Required|Type|Default|Description
----|--------|----|-------|-----------
service|yes|integer||New service id
custom_name|no|string||New custom name

#### Update existed client's base service
```
POST /1.2/services/clientBaseService
```
Name|Required|Type|Default|Description
----|--------|----|-------|-----------
id|yes|integer||Id of client`s base service
service|yes|integer||New service id
custom_name|no|string||New custom name
status|no|ENUM('ACTIVE','SUSPENDED','BLOCKED','CLOSED')||New status

#### Delete existed client's base service
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


