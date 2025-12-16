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
```
dans tp1_part2/5/Dockerfile ajouter un Dockerfile identique à celui du tp1_part2/1
```
### 🌞 Créer un compose.yml
```
voir tp1_part2/5/compose.yml
```
### 🌞 Allumer la stack et prouver que ça fonctionne
```
docker compose up
docker ps

CONTAINER ID   IMAGE                    COMMAND                  CREATED              STATUS          PORTS                                         NAMES
68934a122bcc   mysql:8.4                "docker-entrypoint.s…"   About a minute ago   Up 24 seconds   3306/tcp, 33060/tcp                           5-db-1
6f07d2060d2c   phpmyadmin               "/docker-entrypoint.…"   13 minutes ago       Up 24 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp       5-pma-1
f44c66e6e764   shitty_app_with_db:1.0   "docker-entrypoint.s…"   13 minutes ago       Up 24 seconds   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   5-app-1
```
```
docker compose logs

pma-1  | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.21.0.3. Set the 'ServerName' directive globally to suppress this message
pma-1  | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.21.0.3. Set the 'ServerName' directive globally to suppress this message
pma-1  | [Tue Dec 16 09:03:42.142752 2025] [mpm_prefork:notice] [pid 1:tid 1] AH00163: Apache/2.4.65 (Debian) PHP/8.3.28 configured -- resuming normal operations
pma-1  | [Tue Dec 16 09:03:42.142819 2025] [core:notice] [pid 1:tid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
app-1  | 
app-1  | > Shitty webapp for B3 Dev TP1@1.0.0 dev
app-1  | > nodemon -L src/app.js
app-1  |
app-1  | [nodemon] 3.1.11
app-1  | [nodemon] to restart at any time, enter `rs`
app-1  | [nodemon] watching path(s): *.*
app-1  | [nodemon] watching extensions: js,mjs,cjs,json
app-1  | [nodemon] starting `node src/app.js`
app-1  | Waiting for database... (1/30)
app-1  | Waiting for database... (2/30)
...
app-1  | Waiting for database... (30/30)
app-1  | Failed to start: Error: Database connection failed
app-1  |     at waitForDB (/app/src/app.js:46:9)
app-1  | [nodemon] clean exit - waiting for changes before restart
app-1  |
app-1  | > Shitty webapp for B3 Dev TP1@1.0.0 dev
app-1  | > nodemon -L src/app.js
app-1  |
pma-1  | [Tue Dec 16 09:15:29.581484 2025] [mpm_prefork:notice] [pid 1:tid 1] AH00170: caught SIGWINCH, shutting down gracefully
db-1   | 2025-12-16 09:15:20+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.4.7-1.el9 started.
db-1   | 2025-12-16 09:15:21+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
db-1   | 2025-12-16 09:15:21+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.4.7-1.el9 started.
db-1   | 2025-12-16 09:15:21+00:00 [Note] [Entrypoint]: Initializing database files
db-1   | 2025-12-16T09:15:21.559328Z 0 [System] [MY-015017] [Server] MySQL Server Initialization - start.
db-1   | 2025-12-16T09:15:21.561349Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.4.7) initializing of server in progress as process 80
db-1   | 2025-12-16T09:15:21.576365Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
db-1   | 2025-12-16T09:15:22.257073Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
db-1   | 2025-12-16T09:15:22.964756Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
db-1   | 2025-12-16T09:15:26.050686Z 0 [System] [MY-015018] [Server] MySQL Server Initialization - end.
db-1   | 2025-12-16 09:15:26+00:00 [Note] [Entrypoint]: Database files initialized
db-1   | 2025-12-16 09:15:26+00:00 [Note] [Entrypoint]: Starting temporary server
db-1   | 2025-12-16T09:15:26.128548Z 0 [System] [MY-015015] [Server] MySQL Server - start.
db-1   | 2025-12-16T09:15:26.454584Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.4.7) starting as process 123
db-1   | 2025-12-16T09:15:26.479040Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
db-1   | 2025-12-16T09:15:26.983655Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
db-1   | 2025-12-16T09:15:27.441066Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
db-1   | 2025-12-16T09:15:27.441122Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.     
db-1   | 2025-12-16T09:15:27.445648Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
db-1   | 2025-12-16T09:15:27.469931Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: /var/run/mysqld/mysqlx.sock
db-1   | 2025-12-16T09:15:27.470123Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.4.7'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
db-1   | 2025-12-16 09:15:27+00:00 [Note] [Entrypoint]: Temporary server started.
db-1   | '/var/lib/mysql/mysql.sock' -> '/var/run/mysqld/mysqld.sock'
db-1   | Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
db-1   | Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
db-1   | Warning: Unable to load '/usr/share/zoneinfo/leapseconds' as time zone. Skipping it.
db-1   | 2025-12-16 09:16:26+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.4.7-1.el9 started.
db-1   | 2025-12-16 09:16:27+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
db-1   | 2025-12-16 09:16:27+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.4.7-1.el9 started.
db-1   | '/var/lib/mysql/mysql.sock' -> '/var/run/mysqld/mysqld.sock'
db-1   | 2025-12-16T09:16:27.457589Z 0 [System] [MY-015015] [Server] MySQL Server - start.
db-1   | 2025-12-16T09:16:27.864230Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.4.7) starting as process 1
db-1   | 2025-12-16T09:16:27.873972Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
db-1   | 2025-12-16T09:16:29.636159Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
db-1   | 2025-12-16T09:16:29.776391Z 0 [System] [MY-010229] [Server] Starting XA crash recovery...
db-1   | 2025-12-16T09:16:29.800640Z 0 [System] [MY-010232] [Server] XA crash recovery finished.
db-1   |
db-1   | InnoDB: Progress in percents: 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 272025-12-16T09:16:29.924214Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
db-1   | 2025-12-16T09:16:29.924272Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.     
db-1   |  282025-12-16T09:16:29.932247Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
db-1   |  29 30 31 32 33 34 35 36 372025-12-16T09:16:29.975969Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
db-1   | 2025-12-16T09:16:29.976208Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.4.7'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
pma-1  | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.21.0.3. Set the 'ServerName' directive globally to suppress this message
pma-1  | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.21.0.3. Set the 'ServerName' directive globally to suppress this message
pma-1  | [Tue Dec 16 09:16:27.175426 2025] [mpm_prefork:notice] [pid 1:tid 1] AH00163: Apache/2.4.65 (Debian) PHP/8.3.28 configured -- resuming normal operations
pma-1  | [Tue Dec 16 09:16:27.175525 2025] [core:notice] [pid 1:tid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
app-1  | [nodemon] 3.1.11
app-1  | [nodemon] to restart at any time, enter `rs`
app-1  | [nodemon] watching path(s): *.*
app-1  | [nodemon] watching extensions: js,mjs,cjs,json
app-1  | [nodemon] starting `node src/app.js`
app-1  | Waiting for database... (1/30)
app-1  | Waiting for database... (2/30)
app-1  | Waiting for database... (3/30)
...
app-1  | Waiting for database... (27/30)
app-1  | Waiting for database... (28/30)
app-1  | Waiting for database... (29/30)
app-1  | Waiting for database... (30/30)
```

```
curl http://localhost:8080

StatusCode        : 200
StatusDescription : OK
Content           : <!doctype html>
```
```
curl http://localhost:3000 

StatusCode        : 200
StatusDescription : OK
Content           :
                        <h1>Simple DB App</h1>
```
### 🌞 Mettre en place des données persistentes pour la db
```
volumes:
      - db_data:/var/lib/mysql
```