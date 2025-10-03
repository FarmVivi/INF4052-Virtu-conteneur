# COMPTE RENDU TD 4 - Victor VAIZAND

## Exercice 4

L’objectif est de comprendre les **réseaux docker** et comment **faire communiquer deux conteneurs**.

Dans un premier temps, créez un **Dockerfile** construisant une image avec les contraintes suivantes:

* Image de base: debian
* Les outils pour se connecter à une base de données **postgresql**

Nous allons créer **deux conteneurs** qui seront capables de communiquer entre eux. Pour cela:

* Créez un **réseau docker** que vous nommerez « my-network »

* Créez un conteneur **postgres** que vous connecterez à ce réseau

* Créez un second conteneur (**client1**) issu de l’image que vous avez **créée**, en le connectant également à ce réseau

* Connectez-vous à **se** second conteneur et tentez de vous connecter à la base de données (`psql`)

* Démarrez un troisième conteneur (**client2**) depuis votre image, **ne pas le connecter au réseau**, et tenter de se connecter à la base de données de la même façon

**Quelques informations:**

* Dans le second conteneur (issu de votre image), exécutez la commande `psql --help` pour avoir de l’aide sur comment se connecter à une instance distante de **postgres**
* Se référer à la **cheatsheet** partagée pour savoir comment créer un réseau

---
## Synthèse
- Image cliente basée sur Debian pour accéder à Postgres construite avec succès.
- Réseau `my-network` partagé entre le serveur Postgres et `client1` pour valider la connectivité.
- `client2` isolé confirme l'absence d'accès hors réseau dédié.

## Procédure
### Construction et réseau
```powershell
PS td4> docker build -t td4 .
PS td4> docker network create my-network
```

### Déploiement des conteneurs
```powershell
PS td4> docker run --name postgres --network my-network -e POSTGRES_PASSWORD=monsuperbemotdepassetressecurise -d postgres:17.6-alpine3.22
PS td4> docker run --name client1 --network my-network -it --rm td4
PS td4> docker run --name client2 -it --rm td4
```

### Connexions Postgres
Depuis `client1` connecté au réseau :
```bash
psql -h postgres -U postgres
# mdp : monsuperbemotdepassetressecurise
```

Depuis `client2` hors réseau :
```bash
psql -h postgres -U postgres
# psql: error: could not translate host name "postgres" to address: Name or service not known
```

## Observations
- Le DNS Docker ne résout `postgres` que pour les conteneurs rattachés à `my-network`.
- L'échec attendu de `client2` valide l'isolation réseau.
