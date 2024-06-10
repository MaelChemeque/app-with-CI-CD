> Question 1-1 Document your database container essentials: commands and Dockerfile.

<!-- Make a network so adminer can connect to postgres -->

docker network create app-network

<!-- Build and run postgres image
    Which includes:
    - env variables
    - network declaration
    - port
    - volume to persist data
 -->

docker build -t my-postgres-image .
docker run --name postgres --net=app-network -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -p 5432:5432 -v /my/own/datadir:/var/lib/postgresql/data -d my-postgres-image

#### Adminer

docker run -p "8090:8080" --net=app-network --name=adminer -d adminer

## Backend installation

docker build -t java-image .
docker run --name java --net=app-network -p 8080:8080 java-image:latest

> Question 1-2 Why do we need a multistage build? And explain each step of this dockerfile.

Voici les étapes :

- Récupération d'une image maven
- On se place dans le répertoire de travail
- On copie le fichier pom.xml utilisé par maven pour gérer les dépendances
- On télécharge les dépendances (en les chargeant dans le cache si le fichier pom.xml n'a pas été modifié)
- On récupère le code source
- On le compile en générant un .jar
- Récupération d'un image JRE permettant d'exécuter l'application
- On se place dans le répertoire de travail
- On récupère le fichier compilé
- On définit le programme à exécuter au démarrage du container

Il y a donc deux étapes, la première étant le build du code source, la deuxième son exécution.
Une fois à l'exécution il n'y a plus besoin de maven, donc on peut simplement copier le .jar.
L'image final sera donc plus petite car elle ne contiendra que le JRE et le binaire.

## HTTP server installation

docker build -t my-http-server .
docker run --name my-http-container -p 3000:80 -d my-http-server

> Question 1-3 Document docker-compose most important commands

docker compose -f docker-compose.yml build
docker compose -f docker-compose.yml down
docker compose -f docker-compose.yml up -d

> Question 1-4 Document your docker-compose file

Les lignes importantes de ce fichier docker-compose.yml sont :

- la définition du network qui va permet de faire communiquer les différents containers entre eux sans exposer la base de donnée et le back-end.
- depends_on : permet de lancer les containers dans le bon ordre, par contre cela ne garantit pas que les containers sont bien initialisés (il faut utiliser healthcheck)
- context : correspond à l'environnement de travail
- ports : on expose que le port du server httpd !
- variables d'environnements : probablement pas une bonne idée de stocker le password ici
- volume pour la persistence des données

> Question 1-5 Document your publication commands and published images in dockerhub.

docker tag tp1-database:latest maelchemeque/my-database:1.0
docker tag tp1-backend:latest maelchemeque/my-backend:1.0
docker tag tp1-httpd:latest maelchemeque/my-http-server:1.0

docker push maelchemeque/my-database:1.0
docker push maelchemeque/backend:1.0
docker push maelchemeque/my-http-server:1.0

> 2-1 What are testcontainers ?

Ce sont des bibliothèques Java permettant de faire tourner les tests dans un environnement isolé reflétant celui de production.

> 2-2 Document your Github Actions configurations.

On définit que les actions doivent être exécutées uniquement lors du push sur develop ou main.
On définit qu'on va faire tourner les actions sur ubuntu 22.
On récupère le code source du dépôt pour pouvoir le tester ensuite dans l'environnement de CI.
On configure les environnements dans lesquels on veut tester notre backend.
On exécute la commande maven mvn clean verify dans le répertoire de travail ./backend.

> Document your quality gate configuration.

La commande maven utilisé avec sonar prend un argument --file, donc il a fallu retirer la ligne de contexte.
J'ai utilisé la commande : mvn -B verify sonar:sonar -Dsonar.projectKey=MaelChemeque_app-with-CI-CD -Dsonar.organization=maelchemeque -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file ./backend/pom.xml

Remarque : j'aurais sûrement dû utiliser des noms plus parlants pour "organization" et "projectKey".

> Bonus

cf : ci-test-backend.yml et ci-build-and-push-docker-image.yml
