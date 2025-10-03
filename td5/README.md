# COMPTE RENDU TD 5 - Victor VAIZAND

L’objectif ici est d’utiliser **Docker Compose** sur les exercices précédents (1 à 4).  
Pour cela, créez un dépôt **Git** avec l’architecture à droite, à rendre à la fin.

### Contraintes et conseils
- Utiliser les attributs **configs**, **secrets** et **env_file** où ils sont utiles
- Ne pas exposer vos ports sur `0.0.0.0`, mais uniquement sur `localhost`
- Pour le 3, utilisez un **volume docker**
- Pour le 4 :  
  - uniquement un client **debian_psql**  
  - un réseau docker est créé par `compose.yaml` par défaut, et les services y sont automatiquement inclus  
  - une fois l’archi lancée, vous trouverez le docker **debian_psql** arrêté (`exited with code 0`), car il se connecte à la base Postgres en utilisant les variables d’environnement **PGPASSWORD**, **PGUSER**, **PGDATABASE** et **PGHOST**, seulement une fois la base Postgres prête à recevoir des connexions (**healthcheck**)  

### Arborescence attendue

```

1
├── compose.yaml
├── msg.txt
└── image
└── Dockerfile

2
├── compose.yaml
└── image
├── Dockerfile
├── src/index.js
└── package.json

3
├── compose.yaml
├── db_password.txt
└── .env

4
├── compose.yaml
├── db_password.txt
├── .env
└── debian_psql
└── Dockerfile

```

### Documentation utile

- [Intro Docker Compose](https://docs.docker.com/compose/intro/compose-application-model/)  
- [Configs](https://docs.docker.com/reference/compose-file/configs/)  
- [Build](https://docs.docker.com/reference/compose-file/build/)  
- [Secrets](https://docs.docker.com/compose/how-tos/use-secrets/)  
- [Networking](https://docs.docker.com/compose/how-tos/networking/)  
- [Volumes](https://docs.docker.com/reference/compose-file/volumes/)  
- [Variables d’environnement](https://docs.docker.com/compose/how-tos/environment-variables-set-environment-variables/)  
- [Startup order](https://docs.docker.com/compose/how-tos/startup-order/)  

---

## Synthèse
- Chaque TD (1→4) est encapsulé dans sa propre stack Compose (`1/` à `4/`) afin d’isoler les scénarios.
- La stack 1 injecte le fichier message via l’attribut `configs`, montrant un exemple de configuration read-only.
- Les services exposés restent confinés à `127.0.0.1`; les données Postgres sont persistées dans des volumes Docker nommés.
- Les identifiants sensibles sont fournis via `env_file` et `secrets`, et les ordres de démarrage critiques sont sécurisés par `depends_on` + healthcheck.

## Procédure
### Stack 1 — Cat sur Debian (TD1)
- **Objectif :** reproduire le comportement du conteneur `hello-world` en fournissant le message via un `config` Compose.
1. Construire l’image et exécuter la commande de lecture :
    ```powershell
    PS td5\1> docker compose up --build --abort-on-container-exit
    ```
    Le fichier `/opt/msg.txt` est monté depuis le `config` `message`; le conteneur se termine après l’avoir affiché.
2. Remplacer le message sans reconstruire l’image (montage du fichier alternatif) :
    ```powershell
    PS td5\1> docker compose run --rm -v ${PWD}/../td1/msg2.txt:/opt/msg.txt cat
    ```
    Cela court-circuite le config par un montage ponctuel, comme demandé dans le TD1.

### Stack 2 — Serveur Node.js (TD2)
- **Objectif :** packager le serveur HTTP Node.js avec un build automatisé.
1. Démarrer le service :
    ```powershell
    PS td5\2> docker compose up --build -d
    ```
2. Vérifier depuis le conteneur :
    ```powershell
    PS td5\2> docker compose exec nodejsweb curl http://127.0.0.1:3030
    ```
3. Vérifier depuis l’hôte (port limité à `localhost`) :
    ```powershell
    PS td5\2> curl http://localhost:3030
    ```
4. Nettoyer :
    ```powershell
    PS td5\2> docker compose down
    ```

### Stack 3 — Postgres + secrets (TD3)
- **Objectif :** persister la base Postgres via un volume Docker et externaliser les identifiants.
1. Lancer la stack :
    ```powershell
    PS td5\3> docker compose up -d
    ```
    `POSTGRES_USER`/`POSTGRES_DB` proviennent de `.env` et le mot de passe via le secret `db_password`.
2. Vérifier le volume persistant :
    ```powershell
    PS td5\3> docker volume inspect td5_3_db_data
    ```
3. Tester la connexion :
    ```powershell
    PS td5\3> docker compose exec postgres psql -U john -d doe -c "SELECT current_database();"
    ```
4. Arrêt + nettoyage :
    ```powershell
    PS td5\3> docker compose down -v
    ```

### Stack 4 — Postgres + client Debian (TD4)
- **Objectif :** démontrer l’accès à Postgres depuis un client Debian isolé via le réseau Docker interne.
1. Lancer la stack :
    ```powershell
    PS td5\4> docker compose up --build
    ```
    Le healthcheck `pg_isready` doit passer avant le démarrage du client grâce à `depends_on.condition: service_healthy`.
2. Comprendre l’arrêt du client :
    - `debian_psql` exécute `psql` une seule fois avec les variables `PGPASSWORD`, `PGUSER`, `PGDATABASE`, `PGHOST` puis se termine (`exited with code 0`).
    - Pour rejouer manuellement une connexion :
        ```powershell
        PS td5\4> docker compose run --rm debian_psql psql -c "\conninfo"
        ```
3. Nettoyage complet (services + volumes) :
    ```powershell
    PS td5\4> docker compose down -v
    ```

## Réponses
### Utilisation de `configs`, `secrets`, `env_file`
- `configs` : stack 1 publie `msg.txt` en configuration read-only (`configs.message`) montée sur `/opt/msg.txt`.
- `env_file` : stacks 3 et 4 chargent `POSTGRES_USER`/`POSTGRES_DB` depuis `.env`.
- `secrets` : le mot de passe Postgres est injecté via `db_password.txt` dans les stacks 3 et 4.

### Exposition des ports uniquement sur `localhost`
- Stack 2 : `127.0.0.1:3030:3030`.
- Stacks 3 & 4 : `127.0.0.1:5432:5432`.
- Stack 1 : aucun port exposé.

### Volume Docker pour le TD3
- `td5_3_db_data` est déclaré dans `compose.yaml` et monté sur `/var/lib/postgresql/data`, assurant la persistance des données.

### Particularités du TD4
- Le client `debian_psql` utilise le réseau Compose par défaut et s’arrête immédiatement après une connexion réussie (`code 0`), confirmant que la base est prête grâce au healthcheck.

## Points clés
- La stack 1 illustre l’attribut `configs` de Compose pour distribuer un fichier read-only au conteneur.
- Les stacks 3 et 4 combinent `env_file` + `secrets` pour séparer variables publiques et mots de passe.
- Les ports publiés sont tous bornés sur `127.0.0.1`, limitant l’exposition.
- Le couple healthcheck `pg_isready` + `depends_on.condition: service_healthy` garantit une séquence fiable avant l’exécution du client.
- Les volumes `td5_3_db_data` et `td5_4_db_data` offrent une persistance simple à gérer (et à purger via `docker compose down -v`).