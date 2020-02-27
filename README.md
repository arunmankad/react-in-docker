This is a sample project to demonstrate how React JS app can be build, tested and deployed into an nignx server using docker container. The last part of this doc discuss deploying docker containers to Azure cloud using Azure container registry and azure web app for containers.
The project is progressively build using different feature branches in the following order
__react-dev-server__
__docker-compose__
__test__
__nginx__
__master__ branch now has nginx branch merged into it.

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
Docker volumes => -v /app/node_modules => points node_modules in the container, -v $(pwd):/app maps everything in present working directory to the app folder in the container 
```
docker run -p 30:3000 -v /app/node_modules -v $(pwd):/app IMAGE_ID d0b459a49b68
```

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.<br />
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.<br />
You will also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.<br />
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.<br />
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.<br />
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can’t go back!**

If you aren’t satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you’re on your own.

You don’t have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn’t feel obligated to use this feature. However we understand that this tool wouldn’t be useful if you couldn’t customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: https://facebook.github.io/create-react-app/docs/code-splitting

### Analyzing the Bundle Size

This section has moved here: https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size

### Making a Progressive Web App

This section has moved here: https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app

### Advanced Configuration

This section has moved here: https://facebook.github.io/create-react-app/docs/advanced-configuration

### Deployment

This section has moved here: https://facebook.github.io/create-react-app/docs/deployment

### `npm run build` fails to minify

This section has moved here: https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify
