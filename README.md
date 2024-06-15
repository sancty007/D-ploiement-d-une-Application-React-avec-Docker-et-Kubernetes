# D-ploiement-d-une-Application-React-avec-Docker-et-Kubernetes

# Runbook pour le Développement, Dockerisation et Déploiement d'une Application React avec Docker et Kubernetes

Ce runbook couvre les étapes nécessaires pour créer, dockeriser et déployer une application React en utilisant Docker et Kubernetes.

## Prérequis

- **Node.js** : Assurez-vous que Node.js est installé sur votre machine.
- **Docker** : Assurez-vous que Docker est installé et en cours d'exécution sur votre machine.
- **Kubernetes** : Assurez-vous que Kubernetes est configuré et que `kubectl` est installé sur votre machine.
- **Git** : Assurez-vous que Git est installé pour la gestion du code source.

## Étape 1 : Création de l'Application React

### 1. Initialisation du Projet React

1. Ouvrez un terminal.
2. Exécutez les commandes suivantes pour créer un nouveau projet React :

   ```bash
   npx create-react-app my-react-app
   cd my-react-app
   ```

### 2. Développement de l'Application

1. Ajoutez vos composants, pages, et styles dans le dossier `src`.
2. Par exemple, créez un composant simple dans `src/components/HelloWorld.js` :

   ```jsx
   // src/components/HelloWorld.js
   import React from 'react';

   const HelloWorld = () => {
     return <h1>Hello, World!</h1>;
   };

   export default HelloWorld;
   ```

3. Modifiez `src/App.js` pour utiliser ce composant :

   ```jsx
   // src/App.js
   import React from 'react';
   import HelloWorld from './components/HelloWorld';

   function App() {
     return (
       <div className="App">
         <HelloWorld />
       </div>
     );
   }

   export default App;
   ```

### 3. Test de l'Application en Local

1. Démarrez le serveur de développement React :

   ```bash
   npm start
   ```

2. Ouvrez votre navigateur et allez à `http://localhost:3000` pour voir l'application en cours d'exécution.

## Étape 2 : Dockerisation de l'Application React

### 1. Création d'un Dockerfile

1. À la racine de votre projet, créez un fichier `Dockerfile` avec le contenu suivant :

   ```Dockerfile
   # Utiliser l'image de base Node.js
   FROM node:14-alpine

   # Définir le répertoire de travail dans le conteneur
   WORKDIR /app

   # Copier les fichiers nécessaires pour installer les dépendances
   COPY package*.json ./

   # Installer les dépendances
   RUN npm install

   # Copier le reste des fichiers de l'application
   COPY . .

   # Construire l'application React pour la production
   RUN npm run build

   # Étape finale : servir l'application avec un serveur web (nginx)
   FROM nginx:alpine
   COPY --from=0 /app/build /usr/share/nginx/html

   # Exposer le port 80 pour accéder à l'application
   EXPOSE 80

   # Commande pour démarrer nginx
   CMD ["nginx", "-g", "daemon off;"]
   ```

### 2. Construction et Exécution de l'Image Docker

1. Dans le terminal, exécutez les commandes suivantes pour construire l'image Docker et la démarrer :

   ```bash
   # Construire l'image Docker
   docker build -t my-react-app .

   # Démarrer un conteneur Docker avec l'application React
   docker run -p 8080:80 my-react-app
   ```

2. Ouvrez votre navigateur et allez à `http://localhost:8080` pour voir l'application en cours d'exécution.

## Étape 3 : Déploiement avec Kubernetes (K8s)

### 1. Création des Fichiers Kubernetes

#### Deployment (`deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-react-app
  labels:
    app: my-react-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-react-app
  template:
    metadata:
      labels:
        app: my-react-app
    spec:
      containers:
        - name: my-react-app
          image: my-react-app:latest
          ports:
            - containerPort: 80
```

#### Service (`service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-react-app
spec:
  selector:
    app: my-react-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### 2. Déploiement sur Kubernetes

1. Assurez-vous que Kubernetes est configuré et que `kubectl` est installé sur votre machine.
2. Appliquez les fichiers YAML avec `kubectl` :

   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```

3. Vérifiez l'état du déploiement avec `kubectl get pods`.

4. Exposez le service pour accéder à l'application :

   ```bash
   kubectl expose deployment my-react-app --type=LoadBalancer --name=my-react-app-service
   ```

5. Obtenez l'URL pour accéder à l'application :

   ```bash
   kubectl get service my-react-app-service
   ```

   Utilisez l'adresse IP ou le nom de service fourni pour accéder à votre application React.
