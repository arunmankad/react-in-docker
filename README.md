This is a sample project to demonstrate how React JS app can be build, tested and deployed into an nignx server using docker container. The last part of this doc discuss deploying docker containers to Azure cloud using Azure container registry and azure web app for containers.
The project is progressively build using different feature branches in the following order
```
react-dev-server - custom Dockerfile.dev, and volume mapping
```
```  
docker-compose -  docker-compose.yml added, it will help get rid of lengthy docker run command 
=> docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app IMAGE_ID. 
Instead we call build the image and run the container using "docker-compose up --build",
if alreday build image needs to run use "docker-compose up" 
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
```
git checkout -b react-dev-server origin/react-dev-server
Branch 'react-dev-server' set up to track remote branch 'react-dev-server' from 'origin'.
```

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

```
git checkout -b docker-compose origin/docker-compose
Branch 'docker-compose' set up to track remote branch 'docker-compose' from 'origin'.
```
docker-compose.yml file is created will help avoid lengthy docker run command, with volume mapping and port ,mapping  => docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app IMAGE_ID. 
Instead we call build the image and run the container using "docker-compose up --build",
if image is already built, we needs to run use "docker-compose up" &nbsp;

docker-compose.yml
```
version: '3'
services: 
  react-app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports: 
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```

```
docker-compose up --build
```
```
docker-compose up
```
```
docker-compose down
```
## Unit testing react app
No need to switch branch, discuss first few cases with docker-compose branch itself 
**Unit testing in interactive terminal**
To run test in interactive mode, build the image using Dockerfile.dev and run it by replacing the command in dockerfile

```
docker build -f Dockerfile.dev .
```
```
docker run -it IMAGE_ID npm run test
```
**Live updates in test**
With the above approach live updates on test cases won't get reflected in execution

*Approach 1*

Here we create a single container with docker-compose.yml file and then will get our terminal attached to it by using "docker exec" command 

```
docker-compose up
```
open another terminal and get container id using docker ps command
```
docker exec -it CONTAINER_ID npm run test 
```
Here we will have live updates of the test cases and interactive terminal for testing 

*Approach 2*

Switch to test branch to see the files exactly like it is discribed in this section

```
git checkout -b test origin/test
Branch 'test' set up to track remote branch 'test' from 'origin'.
```
as part of this approach we will add a second service into the docker-compose file, solely responsible for running our test suites.

```
version: '3'
services: 
  react-app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports: 
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
  react-test:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]
```
the second service in decoker compose will give us capability of running the test suite with hot loading of test cases(along with app running in dev mode), but even with docker exec we wn't get the interactive terminal which is a down side to this aaproach.

## Build for prod 
Switch to nginx branch to see the files exactly like it is discribed in this section

```
git checkout -b nginx origin/nginx
Branch 'nginx' set up to track remote branch 'nginx' from 'origin'.
```
Till now, while we were running our app in dev mode, we were relying on the dev server provided as part of create-react-app to serve our app. In prod mode we need another server to serve our app, which is going to be NGINX!.

We will create another Dockerfile meant for prod builds, this time name of the file is going to be the familiar 'Dockerfile' so that while building the image we don't have to explicilty provide the name of the file.

Dockerfile
```
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx
COPY --from=builder /app/build /usr/share/nginx/html
```
This will build the react app using alpine version of node and then copy the content of build folder into nginx server. 

The image created from the docker file has to be run using the below command, default port for nginx is 80 

```
docker run -it -p 8080:80 IMAGE_ID 
```

# **Deploying the containerized react app into Azure** 

**Steps** 
Create Azure container registry
build image in local using Dockefile (taging the image to ACR registry name will help)
login to docker-ACR eg docker login arunm1.azurecr.io
Will prompt for credentials => get it from Access Keys section in Azure CR
Once logged in, push the local image into ACR eg docker push <Image tag>
Create a web app for container in Azure, in its container settings section choose ACR as Image source, then select the registry and image. Switch on Continous deployment option, this will create a webhook and publish it in the resource group.
With the webhook published, every new push of image into ACR will trigger a new deployment into the container. 

