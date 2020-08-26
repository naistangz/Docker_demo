# Dockerising Node.js App and MongoDb :whale:

## Commands used
```
docker-compose build - When changing a service's Dockerfile or contents, run to rebuild
docker-compose up    - Aggregates output of each container
docker ps            - ps stands for process status, shows running containers
```

## Create Dockerfile 
````bash
# Selecting the base image to build our own customised node.js application microservice
FROM node:alpine

# Working directory inside the container
WORKDIR /usr/src/app

# Copying dependencies
COPY package*.json ./

# Installing node package manager
RUN npm install

# Copying everything from current location to default location inside the container
COPY . .

# Expose the port
EXPOSE 3000

# Starting the app with CMD - 
CMD ["node", "app.js"]
````

## Create `docker-compose.yml` in same folder as `Dockerfile`
```yaml
version: '3'
services: 
  mongo: 
    container_name: mongo
    image: mongo
    restart: always
    ports: 
      - "27017:27017"
    #volumes: 
      #- "./data:/data/db"
  
  nodejs: 
    build: .
    container_name: dockerised-nodejs
    environment:
      - DB_HOST=mongodb://mongo:27017/posts
    links: 
      - mongo
    ports: 
      - "3000:3000"
    #command: node seeds/seed.js
```
---
**Troubleshooting:**
- In `docker-compose.yml` database seeded, however, app exited. 

**mongo**
- `image` - instead of building our own image, we pull down the standard `mongo` image from the Docker Hub registry
- `volume` - for persistent storage we mount the host directory `/data` to the container directory `data/db`
    - Mounting volumes gives us persistent storage so when starting a new container, Docker Compose will use the volume of any previous container and copy it to the new container, ensuring that no data is lost.
    - On host we have a physical file system, we plug a the physical file system into the container's virtual file system. The folder in physical hos file system is mounted into the virtual file system of Docker.
- Container runs on the host/client machine, database runs in another container, the container has a virtual file system where the data is stored, meaning data is not persistent. When restarting or removing the container in the host machine, data is gone and starts from a fresh state.
- **Persistent data** means data that doesn't change across time, systems, and memory, we are rendering it static.
- `link` - linking the `nodejs` to the `mongo` container so that the `mongo` service is reachable from the `nodejs` service

**nodejs**
- `version` - specifies version of docker compose we are using
- `services` - defining services `mongo` and `nodejs`
- `container_name`- giving container a memorable name to avoid randomly generated container names
- `restart` - restarting container automatically if it fails
- `build` - using the `Dockerfile` to build image in the `.` current directory
- `ports` - mapping host port to the container port
- `environment` - setting environment variables 

---

**Once you run** `docker-compose up`, **run** `docker ps` **to check that the containers are running**
```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
a1cf5c2ef7a0        app_nodejs          "docker-entrypoint.s…"   28 minutes ago      Up 27 minutes       0.0.0.0:3000->3000/tcp     dockerised-nodejs
3e9dbb0f797c        mongo               "docker-entrypoint.s…"   48 minutes ago      Up 28 minutes       0.0.0.0:27017->27017/tcp   mongo
```

**Run docker images**
```bash
docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
app_nodejs                  latest              72f3f76e2cae        31 minutes ago      471MB
```

**Testing web application on browser**
```bash
http://localhost:3000
http://localhost:3000/fibonacci/10
http://localhost:3000/posts

```

**To rename an image:**
```bash
docker rmi -f app_nodejs
docker tag 72f3f76e2cae naistangz/app_nodejs <image id> <username/repository_name>
```


**Create a repository on [Docker Hub](https://hub.docker.com/repository/docker/) and push docker image to the Docker registry** 
```bash
docker push naistangz/app_nodejs
```

**To access my image:**
```bash
docker pull naistangz/app_nodejs:tagname
```

---

### Docker Volumes
- **Volume types**:
```bash
docker run -v /home/mount/data:/var/lib/mysql/data
docker run -v /var/lib/mysql/data - not specifying where in the host the directory is mounted. Also called anonymous volumes
docker run -v name:/var/lib/mysql/data - named volumes 
```
