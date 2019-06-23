# Mirsal

Mirsal (مرسل) is scalable real-time chat server written in Golang.

It is built with intuitive APIs in mind which make it easy for developers to build chat system on top of it.

Currently there are 2 API classes which supported by Mirsal:

- `HTTP API` => used for operation APIs (e.g sending message)
- `Websocket API` => used for real-time data retrieval (e.g receiving message packet)

Check out `/docs/api_design` for more details on these APIs.

---
