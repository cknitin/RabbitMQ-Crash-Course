**Binding**
→ *the connection between exchange and queue*

A **Binding** is the rule that connects an exchange to a queue.
It defines which messages go into which queue, based on routing keys or headers.

Without bindings, exchanges can’t route anything — they’d just drop messages.

### Example (concept)

```js
channel.bindQueue('emailQueue', 'order_exchange', 'order.email');
```

Whenever a producer sends a message to exchange `order_exchange` with routing key `order.email`,
it will be routed to `emailQueue`.

---

### You can have:

* One exchange bound to many queues
* One queue bound to many exchanges
* Multiple bindings for flexible routing

### Example:

**Exchange:** `order_events`

* (key: `order.created`) → Queue: `inventory`
* (key: `order.created`) → Queue: `email`

---

## 🧩 Same Example in .NET (C#)

Below is the **RabbitMQ binding code using C# (.NET)**:

```csharp
using RabbitMQ.Client;

class RabbitMqBindingExample
{
    static void Main()
    {
        var factory = new ConnectionFactory()
        {
            HostName = "localhost"
        };

        using var connection = factory.CreateConnection();
        using var channel = connection.CreateModel();

        // Declare exchange
        channel.ExchangeDeclare(
            exchange: "order_exchange",
            type: ExchangeType.Direct
        );

        // Declare queue
        channel.QueueDeclare(
            queue: "emailQueue",
            durable: true,
            exclusive: false,
            autoDelete: false,
            arguments: null
        );

        // Bind queue to exchange with routing key
        channel.QueueBind(
            queue: "emailQueue",
            exchange: "order_exchange",
            routingKey: "order.email"
        );

        Console.WriteLine("Queue bound successfully.");
    }
}
```

### 🔁 What this does

* Declares a **direct exchange**
* Declares a **queue**
* Binds the queue to the exchange using a **routing key**
* Messages sent with `order.email` will go to `emailQueue`

---


