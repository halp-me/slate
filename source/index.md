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
  "passwordHash": "5f4dcc3b5aa765d61d8327deb882cf99",
  "pushType": "gcm",
  "pushToken": "<gcm registrationId>"
}

# facebook
{
  "type": "facebook",
  "accessToken": "CAAIMxih9KD0BACKy9y2mhOqwvA8zIsQ2kZBB1zctvlZCZCSZBUhPLODmPltwRbmQvxcwzDNuQaZCZAZB7Gy1udLx8XN6ZB4nUk75ZCq5D0K0MEXZBZBNUDj7xVLxfVyfSjUq2kAuCMypPQeazmWJdagFjS3z2lPKHdABaQ3By8RXzfqZBu2x8Pta9ZBq5aqZBqCZAog9WCAQaksXjVweEEGYYNbwnZBr",
  "pushType": "apn",
  "pushToken": "<apn deviceToken>"
}

# response
{
  "code": "success",
  "sessionId": "49c942f682d3f9329d8f2f1b5696366e",
  "profile": {
    # same object as GET /profile
  }
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
pushType      | enum(apn, gcm)        | (optional) the push notification service to use
pushToken     | string                | (required if pushType set) the deviceToken (apn) or registrationId (gcm) to use to push notifications to this device

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
  "lastname": "User",
  "pushType": "apn",
  "pushToken": "<apn deviceToken>"
}

# response
{
  "code": "success",
  "sessionId": "49c942f682d3f9329d8f2f1b5696366e",
  "profile": {
    # same object as GET /profile
  }
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
pushType      | enum(apn, gcm)        | (optional) the push notification service to use
pushToken     | string                | (required if pushType set) the deviceToken (apn) or registrationId (gcm) to use to push notifications to this device

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
- `invalid_image_format` an image is not an accepted format
- `image_too_large` an image is too large
- `too_many_images` too many images in the request

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
/pins?pinMode=mine

# response
{
  "code": "success",
  "student": {
    # a pin with all the same fields as below
    # or null if the user doesn't have a student pin dropped
  },
  "tutor": {
    # a pin with all the same fields as below
    # or null if the user doesn't have a tutor pin dropped
  }

# request
/pins?pinMode=student&lat1=90&lng1=-180&lat2=-90&lng2=180

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
      "dropTime": 391273011,
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

If pinMode=(student|tutor) return the student or tutor pins *not owned by the
user* within the bounding rectangle specified by (lat1,lng1) (top-left) and
(lat2,lng2) (bottom-right). The bounding rectangle's top edge faces north and
right edge faces east.

If pinMode=mine, return the user's pins. Lat and lng are disregarded if
pinMode=mine.

Note: for testing purposes you can use lat1=90, lng1=-180, lat2=-90, lng2=180
to include all pins.

### Query Parameters

Parameter     |   Type                     | Description
--------------|----------------------------|--------------
pinMode       | enum(student,tutor,mine)   | which mode of pins to *return*?
lat1          | float                      | latitude of *top-left* corner of bounding rectangle
lng1          | float                      | longitude of *top-left* corner of bounding rectangle
lat2          | float                      | latitude of *bottom-right* corner of bounding rectangle
lng2          | float                      | longitude of *bottom-right* corner of bounding rectangle

# Matches

## Get a list of matches

```shell
#request
/matches?mode=student

#response
{
  "code": "success",
  "matches": [
    {
      # same exact fields as a pin with one additional field:
      # ...
      "new" : true
    },
    {
      # ...
      "new" : false
    }
  ]
}

```

`GET /matches`

Get a list of matches that this user has.

Currently a match is defined as follows: a pin in the opposite mode of the
user's pin which matches on at least one course. Recall that courses
are university specific. EG: CPE 101 from AHC != CPE 101 from CP.

Note: matches returned from this endpoint are marked as `new=false` upon
querying this endpoint.

### Query Parameters

Parameter     |   Type                     | Description
--------------|----------------------------|--------------
mode          | enum(student,tutor)        | the mode the current user is in

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

### Failure Codes
- `invalid_image_format` the image is not an accepted format (eg, PNG)
- `image_too_large` the image is too large

# Autofill
Users may need to enter an enumerable type of data such as picking their
university or selecting a subset of skills.

These enumerations (universities, skills, courses) are large and dynamic.
The following endpoints allow us to get the
values of the respective enumeration.

## Universities

```shell
# request
/enum/universities

# response
{
  "code": "success"
  "universities": [
    "San Francisco State University",
    "UC Irvine",
    "Santa Barbara State University",
    "Cal Poly",
    ...
  ]
}
```

`GET /enum/universities`

Get a list of all universities.

## Courses

```shell
# request
/enum/courses?university=California%20Polytechnic%20State%20University

# response
{
  "code": "success"
  "courses": [
    {
      "subject": "CPE",
      "number": 308
    },{
      "subject": "CSC",
      "number": 349 
    }, {
      ...
    }
  ]
}
```

`GET /enum/courses`

Get a list of all courses from `university`.

Parameter            |   Type                                      | Description
---------------------|---------------------------------------------|--------------
university  | string                                      | the full university name that corresponds to the univeristy to search for classes from. This is the same string returned from /enum/universities

### Failure Codes
- `university_length` if university isn't specified

## Skills

```shell
# request
/enum/skills

# response
{
  "code": "success"
  "skills": [
    "Excell",
    "Entreprenuership",
    "Photoshop",
    "Cooking"
    ...
  ]
}
```

`GET /enum/skills`

Get a list of all the existing skills.

# Messaging

## Posting a New Message

```shell
# request
{
  "recipient": 4089,
  "body": "Do you make good pancakes?",
  "senderMode": "student"
}

# response
{
  "code": "success"
}
```

`POST /message`

Send a message to a user.

### Body Parameters

Parameter     |   Type      | Description
--------------|-------------|--------
recipient     | int         | the userId of the user to send the message to
body          | string      | the text body of the message
senderMode    | string      | The mode of the user sending the mesage (the mode the app is currently in)

### Failure Codes
- `body_length` the body does not contain at least one character
- `tutor_profile_missing` the user tried to send a message as a tutor without a tutor profile filled out

## Get a list of conversations

```shell
# request
GET /conversations?mode=tutor

# response
{
  "code": "success",
  "conversations": [
    {
      "otherUser": {
        "userId": 9582,
        "firstname": "James",
      },
      "unreadMessages": 1,
      "lastMessage": {
        "me": false,
        "timestamp": 1425841225,
        "body": "Sweet, lets meet."
      }
    },{
      "otherUser": {
        "userId": 4470,
        "firstname": "Fernando",
      },
      "unreadMessages": 0,
      "lastMessage": {
        "me": true,
        "timestamp": 1425841118,
        "body": "later"
      }
    },{
      ...
    }
  ]
}
```

`GET /conversations`

Get a list of all conversations in the current user-mode.

Conversations are sorted by timestamp such that the conversation 
with the most recent message has the lowest index.

The app should query this endpoint whenever it displays the inbox,
returns to the inbox via back button, or receives a push notification
while at the inbox (messages may have been read on another device).

### Query Parameters

Parameter      |   Type                    | Description
---------------|---------------------------|--------------
mode           | enum('student', 'tutor')  | The mode the current user is in

## Get a conversation's messages

```shell
# request
GET /messages?mode=tutor&otherUserId=9582

# response
{
  "code": "success",
  "otherUser": {
    "userId": 9582,
    "firstname": "James"
  },
  "messages": [
    {
      "me": false,
      "timestamp": 1425841225,
      "body": "Sweet, lets meet."
    },{
    },{
      "me": true,
      "timestamp": 1425841220,
      "body": "Yes I can help you with kinematics."
      ...
    }
  ]
}
```

`GET /messages`

Get all messages that I sent `otherUserId` or that `otherUserId` sent me while
I was in `mode`. (eg. get all the messages between me and user 4492 where I am
a student).

Messages are sorted by timestamp such that the newest message has
the lowest index.

When this endpoint is queried, the messages in this conversation are marked
as read.

The app should query this endpoint whenever it displays a conversation or
receives a push notification while displaying a conversation.

### Query Parameters

Parameter      |   Type                    | Description
---------------|---------------------------|--------------
mode           | enum('student', 'tutor')  | The mode the current user is in
otherUserId    | int                       | The userId of the other user in the conversation

# Push Notifications

```shell
# payload
{
  "studentUnreadMessages": 1,
  "tutorUnreadMessages": 3,
  "studentNewMatches": 0,
  "tutorNewMatches": 1,
  "session": { # or null if no active session
    "otherUserId": 4089,
    "mode": "tutor",
    "startTimestamp": 39384880,
    "rate": 40.55
  },
  # android only
  "message": "You have 1 new match and 4 unread messages."
}
```

### Send To Sync
The apps will only ever have one
push notification awaiting them at a time. This push notification does not
contain new content, rather it tells the app that there is new content;
the app should sync with the server to obtain the new content.

### Payload
The payload of the push notification is shown to the right. When the
application is brought to the foreground or receives the push notification
while it is in the foreground, the application should display these
notifications appropriately around the app
(eg: if in student mode, display studentUnreadMessages next to the inbox).

## Android
In android the application handles how the system displays push notifications.
The app should display `<studentUnreadMessages> + <tutorUnreadMessages> +
<studentNewMatches> + <tutorNewMatches>` as the badge count of the application.
Additionally, it should display `message` in the system tray.

Note: there is an edge case where someone drops a pin then deletes a pin
before the user opens the app. In this case, when the pin is deleted,
a push notification will be sent with `0` as all counts and `''` as the
message. The application should delete the system notification and remove the
application badge count.

## iOS
In iOS the operating system handles displaying the push notifications.

## Badge Count
Some time before being backgrounded, the apps should update the badge count
appropriately.
[ios](http://stackoverflow.com/questions/14038680/how-to-clear-push-notification-badge-count-in-ios)
[android](http://stackoverflow.com/questions/20136483/how-do-you-interface-with-badgeprovider-on-samsung-phones-to-add-a-count-to-the/20136484#20136484)

# Sessions
See the
[sessions doc](https://docs.google.com/document/d/1KII44ezoqBc7rvzaenCXI4E34zfhemRTo-vFQn4NihQ/edit)
for an overview on how sessions work. This section requires knowledge of the
session flow.

## Start Session

```shell
# request (first user)
{
  "mode": "student",
  "otherUserId": 4089
}

# response
{
  "code": "success",
  "complete": false,
  "pollDuration": 2000,
  "pollInterval": 250
}

# request (second user)
{
  "mode": "tutor",
  "otherUserId": 5127 
}

# response
{
  "code": "success",
  "complete": true,
  "startTimestamp": 39384880,
  "rate": 40.55
}
```

`POST /session/start`

This endpoint is how the user sticks out his hand to either initiate or
reciprocate the handshake. It is called every time the user presses the
"Start Session" button.

If this user is initializing the handshake, `complete` will be `false`, and the
user should poll `/session/started` for `pollDuration` milliseconds every
`pollInterval` milliseconds (see below). `pollDuration` and `pollInterval` shall
be respected so we can tweak these values in the production environment without
re-shipping-out our client apps.

If this user is reciprocating the handshake within the time alloted, `complete`
will be `true`, and the app *should not poll for confirmation*. The session has
successfully started. `startTimestamp` is the timestamp that the server has
recorded for session start. `rate` is the rate per hour for the session.

### Body Parameters

Parameter     |   Type      | Description
--------------|-------------|--------
mode          | string      | the mode of *this* user who is pressing the button
otherUserId   | int         | the userId of the user to handshake with (who this user is trying to begin a session with)

## Session Start Confirmation

```shell
# request
GET /session/started?mode=tutor&otherUserId=4089

# response (not started)
{
  "code": "success",
  "complete": false
}

# response (started)
{
  "code": "success",
  "complete": true,
  "startTimestamp": 39384880,
  "rate": 40.55
}
```

`GET /session/started`

Determine whether a session with the specified user has started.

`complete` is false if the session hasn't started, and is true if the session
has started.

### Query Parameters

Parameter     |   Type      | Description
--------------|-------------|--------
mode          | string      | the mode of *this* user who is pressing the button
otherUserId   | int         | the userId of the user to handshake with (who this user is trying to begin a session with)

## End Session
```shell
# request
{
  "duration": 60
}
# response
{
  "success": true
}
```

`POST /session/end`

End the session. This action can only be done once. It releases both users
from the session and performs payment.

### Body Parameters

Parameter     |   Type      | Description
--------------|-------------|--------
duration      | int         | the duration (in seconds) of the session

### Failure Codes
- `duration_too_long` if the duration is longer than `now - sessionStart`
- `no session` if the user is not in a session

## Session End Confirmation
```shell
# request
GET /session/ended

# response
{
  "success": true,
  "ended": false
}
```
`GET /session/ended`

Determine whether the user is in a session. The server will send both users
a push notification when the session has ended, but this endpoint should be
queried every minute or so while the user is in the session just in case
the push doesn't go through.
