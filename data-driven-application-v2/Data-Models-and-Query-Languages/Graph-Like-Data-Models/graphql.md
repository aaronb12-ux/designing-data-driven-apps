# GraphQL

GraphQL is a query language that, by design, is much more restrictive than others we have seen in this chapter. It is intended for **OLTP queries**; its purpose is to allow client software running on a user's device (such as a mobile app or a JavaScript web app frontend) to request a JSON document with a particular structure, containing the fields necessary for rendering its UI.

GraphQL interfaces allow developers to rapidly change queries in client code without changing server-side APIs.

Organizations that adopt GraphQL often need tooling to convert the queries into requests to internal services, which commonly use REST or gRPC. Authorization, rate limiting, and performance are additional concerns.

GraphQL does **not** allow recursive queries, and it does not allow arbitrary search conditions (like our "find people who were born in the US and are now living in Europe" example), unless the service owners specifically choose to offer such search functionality.

## Example: A Group Chat Application

The example below shows how one might implement a group chat application like Discord or Slack using GraphQL.

```graphql
query ChatApp {
  channels {
    name
    recentMessages(latest: 50) {
      timestamp
      content
      sender {
        fullName
        imageUrl
      }
      replyTo {
        content
        sender {
          fullName
        }
      }
    }
  }
}
```

This query gathers the most recent 50 messages sent. For each message, it requests the timestamp, content, and sender info. If the message is a `replyTo`, it also requests the content and sender of the original message.

The response would look like this:

```json
{
  "data": {
    "channels": [
      {
        "name": "#general",
        "recentMessages": [
          {
            "timestamp": 1693143014,
            "content": "Hey! How are y'all doing?",
            "sender": { "fullName": "Aaliyah", "imageUrl": "https://..." },
            "replyTo": null
          },
          {
            "timestamp": 1693143024,
            "content": "Great! And you?",
            "sender": { "fullName": "Caleb", "imageUrl": "https://..." },
            "replyTo": {
              "content": "Hey! How are y'all doing?",
              "sender": { "fullName": "Aaliyah" }
            }
          }
        ]
      }
    ]
  }
}
```

This approach has the advantage that the server does not need to know which attributes the client requires in order to render the user interface — the client can simply request what it needs.

> **GraphQL is nice because instead of returning the entire document or row, you only specifically ask for the fields in the document or row you want returned.**

GraphQL can be implemented on top of any type of database — relational, document, or graph.



