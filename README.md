# micro-service-toolkit
Wire business logic in node with in- and outbound channels (e.g. RabbitMQ) by JSON config. 

## What's it?
We're running a lot of microservices on node.js communicating via RabbitMQ topics. It was annoying to see lot of similar infrastructure glue code.

This is an approach solves that by

1. separating technical infrastructure code completely from business logic
2. avoiding hard coded infrastructure code and use JSON based configuration instead

The business logic is connected to in- and outbound channels (e.g. an RabbitMQ topic) by defining that in the `config.json`.

Have a look at the [example](https://github.com/ma-ha/micro-service-toolkit/tree/master/test) :-)

Published as [npm package](https://www.npmjs.com/package/micro-service-toolkit), you can just do `npm install micro-service-toolkit`.

## Files in this project
* `service.js`: the microservice technical framework
* `channels/inboundRabbitMq.js`: first channel plug in
* `test/myMessageProcessor.js`: dummy microservice business logic part
* `test/config.json`: configuration how to connect in- and output channels to the business logic
* `test/sendTesMsg.js`: create a test message to be processed by the 'MessageProcessor'

## Run the 'MessageProcessor' test service
To start this MessageProcessor micro services:

1. Start RabbitMQ: 
  1. `docker pull rabbitmq`
  2. `docker run -d --hostname my-rabbit --name some-rabbit -e RABBITMQ_DEFAULT_USER=user -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_PASS=password rabbitmq:3-management` 
  3. Console is on `http://localhost:15672`
2. Prepare the service: `npm install`
3. Start the service in the test directory: `node myMessageProcessor.js --rabbitHost=localhost`

To send a test message simply run `node sendTestMsg.js --rabbitHost=localhost` in the test directory.

BTW: The framework uses [amqp-heartbeat](https://www.npmjs.com/package/amqp-heartbeat) to enable a monitoring of the microservices.

## Gernerate Code Template
Call syntax:

```bash
node generateTemplate.js --config=./test/config.json > ./test/myService.js
```

# Command Messages
In an asynchronous environment the best way to manage the services is via messages.

The exchange for command messages is `microservicetoolkit`. Each service is listening to
a topic with the `<service name (lowecase)>.*` as filter.

Message format is:
```js
{
	"cmd":"<command>",
	"authKey":"<secret key>",   // optional -- for authorization commands
	"version":"<no>",           // optional -- to address services of on version
	"serviceId":<UUID>          // optional -- to address a single service (ref. heartbeat)
}
```

Currently supported command messages are
1. *start* starts the message consumers
2. *kill* stops the whole microservice process

By stopping "version-1" and starting "version-2" 
(JS has no `service.start()` in `service.init(..)` code), 
you can do version switch in a running system environment.

In the test folder you can try that:

```bash
node sendStopMsg.js --rabbitHost=localhost --service=MessageProcessor
```

More generic is `node sendCmdMsg.js` (will print out the usage).