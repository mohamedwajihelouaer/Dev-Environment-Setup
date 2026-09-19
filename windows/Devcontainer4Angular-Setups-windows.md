# Angular Projects with Dev container with vs-code
- This is a (not-IDE-agnostic)  step by step guide  to show how to use the devcontainer technology in order to keep local system clean, avoiding any multi-library issues/conflicts specially with node js and angular dev.

## Overview 

- To build angular apps (for this example we will be using angular v17). the following files will be used to provide clean environment
for angular v17 development. The following files will be used: 

| File | Purpose |
| ---- | ------- | 
| .devcontainer/Dockerfile |	Base image, now installs deps in a cached layer (BuildKit cache mount) and runs as the non-root node user |
| .devcontainer/docker-compose.yml | Named container, bind mount for live reload, named volume for node_modules |
| .devcontainer/devcontainer.json | 	Now points at Compose instead of a bare image; won't recreate the container on reopen |
| .dockerignore | 	Keeps build context small/fast |



## Build the base image 

- For all angular v17 projects we will be using the same docker image. (for any version update the Dockerfile)

```bash
docker build -t angular17-dev-base:1.0 -f .devcontainer/Dockerfile .
```

## Day-to-day commands

### first run (or after Dockerfile/deps change)

```bash 
docker compose -f .devcontainer/docker-compose.yml up -d --build
```

### subsequent runs — reuses the existing container, no rebuild/recreate

```bash 
docker compose -f .devcontainer/docker-compose.yml up -d
```

### watch ng serve output

```bash 
docker compose -f .devcontainer/docker-compose.yml logs -f
```

### stop without deleting the container

```bash 
docker compose -f .devcontainer/docker-compose.yml stop
```

## Why containers won't pile up anymore

- Compose: container_name: shopping-cart-container is fixed, so docker compose up -d always reuses/restarts that same container instead of creating a new one each run.

- VS Code Dev Containers: switched from image+runArgs to dockerComposeFile+service. I added "overrideCommand": false (so it keeps running ng serve instead of VS Code's default sleep infinity shim) and "shutdownAction": "none" (so closing the window doesn't stop/remove the container — reopening just reattaches).

- App is then live at http://localhost:4200, and editing files under src/ triggers a live rebuild in the container.

## One optional but bigger lever

- Since your project lives at a native Windows path (C:\Users\...), Docker Desktop's Linux VM still can't watch it natively — polling works but isn't free. If you ever want to eliminate that overhead entirely, cloning the repo into the WSL2 filesystem (e.g. \\wsl$\Ubuntu\home\<you>\shopping-cart / opened via code . from a WSL shell) gets you native inotify and you can drop the CHOKIDAR_* env vars. Not required — just the next performance step if you want it.

- When you add/change npm dependencies: run docker compose -f .devcontainer/docker-compose.yml up -d --build to refresh the baked-in node_modules layer.