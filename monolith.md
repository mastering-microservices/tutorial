# Tutoriel sur les microservices avec JHipster :: Démarrage avec un monolithe

## Plan
* [Sommaire](./README.md)
* [Installation](./install.md)
* Création d'un monolithe
* [Création d'une architecture microservices](./microservice.md)
* [Bonus track](./bonus.md)

## Recupération des fichiers
```bash
mkdir -p ~/github/mastering-microservices/
git clone https://github.com/mastering-microservices/tutorial.git
```

## Création de l'application de base
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

## Génération des entités de l'application store

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

Poussez le projet `online-store` vers un dépot Git (public ou privé) préalablement créé.
```bash
GITHUB_USERNAME=moncomptegithub
git remote add origin git@github.com:$GITHUB_USERNAME/online-store.git
git push -u origin master
```

Ajoutez le pipeline `Jenkinsfile` pour le projet `online-store`.


## Utilisation de l'API REST avec cURL
Executez les commandes suivantes pour invoquer les opérations de l'API du service.

```bash

# Installation
sudo apt-get install curl jq
sudo npm install -g jwt-cli

# Content-Type
ACCEPT_JSON="Accept: application/json"
ACCEPT_CSV="Accept: text/csv"
CONTENT_JSON="Content-Type: application/json"
CONTENT_CSV="Content-Type: text/csv"

# LOCAL
PORT=8080
URL=http://localhost:$PORT

# PROD
#PORT=443
#URL=https://store.mycompany.com:$PORT
#URL=https://microservice-tutorial-store.herukoapp.com:$PORT

# Doc
URL_APIDOC=${URL}/v2/api-docs

# Operations
# CURL="curl --verbose"
CURL="curl -k --verbose"
GET="${CURL} -X GET --header \""$ACCEPT_JSON"\""
POST="${CURL} -X POST --header \""$ACCEPT_JSON"\""
PUT="${CURL} -X PUT --header \""$ACCEPT_JSON"\""
DELETE="${CURL} -X DELETE --header \""$ACCEPT_JSON"\""
OPTIONS="${CURL} -X OPTIONS --header \""$ACCEPT_JSON"\""
HEAD="${CURL} -X HEAD --header \""$ACCEPT_JSON"\""

# ===================================
# Get OpenAPI2.0 specification of the API
# -----------------------------------
${GET} ${URL_SWAGGER} > swagger.json

# ===================================
# Authenfication operations
# -----------------------------------
USERNAME=user
PASSWORD=user

AUTH_JSON="{ \"username\": \"${USERNAME}\", \"password\": \"${PASSWORD}\" }"

# Get the Bearer token for the user
rm $USERNAME.token.json
${POST}  --header "$CONTENT_JSON" -d "$AUTH_JSON" ${URL}/api/authenticate > $USERNAME.token.json
TOKEN=$(jq -r '.id_token' $USERNAME.token.json)
AUTH="Authorization: Bearer $TOKEN"

# Decode JWT Token for fun !
echo "Decode JWT Token $TOKEN"
jwt $TOKEN

# Get account info
${GET} --header "$AUTH" ${URL}/api/account

# Get all products
${GET} ${URL}/api/products
# --> 401 Unauthorized
${GET} --header "$AUTH" ${URL}/api/products
# --> 200
${GET} --header "$AUTH" ${URL}/api/products/1

# Add one new product
PRODUCT='{
  "name": "TEST",
  "description": "This is a test",
  "price": 5,
  "size": "XXL",
  "productCategory": {
    "id": 1,
    "name": "TEST",
    "description": "TESTTEST"
  }
}'
${POST} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products -d "$PRODUCT"

# Get the id of the created product
# TODO with jq ".id"

# Get all products
${GET} ${URL}/api/products

# Update one existing product
CHANGE='{
  "price": 1500
}'
${PUT} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products/3 -d "$CHANGE"
# --> 400

# Update one existing product
CHANGE='{
  "name": "NEWTEST"
  "price": 1500,
  "size": "S",
}'
${PUT} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products/3 -d "$CHANGE"
# --> according the PUT meaning, other properties should not be updated by the operation

# Get the updated product
${GET} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products/3

# Get all products
${GET} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products

# Remove the updated product
${DELETE} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products/3

```

## Géneration et utilisation de l'API REST
Le descripteur du service est disponible ici http://localhost:8080/v2/api-docs . Il est généré à partir des annotations des classes `Resource` du paquetage `com.mycompany.store.web.rest` et des classes Entity ou DTO (en fonction de la [directive de génération `dto`](https://www.jhipster.tech/jdl/)). Vous pouvez compléter les annotations avec les [annotations Swagger](https://github.com/swagger-api/swagger-core/wiki/Annotations-1.5.X) pour améliorer l'information de votre API.

Installez `swagger-codegen` ([pour plus d'information](https://swagger.io/docs/open-source-tools/swagger-codegen/)).
```bash
mkdir -p ~/github/mastering-microservices/swagger-codegen
cd ~/github/mastering-microservices/swagger-codegen
wget https://oss.sonatype.org/content/repositories/releases/io/swagger/swagger-codegen-cli/2.2.1/swagger-codegen-cli-2.2.1.jar
mv swagger-codegen-cli-2.2.1.jar swagger-codegen.jar
java -jar swagger-codegen.jar help
```
> Available languages: [android, aspnet5, async-scala, cwiki, csharp, cpprest, dart, flash, python-flask, go, groovy, java, jaxrs, jaxrs-cxf, jaxrs-resteasy, jaxrs-spec, inflector, javascript, javascript-closure-angular, jmeter, nancyfx, nodejs-server, objc, perl, php, python, qt5cpp, ruby, scala, scalatra, silex-PHP, sinatra, rails5, slim, spring, dynamic-html, html, html2, swagger, swagger-yaml, swift, tizen, typescript-angular2, typescript-angular, typescript-node, typescript-fetch, akka-scala, CsharpDotNet2, clojure, haskell, lumen, go-server]

```bash
codegen() {
  mkdir -p $1
  (cd $1;  java -jar ../swagger-codegen.jar generate -i ../swagger.json -l $1)  
}
```

Récupérez la définition Swagger. Vous pouvez installer l'outil `jq` (https://stedolan.github.io/jq/download/) pour formatter le document `swagger.json`.
```bash
cd ~/github/mastering-microservices/swagger-codegen
wget http://localhost:8080/v2/api-docs -O swagger.json
jq '.' swagger.json
```

> Remarque: il existe un plugin [Swagger Codegen](https://github.com/swagger-api/swagger-codegen/tree/master/modules/swagger-codegen-maven-plugin) pour Maven pour intégrer la génération des documents, des clients et des serveurs dans le build.

### Génération de la documentation HTML
```bash
codegen html2
(cd html2; open index.html)
```

### Génération de clients
```bash
# Client bash basé sur cURL
codegen bash; (cd bash; tree .)

codegen typescript-angular; (cd typescript-angular; tree .)

codegen python; (cd python; tree .)

codegen cpprest; (cd cpprest; tree .)

codegen php; (cd php; tree .)

```

#### Génération de squelettes de serveurs
```bash
codegen python-flask; (cd python-flask; tree .)

codegen nodejs-server; (cd nodejs-server; tree .)

codegen spring; (cd spring; tree .)

codegen go-server; (cd go-server; tree .)
```

### Génération d'un plan de charge pour l'injecteur [Apache JMeter](https://jmeter.apache.org/)
```bash
codegen jmeter; (cd jmeter; ls -al)
```
> Rien pour l'injecteur de charge [Gatling](https://gatling.io/) !

## Lancement l'application store en mode (ie profil) `prod`

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

## Lancement l'application store en mode (ie profil) `prod` sur Heroku
TODO

```bash
./gradlew bootRepackage -x test -Pprod
```
> `-x test` n'exécute pas les tests

## Lancement l'application store en mode (ie profil) `prod` dans un container

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

## Lancement l'injecteur de de charge avec [Gatling](https://gatling.io/)

Suivez la section "Performance tests" de https://www.jhipster.tech/running-tests/

```bash
cd  ~/github/mastering-microservices/
wget https://repo1.maven.org/maven2/io/gatling/highcharts/gatling-charts-highcharts-bundle/2.3.1/gatling-charts-highcharts-bundle-2.3.1-bundle.zip
unzip gatling-charts-highcharts-bundle-2.3.1-bundle.zip
```bash

Lancez le recorder depuis un terminal
```bash
GATLING_HOME=~/github/mastering-microservices/gatling-charts-highcharts-bundle-2.3.1
cd  ~/github/mastering-microservices/online-store
cd src/test/gatling
tree .
$GATLING_HOME/bin/recorder.sh

$GATLING_HOME/bin/gatling.sh
```

Lancez l'application à tester (de préférence, la version de production en modifiant auparavant les loggers positionnés en dessous de WARN).
```bash
cd  ~/github/mastering-microservices/online-store
docker-compose -f src/main/docker/app.yml up -d
docker-compose -f src/main/docker/app.yml logs -f
```

Lancez l'injecteur depuis un autre terminal. Répondez aux questions.
```bash
GATLING_HOME=~/github/mastering-microservices/gatling-charts-highcharts-bundle-2.3.1
cd  ~/github/mastering-microservices/online-store
cd src/test/gatling
$GATLING_HOME/bin/gatling.sh
```

Ouvrez le rapport HTML qui est généré dans le répertoire `$GATLING_HOME/results`.
