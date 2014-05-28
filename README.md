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

Token, received in response, must be send in header "X-Auth-Token" on any other requests to API, which requires authorization. If any credentials ommited, demo user with demo services will auth.

Client chooses active base service on current device by specifying id of service in "service_id" param . If no service specified - the most expensive service chooses.

```
POST /1.2/users/token
```
Deprecated resource, removed in API v.1.3:
```
POST /1.2/users/authorize 
```

Name|Required|Type|Default|Description
----|--------|----|-------|-----------
api_key|yes (since v1.2)|varchar(40)||API key for client identification.
email|no|varchar(40)||User's email, used to identify user on auth server. omitted in demo authorization
password|no|varchar||User's password. Omitted in demo authorization
agent|no|varchar||Agent identifier. Specify agent only in demo authorization, when lodin and password empty.
remember|no|boolean||If set, server generates permanent identificator, which definetly identify client application on server. This hash stores localy on client and will send to server while next authorization request.
permid|no|varchar||Permanent identificator, which definetly identify client application on server. This hash stores localy on client and will send to server while next authorization request. 
service_id|no|integer||Id of base service, which defines set of base and additional services configured in personal page. If omited than it will be chosen automatically as most expensive service
application_version|no|varchar||Client version: “major.minor.build”
os_type|no|varchar||OS type: “IOS”, “BBQNX”, “Linux” etc
os_version|no|varchar||OS version: “3.1.3” etc
device_model|no|varchar||Device model (if appropriate): “iPhone” etc
uuid|no|varchar(16)||UUID generated on first start of the application
device_id|no|varchar||Device ID (if any, used on older iOS)
advertising_id|no|varchar||iOS 6 advertising ID (if any)
vendor_id|no|varchar||iOS6 vendor ID (if any)
mac|no|varchar||Network interface MAC address
ip|no|varchar||Original IP of client. Used if client IP different from IP, from which requests to API sends
referal|no|varchar||Identity of referal. Deprecated at v.1.2 and will be removed in v.1.3 due to passing agent on registration
expiration|no|integer||Used only for DEBUG. May be disabled in production environment. Specify delay of session life in seconds.
session|no|boolean||Start session on billing side. If 1 passed — session initialised and id of session returned

Response:
```json
{
    "status": 0,
    "token": "d41d8cd98f00b204e9800998ecf8427e",
    "profile": {
        "last_name": "Simpson",
        "first_name": "Homer",
        "patronymic": "Jay",
        "gender": "MALE",
        "birthday": "1965-10-01",
        "tester": 1,
        "contract_number": "0001234567"
    },
    "balance": {
        "amount": 55.9,
        "currency": "EUR"
    },
    "baseServices": [
        {
            "id": 19040,
            "custom_name": "In hall",
            "service_id": 1,
            "name": "Advanced",
            "cost": "0.16",
            "ad": 1,
            "catchup": 0,
            "additional": [
                {
                    "id": 36147,
                    "service_id": 4,
                    "name": "Sci-Fi",
                    "cost": "0.12"
                }
            ],
            "total_cost": 0.28
        },
        {
            "id": 36146,
            "custom_name": "In kitchen",
            "service_id": 3,
            "name": "Basic",
            "cost": "0.2",
            "total_cost": 0.2
        }
    ],
    "activeBaseService": 36146,
}
```

Name|Required|Type|Default|Description
----|--------|----|-------|-----------
status|yes|integer||Status code of operation. May be one of these values: 0 - Authorization successfull, 1 - Server error (generic), 2 - Wrong credentials, 3 - Account blocked, 4 - Email not confirmed yet, 5 - Client version not supported, 6 — No active service found (Client myst register some service in personal cabinet)
token|yes|varchar(32)||Authorization token, used as session id to identify any request to API. Send only when status code is 0. This token may expire if client is not active for some time, so new authorization may be needed. Token expiration time updates on every client request to API. Expiration period  not more than few hours. Hash binded to client ip.
permid|no|varchar||Permanent identificator, which definetly identify client application on server. This hash stores localy on client and will send to server while next authorization request. Send only if request parameter "remember" set to "true". Expiration period is up to one month. 
profile|yes|dictionary||Profile dictionary of user, as described below.
balance|yes|dictionary||Balance of user, as described below: 
baseServices|yes|list||Client's base service, as described below.
activeBaseService|yes|integer||Currently active base service
speed|yes|list||List of connect speed where keys are identifiers and values are localized options. One of this keys may be send to API to request streams of some speed
rechargePage|yes|string||Url of page, used for recharging account
profilePage|yes|string||Url of user's profile in billing

Profile dictionary:

Name|Required|Type|Default|Description
----|--------|----|-------|-----------
id|yes|integer||Identifier of user
email|yes|varchar(40)||User's email, used to identify user on auth server
hash|yes|string||Hash whicj uniquely identify user
last_name|no|varchar(30)||Last name
first_name|no|varchar(30)||First name
gender|no|enum('MALE','FEMALE')||Gender of user
birthday|no|string (yyyy-mm-dd), timestamp||Birthday of user
contract_number|yes|varchar||Contract number
status|yes|enum||Status of contract: ACTIVE -abonent active, SUSPENDED - stop by abonent himself, BLOCKED - stop by operator, CLOSED - contract closed
tester|no|int(1,0)||Tester mode status of user. If 1 — user in tester mode. Default — 0. User in tester mode may have some additional functionality in app

Balance dictionary:

Name|Required|Type|Default|Description
----|--------|----|-------|-----------
amount|yes|float||Amount on balance of user
currency|yes|string||Code of currency

Client's base service structire:

Name|Required|Type|Default|Description
----|--------|----|-------|-----------
id|yes|integer||Id of client's base service
custom_name|yes|string||Name of client's base service
service_id|yes|ineger||Id of service
name|yes|string||Name of service
cost|yes|float||cost of service by month
additional|no|list||Related additional services
total_costyes||float||Cost of base and additional services 
ad|no|booleand||Advertising must be shown in shis service
catchup|no|boolean||Catchup enabled for this service


### Logout

### User profile

Authorisation required. Profile of user may be obtained with authentication through email and password or throught passing auth token.
```
GET /1.2/user
```
Deprecated resource, removed in API v.1.3:
```
GET /1.2/users/profile
```
Request params, used only when authorization made througn email and password. This need when selecting of active service while authorization process is undesirable. If auth token passed in header, this params omitted.

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
POST /1.2/services/clientBaseService
```
Name|Required|Type|Default|Description
----|--------|----|-------|-----------
service|yes|integer||New service id
custom_name|no|string||New custom name

### Update existed client's base service
Authorisation required.
```
PUT /1.2/services/clientBaseService
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


