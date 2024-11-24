# Arcadia35 Project

Ce projet est une application web composée d’un back-end en Symfony et d’un front-end, utilisant une base de données MariaDB et un cache Redis. 
Le projet est entièrement containerisé avec Docker pour faciliter le déploiement et l'exécution.

## Prérequis
Avant de commencer, assurez-vous d'avoir les logiciels suivants installés :
- [Docker](https://www.docker.com/products/docker-desktop)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Structure du projet
- **arcadia35back** : Contient le code back-end du projet en Symfony.
- **arcadia35front** : Contient le code front-end du projet HTML / Bootstrap / Javascript / SCSS.
- **docker-compose.yml** : Fichier de configuration pour Docker Compose qui orchestre les conteneurs.

## Installation

### 1. Cloner le projet
   
    Clonez ce dépôt sur votre machine locale :

  - git clone https://github.com/votre-utilisateur/arcadia35.git
  - cd arcadia35

### 2. Construire et démarrer les conteneurs Docker
    Dans le répertoire racine du projet (contenant le fichier docker-compose.yml), exécutez la commande suivante pour construire et lancer les conteneurs :

    docker-compose up --build

    Cette commande va :
    Construire et lancer les services back, front, db, et redis.
    Exposer le back-end Symfony sur http://localhost:8000.
    Exposer le front-end React sur http://localhost:3000.

### 3. Accéder aux services
    Une fois les conteneurs démarrés, vous pouvez accéder aux différentes parties de l'application :

    Back-end Symfony : http://localhost:8000
    Front-end  : http://localhost:3000
    Services Docker

    Le fichier docker-compose.yml configure plusieurs services Docker :

- **Service back**
    Le back-end est une application Symfony configurée pour se connecter à MariaDB et Redis.

    Dockerfile : arcadia35back/Dockerfileback
    Ports exposés : 8000:8000
    Variables d'environnement :
    APP_ENV=dev
    APP_DEBUG=true
    DATABASE_URL=mysql://<VotreNomutilisateur>:<VotreMotdePasse>@db:3306/<LeNomDeVotreBDD>
    OPENREDIS_URL=redis://redis:6379
    Dépendances : db, redis

    - **Service db**
        La base de données est gérée par un conteneur MariaDB.

        Image Docker : mariadb:latest
        Ports exposés : non exposés en externe (utilisé en interne uniquement)
        Variables d'environnement :
        MYSQL_ROOT_PASSWORD=<VotreMotDePasse>
        MYSQL_DATABASE=<LeNomDeVotreBDD>
        MYSQL_USER=<VotreNomutilisateur>
        MYSQL_PASSWORD=<VotreMotdePasse>
        Volume : db_data pour la persistance des données

- **Service Front**
        Le front-end est une application React.

        Dockerfile : arcadia35front/Dockerfile-front
        Ports exposés : 3000:3000
        Volume : monté sur ./arcadia35front:/usr/src/app

- **Service Redis**
        Le service de cache Redis est utilisé pour stocker temporairement certaines données de l'application.

        Image Docker : redis:latest
        Ports exposés : 6379:6379
        Réseau et volumes
        Réseau : app-network pour permettre aux services de communiquer entre eux.
        Volume : db_data pour persister les données de la base de données MariaDB.

-**Exemple d'un fichier docker-compose avec les services back, db, redis et front**
services:
  back:
    build:
      context: ./arcadia35back
      dockerfile: Dockerfileback
    ports:
      - "8000:8000"  
    volumes:
      - ./arcadia35back:/var/www/html
      - ./arcadia35back/vendor:/var/www/html/vendor
      - ./arcadia35back/var:/var/www/html/var
    environment:
      APP_ENV: dev
      APP_DEBUG: true
      DATABASE_URL: "mysql://<VotreNomutilisateur>:<VotreMotDePasse>@db:3306/<NomDeLaBDD> "
      OPENREDIS_URL: "redis://redis:6379"
    depends_on:
      - db
      - redis  
    networks:
      - app-network
    
  db:
    image: mariadb:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: <VotreMotDePasse>
      MYSQL_DATABASE:  <NomDeLaBDD> 
      MYSQL_USER: <VotreNomutilisateur> 
      MYSQL_PASSWORD: <VotreMotDePasse>
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app-network

  redis:
    image: redis:latest
    restart: always
    ports:
      - "6379:6379"  
    networks:
      - app-network

  front:
    build:
      context: ./arcadia35front  
      dockerfile: Dockerfile-front
    ports:
      - "3000:3000" 
    volumes:
      - ./arcadia35front:/usr/src/app  
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db_data:

- **Quelques commandes docker**
    Arrêter les conteneurs
    Pour arrêter tous les conteneurs, exécutez :

    docker-compose down
    Recréer les conteneurs

    Si vous apportez des modifications au Dockerfile ou à la configuration docker-compose.yml, vous pouvez recréer les conteneurs en utilisant :
    docker-compose up --build
    Accéder à un conteneur en ligne de commande

    Vous pouvez accéder à la ligne de commande de l’un des conteneurs en utilisant :
    docker-compose exec <NomDuService> 
    Remplacez <NomDuService> par back, db, front, ou redis.

    Gestion des dépendances
    Back-end Symfony
    Pour gérer les dépendances du projet Symfony, utilisez Composer. Cependant, comme tout est configuré dans le Dockerfile du back-end, les dépendances sont installées automatiquement lors de la construction du conteneur.

    Front-end 
    Pour gérer les dépendances du projet, utilisez NPM. Les dépendances sont également installées automatiquement lors de la construction du conteneur front.

    Configuration des Variables d’Environnement
    Les variables d'environnement requises pour la connexion à la base de données et au cache Redis sont définies directement dans docker-compose.yml et injectées automatiquement dans les conteneurs. Vous n'avez donc pas besoin d'un fichier .env séparé pour ce projet.

    Debugging et Logs
    Les logs des différents services sont accessibles via Docker. Pour visualiser les logs d'un service spécifique, exécutez :

    docker-compose logs <NomDuService>
    Remplacez <NomDuService> par back, db, front, ou redis.


Selon votre environnement, l'installation ou la mise à jour de composer dans le service back sera nécessaire en vous connectant sur ce dernier et en tapant la ligne de commande composer install

Réalisé par :
[Julien DUBOIS](https://github.com/Julien-35)
