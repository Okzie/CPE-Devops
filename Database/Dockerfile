# Utilise l'image Docker officielle pour PostgreSQL version 14.1 basée sur Alpine Linux comme image de base
FROM postgres:14.1-alpine

# Copie les scripts SQL de votre répertoire local ./sql_scripts vers le répertoire /docker-entrypoint-initdb.d dans le conteneur
# Ce répertoire est spécifique à l'image Docker de PostgreSQL. 
# Tous les scripts .sql et .sh placés dans ce répertoire seront exécutés automatiquement à l'initialisation du conteneur.
COPY ./sql_scripts /docker-entrypoint-initdb.d