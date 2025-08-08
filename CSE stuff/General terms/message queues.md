**Difference between message queues and streams:** 
	https://medium.com/@bindubc/distributed-system-difference-between-message-queue-and-streaming-eda505db96cc [[message streams]] vs message queues

A [message queue](https://blog.iron.io/right-on-queue-message-queues-made-simple/), sometimes called a point-to-point communication, is fairly straightforward. A message queue can have one or more consumers and/or producers. In a message queue with multiple consumers, the queue will attempt to distribute the messages evenly across them, with the guarantee being that every message will only be delivered once.

When all consumers are busy, the queue (or “broker”) will store messages, making it a durable queue. To ensure the broker doesn’t drop messages that haven’t been delivered yet, the queue needs to persist. Durability and persistency are two important qualities to look for in a message queue.