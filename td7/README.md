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

## 1. Gitea

1. On prépare les fichiers.

On récupère `compose.yaml` depuis le site officiel de Gitea. A celui-ci, on ajoute un service `gitea-runner` en s’inspirant de la documentation Gitea Actions.

On génère le fichier `config.yaml` par défaut en lançant la commande suivante :

```powershell
PS td7> docker run --entrypoint="" --rm -it docker.io/gitea/act_runner:latest act_runner generate-config > config.yaml
```

Dans la `config.yaml`, on ajoute le réseau docker `gitea` dans la section `container -> network`.

2. On démarre la stack.

```powershell
PS td7> docker compose up
```

3. On finalise Gitea sur `http://localhost:3000` (création admin + instance).
4. On copie le token du runner dans **Site administration ▸ Actions ▸ Runners**.

On modifie le `compose.yaml` pour y insérer le token dans la variable d’environnement `GITEA_RUNNER_REGISTRATION_TOKEN`.

On relance la stack.

```powershell
PS td7> docker compose up
```

5. On vérifie que le nouveau runner est présent dans **Site administration ▸ Actions ▸ Runners**.

All good :)

## 2. Webapp – CI/CD

1. On crée un dépôt `Webapp` sur Gitea et on pousse les sources Rust téléchargées depuis le Moodle.

```powershell
PS td7> mkdir Webapp
PS td7> cd Webapp
PS td7\Webapp> git init
PS td7\Webapp> git remote add origin http://localhost:3000/vvaizand/Webapp.git
PS td7\Webapp> git add .
PS td7\Webapp> git commit -m "Initial commit"
PS td7\Webapp> git push -u origin main
```

2. On crée le fichier `.gitea/workflows/build.yaml` dans le dépôt pour définir la pipeline.
3. On pousse sur `main` pour déclencher l’action et on lit les logs dans l’onglet **Actions**.
4. On vérifie que l'Action a réussi sans erreur dans l'interface Gitea.

Well done !
