
# Demo 1 - Inicializar uma instância Cassandra utilizando Docker

## Baixa a imagem Docker do Cassandra
```bash
docker pull cassandra:latest
```

## Cria uma rede no docker com o nome cassandra
```bash
docker network create cassandra
```

## Inicia o servidor
```bash
docker run --rm -d --name cassandra --hostname cassandra --network cassandra cassandra
```

## Executa o cliente CQLSH para interagir com o cluster
```bash
docker run --rm -it --network cassandra  -v "$(pwd)/data.cql:/scripts/data.cql" nuvo/docker-cqlsh cqlsh cassandra 9042 --cqlversion='3.4.6'
```

## Algubs comandos CQL para teste
```bash
-- Cria keyspace
CREATE KEYSPACE IF NOT EXISTS store WITH REPLICATION = 
{ 'class' : 'SimpleStrategy', 'replication_factor' : '1' };

-- Cria uma tabela
CREATE TABLE IF NOT EXISTS store.shopping_cart (
  userid text PRIMARY KEY,
  item_count int,
  last_update_timestamp timestamp
);

-- Insere dados na tabela
INSERT INTO store.shopping_cart
(userid, item_count, last_update_timestamp)
VALUES ('9876', 2, toTimeStamp(now()));

INSERT INTO store.shopping_cart
(userid, item_count, last_update_timestamp)
VALUES ('1234', 5, toTimeStamp(now()));

INSERT INTO store.shopping_cart
(userid, item_count, last_update_timestamp)
VALUES ('1235', 5, toTimeStamp(now()));
```

# Demo 2 - Inicializar um cluster Cassandra utilizando CCM

Esses scritps foram baseados nos scripts disponíveis no próprio repositório do 
[CCM](https://github.com/riptano/ccm).

No MacOS é necessário criar interfaces extras de rede, no Linux e Windows não é preciso usar o comando.
```bash
sudo ifconfig lo0 alias 127.0.0.2 up
sudo ifconfig lo0 alias 127.0.0.3 up
sudo ifconfig lo0 alias 127.0.0.4 up
```


## Inicializar o cluster do Cassandra usando o ccm
```bash
ccm create usf -v 4.1.1 -n 3 -s
```
## Verifica o status do cluster
```bash
ccm status
```

## Aguardar alguns segundos para o cluster iniciar completamente
```bash
sleep 10
```

## Conectar-se ao cluster e criar um keyspace
```bash
ccm node1 cqlsh -e "CREATE KEYSPACE IF NOT EXISTS demo_keyspace WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};"
```

## Criar tabela no keyspace
```bash
ccm node1 cqlsh -k demo_keyspace -e "CREATE TABLE IF NOT EXISTS users (id UUID PRIMARY KEY, name TEXT, age INT);"
```

## Inserir dados na tabela
```bash
ccm node1 cqlsh -k demo_keyspace -e "INSERT INTO users (id, name, age) VALUES (uuid(), 'John Doe', 30);"
```

## Consultar dados da tabela
```bash
ccm node1 cqlsh -k demo_keyspace -e "SELECT * FROM users;"
```

## Verifica a situação dos nós do cluster
```bash
ccm node1 ring
ccm status 
```

## Adiciona um nó a o cluster
```bash
ccm add node4 -i 127.0.0.4 -j 7400 -b
```

## Parar o cluster
```bash
ccm stop
```

## Excluir o cluster
```bash
ccm remove
```
