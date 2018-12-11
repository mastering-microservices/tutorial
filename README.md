# Tutoriel sur les microservices avec JHipster

## Prerequis


## Démarrage avec un monolithe

## Analyse de la qualité du code

## Mise en place du CI/CD

## Mise en place de l'infrastructure microservices de JHipster

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
