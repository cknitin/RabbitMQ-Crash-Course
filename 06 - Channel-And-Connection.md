## **Internal Components (Under the Hood)**

RabbitMQ has some hidden but critical internal components that make these 5 core parts work efficiently.

---

## **Channel**

A **lightweight virtual connection** inside a TCP connection.

* You can have multiple channels per connection.
* Used for publishing, consuming, and managing queues.

**Why?**
Opening/closing TCP connections is expensive;
channels are cheap.

**Example:**
One app → one TCP connection →
multiple channels for producers/consumers.

---

## **Connection**

A **real TCP connection** between your app and the RabbitMQ broker.

* Established using the AMQP protocol
  (example: `amqp://localhost:5672`)
* RabbitMQ allows multiple concurrent connections
  from different apps or services.

