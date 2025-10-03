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
NAME                           READY   STATUS             RESTARTS       AGE
pod/mongodb-7fd446dc6f-f7zwk   0/1     CrashLoopBackOff   14 (57s ago)   55m
pod/webapp-77c49b79f5-m2krj    1/1     Running            0              42m

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP     56m
service/mongodb      ClusterIP   10.108.207.196   <none>        27017/TCP   55m
service/webapp       ClusterIP   10.106.192.207   <none>        8080/TCP    55m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongodb   0/1     1            0           55m
deployment.apps/webapp    1/1     1            1           55m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/mongodb-7fd446dc6f   1         1         0       55m
replicaset.apps/webapp-77c49b79f5    1         1         1       42m
replicaset.apps/webapp-7f7f69f6f7    0         0         0       55m
```

On convertit le Deployment de la base de données MongoDB en StatefulSet pour bénéficier d'un stockage persistant.
On suit la documentation officielle de Kubernetes pour créer un StatefulSet : https://kubernetes.io/fr/docs/concepts/workloads/controllers/statefulset/

- Dans le fichier `mongodb-service.yaml`, on ajoute `clusterIP: None` pour rendre le service headless.
- On renomme le fichier `mongodb-deployment.yaml` en `mongodb-statefulset.yaml` et on remplace `kind: Deployment` par `kind: StatefulSet`.
- On ajoute le nom du service headless dans `serviceName: mongodb`.
- On supprime la section `strategy` qui n'est pas supportée par les StatefulSets.
- On configure `terminationGracePeriodSeconds` dans la `spec` pour laisser à MongoDB le temps de s'arrêter proprement.
- On augmente la taille du PersistentVolumeClaim à 1Gi dans `td6-db-data-persistentvolumeclaim.yaml` pour éviter les problèmes d'espace disque de MongoDB.

Dans notre cluster local, on supprime l'ancien Deployment et on applique le nouveau StatefulSet et recrée le Service.

```powershell
PS td6/kube> kubectl delete deployment mongodb
PS td6/kube> kubectl delete service mongodb
PS td6/kube> kubectl apply -f mongodb-statefulset.yaml
PS td6/kube> kubectl apply -f mongodb-service.yaml
```

Regardons à nouveau les status des ressources.

```powershell
PS td6/kube> kubectl get all
NAME                          READY   STATUS    RESTARTS       AGE
pod/mongodb-0                 1/1     Running   2 (2m6s ago)   7m6s
pod/webapp-77c49b79f5-m926f   1/1     Running   0              50m

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP     53m
service/mongodb      ClusterIP   None           <none>        27017/TCP   6m26s
service/webapp       ClusterIP   10.100.118.4   <none>        8080/TCP    50m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp   1/1     1            1           50m

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-77c49b79f5   1         1         1       50m

NAME                       READY   AGE
statefulset.apps/mongodb   1/1     7m7s
```

Maintenant on va rajouter des probes de readiness pour s'assurer que les pods sont prêts avant de recevoir du trafic.

Dans le fichier `webapp-deployment.yaml`, on ajoute la section suivante dans le container de la webapp :

```yaml
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 5
```

Dans le fichier `mongodb-statefulset.yaml`, on ajoute la section suivante dans le container de MongoDB :

```yaml
        readinessProbe:
          exec:
            command:
              - mongosh
              - --eval
              - "db.adminCommand('ping')"
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 5
```

On plus dans le fichier `mongodb-statefulset.yaml`, on harmonise la livenessProbe avec la readinessProbe en utilisant la même commande :

```yaml
        livenessProbe:
          exec:
            command:
              - mongosh
              - --eval
              - "db.adminCommand('ping')"
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 5
```

Appliquer les modifications

```powershell
PS td6/kube> kubectl apply -f webapp-deployment.yaml
PS td6/kube> kubectl apply -f mongodb-statefulset.yaml
```

Vérifier que les ressources sont bien déployées avec 2 répliques pour la webapp :

```powershell
PS td6/kube> kubectl get all
NAME                          READY   STATUS    RESTARTS      AGE
pod/mongodb-0                 1/1     Running   1 (11m ago)   105m
pod/webapp-5449674776-q5b7g   1/1     Running   1 (27m ago)   108m

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP     3h28m
service/mongodb      ClusterIP   None           <none>        27017/TCP   160m
service/webapp       ClusterIP   10.100.118.4   <none>        8080/TCP    3h25m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp   1/1     1            1           3h25m

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-5449674776   1         1         1       108m
replicaset.apps/webapp-5dd6b684bb   0         0         0       111m
replicaset.apps/webapp-77c49b79f5   0         0         0       3h25m

NAME                       READY   AGE
statefulset.apps/mongodb   1/1     161m
```

Bon c'est bien tout ça, mais comment on accède à la webapp maintenant ? Le service est de type ClusterIP, donc accessible uniquement depuis le cluster. Pour remédier à cela et y accéder depuis mon Windows, je vais modifier le service de la webapp pour le passer en NodePort.
Ma configuration actuelle ne m'autorise pas à utiliser le port 8080, lorsque j'essaie d'appliquer la modification, j'ai l'erreur suivante :
```
The Service "webapp" is invalid: spec.ports[0].nodePort: Invalid value: 8080: provided port is not in the valid range. The range of valid ports is 30000-32767
```
Je vais donc utiliser le port 32080 à la place.

```yaml
spec:
  type: NodePort
  ports:
    - name: "8080"
      port: 8080
      targetPort: 8080
      nodePort: 32080
```

Appliquer la modification

```powershell
PS td6/kube> kubectl apply -f webapp-service.yaml
```

Désormais, je peux accéder à la webapp depuis mon Windows en utilisant l'URL suivante : `http://localhost:32080` :)

