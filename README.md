# GitOps Project

Simple GitOps-style demo repository that builds an NGINX container image and updates Kubernetes manifests automatically through GitHub Actions.

## Project Structure

.
|- Dockerfile
|- .github/workflows/docker-build-push.yml
|- k8s/
|  |- deployment.yml
|  |- service.yml
|  |- 1.txt
|- README.md

## What This Repo Does

- Builds a Docker image from `Dockerfile` when code is pushed to `main`.
- Pushes the image to Docker Hub as `mohammadkasim/gitops-app:<commit-sha>`.
- Updates `k8s/deployment.yml` image tag to the new SHA.
- Commits the manifest update back to `main` with `[skip ci]` to avoid infinite workflow loops.

## Prerequisites

- Docker
- A Kubernetes cluster (Minikube, Kind, Docker Desktop Kubernetes, or cloud cluster)
- `kubectl`
- GitHub repository secrets configured for workflow image push:
	- `DOCKER_USERNAME`
	- `DOCKER_PASSWORD`

## Local Docker Build and Run

Build image:

```bash
docker build -t gitops-app:local .
```

Run container:

```bash
docker run -d -p 8080:80 --name gitops-app gitops-app:local
```

Open in browser:

```text
http://localhost:8080
```

Stop and remove container:

```bash
docker rm -f gitops-app
```

## Kubernetes Deployment

Apply manifests:

```bash
kubectl apply -f k8s/deployment.yml
kubectl apply -f k8s/service.yml
```

Check status:

```bash
kubectl get pods
kubectl get svc gitops-service
```

## Notes About Current Manifests

- Deployment name: `gitops-app`
- Replicas: `6`
- Service type: `NodePort`
- Exposed container/service port: `80`

To change the deployed image manually, edit the `image:` value in `k8s/deployment.yml` and re-apply.

## GitHub Actions Workflow

Workflow file: `.github/workflows/docker-build-push.yml`

Trigger:

- Push to `main`

High-level pipeline:

1. Checkout repository.
2. Set `TAG` to current commit SHA.
3. Login to Docker Hub.
4. Build and push image `mohammadkasim/gitops-app:$TAG`.
5. Update Kubernetes manifest image reference.
6. Commit and push updated manifest.

## Quick Start

1. Configure Docker Hub secrets in GitHub repository settings.
2. Push any change to `main`.
3. Wait for workflow to publish image and update `k8s/deployment.yml`.
4. Apply Kubernetes manifests to deploy latest image.

## Troubleshooting

- Workflow fails at Docker login:
	- Verify `DOCKER_USERNAME` and `DOCKER_PASSWORD` secrets.
- Pods not starting:
	- Run `kubectl describe pod <pod-name>` and check image pull errors.
- Service not reachable:
	- Check NodePort mapping with `kubectl get svc gitops-service -o wide`.

## License

No license file is currently included. Add a `LICENSE` file if you plan to distribute this project.
