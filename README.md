# TP4 : Docker et Docker Compose (application MERN)

**Étudiant(e) :** <ton nom> · **Classe :** 3ᵉ LIG · **Enseignante :** Zaineb SAKHRAWI

## Ce que j'ai fait

### 1. Cloner le projet
J'ai cloné le dépôt du TP, qui contient deux dossiers : `Client` (React avec Vite) et `Server` (API Express sur le port 9000).
```bash
git clone https://github.com/wiem-kb/MyAppReact.git
cd MyAppReact
```

### 2. Dockerfile du serveur (`Server/Dockerfile`)
```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 9000
CMD ["npm", "start"]
```
J'ai construit l'image avec :
```bash
docker build -t mern-server ./Server
```

### 3. Dockerfile du client (`Client/Dockerfile`)
Le client est construit avec `npm run build`, qui génère le dossier `dist` (Vite). Ce dossier est ensuite servi avec `serve` sur le port 3000.
```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
RUN npm install -g serve
EXPOSE 3000
CMD ["serve", "-s", "dist", "-l", "3000"]
```
J'ai construit l'image avec :
```bash
docker build -t mern-client ./Client
```

### 4. Réseau et conteneurs lancés à la main
```bash
docker network create mern-network
docker run -d --name mongodb --network mern-network mongo
docker run -d --name server --network mern-network -p 9000:9000 mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
```
L'application s'affiche sur http://localhost:3000 et l'API répond sur http://localhost:9000/api/health.

### 5. Docker Compose (`docker-compose.yml`)
J'ai défini 3 services : `mongodb`, `server` et `client`, sur le réseau `mern-network` (driver `bridge`).
Le serveur dépend de `mongodb` et reçoit `MONGO_URI=mongodb://mongodb:27017/merndb`. Le nom du service `mongodb` sert de hostname.
Le client dépend du serveur.
```yaml
services:
  mongodb:
    image: mongo
    container_name: mongodb
    networks:
      - mern-network

  server:
    build: ./Server
    container_name: server
    ports:
      - "9000:9000"
    networks:
      - mern-network
    depends_on:
      - mongodb
    environment:
      - MONGO_URI=mongodb://mongodb:27017/merndb

  client:
    build: ./Client
    container_name: client
    ports:
      - "3000:3000"
    networks:
      - mern-network
    depends_on:
      - server

networks:
  mern-network:
    driver: bridge
```
Lancement :
```bash
docker compose up --build
```

### 6. Publication sur Docker Hub
```bash
docker login
docker tag mern-server <user-dockerhub>/mern-server:latest
docker tag mern-client <user-dockerhub>/mern-client:latest
docker push <user-dockerhub>/mern-server:latest
docker push <user-dockerhub>/mern-client:latest
```

### 7. Pipeline Jenkins (`Jenkinsfile`)
J'ai créé un job Pipeline (« Pipeline script from SCM ») relié à mon dépôt GitHub. Le pipeline a 4 étapes :
1. **Checkout** : récupère le code depuis GitHub.
2. **Build images** : construit `mern-server` et `mern-client`.
3. **Push Docker Hub** : se connecte avec l'identifiant Jenkins `dockerhub` et pousse les deux images.
4. **Deploy (compose)** : relance l'application avec `docker compose up -d --build`.

## Les livrables
- `Server/Dockerfile` et `Client/Dockerfile`
- `docker-compose.yml`
- `Jenkinsfile`
- Ce README

### Captures
**Pipeline Jenkins**

![Pipeline Jenkins](captures/pipeline-jenkins.png)

**Images Docker**

![Images](captures/images.png)

**Conteneurs**

![Conteneurs](captures/conteneurs.png)

### Liens
- GitHub : https://github.com/<user-github>/TP4
- Docker Hub (serveur) : https://hub.docker.com/r/<user-dockerhub>/mern-server
- Docker Hub (client) : https://hub.docker.com/r/<user-dockerhub>/mern-client
