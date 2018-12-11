# Tutoriel sur les microservices avec JHipster

## Prerequis

### Installation JHipster
Suivez les instructions de https://www.jhipster.tech/installation/

#### Java
```bash
scp -i ~/.ssh/xnet jdk-11.0.1_linux-x64_bin.deb tuto@host-1:~
```

#### Nvm et Node
Suivez les instructions de https://github.com/creationix/nvm
```bash
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
nvm ls-remote
nvm install stable
nodejs --version
```

#### Yeoman
```bash
npm install -g yo
yo --version
```

#### JHipster Generator
```bash
npm install -g generator-jhipster
jhipster --version
```

### Heroku
Suivez les instructions de https://devcenter.heroku.com/articles/heroku-cli
```bash
curl https://cli-assets.heroku.com/install-ubuntu.sh | sh
heroku --version
```

Testez votre compte créé sur Heroku via le lien https://signup.heroku.com/
```bash
heroku help
heroku login
heroku login -i

heroku teams
heroku spaces
heroku status
heroku regions
heroku authorizations
heroku releases
heroku notifications
heroku notifications --read
heroku orgs
heroku orgs --json
```

References:
 * https://devcenter.heroku.com/articles/heroku-cli-command

### Google Cloud SDK
Suivez les instructions de https://cloud.google.com/sdk/docs/downloads-interactive
```bash
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
gcloud help
```

Testez votre compte créé sur Google Cloud Platform
```bash
gcloud init
```

### Docker
```bash
TODO
```


### Kubernetes
```bash
TODO
```

## Démarrage avec un monolithe

```bash
mkdir -p ~/github/mastering-microservices/
git clone https://github.com/mastering-microservices/tutorial.git
```

### Création de l'application de base
```bash
mkdir -p ~/github/mastering-microservices/online-store
cd  ~/github/mastering-microservices/online-store
jhipster
```

```bash
./gradlew
```

```bash
open http://localhost:8080
```

### Génération des entités de l'application store

Visualisez le [schéma du service monolithique](./online-store.jh) avec le [JDL Studio](https://start.jhipster.tech/jdl-studio/). L'image est [ici](./online-store.jh.png).

Générez les sources (frontend et backend) relatives aux entités et à leurs relations.
```bash
cd  ~/github/mastering-microservices/online-store
jhipster import-jdl ../tutorial/online-store.jh
```

Lancez l'application en profil `dev`.
```bash
./gradlew
```

Ouvrez l'application dans un browser avec le rafraissement automatique en cas de modification des sources du frontend
```bash
yarn start
```

Ouvrez l'application dans un browser
```bash
open http://localhost:8080
```

Loggez vous en utilisateur `admin` `admin` et parcourez l'API Swagger (A)
```bash
open http://localhost:8080
```

## Lancement des tests générés

### Pour le backend

```bash
cd  ~/github/mastering-microservices/online-store
./gradlew test
```

### Pour le frontend
```bash
yarn test
```

Pour les tests end-to-end, lancez le backend depuis un terminal
```bash
cd  ~/github/mastering-microservices/online-store
./gradlew
```

Depuis un autre terminal, lancez le test e2e
```bash
cd  ~/github/mastering-microservices/online-store
yarn e2e
```

## Analyse de la qualité du code
Lancez un container SonarQube
```bash
cd  ~/github/mastering-microservices/online-store
docker-compose -f src/main/docker/sonar.yml up -d
docker-compose -f src/main/docker/sonar.yml logs -f
^C
```
Attendez qu'il soit démarré et prêt au service.

Lancez l'analyseur SonarQube
```bash
./gradlew sonarqube
```

Visualisez le rapport de l'analyseur SonarQube
```bash
open http://localhost:9000
open http://localhost:9000/dashboard?id=com.mycompany.store%3Astore
```

## Mise en place du CI/CD

Installez et lancez un serveur Jenkins
```bash
mkdir -p ~/github/mastering-microservices/jenkins
wget http://ftp-chi.osuosl.org/pub/jenkins/war-stable/2.150.1/jenkins.war
java -jar jenkins.war --httpPort=8989
```

Ouvrez la page de configuration du serveur Jenkins
```bash
open https://localhost:8989
```

Configurez le serveur Jenkins. Le clé admin se trouve dans la trace de la console de l'application Jenkins. Vous pouvez ajouter des addons comme [Blue Ocean](https://jenkins.io/projects/blueocean/), ...

Générez le pipeline Jenkinsfile pour le projet online-store
```bash
cd  ~/github/mastering-microservices/online-store
jhipster ci-cd
cat Jenkinsfile
```

Poussez le projet online-store vers un dépot Git (public ou privé) préalablement créé.
```bash
GITHUB_USERNAME=moncomptegithub
git remote add origin git@github.com:$GITHUB_USERNAME/online-store.git
git push -u origin master
```

Ajoutez le pipeline Jenkinsfile pour le projet online-store

### Lancement l'application store en mode (ie profil) `prod`

Construisez le WAR de l'application
```bash
cd  ~/github/mastering-microservices/online-store
./gradlew bootRepackage -Pprod
ls -al build/libs/*.war
```

Depuis un autre terminal, lancez la composition docker-compose qui comporte seulement le gestionnaire de base de données' (`store-mysql`).
```bash
cd  ~/github/mastering-microservices/online-store
docker-compose -f src/main/docker/mysql.yml up -d
```

Depuis un autre terminal, visualisez la console du service
```bash
cd  ~/github/mastering-microservices/online-store
docker-compose -f src/main/docker/mysql.yml logs -f
```

Lancez l'application
```bash
java -jar build/libs/store-0.0.1-SNAPSHOT.war
```

Finalement, détruisez le gestionnaire de base de données' (`store-mysql`).
```bash
cd  ~/github/mastering-microservices/online-store
docker-compose -f src/main/docker/mysql.yml down
```


### Lancement l'application store en mode (ie profil) `prod` sur Heroku
TODO

```bash
./gradlew bootRepackage -x test -Pprod
```


### Lancement l'application store en mode (ie profil) `prod` dans un container

Construisez l'image du conteneur
```bash
./gradlew bootRepackage -Pprod buildDocker
docker images | grep store
```

Lancez la composition docker-compose qui comporte l'application (`store-app`) et le gestionnaire de base de données (`store-mysql`).
```bash
docker-compose -f src/main/docker/app.yml up -d
```

Depuis un autre terminal, visualisez les consoles des 2 services
```bash
docker-compose -f src/main/docker/app.yml logs -f
```

Arrêtez le service `store-app` de la composition
```bash
docker-compose -f src/main/docker/app.yml stop store
```

Redémarrez le service `store-app` de la composition
```bash
docker-compose -f src/main/docker/app.yml start store
```

Arrêtez le service `store-mysql` de la composition
```bash
docker-compose -f src/main/docker/app.yml stop mysql
```
> Que se passe t'il ?

Redémarrez le service `store-mysql` de la composition
```bash
docker-compose -f src/main/docker/app.yml start mysql
```
> Que se passe t'il ?

Détruisez la composition.
> Remarque : les données rendues persistances dans les conteneurs sont définitivement perdues.

```bash
docker-compose -f src/main/docker/app.yml down
```

Sauvegardez l'image de l'application
```bash
TODO
```



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


## Bonus track

### Générer mvnw dans un projet Maven
```bash
chmod +x mvnw
~/devtools/apache-maven-X.X.X/bin/mvn -N io.takari:maven:wrapper
./mvnw
```
