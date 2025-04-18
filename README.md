# Flask CI/CD Demo 🚀

This project demonstrates a complete DevOps CI/CD pipeline using:
- Python Flask application
- Docker for containerization
- GitHub Actions for CI/CD automation
- Kubernetes (via K3d) for deployment

---

## 🧱 Flask Application

A minimal Flask app that returns a string on `/`:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from CI/CD!"
```

Unit test (`test_app.py`) ensures endpoint responds with 200 OK and expected content.

---

## 🐳 Docker Setup

The app is containerized using this Dockerfile:

```dockerfile
FROM python:3.13-alpine

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
CMD ["python", "app.py"]
```

Image is built and pushed via GitHub Actions to DockerHub with versioned tags using `GITHUB_RUN_NUMBER`.

---

## ⚙️ GitHub Actions CI/CD Pipeline

### 📌 Workflow Overview

Located in `.github/workflows/ci-cd.yml`

- **Trigger:** On push or pull request to `main`
- **Jobs:**
  - **Build & Test (GitHub-hosted runner):**
    - Installs dependencies
    - Runs unit tests using `pytest`
    - Builds and pushes Docker image to DockerHub
  - **Deploy (self-hosted runner):**
    - Runs locally on developer machine (connected to K3d)
    - Replaces image tag placeholder (`__VERSION__`) in `deployment.yaml`
    - Applies manifests via `kubectl`

### 📌 Docker Image Tagging

Each image is tagged dynamically using:
```yaml
${{ github.run_number }}
```

This ensures every deployment has a unique, traceable version.

---

## ☘️ Kubernetes (K3d) Deployment

The app is deployed to a local Kubernetes cluster using [K3d](https://k3d.io/), which runs lightweight K3s clusters inside Docker.

### ✅ Kubernetes Files:

#### `k8s/deployment.yaml`
```yaml
image: usf277/flask-ci-cd-demo:__VERSION__
```

GitHub Actions replaces `__VERSION__` with `${{ github.run_number }}` using `sed` before applying.

#### `k8s/service.yaml`
Exposes the app on port 80:
```yaml
type: LoadBalancer
targetPort: 5000
```

### 🔁 Port-forward for Local Testing
```bash
kubectl port-forward service/flask-service 8080:80
# Access at http://localhost:8080
```

---

## 🛡️ Secrets Used

In the GitHub repo settings → Secrets:

| Name               | Purpose                      |
|--------------------|------------------------------|
| `DOCKER_USERNAME`  | DockerHub login              |
| `DOCKER_PASSWORD`  | DockerHub password/token     |
| `KUBECONFIG_CONTENT` | Content of ~/.kube/config for self-hosted runner |

---

## 💡 Key Learnings

- Implemented real-world CI/CD from scratch
- Used GitHub Actions for version-aware build/test/deploy
- Combined Docker, Kubernetes, and local runners effectively
- Ensured traceability via dynamic tagging (`GITHUB_RUN_NUMBER`)

---

## 👨‍💻 Author

[Youssef Safwat](https://github.com/Usf277)
