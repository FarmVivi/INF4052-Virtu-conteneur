# COMPTE RENDU TD 1 - Victor VAIZAND

## Exercice 1

```powershell
PS td1> docker build -t td1 .
```

```powershell
PS td1> docker run --rm -it td1
...
contenu
du fichier
...
```

> Q1: Comment modifier le message affiché sans générer une nouvelle image? Donnez la commande qui vous permet d'afficher un fichier depuis la machine hôte.

> A1: Il faut monter un fichier manuellement au lancement du conteneur avec l'option -v de la commande docker run. Par exemple:
```powershell
PS td1> docker run --rm -it -v ${PWD}/msg2.txt:/opt/msg.txt td1
...
not
this
file
...
```
