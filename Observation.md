# Observations

## V1: normal base and normal build [[Dockerfile-v1]]

image info: ![V1 image info](v1.png)

## V2: light weight image [[Dockerfile-v2]]

image info: ![V2 image info](v2.png)

- we need to note that the light-weight image is stripped of several component for exampl I needed to install pnpm manually to run the vite server

## Adding .dockerignore [[.dockerignore]]

Adding .dockerignore to not copy all the files needed funnily enough it increased the image size I have added the .gitignore content fully in the .dockerignore.

![V1 with .dockerignore](v1-dockerignore.png)
![V2 with .dockerignore](v2-dockerignore.png)
``
