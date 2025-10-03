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

## 1. Docker compose

Charger l'image Docker de la webapp 1

```powershell
PS td6> docker load -i .\webapp-1.tar
```

Démarrer les services

```powershell
PS td6> docker compose up -d
```

On remarque que lorsque l'on tente d'ajouter un utilisateur, la webapp n'arrive pas à écrire dans le dossier de log `/var/log/webapp/` du conteneur.

```
[ERROR] unable to create /var/log/webapp/webapp.log (No such file or directory (os error 2)), exiting

thread 'actix-rt|system:0|arbiter:3' panicked at src/main.rs:52:9:

explicit panic
```

Le problème vient du fait que le dossier `/var/log/webapp/` n'existe pas dans le conteneur, et que la webapp n'a pas les droits pour le créer. On peut le vérifier en se connectant au conteneur et en essayant de créer le dossier manuellement.

```powershell
PS td6> docker exec -it td6-webapp bash
root@:/# ll /var/log/
total 276
-rw-r--r-- 1 root root 280819 Oct 31  2024 dnf5.log
root@:/# mkdir /var/log/webapp
root@:/# exit
```

Redémarrer la stack

```powershell
PS td6> docker compose restart
```

Maintenant il n'y a plus d'erreur, on peut ajouter un utilisateur :)

Pour que cela fonctionne automatiquement, Lucas pourrait ajouter la ligne suivante dans son Dockerfile :

```dockerfile
RUN mkdir -p /var/log/webapp
```

Arrêter la stack

```powershell
PS td6> docker compose down
```

Charger l'image Docker de la webapp 2

```powershell
PS td6> docker load -i .\webapp-2.tar
```

Démarrer les services

```powershell
PS td6> docker compose up -d
```

Tout fonctionne correctement avec cette nouvelle version de la webapp.

Arrêter la stack

```powershell
PS td6> docker compose down
```

## 2. Kubernetes

*Pour information, j'utilise l'implementation de kubernetes fournie par Docker Desktop, et non Minikube.*

```powershell
PS td6> kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   33h
No resources found in default namespace.
```

```powershell
PS td6> kompose convert -f compose.yaml -o kube
WARN Restart policy 'unless-stopped' in service webapp is not supported, convert it to 'always' 
WARN Restart policy 'unless-stopped' in service mongodb is not supported, convert it to 'always' 
WARN File don't exist or failed to check if the directory is empty: CreateFile :/data/db: The filename, directory name, or volume label syntax is incorrect. 
INFO Kubernetes file "mongodb-service.yaml" created 
INFO Kubernetes file "webapp-service.yaml" created 
INFO Kubernetes file "mongodb-deployment.yaml" created
INFO Kubernetes file "env-configmap.yaml" created
INFO Kubernetes file "td6-db-data-persistentvolumeclaim.yaml" created
INFO Kubernetes file "webapp-deployment.yaml" created
```

Appliquer les configurations

```powershell
PS td6/kube> kubectl apply -f .
configmap/env created
deployment.apps/mongodb created
service/mongodb created
persistentvolumeclaim/td6-db-data created
deployment.apps/webapp created
service/webapp created
```

Récupérer les status des ressources

```powershell
PS td6/kube> kubectl get all
NAME                           READY   STATUS         RESTARTS   AGE
pod/mongodb-7fd446dc6f-f7zwk   1/1     Running        0          98s
pod/webapp-7f7f69f6f7-zz776    0/1     ErrImagePull   0          98s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP     2m2s
service/mongodb      ClusterIP   10.108.207.196   <none>        27017/TCP   98s
service/webapp       ClusterIP   10.106.192.207   <none>        8080/TCP    98s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongodb   1/1     1            1           98s
deployment.apps/webapp    0/1     1            0           98s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/mongodb-7fd446dc6f   1         1         1       98s
replicaset.apps/webapp-7f7f69f6f7    1         1         0       98s
```

Le pod de la webapp n'a pas pu démarrer car l'image n'a pas été trouvée sur un registry. Normal puisque nous avons build l'image localement. Pour que Kubernetes puisse utiliser cette image, il faut lui indiquer de ne pas tenter de la récupérer depuis un registry. Pour cela, il faut modifier le fichier `webapp-deployment.yaml` et ajouter la ligne `imagePullPolicy: IfNotPresent` dans la section `containers`.

Regardons à nouveau les status des ressources.

```powershell
PS td6/kube> kubectl get all
NAME                           READY   STATUS    RESTARTS      AGE
pod/mongodb-7fd446dc6f-f7zwk   1/1     Running   5 (62s ago)   13m
pod/webapp-77c49b79f5-m2krj    1/1     Running   0             4s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP     13m
service/mongodb      ClusterIP   10.108.207.196   <none>        27017/TCP   13m
service/webapp       ClusterIP   10.106.192.207   <none>        8080/TCP    13m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongodb   1/1     1            1           13m
deployment.apps/webapp    1/1     1            1           13m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/mongodb-7fd446dc6f   1         1         1       13m
replicaset.apps/webapp-77c49b79f5    1         1         1       4s
replicaset.apps/webapp-7f7f69f6f7    0         0         0       13m
```

Tout fonctionne correctement maintenant :)