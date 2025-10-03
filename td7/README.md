# COMPTE RENDU TD 7 - Victor VAIZAND

# Scénario

Vous devez mettre en place une pipeline d’intégration continue pour tester votre application Web à chaque `git push` sur un **Gitea** que vous mettrez en place.  

Il est demandé de rendre un petit rapport pour répondre aux questions et expliquer la démarche. Joignez les fichiers de configuration utilisés au ZIP rendu.

## 1. Gitea
- Depuis la page de l’éditeur, trouvez le fichier `compose.yaml` vous permettant de faire tourner un serveur **Gitea**.
- Pour effectuer des actions à chaque opération de **push**, il existe les **Gitea Actions** (similaires aux Github Actions). Documentez-vous sur les **runners** et ajoutez un service `runner` au `compose.yaml` afin de pouvoir en utiliser un (voir la doc).
- Le réseau docker utilisé par les deux services doit se nommer `gitea`. (Attribut `networks -> gitea -> name`)
- Utilisez des volumes docker (pas **bind mount**) pour Gitea ainsi que pour le `runner`.
- Générez le fichier `config.yaml` comme indiqué dans la documentation. Les conteneurs créés par le `runner` devront tourner dans le réseau nommé `gitea`. (Attribut `container -> network`).  
  Les seuls fichiers dont vous aurez besoin sont :
  - `.env`
  - `compose.yaml`
  - `config.yaml`
- Lorsque vous faites `docker compose up`, puis dans un autre terminal `docker compose ps` (dans le même cwd), vérifiez que le conteneur du `runner` soit bien up. Il se peut que vous ayez à préciser à docker de le redémarrer en cas d’échec dans votre `compose.yaml`.

## 2. CI/CD de la Webapp
- Récupérez le code source de l’application Webapp disponible sur le Moodle, et placez-le dans un nouveau dépôt hébergé dans votre Gitea.
- Créez le fichier d’actions Gitea à la racine du dépôt (`.gitea/workflows/build.yaml`).
- L’application webapp a besoin d’une base de données Postgres.  
  Utilisez une section `services` pour déployer un conteneur `postgres` lors du `run` des Gitea Actions (voir [Creating PostgreSQL service containers](https://docs.github.com/en/actions/using-containerized-services/creating-postgresql-service-containers)).  
  Pour que la Webapp puisse requêter la BDD, vous devez lui passer les variables suivantes en environnement :
  - `MONGODB_USER`
  - `MONGODB_PASSWORD`
  - `MONGODB_HOST`
- Pour tester l’application Webapp, vous devez lancer la commande `cargo test` dans un environnement Rust. Pour cela, utilisez [https://github.com/dtolnay/rust-toolchain](https://github.com/dtolnay/rust-toolchain).

## Quelques conseils :
- Pensez à faire un `docker compose down` entre plusieurs essais, des fois ça aide.

---

## Synthèse
- Stack Gitea + runner opérationnelle sur le réseau Docker `gitea`, configurée via `compose.yaml` et `config.yaml` généré.
- Dépôt Webapp hébergé sur Gitea, pipeline `.gitea/workflows/build.yaml` exécutée avec succès (tests `cargo`).

## Procédure
### 1. Gitea & runner
1. Préparer la configuration :
    ```powershell
    PS td7> docker run --entrypoint="" --rm -it docker.io/gitea/act_runner:latest act_runner generate-config > config.yaml
    ```
    - Ajouter le réseau `gitea` dans `config.yaml` (`container.network`).
    - Étendre `compose.yaml` avec le service `gitea-runner` et les volumes Docker.
2. Démarrer la stack puis finaliser l'instance sur `http://localhost:3000` :
    ```powershell
    PS td7> docker compose up
    ```
3. Récupérer le token Actions (`Site administration ▸ Actions ▸ Runners`), l’injecter dans `compose.yaml` (`GITEA_RUNNER_REGISTRATION_TOKEN`) et relancer :
    ```powershell
    PS td7> docker compose up
    ```
4. Vérifier que le runner est `online` dans l’interface Gitea.

### 2. Webapp – CI/CD
1. Initialiser et pousser le dépôt vers Gitea :
    ```powershell
    PS td7> mkdir Webapp
    PS td7> cd Webapp
    PS td7\Webapp> git init
    PS td7\Webapp> git remote add origin http://localhost:3000/vvaizand/Webapp.git
    PS td7\Webapp> git add .
    PS td7\Webapp> git commit -m "Initial commit"
    PS td7\Webapp> git push -u origin main
    ```
2. Créer `.gitea/workflows/build.yaml` :
    - Image Rust via `dtolnay/rust-toolchain`.
    - Service Postgres déclaré dans `services`.
    - Variables d’environnement `MONGODB_USER/PASSWORD/HOST` pour la webapp.
    - Étape principale `cargo test`.
3. Pousser sur `main`, suivre l’exécution dans l’onglet **Actions** et valider le succès du run.

## Points clés
- Le réseau Docker explicite dans `config.yaml` garantit que les conteneurs lancés par le runner rejoignent `gitea`.
- Le fichier d’action réutilise la logique GitHub Actions, facilitant l’ajout du service Postgres et des variables MongoDB pour les tests Rust.
