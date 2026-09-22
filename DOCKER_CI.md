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

## CI/CD milestone

The first workflow run failed because `Dockerfile` and `index.html` were not yet present in the repository root. This was an intentional real-world failure: CI identified a missing build dependency rather than allowing the issue to remain hidden.

After the repository structure was corrected and both files were committed, the second workflow run passed successfully. The pipeline now validates the Docker build end-to-end on every push to `main` and every pull request targeting `main`.

This workflow is a build check only. It does not publish an image or deploy to a server. Cloud deployment is planned separately as an AWS EC2 migration.
