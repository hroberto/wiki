# REDIS STREAM

## 1. References

[Tutorial Redis Stream](https://redis.io/docs/data-types/streams-tutorial/#creating-a-consumer-group)


## 2. Exemplo de uso


### 2.1. Limpeza de historicos anteriores

```sh
## Apaga todas as chaves
redis-cli -n 8  --scan | xargs redis-cli -n 8  DEL
#(integer) 1

redis-cli -n 8  XADD STM_TRADE_APP1 NOMKSTREAM \* BTC_USD_T  item_trade_1
#(nil)

redis-cli -n 8  EXISTS STM_TRADE_APP1
#(integer) 0
```

----------------------------

### 2.2. Criando novo Stream de leitura

```sh
redis-cli -n 8  XGROUP CREATE  STM_TRADE_APP1  GRP_TRADE_APP1 $ MKSTREAM
#OK

redis-cli -n 8  XADD STM_TRADE_APP1 NOMKSTREAM \* BTC_USD_T  item_trade_1
#"1660236194391-0"

redis-cli -n 8  XINFO GROUPS STM_TRADE_APP1
# 1) 1) "name"
   # 2) "GRP_TRADE_APP1"
   # 3) "consumers"
   # 4) (integer) 0
   # 5) "pending"
   # 6) (integer) 0
   # 7) "last-delivered-id"
   # 8) "0-0"
   
redis-cli -n 8  XREADGROUP GROUP GRP_TRADE_APP1  appconsumer COUNT 1 STREAMS STM_TRADE_APP1 \>
# 1) 1) "STM_TRADE_APP1"
   # 2) 1) 1) "1660236194391-0"
         # 2) 1) "BTC_USD_T"
            # 2) "item_trade_1"

redis-cli -n 8  XADD STM_TRADE_APP1 NOMKSTREAM \* BTC_USD_T  item_trade_1
# "1660236370225-0"

redis-cli -n 8  XADD STM_TRADE_APP1 NOMKSTREAM \* BTC_USD_T  item_trade_1
# "1660236532150-0"

redis-cli -n 8  XREAD STREAMS STM_TRADE_APP1 0
# 1) 1) "STM_TRADE_APP1"
   # 2) 1) 1) "1660236194391-0"
         # 2) 1) "BTC_USD_T"
            # 2) "item_trade_1"
      # 2) 1) "1660236370225-0"
         # 2) 1) "BTC_USD_T"
            # 2) "item_trade_1"
      # 3) 1) "1660236532150-0"
         # 2) 1) "BTC_USD_T"
            # 2) "item_trade_1"


```

----------------------------

### 2.3. Apagando mensagens lidas

```sh
## Verifica o ultimo id entregue pelo XREADGROUP
redis-cli -n 8  XINFO GROUPS STM_TRADE_APP1
# 1) 1) "name"
   # 2) "GRP_TRADE_APP1"
   # 3) "consumers"
   # 4) (integer) 1
   # 5) "pending"
   # 6) (integer) 1
   # 7) "last-delivered-id"
   # 8) "1660236194391-0"

## obtem todos os ID nao utilizados "last-delivered-id"
redis-cli -n 8  XRANGE STM_TRADE_APP1 - "1660236194391-0"
# 1) 1) "1660236194391-0"
   # 2) 1) "BTC_USD_T"
      # 2) "item_trade_1"

redis-cli -n 8  XLEN STM_TRADE_APP1
# (integer) 3

redis-cli -n 8  XDEL STM_TRADE_APP1   "1660236194391-0"
(integer) 1

redis-cli -n 8  XLEN STM_TRADE_APP1
(integer) 2

```

----------------------------

### 2.4. Consulta de informações do Stream

```sh
## obtem todos os ID nao utilizados "last-delivered-id"
redis-cli -n 8  XRANGE STM_TRADE_APP1 - "1660236194391-0"
# 1) 1) "1660236194391-0"
   # 2) 1) "BTC_USD_T"
      # 2) "item_trade_1"

redis-cli -n 8  XREAD STREAMS STM_TRADE_APP1 0
# 1) 1) "STM_TRADE_APP1"
   # 2) 1) 1) "1660236194391-0"
         # 2) 1) "BTC_USD_T"
            # 2) "item_trade_1"
      # 2) 1) "1660236370225-0"
         # 2) 1) "BTC_USD_T"
            # 2) "item_trade_1"
      # 3) 1) "1660236532150-0"
         # 2) 1) "BTC_USD_T"
            # 2) "item_trade_1"

redis-cli -n 8  XREADGROUP GROUP GRP_TRADE_APP1  appconsumer COUNT 1 STREAMS STM_TRADE_APP1 \>
# 1) 1) "STM_TRADE_APP1"
   # 2) 1) 1) "1660236194391-0"
         # 2) 1) "BTC_USD_T"
            # 2) "item_trade_1"

```


----------------------------

### 2.5. Verificando Stream / Groups / Consumers

```sh
redis-cli -n 8  XINFO STREAM STM_TRADE_APP1
 # 1) "length"
 # 2) (integer) 2
 # 3) "radix-tree-keys"
 # 4) (integer) 1
 # 5) "radix-tree-nodes"
 # 6) (integer) 2
 # 7) "last-generated-id"
 # 8) "1660238123345-0"
 # 9) "groups"
# 10) (integer) 1
# 11) "first-entry"
# 12) 1) "1660238122791-0"
    # 2) 1) "BTC_USD_T"
       # 2) "item_trade_1"
# 13) "last-entry"
# 14) 1) "1660238123345-0"
    # 2) 1) "BTC_USD_T"
       # 2) "item_trade_1"

redis-cli -n 8  XINFO GROUPS STM_TRADE_APP1
# 1) 1) "name"
   # 2) "GRP_TRADE_APP1"
   # 3) "consumers"
   # 4) (integer) 1
   # 5) "pending"
   # 6) (integer) 1
   # 7) "last-delivered-id"
   # 8) "1660238122002-0"

redis-cli -n 8  XINFO CONSUMERS STM_TRADE_APP1 GRP_TRADE_APP1
# 1) 1) "name"
   # 2) "appconsumer"
   # 3) "pending"
   # 4) (integer) 1
   # 5) "idle"
   # 6) (integer) 1047148
   
```

----------------------------

### 2.6. Configurando tempo de Expurgo da chave

```sh
redis-cli -n 8  TTL STM_TRADE_APP1
# (integer) -1

redis-cli -n 8  EXPIRE STM_TRADE_APP1 150
# "OK"

redis-cli -n 8  TTL STM_TRADE_APP1
# (integer) 148

redis-cli -n 8  TTL STM_TRADE_APP1
# (integer) 145

redis-cli -n 8  PERSIST STM_TRADE_APP1
# (integer) 1

redis-cli -n 8  TTL STM_TRADE_APP1
# (integer) -1

```
