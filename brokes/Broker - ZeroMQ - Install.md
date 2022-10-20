# RabbitMQ - Install e Samples


[Documentation ZeroMQ](https://zguide.zeromq.org/docs/chapter2/)
[Sockets in ZeroMQ](http://api.zeromq.org/master:zmq-socket)
[Project Sample 1 - ]()



## Install ZeroMQ Docker


```bash
docker pull zeromq/zeromq

docker run -d --rm  \
    --publish 5559:5559 \
    --publish 5560:5560 \
    --publish 5562:5562 \
    --network=zmq  \
    --name zeromq   zeromq/zeromq

# docker run -d --rm  --name zeromq   zeromq/zeromq
```


## Install Library Dependency

```bash
echo "deb http://download.opensuse.org/repositories/network:/messaging:/zeromq:/release-stable/Debian_9.0/ ./" >> /etc/apt/sources.list
wget https://download.opensuse.org/repositories/network:/messaging:/zeromq:/release-stable/Debian_9.0/Release.key -O- | sudo apt-key add
apt-get install libzmq3-dev


git clone https://github.com/zeromq/libzmq.git
cd libzmq
./autogen.sh
./configure
make -j 3
sudo make install

```

## Run Samples C and C++

