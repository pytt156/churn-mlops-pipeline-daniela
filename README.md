# Churn Prediction – MLOps CI/CD Pipeline

An end-to-end MLOps project that automates data validation, model training, evaluation, testing, Docker image publishing, and deployment to Kubernetes using GitHub Actions.

## Tech Stack

Python, scikit-learn, FastAPI, Docker, GitHub Actions, GitHub Container Registry (GHCR), and Kubernetes.

## CI/CD Pipeline

The GitHub Actions workflow includes:

1. **Data validation** – Checks input data quality before training.
2. **Testing** – Runs automated Python tests and dependency security checks.
3. **Model training** – Trains the churn prediction model and saves model artifacts.
4. **Model evaluation** – Validates model performance before further processing.
5. **Docker testing** – Builds and smoke-tests the application containers.
6. **Image publishing** – Publishes the API Docker image to GHCR using the commit SHA as its tag.
7. **Production deployment** – Deploys the approved image to Kubernetes after manual approval.

Model integrity and provenance are verified before the model is used in later pipeline stages.

## Kubernetes Deployment

The API is deployed to the school's Kubernetes cluster using:

- **Deployment:** Two replicas with rolling updates (`maxSurge: 1`, `maxUnavailable: 0`).
- **Service:** ClusterIP, forwarding port 80 to the API on port 8000.
- **Ingress:** Traefik with HTTPS access.
- **Health checks:** Startup, readiness, and liveness probes.
- **Security:** Non-root execution and restricted container privileges.

The Deployment and Service are updated through GitHub Actions. Ingress is currently applied separately.

## Live API

**Health endpoint:**

https://daniela-algerydh.89-167-53-206.sslip.io/health

The Ingress uses a Let's Encrypt staging certificate, which may produce a browser certificate warning.

## Kubernetes Commands

Connect to the school's cluster:

```bash
export KUBECONFIG=~/.kube/mlops-ht26.yaml
kubectl config current-context
```

The expected context is `mlops-ht26`, with namespace `daniela-algerydh`.

Check the deployment:

```bash
kubectl get deployments
kubectl get pods
kubectl get ingress
kubectl describe deployment/saldes-api
```

Test the API:

```bash
curl -k -i https://daniela-algerydh.89-167-53-206.sslip.io/health
```

## Rollback

The Deployment supports manual rollback to a previous Kubernetes revision:

```bash
kubectl rollout history deployment/saldes-api
kubectl rollout undo deployment/saldes-api
kubectl rollout status deployment/saldes-api
```

The pipeline stops if the rollout fails, but does not automatically roll back.

## Future Improvements

- Pin Ubuntu runner, GitHub Actions, and dependency versions for reproducibility.
- Use Helm or Kustomize for flexible Docker image version management.
- Add automated post-deployment smoke tests and Kubernetes manifest validation.
- Introduce monitoring, alerting, and model/data drift detection.
- Improve resource scaling and evaluate automated rollback.
- Integrate Ingress and TLS management into the deployment pipeline.