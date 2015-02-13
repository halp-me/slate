---
title: Halp API - api.halp.me

search: true
---

# Conventions

### Methods
The http methods used to represent our CRUD Rest api are:

- Create: `POST`
- Read: `GET`
- Update: `PUT`
- Delete: `DELETE`

### Request Parameters

`POST` and `PUT`  methods use a single json object as the request body.

`GET` and `DELETE` use `query?parameters=tacked&on=to&the=url`

### Response Bodies
All response bodies are a single json object.

One field is guarenteed in every response-object: `code`.

`code` represents the state of the request (did it succeed or fail?).

A code of `success` means the request succeeded. Anything else is an error.

### Errors

#### Ya 'done fucked up
The generic error `server_error` may be returned for a number of situations in which *"ya 'done fucked up"* (assertion failed because firstname is empty, missing field, &hellip;).

To see exactly how *"ya 'done fucked up"*, take a look at the server logs: [log.api.halp.me](http://log.api.halp.me)

*`server_error` may also be a problem with server-code :P, either way, this isn't a situation we should expect in a production environment (though we should be prepared for it to happen anyway).*

#### How could you have known?
In some circumstances, errors may occur that require the client to do something. For instance, a user may try to register with an email that has already been taken. In this case, the user should be notified of the situation.

Many of the sections below contain a "Failure Codes" section. These codes are the codes that should be explicitly handled for each endpoint.

# Authentication

## Authorized Endpoints

All authorized endpoints (most endpoints) require a `sessionId` http header which
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
- `invalid_facebook_access_token`

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
  "pinMode": "student"
  "latitude": 47.2232931,
  "longitude": -162.9434883,
  "duration": 3600,
# pinMode=student only
  "description": "Paying someone to do my homework for me.",
  "skills": ["archery", "snorkeling", "knitting"],
  "images": ["...", "...", "..."],
  "courses": {
    "Cal Poly": [
      {
        "subject": "PSY",
        "number": 101
      },{
        "subject": "PSY",
        "number": 103
      }
    ],
    "Cuesta": [
      {
        "subject": "PHYS",
        "number": 142
      }
    ]
  }
}

# response
{
  "code": "success"
}
```

`POST /pin`

Drop a pin as the current user.

### Body Parameters

Parameter     |   Type                   | Description
--------------|--------------------------|--------
pinMode       | enum(student,tutor)      | which mode to drop the pin as?
latitude      | float                    | 
longitude     | float                    | 
duration      | int                      | how long the pin should last (in seconds)
description   | string                   | (student) description of the problem
skills        | array of strings         | (student) skills the student needs help with
images        | array of strings         | (student) base64 encoded jpeg or png images
courses       | hash of uname: [course]  | (student) course: {subject, number}

### Failure Codes
- `already_dropped` if the user already has a pin down in this mode
- `max_duration_exceeded` if the duration is too long
- `tutor_profile_missing` if trying to drop pin as tutor but no tutor profile

## Delete a pin

```shell
# request
/pin?pinMode=student

# response
{
  "code": "success"
}
```

`DELETE /pin`

Remove the current user's pin.

### Query Parameters

Parameter     |   Type                | Description
--------------|-----------------------|--------
pinMode       | enum(student,tutor)   | which pin should be removed?

### Failure Codes
- `no_pin` the user doesn't have a pin to remove

## Get a list of pins

```shell
# request
/pins?pinMode=student

# response
{
  "code": "success",
  "pins": [
    {
      "user": {
        "userId": 34848,
        "firstname": "John",
        "lastname": "Doe",
        "rating": 4.72,
        "ratings": 14,
        "image": {original: "http://...."}, # or null if no image
      # pinMode=tutor only
        "bio": "Senior at cal poly with honors in chem..",
        "rate": 23.00
      # end pinMode=tutor only
      },
      "endTime": 391273813, # seconds since epoch
      "latitude": 192.9393,
      "longitude": -44.5821,
    # pinMode=student only
      "description": "a description of the problem",
      "images": [
        {
          "original": "url",
        },{
          "original": "url",
        }
      ],
    # end pinMode=student only
      "courses": {
        "Cal Poly": [
          {
            "subject": "PSY",
            "number": 101
          },{
            "subject": "PSY",
            "number": 103
          }
        ],
        "Cuesta": [
          {
            "subject": "PHYS",
            "number": 142
          }
        ]
      },
      "skills": ["fishing"]
    },{
      ...
    },{
      ...
    }
  ]
}
```

`GET /pins`

Get a list of pins the user is interested in.

### Query Parameters

Parameter     |   Type                | Description
--------------|-----------------------|--------------
pinMode       | enum(student,tutor)   | which mode of pins to *return*?

# Profile

## Get Profile

```shell
#request
/profile

#response
{
  "code": "success",
  "userId": 12,
  "firstname": "Bob",
  "lastname": "Smith",
  "image": {
    "original": "url"
  },
  # tutor = null if no profile
  "tutor": {
    "bio": "I'm awesome."
    "rate": 22.50,
    "skills": ["archery", "wrangling"],
    "courses": {
      "San Jose State University": [
        {
          "subject": "PHIL",
          "number": 182
        }
      ]
    }
  }
}
```

`Get /profile`

Get information about the currently logged in user.

## Update Profile

```shell
#request
# (update the user's image and change the skills of the user's tutor profile)
{
  "image": "base64..",
  "tutor": {
    "skills": ["archery", "wrangling"],
  }
}

#response
{
  "code": "success"
}
```

`PUT /profile`

Update the user's profile.

Leave field out or set to null to not change it.

`profile.image` is the only field that **can be deleted**. Set
`profile.image` to null to delete the user's image.

The tutor portion of the user's profile starts out as null (no profile).
To **create** a tutor profile, simply supply the tutor object with
*all fields*. Creation of the tutor object is the only time where
all fields of the tutor object must be present.

### Body Parameters

Parameter     |   Type                   | Description
--------------|--------------------------|--------
firstname     | string                   | 
lastname      | string                   | 
image         | string                   | base64 encoded png or jpeg image
tutor         | object                   | null=don't change, object=create/update it
tutor.bio         | string                   | short bio of tutor (degrees, bla bla bla)
tutor.rate        | float                    | how much the tutor charges per hour
tutor.skills      | array of strings         | skills the tutor wants to tutor in
tutor.courses     | hash of uname: [course]  | courses the tutor wants to tutor for
