---
title: Halp API - api.halp.me

search: true
---

# Conventions

### Response Bodies
All response bodies are a single json object.

One field is guarenteed in every response-object: `code`.

`code` represents the state of the request (did it succeed or fail?).

### Methods
The http methods used to represent our CRUD Rest api are:

- Create: `POST`
- Read: `GET`
- Update: `PUT`
- Delete: `DELETE`

### Request Parameters

`POST` and `PUT`  methods use a single json object as the request body.

`GET` and `DELETE` use `query?parameters=tacked&on=to&the=url`

### Errors

#### Ya 'done fucked up
The generic error `server_error` may be returned for a number of situations in which *"ya 'done fucked up"* (assertion failed because firstname is empty, missing field, &hellip;).

To see exactly how *"ya 'done fucked up"*, take a look at the server logs: [log.api.halp.me](http://log.api.halp.me)

*`server_error` may also be a problem with server-code :P, either way, this isn't a situation we should expect in a production environemnt*

#### How could you have known?
In some circumstances, errors may occur that require the client to do something. For instance, a user may try to register with an email that has already been taken. In this case, the user should be notified of the situation.

Many of the sections below contain a "Failure Codes" section. These codes are the codes that should be explicitly handled for each endpoint.

# Authentication

## Authorized Endpoints

All authorized endpoints (most endpoints) require a sessionId http header which
communicates the user's identity to the server.

If the sessionId is invalid or not provided, the server will reply with the
error code: `not_authorized`.

The sessionId the user should use is the sessionId returned by the
[Login](#login) and [Register](#register) endpoints.

In the sections below, unauthorized endpoints are marked by the term
`unauthorized`. All other endpoints are authorized.

<!--------------------------------------------------------------------------!-->
## Login

```shell
# halp
{
  "type": "halp",
  "email": "test@halp.me",
  "passwordHash": "5f4dcc3b5aa765d61d8327deb882cf99"
}

# facebook
{
  "type": "facebook",
  "accessToken": "CAAIMxih9KD0BACKy9y2mhOqwvA8zIsQ2kZBB1zctvlZCZCSZBUhPLODmPltwRbmQvxcwzDNuQaZCZAZB7Gy1udLx8XN6ZB4nUk75ZCq5D0K0MEXZBZBNUDj7xVLxfVyfSjUq2kAuCMypPQeazmWJdagFjS3z2lPKHdABaQ3By8RXzfqZBu2x8Pta9ZBq5aqZBqCZAog9WCAQaksXjVweEEGYYNbwnZBr"
}

# response
{
  "code": "success",
  "sessionId": "49c942f682d3f9329d8f2f1b5696366e"
}
```

`POST /login` &nbsp; &nbsp; `unauthorized`

Attempt to login the user.

### Body Parameters

Parameter     |   Type                | Description
--------------|-----------------------|--------
type          | enum(halp, facebook)  |
email         | string                | (halp) user's email
passwordHash  | string                | (halp) md5 hash of user's password
accessToken   | string                | (facebook) user's facebook access token

### Failure Codes
- `invalid_credentials`
- `invalid_facebook_access_token` TODO: currently this is `...auth_token`

<!--------------------------------------------------------------------------!-->
## Register

```shell
# request
{
  "email": "test@halp.me",
  "passwordHash": "5f4dcc3b5aa765d61d8327deb882cf99",
  "firstname": "Test",
  "lastname": "User"
}

# response
{
  "code": "success",
  "sessionId": "49c942f682d3f9329d8f2f1b5696366e"
}
```

`POST /register` &nbsp; &nbsp; `unauthorized`

Register the user using the `halp` authentication system.

### Body Parameters

Parameter     |   Type                | Description
--------------|-----------------------|--------
email         | string                | user's email
passwordHash  | string                | md5 hash of user's password
firstname     | string                | user's firstname
lastname      | string                | user's lastname

### Failure Codes
- `email_taken`

# Pins

## Drop a pin

```shell
# request
{
  "latitude": 47.2232931,
  "longitude": -162.9434883,
  "university": "Cal Poly",
  "course": {
    "subject": "PSY",
    "number": 101
  },
  "description": "Paying someone to do my homework for me.",
  "images": ["...", "...", "..."],
  "skills": ["archery", "snorkeling", "knitting"]
}

# response
{
  "code": "success"
}
```

<aside class="warning">This endpoint is undergoing major revisions</aside>

`POST /pin`

Drop a pin as the current user.

### Body Parameters

Parameter     |   Type                | Description
--------------|-----------------------|--------
latitude      | float                 | 
longitude     | float                 | 
university    | string                | 
course        | object                | subject (string) and number (int)
description   | string                | description of the problem
images        | array of strings      | each string is a base64 encoded jpeg or png image
skills        | array of strings      | the skills the user needs help with

### Failure Codes
- `already_dropped` if the user already has a pin down

## Delete a pin

```shell
# request
/pin?mode=student

# response
{
  "code": "success"
}
```

`DELETE /pin`

<aside class="warning">This endpoint is undergoing major revisions</aside>

Remove the current user's pin.

### Query Parameters

Parameter     |   Type                | Description
--------------|-----------------------|--------
mode          | enum(student,tutor)   | which pin should be removed?

### Failure Codes
- `no_pin` the user doesn't have a pin to remove

## Get a list of pins

```shell
# request
/pins

# response
{
  "code": "success",
  "pins": [
    {
      "userId": 34848,
      "latitude": 192.9393,
      "longitude": -44.5821,
      "course": {
        "subject": "PSY",
        "number": 101
      },
      "university": "Cal Poly",
      "skills": ["fishing"],
      "firstname": "John",
      "lastname": "Doe",
      "rating": 5.0,
      "profileImageUrl": "http://....",
      "images": ["url", "url"]
    },
    {
      ...
    }
  ]
}
```

`GET /pins`

<aside class="warning">This endpoint is undergoing major revisions</aside>

Get a list of pins the user is interested in. For now, return all pins. Soon this will involve returning only pins that are close or that the user is interested in.
