# REDIS - Guia básico de utilização

<details><summary>1. Instalando o Ubuntu via WSL</summary>
<p>


## 1.1. Configuração o WSL

```bash
wsl -s -v
    NAME            STATE           VERSION
    * Ubuntu-20.04    Running         1

wsl --set-version  Ubuntu-20.04  2
    Conversão em andamento. Isso pode levar alguns minutos...
    Para obter informações sobre as principais diferenças em relação ao WSL 2, visite https://aka.ms/wsl2

 wsl -l -v
  NAME            STATE           VERSION
* Ubuntu-20.04    Stopped         2

```

## 1.2. Baixando o Docker Redis


```sh
sudo service docker start
# sudo systemctl start docker

docker pull redis:latest

# Instancia temporia
docker run --rm --name rdsTemp --publish 6378:6379 -d redis:latest

docker run --rm  --detach   --name rdsVol \
    --publish 6379:6379 \
    -v /docker/host/dir:$HOME/redis-data \
    redis:latest

docker run --rm  --detach --name rdsWithPass \
    --publish 6379:6379 \
    -v /docker/host/dir:$HOME/redis-data \
    -e REDISCLI_AUTH=96b19ef2495a2979a97a4a5a8c578b6ca56ade1e \
    redis:latest \
    /bin/sh -c 'redis-server --requirepass ${REDISCLI_AUTH}'


docker exec -ti <Name Conteiner>   /bin/bash

```
</p>
</details>




<details><summary>2. Gerenciando o servidor Redis</summary>
<p>

## 2.1. Desligando o servidor Redis

```bash
# SHUTDOWN   [NOSAVE | SAVE]   [NOW] [FORCE] [ABORT]
redis-cli SHUTDOWN SAVE

## OU via systemctl
sudo shutdown restart redis

# Visualizando o log do Redis
sudo tail -f  /var/log/redis/redis-server.log

```


## 2.2. Limitando a quantidade de memoria RAM utilizada no servidor

Abaixo temos o comando utilizado para alterar o parametro da quantidade de memoria limite a ser utilizdo pelo Redis. Por padrão, o Redis não possui limitação de memoria.
Uma opção alternativva para a alteração, é realizar a mudança ONLINE via comando "CONFIG SET" e posteriormente registrar a mudança no arquivo "redis.conf" em caso de posterior reinicialização do Redis.

Uma opção alternativa para o comando abaixo, é a edição do arquivo ( /etc/redis/redis.conf ), habilitando os parametros abaixo, e a posterior reinicialização do Redis.


```bash

redis-cli CONFIG SET  maxmemory  6g
redis-cli CONFIG SET  maxmemory-policy  allkeys-lfu

redis-cli CONFIG GET maxmemory*
# 1) "maxmemory-policy"
# 2) "allkeys-lfu"
# 3) "maxmemory-samples"
# 4) "5"
# 5) "maxmemory"
# 6) "6442450944"

sudo vi  /etc/redis/redis.conf
> maxmemory  6g
> maxmemory-policy  allkeys-lfu

```


## 2.3. Otimizando a persistência manualmente

> [Persistência do Redis](https://redis.io/docs/manual/persistence/)
> [Redis - Comando BGREWRITEAOF](https://redis.io/commands/bgrewriteaof/)

```bash
redis-cli -n <N_DATABASE> MEMORY PURGE

redis-cli  SAVE
# OK
# (3.45s)

redis-cli  BGREWRITEAOF
# Background append only file rewriting started

# Visualizando o log do Redis
sudo tail -f  /var/log/redis/redis-server.log

```

- Visualizando configurações atuais

```bash
redis-cli CONFIG GET append*
# 1) "appendonly"
# 2) "yes"
# 3) "appendfilename"
# 4) "appendonly.aof"
# 5) "appendfsync"
# 6) "everysec"

redis-cli CONFIG GET save
# 1) "save"
# 2) "900 1 300 10 60 10000"

redis-cli CONFIG GET *aof*
#  1) "aof-rewrite-incremental-fsync"
#  2) "yes"
#  3) "aof-load-truncated"
#  4) "yes"
#  5) "aof-use-rdb-preamble"
#  6) "yes"
#  7) "aof_rewrite_cpulist"
#  8) ""
#  9) "auto-aof-rewrite-percentage"
# 10) "100"
# 11) "auto-aof-rewrite-min-size"
# 12) "67108864"
# 13) "replicaof"
# 14) ""

```



## 2.4. Monitorando as alocações no Redis.


* Visualizando resumo de chaves por database

```bash
redis-cli info keyspace
    # # Keyspace
    # db5:keys=124,expires=0,avg_ttl=0
    # db6:keys=17,expires=0,avg_ttl=0
    # db8:keys=1303,expires=1301,avg_ttl=61822
```


* Detalhamento de alocação do Database

```bash
redis-cli -n <N_DATABASE> --memkeys

# # Scanning the entire keyspace to find biggest keys as well as
# # average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# # per 100 SCAN commands (not usually needed).

# [00.00%] Biggest zset   found so far '"ALGO_USD_CFD_1"' with 3658568 bytes
# [00.00%] Biggest string found so far '"BTC_BRL"' with 312 bytes
# [00.00%] Biggest zset   found so far '"BTC_USD_T"' with 103111379 bytes
# [08.87%] Biggest hash   found so far '"exchange-rate"' with 1336 bytes
# [16.94%] Biggest string found so far '"BTC_BRL_DAY"' with 376 bytes
# [49.19%] Biggest hash   found so far '"listSymbols"' with 2808 bytes

# -------- summary -------

# Sampled 124 keys in the keyspace!
# Total key length in bytes is 1284 (avg len 10.35)

# Biggest   hash found '"listSymbols"' has 2808 bytes
# Biggest string found '"BTC_BRL_DAY"' has 376 bytes
# Biggest   zset found '"BTC_USD_T"' has 103111379 bytes

# 0 lists with 0 bytes (00.00% of keys, avg size 0.00)
# 2 hashs with 4144 bytes (01.61% of keys, avg size 2072.00)
# 40 strings with 12096 bytes (32.26% of keys, avg size 302.40)
# 0 streams with 0 bytes (00.00% of keys, avg size 0.00)
# 0 sets with 0 bytes (00.00% of keys, avg size 0.00)
# 82 zsets with 355773059 bytes (66.13% of keys, avg size 4338695.84)
```


## 2.5. Controlando a instancia como replica ( Slave )

Na instancia que receberá a replicação, execute:

- Para Habilitar a replicação

```sh
# Definir a senha de autenticação da instancia MASTER
export REDISCLI_AUTH='<Senha do Host Redis>'
redis-cli  CONFIG  SET  masterauth  ${REDISCLI_AUTH}

redis-cli REPLICAOF hostname 6379
```

- Voltar a instancia como MASTER e Desligar a replicação

```bash
redis-cli REPLICAOF NO ONE
```


## 2.6. Verificando uso utilizado por Chaves

```bash

clear ; \
NDB=6 ;\
for KEY in `redis-cli -n ${NDB} --scan --pattern "*" `; do  \
    KEY_USAGE=`redis-cli -n ${NDB}  MEMORY USAGE ${KEY}` ; \
    echo "KEY := ${KEY}  - Memory usage ${KEY_USAGE} "; \
done

```


## 2.7. Limpando chaves ZSET pelo EPOCH

```bash
clear ; \
NDB=6 ;\
ETODAY=`date +%Y-%m-%d`;  \
EPOCH_MAX=`date +%s -d "${ETODAY}"`;\
for KEY in `redis-cli -n ${NDB} --scan --pattern "*" `; do  \
    KEY_TYPE=`redis-cli -n ${NDB}  TYPE  ${KEY}`; \
    if [ "${KEY_TYPE}" == "zset" ]; then
        echo "KEY := ${KEY} - EPOCH_MAX := ${EPOCH_MAX} "; \
        redis-cli -n ${NDB}  ZCOUNT  ${KEY} -inf "(${EPOCH_MAX}" ; \
        redis-cli -n ${NDB}  ZREMRANGEBYSCORE  ${KEY} -inf "(${EPOCH_MAX}" ; \
    fi
done
```


</p>
</details>



<details><summary>3. Gerenciando sessoes no Redis</summary>
<p>


## 3.1. Visualizando sessões ativas

```sh
redis-cli  CLIENT ID
# (integer) 67

redis-cli  CLIENT INFO
# id=67 addr=127.0.0.1:45742 laddr=127.0.0.1:6379 fd=9 name= age=170 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=40928 argv-mem=10 obl=0 oll=0 omem=0 tot-mem=61466 events=r cmd=client user=default redir=-1

redis-cli  CLIENT LIST
# id=67 addr=127.0.0.1:45742 laddr=127.0.0.1:6379 fd=9 name= age=296 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=40928 argv-mem=10 obl=0 oll=0 omem=0 tot-mem=61466 events=r cmd=client user=default redir=-1
# id=71 addr=172.17.0.1:50840 laddr=172.17.0.2:6379 fd=8 name= age=148 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 argv-mem=0 obl=0 oll=0 omem=0 tot-mem=20520 events=r cmd=zremrangebyscore user=default redir=-1
```

## 3.2. Bloqueando atividade no Redis

```bash
# CLIENT PAUSE timeout [WRITE | ALL]
redis-cli  CLIENT PAUSE 1000  WRITE
```

## 3.3. Desconectando uma sessão

```bash
redis-cli  CLIENT KILL ID 71
# (integer) 1
```

</p>
</details>



<details><summary>4. Comandos básicos</summary>
<p>


Para acesso a documentação completa acesse este [link](https://bestofcpp.com/repo/sewenew-redis-plus-plus-cpp-database).


## 4.1. Pesquisas básicas


```sh
redis-cli  --scan --pattern 'exchange*'

redis-cli  type listSymbols
# hash

redis-cli DEL listSymbols
# (integer) 1

```


## 4.2. Trabalando com HASH

```bash

redis-cli type exchange-rate
# hash

redis-cli  HKEYS exchange-rate
# 1) "USD_BEST_ASK"
# 2) "USD_BEST_BID"
# 3) "USD"


redis-cli  HGET exchange-rate USD
# "5.701500"

redis-cli  HGETALL exchange-rate
#1) "USD"
#2) "5.701500"
#3) "USD_BEST_BID"
#4) "5.699500"
#5) "USD_BEST_ASK"
#6) "5.701500"

redis-cli  HSET listSymbols  "BTC_BRL"  "COINBASE_SPOT_BTC_BRL;COINBASE;SPOT;BTC;BRL;;BTC/BRL;BTC;BRL;0,01;1E-08;"

```


## 4.3. Trabalhando com chaves STRING

```bash

redis-cli  --scan --pattern BTC_USD
# BTC_USD

redis-cli  TYPE BTC_USD
# string

redis-cli  GET BTC_USD
# "COINBASE;SPOT;BTC;USD;1MIN;2022-02-02T20:07:00.0000000Z;2022-02-02T20:08:00.0000000Z;2022-02-02T20:07:00.3842930Z;2022-02-02T20:07:27.6100660Z;37602.58;37617;37588.04;37603.08;4.68219980;187;108057590;ohlcv;;;"

redis-cli  SET TESTE ola
# OK

redis-cli  GET TESTE
# "ola"

redis-cli  DEL TESTE
# (integer) 1

```


## 4.4. Trabalhando com chaves ZSET


```sh
redis-cli  --scan --pattern BTC_USD_T
# BTC_USD_T

redis-cli  TYPE BTC_USD_T
# zset

redis-cli  KEYS DOGE_*
# 1) "DOGE_USD"
# 2) "DOGE_USD_1"
# 3) "DOGE_USD_T"

redis-cli type DOGE_USD_T
# zset

redis-cli  ZRANGE  BTC_USD_T  0  2  REV  WITHSCORES
# 1) "COINBASE;SPOT;BTC;USD;2022-02-01T19:47:48.0520560Z;2022-02-01T19:47:48.1237110Z;21205f7a-401a-4bee-a0e9-186995cf8b25;38404.13;0.00251216;BUY;9442300;trade;;;"

redis-cli  ZREVRANGE  BTC_USD_T 0 2 withscores
# 1) "COINBASE;SPOT;BTC;USD;2022-02-02T19:57:01.2638380Z;2022-02-02T19:57:01.2746007Z;e10fa07e-cb60-44a6-8f4f-4884b57f5abb;37613.67;0.00416902;SELL;9819940;trade;;;"

redis-cli  ZRANGEBYSCORE BTC_USD_T  "(1643831821.1993799"  "(1643831821.1993999" 
# 1) "COINBASE;SPOT;BTC;USD;2022-02-02T19:57:01.2638380Z;2022-02-02T19:57:01.2745961Z;d3efabba-2e35-4a48-864a-e34e6bc3bf7b;37613.83;0.00664659;SELL;9819939;trade;;;"

```

</p>
</details>
