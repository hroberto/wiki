# RabbitMQ - Install e Samples


[RabbitMQ - Documentation](https://www.rabbitmq.com/documentation.html)
[RabbitMQ - Get Started](https://www.rabbitmq.com/getstarted.html)
[Project Sample 1 - SimpleAmqpClient](https://github.com/alanxz/SimpleAmqpClient)
[Project Sample 2 - amqpcpp](https://github.com/akalend/amqpcpp)
[Project Sample 3 - AMQP-CPP](https://github.com/CopernicaMarketingSoftware/AMQP-CPP)



## Install RabbitMQ Docker

```bash
docker pull rabbitmq:3.10-management
# interactive + tty
##docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.10-management
docker run --detach   --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.10-management


docker exec -ti rabbitmq /bin/bash

```

## Install Project C/C++ Samples

```bash
## RabbitMQ C client
git clone https://github.com/alanxz/rabbitmq-c

cd rabbitmq-c
mkdir build && cd build
cmake ..
cmake  -DBUILD_EXAMPLES=ON  --build .
```


## Run Samples C and C++

-  Sample 1 - Echo

```bash
./examples/amqp_listen localhost 5672 amq.direct test &
./examples/amqp_sendstring localhost 5672 amq.direct test "hello world"
fg

- Log RabbitMQ
<!-- 2022-09-29 12:50:07.690176+00:00 [info] <0.968.0> accepting AMQP connection <0.968.0> (172.17.0.1:41090 -> 172.17.0.2:5672)
2022-09-29 12:50:07.693591+00:00 [info] <0.968.0> connection <0.968.0> (172.17.0.1:41090 -> 172.17.0.2:5672): user 'guest' authenticated and granted access to vhost '/'
2022-09-29 12:50:11.472361+00:00 [info] <0.985.0> accepting AMQP connection <0.985.0> (172.17.0.1:41094 -> 172.17.0.2:5672)
2022-09-29 12:50:11.477081+00:00 [info] <0.985.0> connection <0.985.0> (172.17.0.1:41094 -> 172.17.0.2:5672): user 'guest' authenticated and granted access to vhost '/'
2022-09-29 12:50:11.480192+00:00 [info] <0.985.0> closing AMQP connection <0.985.0> (172.17.0.1:41094 -> 172.17.0.2:5672, vhost: '/', user: 'guest') -->
```


- Sample 2 - Consumer e Producer

```bash 
# host port exchange  bindingkey
./examples/amqp_consumer localhost 5672 amq.direct test &
# host port rate_limit  message_count
./examples/amqp_producer localhost 5672  1 10

```



## Run Samples 

```
cd build
make 
./bin/Debug/samples_rabbitmq/amqp_producer localhost 5672  1 10 'test queue'
../bin/Debug/samples_rabbitmq/amqp_consumer    localhost 5672  'amq.direct'  'test queue'


../bin/Debug/samples_rabbitmq/amqp_test_consumer    localhost 5672 'Exchange-Test'   'topic'  'test.#' 'test.client1'
../bin/Debug/samples_rabbitmq/amqp_test_consumer    localhost 5672 'Exchange-Test'   'topic'  'test.#' 'test.client2'
../bin/Debug/samples_rabbitmq/amqp_sendstring  localhost 5672  'Exchange-Test'   'test.#'  'Hello !!'

clear ; rabbitmqctl  report

rabbitmqctl  delete_queue   test.client2 ; \
rabbitmqctl  delete_queue  test.client1
rabbitmqctl list_channels connection messages_unacknowledged



