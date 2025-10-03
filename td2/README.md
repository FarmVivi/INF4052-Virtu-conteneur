# COMPTE RENDU TD 2 - Victor VAIZAND

## Exercice 2

Créez les fichiers suivants à la racine de votre projet :

### `src/index.js`
```js
const http = require('http');
const port = 3030;
const requestHandler = (request, response) => {
    console.log(request.url);
    response.end("Hello Node.js Server!");
};
const server = http.createServer(requestHandler);
server.listen(port, (err) => {
    if (err) {
        return console.log("something bad happened", err);
    }
    console.log('server is listening on port', port);
});
```

### `package.json`

```json
{
  "name": "docker-exo-2",
  "version": "1.0.0",
  "description": "",
  "main": "src/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node src/index.js"
  },
  "keywords": [],
  "author": "",
  "license": "MIT"
}
```

### Étapes

1. **Testez localement** dans un premier temps. Installez Node.js sur votre machine et veillez à ce que les commandes suivantes fonctionnent :

   * `npm install` : permet d’installer les paquets, doit être exécutée dans le même répertoire que `package.json`
   * `npm start` : démarre le serveur localement
   * `curl http://localhost:3030` pour valider que ça fonctionne

2. **Créez un Dockerfile** avec les contraintes suivantes :

   * Utiliser une image de base Node.js (peu importe la version)
   * Copier le `package.json` depuis votre projet local vers votre image
   * Installer les dépendances
   * Lancer la commande `npm start` au démarrage du conteneur

3. **Construisez l’image**, démarrez un conteneur avec cette dernière et connectez-vous y.

   * Exposer le port `3030`
   * Vérifiez que vous pouvez communiquer avec le serveur **de l’intérieur du conteneur**
   * Vérifiez que vous pouvez communiquer avec le serveur **de l’extérieur du conteneur**

---

## Synthèse
- Image Node.js construite avec les dépendances et le serveur HTTP.
- Conteneur exposé sur `3030`, testé depuis l’hôte et depuis l’environnement du conteneur.

## Procédure
### Construction de l'image
```powershell
PS td2> docker build -t td2 .
```

### Exécution et vérifications
```powershell
PS td2> docker run --rm -it -p 3030:3030 td2
```

Depuis l'hôte :
```powershell
PS td2> curl http://localhost:3030
```

Depuis le conteneur (session interactive ouverte par `docker run`) :
```bash
curl http://localhost:3030
```
