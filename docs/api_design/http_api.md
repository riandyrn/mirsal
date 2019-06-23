# HTTP API

## User Login

POST: `/chat/sessions`

This API is used for generating access token by using username & password.

**Body:**

```json
{
    "username": "riandyrn",
    "password": "hello_world!"
}
```

**Responses:**

* Success (`200 OK`)

    ```json
    {
        "status": 200,
        "data": {
            "user_id": 1,
            "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0IjoiYSIsImlkIjoiMSIsImV4cCI6MTU5MjgyNzYxMSwiaWF0IjoxNTYxMjkxNjExfQ.5VdT7xERq-EnXhys_UrMcXKc-a_FC5rTzzi-1yKv8zY",
            "expires": "2020-06-22T12:06:51Z",
            "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0IjoiciIsImlkIjoiMSIsImV4cCI6MTU5MjgyNzYxMSwiaWF0IjoxNTYxMjkxNjExfQ.iD_5I6QrLluS-H0kvI0wMjYgX_c4P1iNrLsOjQqcFYA"
        },
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

* Mismatch Credentials (`401 Unauthorized`)

    ```json
    {
        "status": 401,
        "err": "ERR_MISMATCH_CREDENTIALS",
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

    Client will receive this error when either provided `username` or `password` is invalid.

[Back to Top](#http-api)

---

## Refresh Access Token

PUT: `/chat/sessions/{user_id}`

This API is used for generating new access token by using refresh token. Supposedly being called when current access token is already invalid.

**Header:**

```bash
Authorization: Bearer <refresh_token>
```

**Responses:**

* Success (`200 OK`)

    ```json
    {
        "status": 200,
        "data": {
            "user_id": 1,
            "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0IjoiYSIsImlkIjoiMSIsImV4cCI6MTU5MjgyNzYxMSwiaWF0IjoxNTYxMjkxNjExfQ.5VdT7xERq-EnXhys_UrMcXKc-a_FC5rTzzi-1yKv8zY",
            "expires": "2020-06-22T12:06:51Z",
            "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0IjoiciIsImlkIjoiMSIsImV4cCI6MTU5MjgyNzYxMSwiaWF0IjoxNTYxMjkxNjExfQ.iD_5I6QrLluS-H0kvI0wMjYgX_c4P1iNrLsOjQqcFYA"
        },
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

* Invalid Refresh Token (`401 Unauthorized`)

    ```json
    {
        "status": 401,
        "err": "ERR_INVALID_REFRESH_TOKEN",
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

    Client will receive this error when provided `refresh_token` is invalid. When client receive this error, client should redirect user to login page.


[Back to Top](#http-api)

---

## Create Topic

POST: `/chat/topics`

This API is used for creating new topic (either `p2p` or `group` topic).

On `p2p` topic, the behavior of this API is idempotent rather than creating new topic.

**Header:**

```bash
Authorization: Bearer <access_token>
```

**Body:**

* Issue `p2p` topic:

    ```json
    {
        "type": "p2p",
        "peer_id": 2
    }
    ```

* Issue `group` topic:

    ```json
    {
        "type": "group"
    }
    ```

**Responses:**

* Success for `p2p` topic (`200 OK`):

    ```json
    {
        "status": 200,
        "data": {
            "user_id": 1,
            "topic_id": "p2p1_2",
            "peer_id": 2
        },
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

* Success for `group` topic (`200 OK`):

    ```json
    {
        "status": 200,
        "data": {
            "user_id": 1,
            "topic_id": "grp1561294746123"
        },
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

[Back to Top](#http-api)

---

## Get User Latest Topics

GET: `/chat/topics`

This API is used for fetching user latest topics sorted from latest updated topic to the oldest one.

**Header:**

```bash
Authorization: Bearer <access_token>
```

**Parameters:**

* `size` (Int): the maximum number of topics being fetched.
* `start_index` (Int, _optional_): start index of next batch of items to be fetched, the value started from `0`.

**Example Request:**

```curl
GET /chat/topics?size=10&start_index=5
```

This request will fetch up to `10` next least updated topics starting from topic with index `5` from the overall topics in database.

**Responses:**

* Success (`200 OK`):

    ```json
    {
        "status": 200,
        "data": {
            "user_id": 1,
            "topics": [
                {
                    "topic_id": "p2p1_2",
                    "title": "salim_abdullah",
                    "last_message": {
                        "seq_id": 1536394351994,
                        "sender_id": 2,
                        "sender_name": "salim_abdullah",
                        "type": "text/plain",
                        "content": "Hello World!"
                    },
                    "last_read": {
                        "1": 1536394340438,
                        "2": 1536394351994
                    },
                    "updated_at": "2019-06-23T12:06:51.028Z",
                    "unread": 2
                },
                {
                    "topic_id": "grp1561294746123",
                    "title": "Awesome Group!",
                    "last_message": {
                        "seq_id": 1536394352345,
                        "sender_id": 1,
                        "sender_name": "riandyrn",
                        "type": "text/plain",
                        "content": "Hello Guys!",
                    },
                    "last_read": {
                        "1": 1536394352345,
                        "2": 1536394350000,
                        "3": 1536394350000
                    },
                    "updated_at": "2019-06-23T08:07:00.128Z",
                    "unread": 0
                }
            ]
        },
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

* Invalid Size (`400 Bad Request`)

    ```json
    {
        "status": 400,
        "err": "ERR_INVALID_SIZE",
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

    Client will receive this error when provided `size` value is invalid. By default it should higher than `0` & less or equal than `20`.

* Invalid Start Index (`400 Bad Request`)

    ```json
    {
        "status": 400,
        "err": "ERR_INVALID_START_INDEX",
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

    Client will receive this error when provided `start_index` value is invalid. The value should be greater than `0`.

[Back to Top](#http-api)

---

## Send Message

POST: `/chat/topics/{topic_id}/messages`

[Back to Top](#http-api)

---

## Get Latest Messages

GET: `/chat/topics/{topic_id}/messages`

[Back to Top](#http-api)

---

## Confirm Reads

PUT: `/chat/topics/{topic_id}/reads/{user_id}`

[Back to Top](#http-api)

---

## Delete Topic

DELETE: `/chat/topics/{topic_id}`

[Back to Top](#http-api)

---

## Get Topic Member List

GET: `/chat/topics/{topic_id}/users`

[Back to Top](#http-api)

---

## Add User to Topic

POST: `/chat/topics/{topic_id}/users`

[Back to Top](#http-api)

---

## Remove User from Topic

DELETE: `/chat/topics/{topic_id}/users/{user_id}`

[Back to Top](#http-api)

---

## Promote as Admin

PATCH: `/chat/topics/{topic_id}/users/{user_id}`

[Back to Top](#http-api)

---

## Demote as Admin

PATCH: `/chat/topics/{topic_id}/users/{user_id}`

[Back to Top](#http-api)

---
