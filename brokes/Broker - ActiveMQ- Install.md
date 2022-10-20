# Apache ActiveMQ Artemis 2.25.0 - Install e Samples

[Documentation Artemis](https://activemq.apache.org/components/artemis/documentation/)
[github](https://hub.docker.com/r/cloudesire/activemq)
[examples](https://qpid.apache.org/releases/qpid-proton-0.37.0/proton/cpp/api/simple_send_8cpp-example.html)



## Install ZeroMQ Docker

```bash
docker pull cloudesire/activemq

export AMQ_USER=admin ; \
export AMQ_PASSWORD=admin ; \
docker run --detach --rm \
    -e AMQ_USER=${AMQ_USER} -e AMQ_PASSWORD=${AMQ_PASSWORD} \
    --publish 61616:61616  --publish 61617:61617  --publish 8161:8161 --publish 95672:5672 \
    --name artemis quay.io/artemiscloud/activemq-artemis-broker


export AMQ_USER=admin ; \
export AMQ_PASSWORD=admin ; \
docker run -ti  --rm \
    -e AMQ_USER=${AMQ_USER} -e AMQ_PASSWORD=${AMQ_PASSWORD} \
    --publish 61616:61616/tcp  --publish 61617:61617/tcp \
    --name artemis quay.io/artemiscloud/activemq-artemis-broker

export AMQ_USER=admin ; \
export AMQ_PASSWORD=admin ; \
docker run --detach --rm \
    -e AMQ_USER=${AMQ_USER} -e AMQ_PASSWORD=${AMQ_PASSWORD} \
    --publish 61616:61616/tcp  --publish 61617:61617/tcp \
    --name artemis quay.io/artemiscloud/activemq-artemis-broker

docker exec -ti artemis /bin/bash

```

## Install Libraries


### AMQP CPP - ProtonCPP

[docs](https://qpid.apache.org/releases/qpid-proton-0.37.0/proton/cpp/api/index.html)
[ProtonCPP - AMQP CPP example](https://github.com/apache/activemq-artemis/tree/main/examples/protocols/amqp/proton-cpp)

```bash
sudo apt install -y  libqpid-proton-cpp12  libqpid-proton-cpp12-dev

```
<!-- 

### CMS Client - C++ Messaging API

[about](https://activemq.apache.org/components/cms/)
[Read the Docs](https://activemq.apache.org/components/cms/documentation)

```bash
pushd ~/lib-external/
git clone https://gitbox.apache.org/repos/asf/activemq-cpp.git
cd activemq-cpp
git checkout tags/3.9.5
sudo apt-get install uuid-dev libcppunit-dev

cd activemq-cpp
./autogen.sh 
sudo apt-get install -y libcppunit-dev libapr1 libapr1-dev
./configure
make -j 6
make install

``` -->

