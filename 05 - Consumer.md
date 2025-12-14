### **Consumer**

A **Consumer** is an application or service that reads messages from a queue and processes them.

It connects to RabbitMQ, subscribes to a queue, and waits for new messages to arrive.

### Consumers can:

* Auto-acknowledge messages (less reliable)
* Manually acknowledge after processing (recommended)
* NACK messages to requeue or send to DLQ

---

### Acknowledgement behaviors

```
channel.ack(msg)
→ Remove message from queue permanently

channel.nack(msg, false, true)
→ Requeue message for retry

channel.nack(msg, false, false)
→ Drop message (or send to DLQ)

(Consumer crash before ACK)
→ RabbitMQ requeues automatically
```

---

### Responsibilities

* Retrieve and process messages
* Send ACK / NACK
* Handle retries or DLQ
* Scale horizontally for parallel processing

---

### Example (concept – JS)

```js
channel.consume('orderQueue', (msg) => {
  const order = JSON.parse(msg.content.toString());
  console.log('Processing order:', order.orderId);

  // Simulate processing
  channel.ack(msg);
});
```

---

### Flow

1. RabbitMQ delivers message → Consumer
2. Consumer processes it
3. Consumer sends ACK → RabbitMQ removes message

---

## 🧩 Same Example in .NET (C#)

### RabbitMQ Consumer with Manual ACK

```csharp
using System;
using System.Text;
using RabbitMQ.Client;
using RabbitMQ.Client.Events;

class RabbitMqConsumer
{
    static void Main()
    {
        var factory = new ConnectionFactory()
        {
            HostName = "localhost"
        };

        using var connection = factory.CreateConnection();
        using var channel = connection.CreateModel();

        channel.QueueDeclare(
            queue: "orderQueue",
            durable: true,
            exclusive: false,
            autoDelete: false,
            arguments: null
        );

        // Manual acknowledgment (autoAck = false)
        var consumer = new EventingBasicConsumer(channel);

        consumer.Received += (sender, ea) =>
        {
            try
            {
                var body = ea.Body.ToArray();
                var message = Encoding.UTF8.GetString(body);

                Console.WriteLine($"Processing order: {message}");

                // Simulate processing success
                channel.BasicAck(
                    deliveryTag: ea.DeliveryTag,
                    multiple: false
                );
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error: {ex.Message}");

                // NACK and requeue message
                channel.BasicNack(
                    deliveryTag: ea.DeliveryTag,
                    multiple: false,
                    requeue: true
                );
            }
        };

        channel.BasicConsume(
            queue: "orderQueue",
            autoAck: false,
            consumer: consumer
        );

        Console.WriteLine("Consumer started. Press [Enter] to exit.");
        Console.ReadLine();
    }
}
```

---

## ✅ Key .NET Mappings

| Concept        | JavaScript                       | .NET (C#)              |
| -------------- | -------------------------------- | ---------------------- |
| Consume        | `channel.consume()`              | `BasicConsume()`       |
| ACK            | `channel.ack(msg)`               | `BasicAck()`           |
| NACK (requeue) | `channel.nack(msg, false, true)` | `BasicNack(..., true)` |
| Manual ACK     | default                          | `autoAck: false`       |

---

