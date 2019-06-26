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

GET: `/chat/topics?size=<size>&start_index=<start_index>`

This API is used for fetching user latest topics sorted from latest updated topic to the oldest one.

// TODO: tambahkan penjelasan mengenai `last_read` & `unread_by`.

// TODO: tambahkan is owner / is admin parameter di result + penjelasannya --> tambahkan jg parameter untuk ngecek apakah ada admin selain owner atau gimana --> list mungkin ya?

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

This request will fetch up to `10` next least updated topics starting from topic with index `5` from the overall topics exist in database.

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
                        "is_read": true
                    },
                    "last_read": {
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

This API is used for publishing message to topic.

All participants which currently online (opening websocket connection to server) will receive the message packet including sender websocket session. The rationale behind this is because sender may have more than one client active at the same time (e.g using both web & app client at the same time). For more details about this behavior, please check out `docs/api_design/ws_api.md` document.

Notice that when this API is successfully called, it doesn't guarantee the message has been processed by server, it just mark the message has been successfully delivered to server to be processed by it later.

Message is guaranteed to be processed by server when client already receive the message packet echo through the websocket session or the message has been appeared on the result of [Get User Latest Topics](#get-user-latest-topics). This is similar to the [Facebook's message delivery confirmation mechanism](https://www.facebook.com/help/messenger-app/iphone/926389207386625/).

[Back to Top](#http-api)

---

## Get Latest Messages

GET: `/chat/topics/{topic_id}/messages?size=<size>&start_index=<start_index>`

This API is used to fetch user latest messages started from the latest one but the order is reversed in every fetch of messages. So for example we have following messages in database:

* `msg_0` (this is the oldest message)
* `msg_1`
* `msg_2`
* ...
* `msg_9` (this is the latest message)

For example we want to fetch max `5` messages in every fetch operation. So in first operation we would get:

* `msg_5`
* `msg_6`
* ...
* `msg_9`

In the second operation we would get:

* `msg_0`
* `msg_1`
* ...
* `msg_4`

To put it simply this behavior is similar to behavior of common messengers we know today (Whatsapp, Facebook Messenger, etc) when loading messages in the topic.

[Back to Top](#http-api)

---

## Confirm Reads

PUT: `/chat/topics/{topic_id}/reads/{user_id}`

This API is used for submitting `seq_id` of last message being read in the topic. All participants which currently online except sender will receive the read packet (this behavior is different with message packet in which sender also receive the packet).

[Back to Top](#http-api)

---

## Delete Topic

DELETE: `/chat/topics/{topic_id}?delete=true`

This API is used to make either `p2p` or `group` topic disappear from the result of [Get User Latest Topics](#get-user-latest-topics). Underneath though, `p2p` & `group` topics are being treated differently.

When this function is being called on `p2p` topic, it would not remove the topic entry from database, only hide it. The reason behind this behavior is because we want to make it possible for peer to still contact the user after user deleting the topic because essentially what user did is relatively delete the topic only for himself. So it would be very weird from peer perspective if all of sudden he cannot send the message to user just because user delete the topic from his side (e.g to clear up his storage). This behavior is actually the same behavior like Whatsapp.

When this function is being called on `group` topic, there are 2 possible actions which could be done:

1. Literally deleting the topic, in which means all of the current members would be kicked out
2. Leave the topic

The first action would only be possible to be done by topic `owner`. As for the second action it could be done by every member in the topic.

When `owner` leaving the topic, one of topic `admin` would be randomly selected as the new `owner`, when there is no `admin` in the topic, the request for `owner` to leave the topic would be rejected (he need to explicitly execute the first action). Client could check whether there is admin in the topic by looking at the result of [Get User Latest Topics](#get-user-latest-topics).

To literally delete the topic (first action), client need to specify `delete=true` on query parameters, otherwise it would be leaving the topic.

Client could recognize whether user is `owner` or `admin` or normal user on the `group` topic by looking at the result of [Get User Latest Topics](#get-user-latest-topics). Notice that this structure of authorization only exist on `group` topic.

When a user is being promoted to `admin` or `owner`, all online members on the topic would be notified. Please check out `docs/api_design/ws_api.md` for details.

[Back to Top](#http-api)

---

## Get Topic Member List

GET: `/chat/topics/{topic_id}/users`

This API is used for getting current members of the `group` topic along with their authorization level.

[Back to Top](#http-api)

---

## Add User to Topic

POST: `/chat/topics/{topic_id}/users`

This API is used by `admin` to add member to `group` topic.

[Back to Top](#http-api)

---

## Remove User from Topic

DELETE: `/chat/topics/{topic_id}/users/{user_id}`

This API is used by `admin` to evict member from `group` topic.

[Back to Top](#http-api)

---

## Promote as Admin

PATCH: `/chat/topics/{topic_id}/users/{user_id}`

This API is used by `admin` to promote a member into `admin` in `group` topic.

An `admin` may add or remove member from the topic.

When member promotion is successful, all online members will receive the notification packet via websocket session. For more details please check `docs/api_design/ws_api.md`.

[Back to Top](#http-api)

---

## Demote as Admin

PATCH: `/chat/topics/{topic_id}/users/{user_id}`

This API is used by `admin` to demote another admin into normal user in `group` topic.

When admin demotion is successful, all online members will be notified via websocket session. For more details please check `docs/api_design/ws_api.md`.

[Back to Top](#http-api)

---
