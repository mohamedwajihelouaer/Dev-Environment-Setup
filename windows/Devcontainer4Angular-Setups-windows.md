# Angular Projects with Dev container with vs-code
- This is a (not-IDE-agnostic)  step by step guide  to show how to use the devcontainer technology in order to keep local system clean, avoiding any multi-library issues/conflicts specially with node js and angular dev.

## Build the base image

- First,  let's pick one version of angular to implement our application. In this guide we pick the angular version 17. This guide should apply to any version resulting in possible multiple version app dev on a same host.
- We build the docker image from a simple Dockerfile and keep under local registry or push it. (To keep things simple no push to any remote will be run).
- Content of Dockerfile

```dockerfile
FROM node:20-bullseye

# Install Angular CLI v17 globally, pinned to match your app version
RUN npm install -g @angular/cli@17

# Set working directory inside the container
WORKDIR /workspace
```

- Time to build with a tag (as best practice specially if we want to push ato remote registry)
```bash
docker build -t angular17-dev-base:1.0 -f Dockerfile .
```

## Load the image in a project

- Assuming i want to write an angular client using version 17, here are the steps to perform:

```bash
# step1
mkdir [root-app-folder]

# step2
mkdir .devcontainer
cd .devcontainer
touch devcontainer.json
```

- The content of the ```devcontainer.json``` file should be similar to:

```json
{
  "name": "my-application",
  "image": "angular17-dev-base:1.0",
  "workspaceFolder": "/workspace",
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind,consistency=cached",
  "forwardPorts": [4200],
  "postCreateCommand": "ng version",
  "customizations": {
    "vscode": {
      "extensions": [
        "Angular.ng-template",
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode"
      ],
      "settings": {
        "terminal.integrated.defaultProfile.linux": "bash"
      }
    }
  },
  "remoteUser": "node"
}
```

- The ```devcontainer.json``` is shown as an example, lot more customization can be added based on each project's requirements.
- The second entry wih the "image" key shows how to reference the previously built docker image
- The linux user inside the running container, as shown in the "remoteUser" entry is "node" which means, once we are in the container, we have the node username.  

 
