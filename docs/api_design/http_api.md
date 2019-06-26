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

GET: `/chat/topics?size=<size>&last_key=<last_key>`

This API is used for fetching user latest topics (subscriptions) sorted from latest updated topic to the oldest one. Client could check various information regarding user subscription to the topic from the result of this API.

In `last_message` object (both in `p2p` & `group` topic), when the last message is being sent by user, there will be extra field called `is_read` which indicate whether that message has been read by all members in the topic or not.

Object `last_read` is being used to give other members read information. This information is expected to be used when later client called [Get Latest Messages](#get-latest-messages).

Field `unread` is indicating how many messages in the topic which has not been read by user.

In `group` topic, there will be extra field called `is_admin` which represents whether user is an `admin` in that particular topic. This information could be used by client to show commands which only available for `admin` when user loading the topic later.

Field `last_key` is special string which represents last element in the topic iterator which could be used as offset to fetch next topics. This field exists when user potentially has more topics to be fetched.

**Header:**

```bash
Authorization: Bearer <access_token>
```

**Parameters:**

* `size` (Int): the maximum number of topics being fetched.
* `last_key` (String, _optional_): special string which represents last element in the topic iterator which could be used as offset to fetch next topics.

**Example Request:**

```curl
GET /chat/topics?size=10&last_key=MXxwMnAxXzJ8MTUxODc2NTUxNTIyMg
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
                    "unread": 0,
                    "is_admin": true
                }
            ],
            "last_key": "MXxwMnAxXzJ8MTUxODc2NTUxNTIyMg",
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

* Invalid Last Key (`400 Bad Request`)

    ```json
    {
        "status": 400,
        "err": "ERR_INVALID_LAST_KEY",
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

    Client will receive this error when provided `last_key` value is invalid. Make sure to use value which provided by previous API response.

[Back to Top](#http-api)

---

## Send Message

POST: `/chat/topics/{topic_id}/messages`

This API is used for publishing message to topic.

All participants which currently online (opening websocket connection to server) will receive the message packet including sender websocket session. The rationale behind this is because sender may have more than one client active at the same time (e.g using both web & app client at the same time). For more details about this behavior, please check out `docs/api_design/ws_api.md` document.

Notice that when this API is successfully called, it doesn't guarantee the message has been processed by server, it just mark the message has been successfully delivered to server to be processed by it later.

Message is guaranteed to be processed by server when client already receive the message packet echo through the websocket session or the message has been appeared on the result of [Get User Latest Topics](#get-user-latest-topics). This is similar to the [Facebook's message delivery confirmation mechanism](https://www.facebook.com/help/messenger-app/iphone/926389207386625/).

**Header:**

**Body:**

**Responses:**

[Back to Top](#http-api)

---

## Get Latest Messages

GET: `/chat/topics/{topic_id}/messages?size=<size>&last_key=<last_key>`

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

**Header:**

```bash
Authorization: Bearer <access_token>
```

**Parameters:**

* `size` (Int): the maximum number of messages being fetched.
* `last_key` (String, _optional_): special string which represents last element in the message iterator which could be used as offset to fetch next messages.

**Example Request:**

```curl
GET /chat/topics/p2p1_2/messages?size=10&last_key=cDJwMV8yfDE1MTkwMjMxOTIxMjM
```

**Responses:**

* Success (`200 OK`)

    ```json
    {
        "status": 200,
        "data": {
            "user_id": 1,
            "topic_id": "p2p1_2",
            "messages": [
                {
                    "seq_id": 1536393052771,
                    "sender_id": 2,
                    "sender_name": "salim_abdullah",
                    "type": "text/plain",
                    "content": "This is slightly older message",
                    "created_at": "2018-09-08T07:50:52Z.771"
                },
                {
                    "seq_id": 1536394351994,
                    "sender_id": 2,
                    "sender_name": "salim_abdullah",
                    "type": "text/plain",
                    "content": "This is latest message!",
                    "created_at": "2018-09-08T08:12:31Z.994"
                }
            ],
            "last_key": "cDJwMV8yfDE1MTkwMjMxOTIxMjM"
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

* Invalid Last Key (`400 Bad Request`)

    ```json
    {
        "status": 400,
        "err": "ERR_INVALID_LAST_KEY",
        "ts": "2019-06-23T12:06:51.028Z"
    }
    ```

    Client will receive this error when provided `last_key` value is invalid. Make sure to use value which provided by previous API response.

[Back to Top](#http-api)

---

## Confirm Reads

PUT: `/chat/topics/{topic_id}/reads/{user_id}`

This API is used for submitting `seq_id` of last message being read in the topic. All participants which currently online except sender will receive the read packet (this behavior is different with message packet in which sender also receive the packet).

[Back to Top](#http-api)

---

## Delete Topic

DELETE: `/chat/topics/{topic_id}`

This API is used to make either `p2p` or `group` topic disappear from the result of [Get User Latest Topics](#get-user-latest-topics). Underneath though, `p2p` & `group` topics are being treated differently.

When this function is being called on `p2p` topic, it would not remove user subscription to the topic, only hide it. The reason behind this behavior is because we want to make it possible for peer to still contact the user after user deleting the topic because essentially what user did is relatively delete the topic only for himself. So it would be very weird from peer perspective if all of sudden peer cannot send the message to user just because user delete the topic (e.g to clear up his storage). This behavior is actually the same behavior like Whatsapp.

When this function is being called on `group` topic, user subscription would be literally deleted from database. So user would no longer be able to receive any notification from the topic nor send message to the topic.

[Back to Top](#http-api)

---

## Get Topic Member List

GET: `/chat/topics/{topic_id}/users`

This API is used for getting current members of the `group` topic along with their authorization level (either `normal` or `admin` user).

[Back to Top](#http-api)

---

## Add User to Topic

POST: `/chat/topics/{topic_id}/users`

This API is used by `admin` to add member to `group` topic.

When new member is successfully being added to the group, the event would broadcasted to all online members. For more details please check `docs/api_design/ws_api.md`.

[Back to Top](#http-api)

---

## Remove User from Topic

DELETE: `/chat/topics/{topic_id}/users/{user_id}`

This API is used by `admin` to remove member from `group` topic.

When member is successfully removed from the group, the event would broadcasted to all online members. For more details please check `docs/api_design/ws_api.md`.

[Back to Top](#http-api)

---

## Promote as Admin

PATCH: `/chat/topics/{topic_id}/users/{user_id}`

This API is used by `admin` to promote a member into new `admin` in `group` topic.

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
