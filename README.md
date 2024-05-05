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

Exemples de requêtes avec des critères spécifiques :
```
// Obtenir la longueur du nom d'utilisateur pour user123 :
STRLEN username:user123

// Obtenir le nombre d'utilisateurs en ligne :
LLEN online_users

// Vérifier si user123 a le rôle "admin" :
SISMEMBER user_roles:user123 admin

// Obtenir le nombre de rôles attribués à user123 :
SCARD user_roles:user123

// Vérifier si user123 a un champ d'e-mail :
HEXISTS user:user123 email

// Obtenir le nombre de champs pour user123 :
HLEN user:user123

// Obtenir le score de user123 dans le jeu de données trié high_scores :
ZSCORE high_scores user123
```

# Partie 3 : Intégration de Redis dans un Projet

Préalable : Installer la bibliothèque cliente Redis : Selon votre langage de programmation, vous devrez installer une bibliothèque cliente Redis. Parmi les plus populaires, on trouve redis-py pour Python, redis pour Node.js, et StackExchange.Redis pour C#/.NET.

Exemple pour connecter une BDD Redis à un projet Python en utilisant la bibliothèque `redis-py` :
```
import redis

# Establish connection to Redis
redis_client = redis.StrictRedis(host='localhost', port=6379, db=0)

# Set a value
redis_client.set('mykey', 'myvalue')

# Get a value
value = redis_client.get('mykey')
print(value)
```
Remplacez 'localhost' et 6379 par l'hôte et le port appropriés de votre base de données Redis.


# Partie 4 : Introspection sur l'Intégration de Redis

Intégrer Redis dans des projets existants peut apporter plusieurs avantages notamment en termes de performances, de scalabilité et de fonctionnalités. 

### Pour les Applications Web en Temps Réel :
  - On pourrait utiliser Redis comme cache pour stocker des données fréquemment consultées en mémoire, réduisant ainsi les temps de latence et améliorant les performances globales de l'application.

  - Pub/Sub : Exploiter les fonctionnalités de pub/sub de Redis pour mettre en place des communications en temps réel entre les clients et le serveur, idéal pour les applications de chat, les notifications en direct, etc.

  - Sessions Utilisateur : Stocker les sessions utilisateur en mémoire avec Redis pour une gestion efficace des sessions et une répartition de charge plus équilibrée sur les serveurs.
    
### Applications E-Commerce :

  - Cache de Catalogue : Mettre en cache les données de catalogue, les informations produit et les résultats de recherche dans Redis pour réduire la charge sur la base de données principale et améliorer les temps de réponse des requêtes.

  - Paniers Utilisateurs : Stocker temporairement les paniers utilisateurs en mémoire avec Redis pour une expérience d'achat fluide et rapide, même en cas de trafic élevé.
    
  - Gestion des Stocks : Utiliser des structures de données Redis telles que les ensembles ordonnés pour suivre et gérer efficacement les niveaux de stock en temps réel.
    
### Applications Mobiles :

  - Cache Mobile : Intégrer Redis en tant que cache côté serveur pour les applications mobiles afin de réduire les délais de chargement et d'améliorer la réactivité de l'application.
    
  - Notifications Push : Utiliser les fonctionnalités de pub/sub de Redis pour gérer les notifications push et les messages en temps réel envoyés aux utilisateurs de l'application.
    
  - Gestion de Sessions : Stocker les sessions utilisateur et les informations de connexion en mémoire avec Redis pour une authentification sécurisée et une expérience utilisateur transparente.
    
### Applications de Jeux en Ligne :
  - Cache de Données de Jeu : Utiliser Redis comme cache pour stocker les données de jeu en mémoire, telles que les scores, les classements et les états de jeu, pour des performances optimales et une expérience de jeu fluide.
    
  - Messagerie en Temps Réel : Mettre en place un système de messagerie en temps réel avec Redis pour permettre aux joueurs de communiquer, de rejoindre des salons de discussion et de partager des mises à jour en direct.
    
  - Gestion de Sessions de Jeu : Stocker les sessions de jeu en mémoire avec Redis pour une transition fluide entre les différentes plates-formes et appareils, garantissant une expérience de jeu cohérente pour les utilisateurs.
    


