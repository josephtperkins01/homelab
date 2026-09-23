# Docker Build and GitHub Actions

I added a small Dockerized Nginx site and a GitHub Actions build check so the repository can prove that the image still builds after each change.

## The image

The `Dockerfile` intentionally stays small:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

I chose the Alpine Nginx image because this lab only needs a lightweight web server for the example site.

To build and test locally:

```bash
docker build -t homelab-docker-site .
docker run --name homelab-docker-site -d -p 8080:80 homelab-docker-site
```

I can open `http://localhost:8080` to verify the container page, then clean it up:

```bash
docker stop homelab-docker-site
docker rm homelab-docker-site
```

## The CI workflow

`.github/workflows/docker-build.yml` runs on pushes to `main` and pull requests targeting `main`. It checks out the repository and runs the same build command I use locally:

```bash
docker build -t homelab-docker-site .
```

The workflow is a build check only. It does not publish an image or deploy to a server.

## The failure that proved the workflow mattered

The first workflow run failed because `Dockerfile` and `index.html` were not present at the repository root. Instead of treating that as a generic CI problem, I used the failure to identify the missing build context, added both files, committed the correction, and pushed again.

The second run passed. That gives this project a more useful CI story than a green run from the start: GitHub Actions caught a real repository mistake, and the follow-up run verified the fix end to end.

Cloud deployment is intentionally deferred to the separate AWS EC2 project.
