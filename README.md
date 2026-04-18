# Docker Image Optimization Practice

This repository contains practice files and observations on optimizing Docker images, following the principles from the video [Docker Image BEST Practices - From 1.2GB to 10MB](https://www.youtube.com/watch?v=t779DVjCKCs).

The goal of this project is to take a standard Node.js (Vite/React) application and progressively optimize its Docker image size, build time, and security profile.

## Project Structure

- `vite-react-template/`: The sample React application built with Vite.
- `Dockerfile-v*`: Progressive iterations of Dockerfiles, from basic to highly optimized multi-stage builds.
- `03 Docker Best Practices.md`: Comprehensive notes on Docker best practices and advanced techniques.
- `Observation.md`: Observations, layer sizes, and lessons learned from each optimization step.

## Optimization Stages

### V1: The Baseline (`Dockerfile-v1`)

A standard, unoptimized Dockerfile using the full `node:22` base image.

- **Issues:** Extremely large image size (~1.2GB). In the initial version, it suffered from poor layer caching (re-installing packages on every source code change), which was later fixed.

### V2: Lightweight Base Image (`Dockerfile-v2`)

Switched to a lightweight base image (`node:22-alpine`).

- **Improvements:** Drastically reduced base image size (down to ~350MB).
- **Notes:** Alpine Linux strips out many underlying utilities. Depending on your needs, you might have to manually install certain global packages (like `pnpm`).

### V3: Layer Squashing & Caching (`Dockerfile-v3`)

Introduced proper layer caching by copying `package*.json` and running `npm install` *before* copying the rest of the source code. Combined commands and cleaned up caches in a single `RUN` instruction.

- **Improvements:** Faster subsequent builds due to cached dependencies. Smaller layer sizes by removing the `.npm` cache in the same layer.
- **Limitations:** We cannot delete `node_modules` here because the command (`npm run preview`) requires the Vite CLI to serve the application. The image is still heavier than it needs to be.

### V4: Multi-Stage Builds (`Dockerfile-v4` & `Dockerfile-v4.1`)

The ultimate optimization. Uses a `builder` stage to install dependencies and compile the static assets, then copies *only* the compiled assets (`dist/`) into a lightweight `nginx:alpine` image.

- **Improvements:** Tiny final image size (often < 50MB). Contains no source code, package managers, or `node_modules` in the production image. Drastically reduced attack surface and improved security.

## Key Takeaways

1. **Leverage Layer Caching:** Order commands from least frequently changed to most frequently changed. Always copy `package.json` and install dependencies *before* copying source code. This ensures `npm install` is cached unless dependencies actually change.
2. **Use Lightweight Base Images:** Prefer `alpine` or `distroless` images over full OS images (like Ubuntu/Debian) to reduce size, bandwidth, and attack surface.
3. **Clean Up in the Same Layer:** When installing tools or packages, clean up the cache (`npm cache clean --force`, `rm -rf /var/cache/apk/*`) in the *same* `RUN` instruction (using `&& \`) to prevent the cache from being permanently stored in an immutable layer.
4. **Multi-Stage Builds are Essential:** For frontend frameworks (React, Vue, etc.) or compiled languages (Go, Rust), always use multi-stage builds. It strictly separates the heavy build environment from the minimal runtime environment.

## Resources

- [Docker Image BEST Practices - From 1.2GB to 10MB (YouTube)](https://www.youtube.com/watch?v=t779DVjCKCs)
