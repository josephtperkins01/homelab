# Docker Build and GitHub Actions

This repository includes a minimal Dockerized Nginx site and a GitHub Actions build check.

## Docker image

`Dockerfile` uses `nginx:alpine` as the base image and copies `index.html` into Nginx's default document root.

Build and run locally:

```bash
docker build -t homelab-docker-site .
docker run --name homelab-docker-site -d -p 8080:80 homelab-docker-site
```

Open `http://localhost:8080` to verify the containerized page. Stop and remove it when finished:

```bash
docker stop homelab-docker-site
docker rm homelab-docker-site
```

## Continuous integration

`.github/workflows/docker-build.yml` runs on pushes to `main` and pull requests targeting `main`. It checks out the repository and builds the Docker image with the same command used locally:

```bash
docker build -t homelab-docker-site .
```

A successful workflow confirms that the Dockerfile and build context can produce an image. It does not publish the image to a registry or deploy it.
