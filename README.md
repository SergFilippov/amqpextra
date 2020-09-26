# Extra features for streadway/amqp package. 
<a href="https://travis-ci.org/makasim/amqpextra"><img src="https://travis-ci.org/makasim/amqpextra.png?branch=master" alt="Build Status"></a>

## Auto reconnecting.

The package provides an auto reconnect feature for [streadway/amqp](https://github.com/streadway/amqp). The approach idea is to add as little abstraction as possible. In your code instead of using `*amqp.Connection` you should use `<-chan *amqp.Connection`. The channel returns a healthy connection. You should subscribe to `chan *amqp.Error` to get notified when a connection is not helthy any more and you should request a new one via  `<-chan *amqp.Connection`. The channel `<-chan *amqp.Connection` is closed when you explicitly closed it by calling `connextra.Close()` method, otherwise, it tries to reconnect in background.

## Dial multiple hosts

The Dial method accepts a slice of connection URLs. It would round robin them till one works.

## Consumer.

The package provides a handy consumer abstraction that works on top of `<-chan *amqp.Connection` and `<-chan *amqp.Error` channels.

#### Workers

Consumer can start multipe works and spread the processing between them.

#### Context

Consumer supports context.Context. The context is passed to worker function. You can build timeout, cancelation strategies on top of it.

#### Middleware

The consumer could chain middlewares for pre precessing received message. 
Check an example that rejects messages without correlation_id and reply_to properties.  

Some built-in middlewares:

* [HasCorrelationID](consumer/middleware/has_correlation_id.go) - Nack message if has no correlation id
* [HasReplyTo](consumer/middleware/has_reply_to.go) - Nack message if has no reply to.
* [Logger](consumer/middleware/logger.go) - Context with logger.
* [Recover](consumer/middleware/recover.go) - Recover worker from panic, nack message.
* [Expire](consumer/middleware/expire.go) - Convert Message expiration to context with timeout.
* [AckNack](consumer/middleware/ack_nack.go) - Return middleware.Ack to ack message.

## Publisher.

The package provides a handy publisher. 
* Handles re-connection, channel close.
* Context aware.
* Wait between re-connections.
* Provides ready\unready\closed status channels.
* An easy configuration (WithXXX).
* Support [flow control](https://www.rabbitmq.com/flow-control.html). 
