RabbitMQ Dead lettering
---

Dead lettering concept has been around for quite sometime.

We had a requirement, where we had to publish messages after a delay to the desired queue.
Now there are many ways to simulate a delay, like:

- Java Scheduler
- Databases techniques
- etc

But RabbitMQ has this interesting concept of dead lettering.

**Concept:**

In a nutshell, we create 2 queues, the main queue and the delay queue.
We create the main queue normally.
We create the delay queue by defining the Dead letter exchange (or "x-dead-letter-exchange") argument along with Dead letter routing key (or "x-dead-letter-routing-key") argument. The value for the "Dead letter Exchange" would be exchange name of the main queue, and the value for "Dead letter routing key" is the routing key used while defining the binding between the main exchange and the main queue.

Now when a message is published with a delay, we publish the message to the delay queue and set the message "expiration" to the delay seconds (in millis).
Internally what happens is, when the message expires on the delay queue, RabbitMQ removes the messages from the delay queue and makes it available on the main queue.

Using this technique has many advantages:

- No loss of messages if JVM crashes, and the message was in memory, waiting to be published
- No heavy database solutions required
- No need of schedulers

Hope you find this useful, like I did!!!

Here is the official link [Dead Letter Exchange](http://www.rabbitmq.com/dlx.html)
