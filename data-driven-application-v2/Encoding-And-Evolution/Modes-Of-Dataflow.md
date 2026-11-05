# Modes of Dataflow

We will discuss the most common ways data flows between processes:

- Through databases
- Through service calls
- Through workflow engines
- Through asynchronous messages

## Dataflow Through Databases

In a database, the process that writes data encodes it, while the process that reads the data decodes it.

Think of storing something in a database as **sending a message to your future self**.

It is common for multiple processes to access the same database simultaneously. These processes might be:

- Different applications or services.
- Multiple instances of the same service running in parallel for scalability or fault tolerance.

As a result, some processes may be running newer versions of the code while others are still running older versions.

This means a value in the database could be written by a newer version of the code and subsequently read by an older version.

Therefore, **forward compatibility is particularly important when using databases**.

### Different Values Written at Different Times

Rewriting (migrating) existing data into a new schema is possible, but it can be expensive when dealing with large datasets.

Some storage engines handle schema changes without requiring an immediate rewrite of all existing data:

- **LSM-tree storage engines:** Data is rewritten into the latest format during compaction.
- **Relational databases:** Many allow simple schema changes, such as adding a new column with a `NULL` default value, without immediately rewriting existing rows.

When an old row is read, the database can supply `NULL` for any columns that are missing from the data stored on disk.

Schema evolution therefore allows the database to appear as though all records use a single, consistent schema, even though the underlying storage may contain records written using different historical versions of the schema.

More complex schema changes may require rewriting data at the application level. Examples include:

- Changing a single-valued attribute into a multivalued attribute.
- Moving data from an existing table into a separate table.

### Archival Storage

It is common to take snapshots of a database for backup purposes or to load data into a data warehouse.

It is useful to encode data consistently across different schema versions because snapshots taken at different points in time may contain data written using different historical schemas.

Since a database dump is typically written in one operation and remains immutable afterward, formats such as **Avro Object Container Files** are a good fit.

They allow records to be stored together with the schema needed to interpret them, making them suitable for large, immutable data dumps.

## Dataflow Through Services: REST and RPC

The most common way for processes to communicate over a network is through services:

- **Servers:** Expose an API over a network.
- **Clients:** Connect to servers and make requests to their APIs.
- **Services:** Provide functionality through an API exposed by a server.

The web works this way. Clients (web browsers) make requests to web servers:

- `GET` requests retrieve resources such as HTML, CSS, JavaScript, and images.
- `POST` requests submit data to the server.

In some ways, services are similar to databases because they typically allow clients to submit and retrieve data.

However, services expose an **application-specific API** that only allows inputs and outputs predetermined by the business logic of the application.

This means services can impose fine-grained restrictions on what clients can and cannot do.

A key design goal of a **service-oriented architecture (SOA)** or **microservice architecture** is to make applications easier to change and maintain by making services independently deployable and evolvable.

### Web Services

A service that uses HTTP as its communication protocol is known as a **web service**.

Web services are used in several contexts, not just on the public web:

- A client application running on a user's device makes requests to a service over HTTP through the public internet.
- One service makes requests to another service within the same organization.
- One service makes requests to a service owned by a different organization, usually over the internet.

### Interface Description Languages (IDLs)

Service developers often use an **IDL (Interface Description Language)** to define and document their service's API endpoints and data models, as well as manage changes to them over time.

An IDL is a specialized, human-readable language used to define the API or communication interface between different software components, regardless of the programming languages they are written in.

For example, **OpenAPI** allows developers to describe HTTP APIs using YAML or JSON.

#### Example: OpenAPI Service Definition

```yaml
openapi: 3.0.0
info:
  title: Ping, Pong
  version: 1.0.0
servers:
  - url: http://localhost:8080
paths:
  /ping:
    get:
      summary: Given a ping, returns a pong message
      responses:
        '200':
          description: A pong
          content:
            application/json:
              schema:
                type: object
                properties:
                  message:
                    type: string
                    example: Pong!
```

### Service Frameworks

Service frameworks allow developers to focus on writing the business logic for each API endpoint, while the framework handles common infrastructure tasks, such as:

- Request routing
- Metrics and monitoring
- Caching
- Authentication
- Request and response handling

#### Example: FastAPI Service Implementation

The following FastAPI application implements the `/ping` endpoint defined in the OpenAPI example.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="Ping, Pong", version="1.0.0")

class PongResponse(BaseModel):
    message: str = "Pong!"

@app.get(
    "/ping",
    response_model=PongResponse,
    summary="Given a ping, returns a pong message"
)
async def ping():
    return PongResponse()
```

FastAPI uses the route decorator to define the endpoint and Pydantic to define the response data model. It can also automatically generate OpenAPI documentation from the application.

### The Problems with Remote Procedure Calls (RPC)

A **Remote Procedure Call (RPC)** model attempts to make a request to a remote network service look like calling a function or method within the same process.

This abstraction is sometimes called **location transparency**: the idea that the location of the code being executed should not matter to the caller.

However, a network request is fundamentally different from a local function call.

#### Why Remote Calls Are Different

- **Unpredictability:** A local function call typically executes within the same process and is not dependent on network connectivity. A network request can fail because of network problems, server failures, or other external conditions.
- **Timeouts:** A local function call either returns a result, throws an exception, or never returns. A network request introduces another possible outcome: it may time out without returning a result. Importantly, the server may have successfully processed the request even though the client never received the response.
- **Latency:** A network request is significantly slower than a local function call, and its latency is highly variable. It may respond quickly under normal conditions but become much slower when the network or server is overloaded.
- **Data serialization:** When calling a local function, we can efficiently pass references (pointers) to objects in memory. With network requests, parameters must first be encoded into a sequence of bytes and transmitted over the network.

All of these differences mean there is little benefit in trying to make a remote service behave exactly like a local object in a programming language.

Part of the appeal of **REST** is that it treats communication over a network as a distinct process involving the transfer of resources and representations, rather than pretending that a remote operation is simply a local function call.
    

    

  
  
