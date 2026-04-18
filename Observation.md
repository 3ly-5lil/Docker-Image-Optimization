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

## Using layer caching

By setting the unchanged lines first like exposing port installing needed packages makes the image build faster on future edits.

## Layer squaching [[Dockerfile-v3]]

Setting the dependent command together and deleting cache afterward can save a lot of space.

Instead of installing package then building we can do this in the same layer and deleting the installed packages afterward.

![V3 image info](v3.png)

## Multi-Stage Build [[Dockerfile-v4]]

- Why use multi-stage build?
It splits the operations in different image for example we have split the image into 2 stages instead of using npm to distribute the application we just use it to build the html & scripts and after that use the nginx to serve these files.

- the only remaining files in the image is the files of the last layer the previous layers isn't maintained in the image unless referenced so even if we use the full image for the build the sized remain the same as long as the last stage image is the same.

![V4 image info](v4.png)
![V4.1 image info](v4.1.png)
