
# Docker Best Practices

## Light-weight base images

- Using light weight base image ensure that the final image is light weight
  - using nodejs image create a 1 gb image while using a nodejs-alpine create a 350 mb image
  - using light weight image can cause compatibility issues so using it for development isn't suggested

## .dockerignore

- don't copy all the files, specify the needed files
- Use .dockerignore file to specify the unneeded files
  - using .dockerignore ensure that unneeded files won't be carried when copying directory

## Layers Caching

- Each layer in the docker file is immutable so it doesn't get recreate if no changes happens to it,
    we can benefit from this by making the unchanged layers first and setting the changed frequently layers later

### What cause caching invalidation

1. changes to the file copied
2. changes to the docker file instruction
3. changes in the previous layer

## Layer Squashing

- Using commands that create files in a layer then deleting them in the next layer doesn't reduce the image size **the previous layers are immutable** deleting the file doesn't reduce the space of the files created the files just labeled as inaccessible

## Multi-Stage Image

- in production when building application we don't actually want the project itself on the server we just want the artifact built from it and that's where the multi-stage build come in
- using multi-stage image nothing is retained from the previous stage unless explicitly specified otherwise only the last stage is retained in the image

## Example

### Base 1.21 GB

```Dockerfile
FROM node:22
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
EXPOSE 3000
CMD ["npm", "run"]
```

### Optimization

```Dockerfile
FROM node:22-alpine # use light weight image
WORKDIR /app
COPY package*.json ./ # caching the files that doesn't change frequently
RUN npm install
COPY . .
RUN npm run build && \
 npm cache clean --force && \
 rm -rf /root/.npm
# NOTE: We CANNOT delete node_modules here if we plan to use `npm run preview` 
# because Vite CLI lives in node_modules and is required to run the preview server.
EXPOSE 3000
CMD ["npm", "run", "preview"]
```

1. Installing packages BEFORE copying the rest of the source code ensures we use **layer caching** effectively.
2. After building the artifact we don't need the npm cache.
3. We delete the `.npm` cache as we don't need it for anything after the build.
4. *Limitation*: We are forced to keep `node_modules` because our runner (`npm run preview`) needs it. This is why we need **Multi-Stage Builds** to truly optimize the image.
5. *Note on `--production`*: We cannot use `npm install --production` here because building a Vite app requires `devDependencies` (like `vite`, `typescript`, etc.). We need the full `node_modules` just to build the app!

> [!note] NOTE
> All this must be done in the same command as to not leave anything in the cached layers

### Multi-Stage

- Why use multi-stage build?
    It splits the operations in different image for example we have split the image into 2 stages instead of using npm to distribute the application we just use it to build the html & scripts and after that use the nginx to serve these files.

- the only remaining files in the image is the files of the last layer the previous layers isn't maintained in the image unless referenced so even if we use the full image for the build the sized remain the same as long as the last stage image is the same.

```Dockerfile
FROM node:22-alpine AS builder
RUN npm install -g pnpm

WORKDIR /app

COPY ./vite-react-template/package*.json .
RUN npm install

COPY ./vite-react-template .
RUN npm run build

# ========================= #

FROM nginx:stable-alpine3.23

EXPOSE 80

COPY --from=builder /app/dist /usr/share/nginx/html

CMD [ "nginx", "-g", "daemon off;" ]
```

## Distroless Images

- Distroless images contain only your application and its runtime dependencies. They do not contain package managers, shells, or other utility programs you would expect to find in a standard Linux distribution.
- **Benefits:** Dramatically smaller size, improved security (reduced attack surface), and fewer false positives in vulnerability scanners.

## Security & Best Practices

- **Non-root user:** By default, Docker runs containers as root. It is a critical best practice to use the `USER` instruction to run your application as a non-root user.
- **Reduced attack surface:** Using minimal base images means fewer utilities and libraries are present, reducing the potential vectors for an attack.

## Professional Tooling

- **Dive:** A great CLI tool for exploring a Docker image, layer by layer. It helps you discover exactly what files are wasting space and what changes between layers.
- **Slim (formerly DockerSlim):** An automated tool that analyzes your application and creates a minimized, secure container by automatically stripping out unused files.
