---
layout: post
title: "Scheduling delayed messages with RabbitMQ using a Dead Letter Exchange"
description: ""
category: 
tags: [c#, rabbitmq]
---
{% include JB/setup %}

Sometimes you don't want messages in the queue to be read immediately. For example, you have a message that can't be procesed *right now* but you want to requeue it and try again in 5 minutes. Unfortunately, [RabbitMQ](http://www.rabbitmq.com/) doesn't come with native support for delayed or scheduled messages. Fortunately, RabbitMQ 2.8.0 introduced [Dead Letter Exchanges](http://www.rabbitmq.com/dlx.html) (DLX), which allows us to simulate message scheduling. 

#### Dead what? 

When you create a queue (A) bound to exchange (X), you can also specify an exchange (DLX) to redirect expired and rejected messages to. This is the dead letter exchange. Then you have another queue (B) bound to this dead letter exchange (DLX) to consume messages as they drop off the original queue (A).

In practice, if we wanted to enable retry on failure every 5 minutes, the flow would look like this:

1. Create `WorkQueue` bound to `WorkExchange`
2. Create `RetryQueue` bound to `RetryExchange`
  * Set `x-dead-letter-exchange` to `WorkExchange`
  * Set `x-message-ttl` to 300000 ms (5 minutes)
3. Publish message to `WorkQueue`
4. Client reads message from `WorkQueue` and attempts to process it
5. Process fails and client publishes to `RetryQueue`
6. Messages sits in `RetryQueue` for 5 minutes
7. When message expires, it is requeued to `WorkQueue` for another attempt at processing
8. Repeat steps 4-7
  
Here's how you do it in C#:

#### Create the WorkQueue

    // using RabbitMQ.Client;

    private const string WORK_QUEUE = "WorkQueue";
    private const string WORK_EXCHANGE = "WorkExchange"; // dead letter exchange
    
    ConnectionFactory factory = new ConnectionFactory();
    factory.HostName = "localhost";
     
    IConnection connection = factory.CreateConnection();
    IModel channel = connection.CreateModel();
     
    channel.ExchangeDeclare(WORK_EXCHANGE, "direct");
    channel.QueueDeclare(WORK_QUEUE, true, false, false, null);
    channel.QueueBind(WORK_QUEUE, WORK_EXCHANGE, string.empty, null);

#### Create the RetryQueue

    private const string RETRY_EXCHANGE = "RetryExchange";
    private const string RETRY_QUEUE = "RetryQueue";
    private const int RETRY_DELAY = 300000; // in ms

    // messages will drop off RetryQueue into WorkExchange for reprocessing
    var queueArgs = new Dictionary<string, object> {
        { "x-dead-letter-exchange", WORK_EXCHANGE },
        { "x-message-ttl", RETRY_DELAY }
    };

    channel.ExchangeDeclare(RETRY_EXCHANGE, "direct");
    channel.QueueDeclare(RETRY_QUEUE, true, false, false, queueArgs);
    channel.QueueBind(RETRY_QUEUE, RETRY_EXCHANGE, string.empty, null);

#### Read from WorkQueue

    QueueingBasicConsumer consumer = new QueueingBasicConsumer(channel);
    channel.BasicConsume(WORK_QUEUE, true, consumer);

    while (true) {
        BasicDeliverEventArgs e = (BasicDeliverEventArgs) consumer.Queue.Dequeue();
        var message = Encoding.UTF8.GetString(e.Body);
        channel.BasicAck(e.DeliveryTag);

        if (!DoSomething(message)) {
            TryAgainLater(message);
        }
    }

#### Publish to RetryQueue on failure

    var data = Encoding.UTF8.GetBytes(message)
    channel.BasicPublish(RETRY_EXCHANGE, string.empty, null, data);

The code above will keep retrying indefinitely. It's a good idea to check against some sort of MAX_RETRY constant. If your messages are JSON strings, it's easy to encode the retry count in the message itself.


