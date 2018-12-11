# Tutoriel sur les microservices avec JHipster :: Bonus Track

## Plan
* [Sommaire](./README.md)
* [Installation](./install.md)
* [Création d'un monolithe](./monolith.md)
* [Création d'une architecture microservices](./microservice.md)
* Bonus track

## Générer mvnw dans un projet Maven
```bash
chmod +x mvnw
~/devtools/apache-maven-X.X.X/bin/mvn -N io.takari:maven:wrapper
./mvnw
```

## Changer de version du generateur de JHipster
```bash
# Supérieur ou égal (dans la majeure) à 4.13.2
yarn global upgrade generator-jhipster@^4.13.2
jhipster --version

# Exactement 4.13.2
yarn global upgrade generator-jhipster@4.13.2
jhipster --version

# Supérieur ou égal (dans la majeure) à 5.7.0
yarn global upgrade generator-jhipster@^5.7.0
jhipster --version

# Exactement 5.1.0
yarn global upgrade generator-jhipster@5.1.0
jhipster --version
```
