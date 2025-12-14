## Core Components

### 1. Producer

**Sends (publishes) messages to an Exchange.**

* A producer **never sends messages directly to a queue**
* It always sends messages to an **Exchange**
* The **Exchange decides** where the message should go

---

## Technical Behavior

A Producer:

* Opens a **connection** and a **channel** with RabbitMQ
* Publishes a message with:

  * **Exchange name**
  * **Routing key**
  * **Message body (payload)**
  * **Properties** (headers, persistence flag, priority, etc.)

---

## Important Note

* Every message sent by a producer reaches the **exchange in binary form**
* RabbitMQ uses **AMQP**, which is a **binary protocol**
* The **developer is responsible** for converting structured data (like JSON) into bytes before sending

---

## Example (C# / .NET)

```csharp
using RabbitMQ.Client;
using System.Text;
using System.Text.Json;

// Create connection factory
var factory = new ConnectionFactory()
{
    HostName = "localhost"
};

// Open connection and channel
using var connection = factory.CreateConnection();
using var channel = connection.CreateModel();

// Create message object
var order = new
{
    orderId = 101,
    user = "Aniket"
};

// Serialize object to JSON
var messageBody = JsonSerializer.Serialize(order);
var body = Encoding.UTF8.GetBytes(messageBody);

// Set message properties
var properties = channel.CreateBasicProperties();
properties.Persistent = true;

// Publish message
channel.BasicPublish(
    exchange: "orders_exchange",
    routingKey: "order.created",
    basicProperties: properties,
    body: body
);
```

---

## Responsibilities of a Producer

* **Serialize data** (e.g., convert object to JSON)
* **Define routing key**
* **Ensure persistence** (so messages survive broker restart)
* **Handle delivery confirmations** (optional but recommended)


