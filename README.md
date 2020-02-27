This is a sample project to demonstrate how React JS app can be build, tested and deployed into an nignx server using docker container. The last part of this doc discuss deploying docker containers to Azure cloud using Azure container registry and azure web app for containers.
The project is progressively build using different feature branches in the following order
```
react-dev-server - custom Dockerfile.dev, and volume mapping
```
```  
docker-compose -  docker-compose.yml added
```
```
test - 2 step testing strategy 
```
```
nginx - nginx in prod
```
```
master branch now has nginx branch merged into it.
```

# react-dev-server 
Switching react-dev-server branch will help see only those files that are required for this step.

The react project is created using create-recat-app 

```
npx create-react-app PROJECT_NAME
```
Now change directory into the project folder, now we will create a custom Dockerfile named Dockerfile.dev
```
FROM node:alpine
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "run", "start"]
```
To build the docker image using custom docker file
```
docker build -f Dockerfile.dev .
```
## Remove duplicate dependencies
The __node_modules__ folder created by create-react-app can be removed from the project folder

## start the container
The container can be started using the command - __docker run__, since the dev server used in create-react-app is served over port 3000, port 3000 of the conatiner has to be mapped to some port of the local machine(localhost) for it to be served over a browser.

```
docker run -it -p 3000:3000 IMAGE_ID
```

**Docker Volumes**
Hot reload of changes to the source file won't be available if we run the continer as in the above step.
Docker volumes => -v /app/node_modules => points node_modules in the container, -v $(pwd):/app maps everything in present working directory to the app folder in the container. The auto refresh of the changes in react app is a function of create-react-app, but for that to happen any changes that we make to files in local file system should propagate to the container and that is what we achieve through the volume mapping. 
```
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app IMAGE_ID 
```
## Shorthening the command with docker compose
Switch to docker-compose branch to see the files exactly like it is discribed in this section