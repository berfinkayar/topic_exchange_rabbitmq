## Sending and Receiving Messages with RabbitMQ using Python

Firstly we're going to install the Pika which is a Python client recommended by RabbitMQ.
```
python -m pip install pika --upgrade
```
Then, on the producer part, we're going to establish a connection with our RabbitMQ server via this client.
```
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
```
Next, we have to create the queue which our message is going to be delivered through.
```
channel.queue_declare(queue='queue_name')
```
But before it gets enqueued, our message needs to go through an exchange. We define a default exchange with the empty string and simply specify the queue name in the routing_key parameter. We pass our message with the body parameter.
```
channel.basic_publish(exchange='',
                      routing_key='hello',
                      body='Hello World!')
```
On the consumer side, we first connect to the RabbitMQ server as well. Since we may not know which program is going to be run first, to declare the same queue again is a good practice. 
Then, our consumer has to subscribe a callback function to the queue, in order to be informed whenever a new message arrives. 
```
def callback(ch, method, properties, body):
    print(f" [x] Received {body}")
```
Now we have to let RabbitMQ know that this callback function should receive messages from our queue.
```
channel.basic_consume(queue='hello',
                      auto_ack=True,
                      on_message_callback=callback)
```
As a result, we enter a loop that is waiting for data and runs callbacks when it's necessary.
```
channel.start_consuming()
```
Now our consumer listens constantly, and producer is able to send messages whenever it's run!

![alt text](screenshots/sender.png)
![alt text](screenshots/receiver.png)

## Topic Exchange

This time let's use a different exchange type. In topic exchange, the producer sends messages to the topic, and the consumers which are interested in that topic follows that topic and receives the messages. Their routing_key can contain an asterisk (*), which matches to exactly one key, and a hash (#), which matches to zero ore more keys.
We firstly connect to are server by using the same functions. Then we define our queue with the exchange_type parameter set to 'topic'.
```
channel.exchange_declare(exchange='topic_logs', exchange_type='topic')
```
Then we publish our message with the following function. The routing_key and the message are defined from command line.
```
channel.basic_publish(
    exchange='topic_logs', routing_key=routing_key, body=message)
```
On the consumer side, we check the binding keys for that consumer, defined from the command line. These binding keys can be thought as the topics the consumer is interested in. Then we bind the exchange name with the queue and the binding key.  Now, when a message is published to 'topic_logs' with the routing_key equal to binding_key, it will be routed to the specified queue.
```
for binding_key in binding_keys:
    channel.queue_bind(
        exchange='topic_logs', queue=queue_name, routing_key=binding_key)

```
Now we create three consumers. Upper right one is strictly interested in calm and orange dogs. So it doesn't receive any messages sent by the producer. Lower left one is interested in any crazy animal. So it receives the messages with the binding keys "crazy.orange.cat" and "crazy.orange.kitten". Finally, the lower right one is interested in all cats and dogs, so it receives the messages with the binding keys "crazy.orange.cat" and "calm.golden.dog".
![alt text](screenshots/topic_exchange.png)
