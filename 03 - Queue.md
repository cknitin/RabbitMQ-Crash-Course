## Queue — the message buffer

A **Queue** is a storage unit that holds messages until they are consumed.
Think of it as a **FIFO (First-In, First-Out)** message buffer.

Messages stay in the queue until:

* A consumer retrieves and **acknowledges** them
* The message **expires (TTL)**
* The **queue is deleted**

---

## Responsibilities

* Hold messages safely until processed
* Manage message order and delivery
* Support reliability (**acknowledgements, persistence, DLQ**)
* Support scaling (**multiple consumers**)

---

## Key Properties

| Property    | Description                              |
| ----------- | ---------------------------------------- |
| Durable     | Survives broker restarts                 |
| Exclusive   | Used only by one connection              |
| Auto-delete | Deleted when no consumers remain         |
| TTL         | Message expiry time                      |
| DLX         | Dead-letter exchange for failed messages |
| Lazy mode   | Stores messages on disk to save RAM      |

---

## Queue Declaration (C# / .NET)

```csharp
using RabbitMQ.Client;

// Create connection factory
var factory = new ConnectionFactory()
{
    HostName = "localhost"
};

// Open connection and channel
using var connection = factory.CreateConnection();
using var channel = connection.CreateModel();

// Declare queue
channel.QueueDeclare(
    queue: "orderQueue",
    durable: true,
    exclusive: false,
    autoDelete: false,
    arguments: null
);
```

---

### Example with Advanced Arguments (TTL & DLX)

```csharp
var args = new Dictionary<string, object>
{
    { "x-message-ttl", 60000 },                // 60 seconds
    { "x-dead-letter-exchange", "dlx_exchange" }
};

channel.QueueDeclare(
    queue: "orderQueue",
    durable: true,
    exclusive: false,
    autoDelete: false,
    arguments: args
);
```

---

### One-Line Summary ⭐

**A Queue safely stores messages and delivers them to consumers in order, reliably and at scale.**

Here’s the **text extracted from the image**, clearly transcribed:

---

## Queue Types (Special Behavior Queues)

RabbitMQ doesn’t have **“named queue types”** like *direct queue* or *topic queue*.
The **exchange** defines routing,
but **queue types here mean specialized behavior queues based on their arguments**.

Let’s explore the most important ones.

---

## 1. Standard Queue (Default)

* The most common queue type
* **FIFO (First-In-First-Out)** order is preserved (except when using priority)
* Messages are stored in **memory or disk** (depending on mode)
* Can have **multiple consumers** (for load balancing)

**Use case:**
General background jobs, microservices messaging.

Here’s the **extracted text from the image**, with the **code rewritten in .NET (C#)** instead of JavaScript.

---

## 2. Priority Queue

Messages can have a **priority level (integer)**.

Higher-priority messages are delivered first, even if others arrived earlier.

---

## Setup (C# – RabbitMQ)

```csharp
using RabbitMQ.Client;
using System.Collections.Generic;

var factory = new ConnectionFactory() { HostName = "localhost" };
using var connection = factory.CreateConnection();
using var channel = connection.CreateModel();

var arguments = new Dictionary<string, object>
{
    { "x-max-priority", 10 }
};

channel.QueueDeclare(
    queue: "priorityQueue",
    durable: true,
    exclusive: false,
    autoDelete: false,
    arguments: arguments
);
```

---

## Producer (C#)

```csharp
using RabbitMQ.Client;
using System.Text;

var factory = new ConnectionFactory() { HostName = "localhost" };
using var connection = factory.CreateConnection();
using var channel = connection.CreateModel();

var properties = channel.CreateBasicProperties();
properties.Priority = 8;

var message = "Urgent task";
var body = Encoding.UTF8.GetBytes(message);

channel.BasicPublish(
    exchange: "",
    routingKey: "priorityQueue",
    basicProperties: properties,
    body: body
);
```

---

## Use Cases

* Payment alerts
* Urgent notifications
* Critical updates

---

## 3. Dead Letter Queue (DLQ)

Stores failed messages that couldn’t be processed successfully.

Typically connected via a **Dead Letter Exchange (DLX)**.

---

### Setup Main Queue with DLQ (C#)

```csharp
using RabbitMQ.Client;
using System.Collections.Generic;

var factory = new ConnectionFactory() { HostName = "localhost" };
using var connection = factory.CreateConnection();
using var channel = connection.CreateModel();

var mainQueueArgs = new Dictionary<string, object>
{
    { "x-dead-letter-exchange", "dlx_exchange" },
    { "x-dead-letter-routing-key", "dead.msg" }
};

channel.QueueDeclare(
    queue: "mainQueue",
    durable: true,
    exclusive: false,
    autoDelete: false,
    arguments: mainQueueArgs
);
```

---

### Declare DLQ and Bind to DLX (C#)

```csharp
channel.ExchangeDeclare(
    exchange: "dlx_exchange",
    type: ExchangeType.Direct,
    durable: true
);

channel.QueueDeclare(
    queue: "dlqQueue",
    durable: true,
    exclusive: false,
    autoDelete: false
);

channel.QueueBind(
    queue: "dlqQueue",
    exchange: "dlx_exchange",
    routingKey: "dead.msg"
);
```

---

### Use Cases

* Error handling
* Retries
* Message auditing

---

---

## 4. Lazy Queue

Stores messages on **disk immediately instead of memory**.

Reduces **RAM usage** when message volume is high.

Slightly slower, but very **memory efficient**.

---

### Setup Lazy Queue (C#)

```csharp
var lazyQueueArgs = new Dictionary<string, object>
{
    { "x-queue-mode", "lazy" }
};

channel.QueueDeclare(
    queue: "lazyQueue",
    durable: true,
    exclusive: false,
    autoDelete: false,
    arguments: lazyQueueArgs
);
```

---

### Use Cases

* Large queues
* Background workloads
* High backlog systems

---


