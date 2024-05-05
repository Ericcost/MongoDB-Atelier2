# MongoDB-Atelier2

# Partie 1 : Installation et Configuration du Cluster Redis

## J'ai choisis de faire un docker-compose.yml

```
version: '3'

services:
  redis-node1:
    image: redis
    command: ["redis-server", "--port", "7000", "--cluster-enabled", "yes"]
    ports:
      - "7000:7000"
    networks:
      - redis-cluster

  redis-node2:
    image: redis
    command: ["redis-server", "--port", "7001", "--cluster-enabled", "yes"]
    ports:
      - "7001:7001"
    networks:
      - redis-cluster

  redis-node3:
    image: redis
    command: ["redis-server", "--port", "7002", "--cluster-enabled", "yes"]
    ports:
      - "7002:7002"
    networks:
      - redis-cluster

  redis-node4:
    image: redis
    command: ["redis-server", "--port", "7003", "--cluster-enabled", "yes"]
    ports:
      - "7003:7003"
    networks:
      - redis-cluster

  redis-node5:
    image: redis
    command: ["redis-server", "--port", "7004", "--cluster-enabled", "yes"]
    ports:
      - "7004:7004"
    networks:
      - redis-cluster

  redis-node6:
    image: redis
    command: ["redis-server", "--port", "7005", "--cluster-enabled", "yes"]
    ports:
      - "7005:7005"
    networks:
      - redis-cluster

networks:
  redis-cluster:
    driver: bridge

```


Ensuite il faut run la commande suivante pour créer une ccluster redis : 
```
docker compose up -d
```

Dans la mesure où vous ne connaissez pas l'addresse IP du container et/ou le port, veuillez utiliser cette commande pour connaître les valeurs :

```
sudo docker inspect <container name>
```

Pour pouvoir ensuite se rendre dans la console redis d'une des instances : 
```
sudo docker exec -it atelier2-redis-node1-1 redis-cli -h 172.20.0.4 -p 7000 // Répétez le process pour chaque instance
```

De là vérifiez qu'il y a bien un cluster : 
```
cluster nodes
fec53df76171a7c13361142b1be4e40420845f86 :7003@17003 myself,master - 0 0 0 connected
```

# Partie 2 : Premiers Pas avec le Cluster Redis

## Injection de données 

De là lorsqu'on essaie d'injecter des données depuis la console redis-cli on obtient une erreur :
```
SET mykey "Hello"
(error) CLUSTERDOWN Hash slot not served
```

C'est important alors de vérifier les informations liées à notre cluster depuis la console :
```
CLUSTER INFO
cluster_state:fail
cluster_slots_assigned:0
cluster_slots_ok:0
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:1
cluster_size:0
cluster_current_epoch:0
cluster_my_epoch:0
cluster_stats_messages_sent:0
cluster_stats_messages_received:0
total_cluster_links_buffer_limit_exceeded:0
```

Il faut alors revenir au terminal initial et lancer la commande suivante : 
```
sudo docker exec -it atelier2-redis-node2-1 redis-cli --cluster fix localhost:7001 // En faisant correspondre le bon container avec son addresse IP
```

Si on run la commande suivante à nouveau :
```
CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:1
cluster_size:1
cluster_current_epoch:1
cluster_my_epoch:1
cluster_stats_messages_sent:0
cluster_stats_messages_received:0
total_cluster_links_buffer_limit_exceeded:0
```

Puis répétez le process pour chacune des instances.

Pour l'injection des données cela ce ce fait comme suit :
```
// Strings
SET mykey "Hello"

// Lists
LPUSH mylist "World"
LPUSH mylist "Redis"

//Sets
SADD myset "Apple"
SADD myset "Banana"
SADD myset "Orange"

//Hashes
HSET myhash field1 "value1"
HSET myhash field2 "value2"

//Sorted Sets
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
```

## Requêtage des données

```
// Lecture des valeurs
GET mykey

// Lecture des éléments d'une liste:
LRANGE mylist 0 -1

// Lecture des éléments d'un ensemble:
SMEMBERS myset

// Lecture des champs et valeurs d'un hash:
HGETALL myhash

// Lecture des éléments d'un ensemble trié par score:
ZRANGE myzset 0 -1 WITHSCORES
```

Exemple pour ajouter un user dans la BDD Redis: 
```
//Set String
SET username:user123 "Alice"

//Push Data to List
LPUSH online_users user123
LPUSH online_users user456

//Add Data to set
SADD user_roles:user123 admin
SADD user_roles:user456 user

//Set Hash Field
HSET user:user123 email alice@example.com
HSET user:user123 age 30

//Add Data to Sorted Set
ZADD high_scores 100 user123
ZADD high_scores 90 user456
```



