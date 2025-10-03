# COMPTE RENDU TD 3 - Victor VAIZAND

## Exercice 3

L’objectif est de comprendre l’intérêt des volumes dans un environnement Docker.  

Il est demandé ici d’écrire un fichier **bash** et **txt** regroupant les opérations suivantes :

1. **Démarrer un conteneur** se basant sur l’image officielle `postgres` avec les informations suivantes uniquement en CLI :
   - Nom d’utilisateur : `john`  
   - Nom de la base de données : `doe`  
   - Mot de passe : `johndoe`  

   ```bash
   docker run -e KEY1=VALUE1 -e KEY2=VALUE2 myimage
   ```

2. La commande suivante peut être étendue pour spécifier un fichier d’environnement :

   ```bash
   docker run --env-file /path/to/env.txt myimage
   ```

   Le fichier doit être un fichier contenant un couple clé-valeur par ligne (`KEY=VALUE`).

3. **Cherchez et notez le dossier** (pour Postgres) qui contiendra les données de la future base de données.

4. **Démarrez un second conteneur**, mais avec un **volume système partagé (bind mount)** (dans le dossier courant par exemple, que l’on appellera `data`) avec Postgres, qui contiendra les données à persister.

5. **Démarrez un troisième conteneur**, mais avec un **volume Docker** cette fois-ci (même consigne que précédemment).

---

## Synthèse
- Initialisation d'un conteneur Postgres via variables d'environnement ou fichier `.env`.
- Localisation du dossier de données persistant : `/var/lib/postgresql/data`.
- Tests de persistance avec bind mount local puis volume Docker nommé.

## Procédure
### Lancement en CLI
```powershell
PS td3> docker run -e POSTGRES_PASSWORD=johndoe -e POSTGRES_USER=john -e POSTGRES_DB=doe -p 5432:5432 --rm -it postgres:17.6-alpine3.22
```

### Lancement via fichier d'environnement
```powershell
PS td3> docker run --env-file postgres.env -p 5432:5432 --rm -it postgres:17.6-alpine3.22
```

### Bind mount du dossier `data`
```powershell
PS td3> docker run --env-file postgres.env -p 5432:5432 --rm -it -v ${PWD}/data:/var/lib/postgresql/data postgres:17.6-alpine3.22
```

### Volume Docker nommé
```powershell
PS td3> docker run --env-file postgres.env -p 5432:5432 --rm -it -v td3_postgres_data:/var/lib/postgresql/data postgres:17.6-alpine3.22
```

## Réponses
### Q1 — Lancement Postgres uniquement en CLI
- Variables `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` passées avec `-e` pour créer la base `doe` et l'utilisateur `john`.

### Q2 — Utilisation d'un fichier d'environnement
- Passage des mêmes paires clé/valeur via `--env-file postgres.env`.

### Q3 — Localisation des données Postgres
- Chemin de stockage : `/var/lib/postgresql/data`.

### Q4 — Persistance via bind mount
- Montage du dossier `data` courant dans le chemin de données pour valider la persistance.

### Q5 — Persistance via volume Docker
- Création implicite du volume `td3_postgres_data` monté sur `/var/lib/postgresql/data`.
