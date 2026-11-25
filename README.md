# TP4 : Docker w Docker Compose (application MERN)

## 1. Clonit el repo
Mel loul clonit el repo mta3 el projet w d5alt lel dossier :

```bash
git clone https://github.com/wiem-kb/MyAppReact.git
cd MyAppReact
```

Fih zouz dossiers : `Server` w `Client`.

## 2. 3malt el Dockerfile mta3 el server
Fil dossier `Server` 3malt fichier `Dockerfile` w 7attit fih el code hedha :

```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 9000
CMD ["npm", "start"]
```

Ba3d buildit l'image mta3 el server b el commande hedhi :

```bash
docker build -t mern-server ./Server
```

## 3. 3malt el Dockerfile mta3 el client
Fil dossier `Client` 3malt zeda `Dockerfile` w 7attit fih :

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

W buildit l'image mta3 el client :

```bash
docker build -t mern-client ./Client
```

## 4. 3malt el réseau
3malt réseau pour les conteneurs:

```bash
docker network create mern-network
```

## 5. Lancit MongoDB
Lancit conteneur MongoDB m3a il  réseau :

```bash
docker run -d --name mongodb --network mern-network mongo
```

## 6. Lancit el server w el client

```bash
docker run -d --name server --network mern-network -p 9000:9000 mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
```

7allit `http://localhost:3000` w bil port 9000 zeda fil navigateur : l9it fihom zouz li « API is running ».

## 7. 3malt el docker-compose
Fil racine mta3 el projet 3malt fichier `docker-compose.yml` w 7attit :

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

j ai lancé tous ce la b docker compose :

```bash
docker compose up --build
```

## 8. Pushit les images 3al Docker Hub

```
docker tag mern-server chaymaamara2005/mern-server:latest
docker tag mern-client chaymaamara2005/mern-client:latest
docker push chaymaamara2005/mern-server:latest
docker push chaymaamara2005/mern-client:latest
```

## 9. 3malt el pipeline Jenkins
Pushit el projet 3al GitHub, w 3malt fichier `Jenkinsfile` Ba3d fi Jenkins 3malt job Pipeline `TP4_MERN_Pipeline`, rbattou bel repo mta3 GitHub w lancit el build : les étapes lkol 5edmo.
