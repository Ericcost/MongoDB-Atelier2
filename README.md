# MongoDB-Atelier2

# Partie 1 :

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

Pour se rendre dans la console redis d'une des instances : 
```
sudo docker exec -it atelier2-redis-node1-1 redis-cli -h 172.20.0.4 -p 7000 
```

Dans la mesure où l'addresse IP du container et/ou le port ne sont pas les bons veuillez utiliser cette commande pour connaître les valeurs 

# Partie 2 : 

 
