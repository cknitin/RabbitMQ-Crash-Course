

### **Virtual Host (vHost)**

Logical namespace for isolation.

Think of it like **“databases” inside a DBMS**.

Each vHost has its own **exchanges, queues, and permissions**.

**Default vHost:** `/`

Useful for **multi-tenant systems**
or **environment separation** (`/dev`, `/prod`).

---

### **Message (Payload + Metadata)**

```json
{
  "body": { "orderId": 123 },
  "properties": {
    "contentType": "application/json",
    "deliveryMode": 2,
    "headers": { "region": "india" }
  }
}
```

---

### **Message Flow**

```
PRODUCER
   ↓
[Connection + Channel]
   ↓
EXCHANGE
   ↓
(Bindings)
   ↓
QUEUE
   ↓
CONSUMER
```

