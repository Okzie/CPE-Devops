# TP part 01 - Docker

## Database

### 1-1

```bash
docker build -t okzie/database .
```

```bash
docker run -d -p 5432:5432 --name database --network app-network -e POSTGRES_DB=db -e POSTGRES_USER=user -e POSTGRES_PASSWORD="pwd" -v /my/own/datadir:/var/lib/postgresql/data okzie/database 
```

```bash
docker run -d --network app-network -p 8080:8080 adminer
```

>Lien vers le dockerfile documenté : [Lien](/Database/Dockerfile)

## Backend API

### 1-2 : Why do we need a multistage build? And explain each step of this dockerfile.

> L'intéret du multistage est de récupérer une image finale moins lourde car elle ne contient rien qui est en rapport avec le build (dépendances, etc).


```dockerfile
# Build
# Utilise l'image Docker officielle pour Maven 3.8.6 avec Amazon Corretto 17 comme image de base pour la phase de build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build

# Définit une variable d'environnement pour le répertoire de l'application
ENV MYAPP_HOME /opt/myapp

# Définit le répertoire de travail dans le conteneur
WORKDIR $MYAPP_HOME

# Copie le fichier pom.xml dans le répertoire de travail du conteneur
COPY pom.xml .

# Copie le répertoire src dans le répertoire de travail du conteneur
COPY src ./src

# Exécute mvn package pour construire l'application, en sautant les tests
RUN mvn package -DskipTests

# Run
# Utilise l'image Docker officielle pour Amazon Corretto 17 comme image de base pour la phase de run
FROM amazoncorretto:17

# Réutilise la variable d'environnement pour le répertoire de l'application
ENV MYAPP_HOME /opt/myapp

# Définit le répertoire de travail dans le conteneur
WORKDIR $MYAPP_HOME

# Copie le fichier JAR construit dans la phase de build dans le répertoire de travail du conteneur
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

# Définit la commande à exécuter lorsque le conteneur est lancé
ENTRYPOINT java -jar myapp.jar
```

```bash
docker run -d --name backend -p 8082:8080 --network app-network -e DB_URL="jdbc:postgresql://database:5432/db" -e DB_USERNAME="user" -e DB_PASSWORD="pwd" okzie/simple-api
```

## Http server

```bash
docker run -d --name httpserv -p 80:80 --network app-network okzie/httpserv
```

## Docker-compose

### 1-3

```bash
docker-compose up -d
```

```bash
docker-compose ps 
```

### 1-3

>Lien vers le fichier docker-compose documenté : [Lien](/Docker-compose/docker-compose.yml)

## Publish

### 1-5

```bash
docker tag okzie/database okzie/database:1.0
```

```bash
docker tag okzie/backend okzie/backend:1.0
```

```bash
docker tag okzie/httpserv okzie/httpserv:1.0
```

```bash
docker push okzie/database:1.0
```

```sh
docker push okzie/backend:1.0
```

```sh
docker push okzie/httpserv:1.0
```

> Toute mes images sont maintenant disponible : https://hub.docker.com/u/okzie 

<br><br>

# TP part 02 - GitHub Action

## Setup GitHub Actions

### 2-1 : What are testcontainers

> Testcontainers est une bibliothèque Java qui fournit une API légère, réutilisable et flexible pour lancer des instances de Docker à partir de tests JUnit. Elle est principalement utilisée pour tester les interactions avec des bases de données, des API web ou d'autres ressources qui peuvent être encapsulées dans un conteneur Docker.

### 2-2

#### Avant slip des pipelines
>Lien vers le fichier de conf de la pipeline GitHub actions documenté : [Lien](/.github/workflows/main.yml.ignore)

#### Après slip des pipelines
>Lien vers le fichier de conf de la pipeline test-backend : [Lien](/.github/workflows/deploy-app.yml)

>Lien vers le fichier de conf de la pipeline build-and-push-docker-images : [Lien](/.github/workflows/build-and-push-docker-image.yml)

<br><br>

# TP part 03 - Ansible

## Introduction

### 3-1

>Lien vers le fichier inventories setup.yml : [Lien](/ansible/inventories/setup.yml)

### 3-2

>Lien vers le fichier playbook : [Lien](/ansible/playbook.yml)

## Deploy your App

>Lien vers la config de docker_container pour le backend : [Lien](/ansible/roles/app_java/tasks/main.yml)

>La configuration pour les autres service sont globalement les mêmes.

>Lien pour la database : [Lien](/ansible/roles/database/tasks/main.yml)

>Lien pour le proxy : [Lien](/ansible/roles/proxy/tasks/main.yml)

>Lien pour le front : [Lien](/ansible/roles/front/tasks/main.yml)