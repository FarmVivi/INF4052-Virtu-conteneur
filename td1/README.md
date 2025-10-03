# COMPTE RENDU TD 1 - Victor VAIZAND

## Exercice 1

L’objectif ici est de reproduire ce que fait le conteneur hello-world qu’on a pu voir sur la présentation.

### Contraintes :
- conteneur basé sur une image **Debian**  
- un fichier `/opt/msg.txt` sera copié depuis le dossier courant au moment de la construction de l’image  
- le conteneur utilisera la commande `/bin/cat` pour afficher le contenu du fichier `/opt/msg.txt`

```bash
docker run -it my-image
...
contenu
du fichier
...
```

**Question :**
Comment modifier le message affiché sans générer une nouvelle image ? Donnez la commande qui vous permet d’afficher un fichier depuis la machine hôte.

---

## Synthèse
- Image `td1` construite localement à partir du Dockerfile fourni.
- Exécution du conteneur pour afficher le contenu de `/opt/msg.txt`.
- Variation du message en montant un fichier alternatif depuis l'hôte.

## Procédure
### Construction de l'image
```powershell
PS td1> docker build -t td1 .
```

### Exécution par défaut
```powershell
PS td1> docker run --rm -it td1
...
contenu
du fichier
...
```

## Réponses
### Q1 — Modifier le message sans reconstruire l'image
- Monter un fichier local au démarrage grâce à `-v` pour remplacer `/opt/msg.txt`.
```powershell
PS td1> docker run --rm -it -v ${PWD}/msg2.txt:/opt/msg.txt td1
...
not
this
file
...
```
