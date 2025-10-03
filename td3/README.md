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

> Q1: Démarrer un conteneur se basant sur l'image officielle postgres avec les informations suivantes uniquement en CLI
> - Nom d'utilisateur : john
> - Nom de la base de données : doe
> - Mot de passe : johndoe

> A1:
> ```powershell
> PS td3> docker run -e POSTGRES_PASSWORD=johndoe -e POSTGRES_USER=john -e POSTGRES_DB=doe -p 5432:5432 --rm -it postgres:17.6-alpine3.22
> ```

> Q2: Spécifier un fichier d'environnement pour démarrer le conteneur

> A2:
> ```powershell
> PS td3> docker run --env-file postgres.env -p 5432:5432 --rm -it postgres:17.6-alpine3.22
> ```

> Q3: Rechercher dans quel dossier l'image postgres officielle stocke les bases de données

> A3: Sur l'image postgres 17, le volume contenant les bases de données est /var/lib/postgresql/data

> Q4: Démarrer un second conteneur, mais avec un volume système partagé (bind mount) (dans le dossier courant par exemple, que l'on appellera `data`) avec postgres, qui contiendra les données à persister.

> A4:
> ```powershell
> PS td3> docker run --env-file postgres.env -p 5432:5432 --rm -it -v ${PWD}/data:/var/lib/postgresql/data postgres:17.6-alpine3.22
> ```

> Q5: Démarrer un troisième conteneur, mais avec un volume docker cette fois-ci (même consigne que précédemment)

> A5:
> ```powershell
> PS td3> docker run --env-file postgres.env -p 5432:5432 --rm -it -v td3_postgres_data:/var/lib/postgresql/data postgres:17.6-alpine3.22
> ```
