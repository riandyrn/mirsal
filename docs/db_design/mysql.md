# MySQL Schema

## Tables `users`

**Fields:**

* `id`: INT(11)
* `username`: VARCHAR(30)
* `avatar_url`: TEXT
* `password`: TEXT
* `salt`: TEXT

[Back to Top](#mysql-schema)

---

## Table `topics`

**Fields:**

* `id`: VARCHAR(255)
* `last_message_content`: TEXT
* `last_message_id`: BIGINT(20)
* `updated_at`: BIGINT(20)
* `expired_at`: BIGINT(20)

[Back to Top](#mysql-schema)

---

## Table `subscriptions`

**Fields:**

* `user_id`: INT(11)
* `topic_id`: VARCHAR(255)
* `updated_at`: BIGINT(20)
* `unreads`: INT(11)
* `last_read`: BIGINT(20)
* `offset_read`: BIGINT(20)
* `expired_at`: BIGINT(20)
* `is_admin`: TINYINT(1)

[Back to Top](#mysql-schema)

---

## Table `messages`

**Fields:**

* `topic_id`: VARCHAR(255)
* `seq_id`: BIGINT(20)
* `created_at`: BIGINT(20)
* `content`: TEXT
* `sender_id`: INT(11)
* `content_type`: INT(2)
* `expired_at`: BIGINT(20)

[Back to Top](#mysql-schema)

---
