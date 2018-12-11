# Tutoriel sur les microservices avec JHipster :: Microservices

L'objectif de cette partie est la mise en place de microservices avec l'infrastructure microservices de JHipster.

## Plan
* [Sommaire](./README.md)
* [Installation](./install.md)
* [Création d'un monolithe](./monolith.md)
* Création d'une architecture microservices
* [Bonus track](./bonus.md)

## Création de la gateway et des microservices
L'infrastructure de base est constitué de :
* du registre de services de JHipster
* de l'API Gateway de JHipster
* d'un microservice invoice (utilisant une base de données mysql)
* d'un microservice notification (utilisant une base de données mongodb)


### Création de la gateway (vierge)
```bash
mkdir -p ~/github/mastering-microservices/gateway
cd  ~/github/mastering-microservices/gateway
jhipster
```

### Création d'un microservice (vierge) invoice
```bash
mkdir -p ~/github/mastering-microservices/invoice
cd  ~/github/mastering-microservices/invoice
jhipster
```

### Création d'un microservice (vierge) notification
```bash
mkdir -p ~/github/mastering-microservices/invoice
cd  ~/github/mastering-microservices/invoice
jhipster
```

### Lancement du registre
```bash
cd ~/github/mastering-microservices/gateway
docker-compose -f src/main/docker/jhipster-registry.yml up
```

### Lancement de la gateway
Lancez l'application en profil `dev`.
```bash
cd ~/github/mastering-microservices/gateway
./gradlew
```

### Visite guidée de l'application gateway
```bash
open http://localhost:8080
```
[Plus d'information ...](https://www.jhipster.tech/api-gateway/)

### Visite guidée du registre
```bash
open http://localhost:8761
```
[Plus d'information ...](https://www.jhipster.tech/jhipster-registry/)

### Lancement du  microservice invoice
Lancez l'application en profil `dev`.
```bash
cd  ~/github/mastering-microservices/invoice
./gradlew
```
> Remarque : La base de données utilisée est H2 pour le profil `dev`. En profil `prod`, il y aura un conflit de prot (`3306`) entre le container mysql utilisé par la gateway et celui utilisé par le microservice.

Consultez le descripteur Swagger
```bash
wget http://localhost:8081/v2/api-docs -O swagger.json
jq "." swagger.json
```

### Création d'un microservice (vierge) notification
Lancez le container mongodb.
```bash
cd  ~/github/mastering-microservices/invoice
docker-compose -f src/main/docker/mongodb.yml up -d
docker-compose -f src/main/docker/mongodb.yml logs -f
```

Lancez l'application en profil `dev` (attendre que mongodb soit prêt au service).
```bash
cd  ~/github/mastering-microservices/invoice
./gradlew
```

Consultez le descripteur Swagger
```bash
wget http://localhost:8082/v2/api-docs -O swagger.json
jq "." swagger.json
```

### Visite guidée de l'application gateway
```bash
open http://localhost:8080
```
> Que voyez vous ?
> Que se passe t'il si un des microservices est arrêté (simulation d'une panne, ...)?

### Visite guidée du registre
```bash
open http://localhost:8761
```
> Que voyez vous ?
> Que se passe t'il si un des microservices est arrêté (simulation d'une panne, ...)?

##





## Communication entre microservices par envoi asynchrone de messages

JHipster permet une communication entre microservices par l'envoi asynchrone de messages via des brokers de messages tels que RabbitMQ ou Apache Kafka. Cette fonction fait partie de [Spring Cloud Stream](https://cloud.spring.io/spring-cloud-static/Dalston.SR5/multi/multi__introducing_spring_cloud_stream.html).

Installez le générateur generator-jhipster-spring-cloud-stream
```bash
yarn global add generator-jhipster-spring-cloud-stream
```

Lancez le générateur generator-jhipster-spring-cloud-stream avec le microservice `notification`
```bash

cd notification
yo jhipster-spring-cloud-stream
git status
```

Plusieurs classes interessantes sont générées:

La classe `JhiNotificationEvent` est un exemple de message échangeable via RabbitMQ

La classe `NotificationEventSink` est un exemple de souscripteur de messages. Les messages recus sont stockés dans une liste (non persistante). Le topic de souscription est (par défaut) défini dans la propriété `spring.cloud.stream.bindings.input.destination` (`application-*.yml`)

La classe NotificationEventResource est un exemple de ressource
* avec une opération POST pour publier un message sur le topic (par défaut) défini dans la propriété `spring.cloud.stream.bindings.output.destination` (`application-*.yml`)
* avec une opération GET pour récupérer la liste non persistance des messages recus du topic

Lancez de RabbitMQ sur Term 1:
```bash
docker-compose -f src/main/docker/rabbitmq.yml up -d
docker-compose -f src/main/docker/rabbitmq.yml log -f
```

Lancez le micro-service `notification` sur Term 1:
```bash
./gradlew
```

Ouvrez la console de management de Rabbitmq (avec les crententials par défaut `guest` `guest`)

```bash
open http://localhost:15672/#/
```

Testez l’envoi de messages via l’interface Swagger UI (disponible depuis `Administration > API > notification`)

> NB: les messages peuvent être sérialisés dans un format comme JSON pour être échangés avec des micro-services non-Java. [Plus de détails](https://cloud.spring.io/spring-cloud-static/Dalston.SR5/multi/multi_contenttypemanagement.html)

Vérifiez la non persistance des messages en redémarrant le micro-service `notification`. L'opération GET retourne une liste vide.

References:
* https://github.com/hipster-labs/generator-jhipster-spring-cloud-stream
* https://www.jhipster.tech/using-kafka/
