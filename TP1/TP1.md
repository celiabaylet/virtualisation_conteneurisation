# TP1 Part 1 : Just run things ffs

## 1. Une ptite db MySQL

### 🌞 Lancez un conteneur mysql en version 8.4
```docker run --name mysql mysql:8.4```
### 🌞 Vérifier que le conteneur est actif
```docker ps -a```
```
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS                    PORTS                    NAMES
1e127a522386   mysql:8.4             "docker-entrypoint.s…"   4 seconds ago   Up 4 seconds              3306/tcp, 33060/tcp      mysql
```
### 🌞 Consulter les logs du conteneur
    You need to specify one of the following as an environment variable:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_ALLOW_EMPTY_PASSWORD
    - MYSQL_RANDOM_ROOT_PASSWORD
### 🌞 Supprimer le conteneur inactif
```docker rm 938bb3f7c84c```
### 🌞 Lancer un nouveau conteneur MySQL
```docker run --name mysql -e MYSQL_ROOT_PASSWORD=mysql -d mysql:8.4```

## 2. Un PMA pour accompagner la DB

### 🌞 Lancer un conteneur PHPMyAdmin
```docker run --name phpmyadmin -d --link mysql:db -p 8080:80 phpmyadmin```
### 🌞 Visitez l'interface web de PHPMyAdmin
```curl http://localhost:8080```
<br>
```
<!doctype html>
<html lang="en" dir="ltr">
<head>
```

## 3. Compose
### 🌞 Créer un fichier compose.yml qui a le contenu suivant :
```
Création du fichier compose.yml
```
### 🌞 Remplacer par les bonnes valeurs
```
services:
  db:
    image: mysql:8.4
    environment:
      - MYSQL_ROOT_PASSWORD=<MON_PASSWORD>

  pma:
    image: phpmyadmin
    ports:
      - "8080:80"
```
### 🌞 Lancer la stack compose
```cd dans le dossier qui contient le fichier compose.yml```
```docker compose up```
```
docker compose ps 
NAME       IMAGE       COMMAND                  SERVICE   CREATED          STATUS          PORTS
tp1-db-1   mysql:8.4   "docker-entrypoint.s…"   db        53 seconds ago   Up 52 seconds   3306/tcp, 33060/tcp
```
### 🌞 Visitez l'interface Web de PHPMyAdmin
```curl http://localhost:8080```
<br>
```
<!doctype html>
<html lang="en" dir="ltr">
<head>
```

## 4. Donnée persistantes
### 🌞 Ajouter la gestion d'un volume à votre compose.yml
```
services:
  db:
    image: mysql:8.4
    environment:
      - MYSQL_ROOT_PASSWORD=<MON_PASSWORD>
    volumes:
      - db_data:/var/lib/mysql

  pma:
    image: phpmyadmin
    ports:
      - "8080:80"

volumes:
  db_data:
```
```
on déclare un volume. Docker s'occupe de créer un dossier appelé db_data.
```
### 🌞 Prouver que le volume est bien utilisé
```
docker compose up dans le dossier qui contient le fichier compose.yml
```
```
docker volume ls
```
```
DRIVER    VOLUME NAME
local     tp1_db_data
```

## 5. Changing database easily
### 🌞 Proposer un nouveau compose.yml
```
plutôt que lancer un MySQL, il lance un serveur PostgreSQL 
plutôt qu'un PHPMyAdmin, il lance un adminer 
```
```
services:
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=<MON_PASSWORD>
    volumes:
      - db_data:/var/lib/postgresql/data

  pma:
    image: adminer
    ports:
      - "8080:8080"

volumes:
  db_data:
```

### 🌞 Prouver que ça run !
```docker ps -a```
```
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS                      PORTS                                     NAMES
f670d76cd8f3   postgres:16           "docker-entrypoint.s…"   53 seconds ago   Exited (1) 47 seconds ago                                             tp1-db-1
70629afda65d   adminer               "entrypoint.sh docke…"   53 seconds ago   Created 
```


# TP1 Part 2 : Dev environment
## 1. Introoo
## 2. Dockerfile
### 🌞 Récupérez le package.json et app.js 
```
les fichiers sont ici :
tp1_part2/1/package.json
tp1_part2/1/src/app.js
```
### 🌞 Ajout d'un fichier tp1_part2/Dockerfile 

### 🌞 Construire l'image shitty_app à partir de ce Dockerfile :
```
toujours depuis un shell, déplacez vous dans le dossier qui contient ce Dockerfile
exécutez la commande docker build suivante :
```
```
# Déplacez vous dans le dossier qui contient vos fichiers
❯ cd tp1_part2/1

.
├── Dockerfile
├── package.json
└── src
    └── app.js

# le "." c'est pour indiquer le chemin vers le "contexte de build" : là où on trouve le Dockerfile notamment
❯ docker build -t shitty_app:1.0 .
```
### 🌞 Lancer l'application avec cette commande :
```
docker run -p 3000:3000 -d shitty_app:1.0
```
### 🌞 Prouvez que ça tourne :
```
docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                                         NAMES
d59df0ab27f5   shitty_app:1.0   "docker-entrypoint.s…"   16 seconds ago   Up 16 seconds   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   trusting_roentgen
```
```
docker logs trusting_roentgen

> Shitty webapp for B3 Dev TP1@1.0.0 dev
> nodemon -L src/app.js

[nodemon] 3.1.11
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,cjs,json
[nodemon] starting `node src/app.js`
Server running at http://localhost:3000
```
```
curl http://localhost:3000
```
```
    <!DOCTYPE html>
    <html>
      <head>
        <title>HTTP Cat</title>
```

## 3. Hot reload pleaaaase
### 🌞 Lancer un nouveau conteneur à partir de l'image shitty_app
```
docker run -p 3000:3000 -d -v "C:\Users\Utilisateur\Documents\Bachelor_dev\COURS\virtualisation conteneurisation\TP1\tp1_part2\1\src:/app/src" shitty_app:1.0
9e49a18dc7e6925b42b770170d8ddbd569579bbbb02e9c334c7b4a0deed041c3
```
### 🌞 Vérifier que ça fonctionne
```
docker logs 9e4 -f

> Shitty webapp for B3 Dev TP1@1.0.0 dev
> nodemon -L src/app.js

[nodemon] 3.1.11
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,cjs,json
[nodemon] starting `node src/app.js`
Server running at http://localhost:3000
[nodemon] restarting due to changes...
[nodemon] starting `node src/app.js`
Server running at http://localhost:3000
```

## 4. Compose please
### 🌞 Transformer ce docker run en compose.yml
```
services:
  app:
    image: shitty_app:1.0
    ports:
      - "3000:3000"
    volumes:
      - ./src:/app/src
```

## 5. DB please
### 🌞 Créer un Dockerfile

