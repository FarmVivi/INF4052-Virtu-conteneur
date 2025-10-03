# COMPTE RENDU TD 6 - Victor VAIZAND

# Scénario

Votre collègue Lucas vous fournit l’image Docker de la webapp que vous devez packager et déployer.

Il est demandé de rendre un petit rapport pour répondre aux questions et expliquer la démarche. Joignez les fichiers de configuration utilisés au ZIP rendu.

## Message de Lucas

> Coucou,
>
> J’ai fini mon dev sur « webapp », je t’ai uploadé l’image Docker. Elle écoute sur le port 8080. Elle met un peu de temps à démarrer, environ 20 secondes.
>
> Pour t’en servir, t’as juste besoin de brancher dessus un MongoDB en lui passant les variables d’environnement **MONGODB_USER**, **MONGODB_PASSWORD**, et **MONGODB_HOST**.
>
> Pas le temps de tester, faut que ça soit en prod le plus vite possible.
>
> PS : je pars en vacances, bon chance !
>
> _Lucas E._

## 1. Docker compose

a. Chargez l’image Docker **webapp-1.tar** et créez un fichier **compose.yaml** avec un service `mongodb/mongodb-community-server` pour la démarrer. (images sur le Moodle)

b. Essayer d’ajouter un utilisateur.

c. Il est possible que Lucas n’ait pas bien testé. Quel est le problème ? Comment le voyez-vous ?

d. **« Faut qu’ça tourne ! »** Redémarrez le conteneur **webapp** et connectez-y vous pour le faire fonctionner tout de même.

e. Quelle ligne Lucas pourrait-il rajouter à son **Dockerfile** pour le résoudre ?

f. Remplacez l’image de la webapp par celle de **webapp-2.tar** et testez que l’ajout d’utilisateur soit fonctionnel.

## 2. Kubernetes

a. Installez et démarrez Minikube (`minikube start --driver=docker`).

b. Monitorer l’état de votre cluster Minikube avec `kubectl get all`.

c. Utilisez l’utilitaire **kompose** pour convertir votre **compose.yaml** en configurations K8s.

d. Appliquez les configurations. Quelles corrections avez-vous dû leur apporter ? Dans quel ordre les appliquez-vous, et pourquoi ? Utilisez les attributs **readinessProbe** sur les pods MongoDB et webapp.

e. Pourquoi la webapp n’a pas été déployée ? Corrigez cela avec `minikube docker-env`.

f. Paramétrez le déploiement de la webapp pour avoir deux répliques. Testez la redondance en tuant un des pods de la webapp.

---
## Synthèse
- Stack Docker Compose fonctionnelle après création du dossier `/var/log/webapp` manquant pour *webapp-1* ; *webapp-2* fonctionne sans correctif.
- Workflow Kubernetes généré avec `kompose`, adapté (StatefulSet MongoDB, probes, NodePort) et testé sur Docker Desktop.
- Mise en place de 2 répliques pour la webapp et validation de la résilience via suppression d'un pod.

## Procédure
### Docker Compose
1. Charger les images fournies :
    ```powershell
    PS td6> docker load -i .\webapp-1.tar
    ```
2. Démarrer la stack :
    ```powershell
    PS td6> docker compose up -d
    ```
3. Constater l'échec d'écriture du log :
    ```
    [ERROR] unable to create /var/log/webapp/webapp.log (No such file or directory (os error 2))
    ```
4. Créer le dossier manquant puis redémarrer :
    ```powershell
    PS td6> docker exec -it td6-webapp bash -c "mkdir -p /var/log/webapp"
    PS td6> docker compose restart
    ```
  → Correctif pérenne conseillé : `RUN mkdir -p /var/log/webapp` dans le Dockerfile.
5. Arrêter la stack et charger la V2 :
    ```powershell
    PS td6> docker compose down
    PS td6> docker load -i .\webapp-2.tar
    PS td6> docker compose up -d
    PS td6> docker compose down
    ```

### Kubernetes (Docker Desktop au lieu de Minikube)
1. Vérifier le cluster puis convertir Compose :
    ```powershell
    PS td6> kubectl get all
    PS td6> kompose convert -f compose.yaml -o kube
    ```
2. Appliquer les manifestes générés :
    ```powershell
    PS td6/kube> kubectl apply -f .
    ```
3. Corriger `ErrImagePull` sur la webapp (image locale) : ajouter `imagePullPolicy: IfNotPresent` dans `webapp-deployment.yaml` et ré-appliquer.
4. Durcir MongoDB : transformer le Deployment en StatefulSet, rendre le service headless, augmenter le PVC à 1 Gi, ajouter `terminationGracePeriodSeconds`, puis :
    ```powershell
    PS td6/kube> kubectl delete deployment mongodb
    PS td6/kube> kubectl delete service mongodb
    PS td6/kube> kubectl apply -f mongodb-statefulset.yaml
    PS td6/kube> kubectl apply -f mongodb-service.yaml
    ```
5. Ajouter probes :
    - Webapp readiness HTTP `/` port 8080.
    - MongoDB readiness/liveness via `mongosh --eval "db.adminCommand('ping')"`.
    ```powershell
    PS td6/kube> kubectl apply -f webapp-deployment.yaml
    PS td6/kube> kubectl apply -f mongodb-statefulset.yaml
    ```
6. Exposer la webapp via NodePort (32080) dans `webapp-service.yaml` puis appliquer pour un accès `http://localhost:32080`.
7. Monter à deux répliques (`replicas: 2`) et surveiller :
    ```powershell
    PS td6/kube> kubectl apply -f webapp-deployment.yaml
    PS td6/kube> kubectl get pods -w
    ```
8. Tester la résilience :
    ```powershell
    PS td6/kube> kubectl delete pod webapp-<pod-id>
    ```
9. Nettoyage final :
    ```powershell
    PS td6/kube> kubectl delete -f .
    ```

## Réponses
### 1.a — Charger l’image webapp-1 et composer avec MongoDB
- `docker load -i webapp-1.tar` puis `docker compose up -d` pour lancer webapp + `mongodb/mongodb-community-server` sur le même réseau.

### 1.b — Tentative d’ajout d’utilisateur
- L’appel POST depuis la webapp déclenche immédiatement une erreur d’écriture dans `/var/log/webapp`, ce qui empêche la création d’utilisateur.

### 1.c — Problème constaté et preuve
- Le conteneur webapp plante car le dossier `/var/log/webapp` est absent ; `docker compose logs webapp` remonte `unable to create /var/log/webapp/webapp.log`.

### 1.d — Remise en route manuelle
- Connexion interactive : `docker compose exec webapp bash` puis `mkdir -p /var/log/webapp`; un `docker compose restart` suffit ensuite pour que la webapp redémarre correctement.

### 1.e — Correctif à ajouter au Dockerfile
- Ajouter `RUN mkdir -p /var/log/webapp` dans l’image pour créer le dossier au build et éviter toute intervention manuelle.

### 1.f — Passage à webapp-2
- Après `docker load -i webapp-2.tar` et relance de la stack, l’ajout d’utilisateur fonctionne et le log est créé automatiquement.

### 2.a — Cluster local démarré
- Utilisation de l’orchestrateur Docker Desktop (équivalent Minikube) ; commande de référence : `minikube start --driver=docker` si nécessaire.

### 2.b — Supervision du cluster
- `kubectl get all` utilisé en continu pour suivre pods, services et replicaSets pendant le déploiement.

### 2.c — Conversion Compose → K8s
- `kompose convert -f compose.yaml -o kube` génère les manifests de base (services, déploiements, configmap, PVC).

### 2.d — Corrections appliquées et ordre
- Ordre : configmap & PVC → MongoDB (StatefulSet + service headless) → webapp. Ajouts : `imagePullPolicy`, probes HTTP & `mongosh`, volume de 1 Gi, `terminationGracePeriodSeconds` et transformation du Deployment MongoDB en StatefulSet.

### 2.e — Webapp non déployée initialement
- Cause : l’image n’était disponible que localement. Solution avec Minikube : `minikube docker-env` puis rebuild/push dans l’environnement du cluster (ou `imagePullPolicy: IfNotPresent` après chargement local sous Docker Desktop).

### 2.f — Redondance de la webapp
- `replicas: 2` ajouté dans `webapp-deployment.yaml`. Tests OK : suppression d’un pod (`kubectl delete pod …`) pendant que l’autre continue de répondre via le service NodePort.

## Points clés
- L'ajout du dossier `/var/log/webapp` règle le crash de la v1 ; la v2 embarque déjà le correctif.
- `kompose` génère une base solide mais nécessite ajustements manuels pour MongoDB (StatefulSet) et pour l'image locale de la webapp.
- Les probes et le scaling à 2 répliques garantissent la tolérance aux pannes lors de la suppression d'un pod.
