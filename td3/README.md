# COMPTE RENDU TD 3 - Victor VAIZAND

## Exercice 3

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
