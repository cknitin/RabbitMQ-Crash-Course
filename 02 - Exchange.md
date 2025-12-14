## 2. Exchange — the message router

The **Exchange** is the brain of RabbitMQ’s routing system.
It decides how messages flow from producers to one or more queues.

When a message arrives, the exchange looks at its **type**
and **bindings** to decide which queues should receive the message.

---

## Responsibilities

* Receive messages from producers
* Match routing keys against bindings
* Route copies of messages into the correct queues

---

## Main Exchange Types

| Type    | Routing Logic                  | Use Case                                       |
| ------- | ------------------------------ | ---------------------------------------------- |
| Direct  | Exact routing key match        | Logs, targeted routing                         |
| Fanout  | Broadcasts to all queues bound | Notifications, broadcasts                      |
| Topic   | Pattern-based routing          | Multi-service routing (`*.created`, `india.*`) |
| Headers | Route based on headers         | Advanced filtering                             |


