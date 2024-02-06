# TP part 01 - Docker

## Database

```bash
docker build -t okzie/database .
```

```bash
docker run -d -p 5432:5432 --name database --network app-network -e POSTGRES_DB=db -e POSTGRES_USER=user -e POSTGRES_PASSWORD="pwd" -v /my/own/datadir:/var/lib/postgresql/data okzie/database 
```

```bash
docker run -d --network app-network -p 8080:8080 adminer
```

## Backend API

1-2 Why do we need a multistage build? And explain each step of this dockerfile.

```
L'intéret du multistage est de récupérer une image finale moins lourde car elle ne contient rien qui est en rapport avec le build (dépendances, etc).
```

```Dockerfile
# Utilisation de l'image maven:3.8.6-amazoncorretto-17 comme image de base pour la phase de construction
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
# Définition d'une variable d'environnement pour le répertoire de l'application
ENV MYAPP_HOME /opt/myapp 
# Définition du répertoire de travail dans le conteneur
WORKDIR $MYAPP_HOME
# Copier le fichier pom.xml dans le répertoire de travail du conteneur
COPY pom.xml .
# Copier le répertoire src dans le répertoire de travail du conteneur
COPY src ./src
# Exécuter la commande mvn package pour construire l'application, en sautant les tests
RUN mvn package -DskipTests

# Début d'une nouvelle phase avec l'image amazoncorretto:17 comme image de base pour l'exécution de l'application
FROM amazoncorretto:17
# Réutilisation de la variable d'environnement pour le répertoire de l'application
ENV MYAPP_HOME /opt/myapp
# Définition du répertoire de travail dans le conteneur
WORKDIR $MYAPP_HOME
# Copier le fichier jar construit dans la phase de construction dans le répertoire de travail du conteneur
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
# Commande pour exécuter l'application
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

```bash
docker run -d --name backend -p 8082:8080 --network app-network -e DB_URL="jdbc:postgresql://database:5432/db" -e DB_USERNAME="user" -e DB_PASSWORD="pwd" okzie/simple-api
```

## Http server

```bash
docker run -d --name httpserv -p 80:80 --network app-network okzie/httpserv
```

## Docker-compose

```bash
docker-compose up -d
```

```bash
docker-compose ps 
```

## Publish

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
docker push okzie/simpleapi-student:1.0
```

```sh
docker push okzie/httpserv:1.0
```

Toute mes images sont maintenant disponible : https://hub.docker.com/u/okzie