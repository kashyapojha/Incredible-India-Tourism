# 🇮🇳 Incredible India — Flask Tourism Web App

A production-ready Flask web application for exploring Indian state tourism, with a full DevOps stack.

---

## Project Structure

```
flask_tourism/
├── app.py                          # Flask application
├── requirements.txt
├── Dockerfile                      # Multi-stage production build
├── docker-compose.yml              # Local dev with Nginx
├── nginx.conf                      # Nginx reverse proxy config
├── sonar-project.properties        # SonarQube config
├── .dockerignore
├── templates/
│   ├── base.html                   # Shared layout + nav
│   ├── login.html
│   ├── signup.html
│   ├── home.html                   # State explorer grid
│   ├── state.html                  # State detail with tabs
│   ├── quiz.html                   # Quiz question page
│   ├── quiz_result.html
│   └── profile.html
├── data/                           # Auto-created at runtime
│   ├── users.json                  # Hashed credentials
│   └── userdata.json               # Favourites, visited, scores
├── k8s/
│   └── manifests.yaml              # Namespace, Deployment, Service, Ingress, HPA
├── terraform/
│   └── main.tf                     # AWS EKS + VPC + ECR
└── .github/
    └── workflows/
        └── ci-cd.yml               # 9-job CI/CD pipeline
```

---

## Quick Start — Local Development

```bash
# 1. Clone and install
git clone <your-repo>
cd flask_tourism
pip install -r requirements.txt

# 2. Run locally
python app.py
# Open http://localhost:5000
```

## Docker (Recommended)

```bash
# Build and run with Docker Compose (includes Nginx)
docker compose up --build

# App at http://localhost:80
# Direct Flask at http://localhost:5000
```

---

## CI/CD Pipeline (9 Jobs)

| # | Job | Trigger | What it does |
|---|-----|---------|-------------|
| 1 | 🔍 Lint | Every push/PR | `black`, `isort`, `flake8` |
| 2 | 🧪 Tests | After lint | `pytest` + coverage report |
| 3 | 🔒 Security | After lint | `bandit` + `safety` scan |
| 4 | 📊 SonarQube | After tests | Code quality gate |
| 5 | 🐳 Docker | After jobs 2,3,4 | Build & push to registry |
| 6 | 🏗️ Terraform | After Docker | Plan always; Apply on main |
| 7 | 🚀 Deploy | After Terraform (main only) | Rolling deploy to EKS |
| 8 | 📦 Release | After Deploy | GitHub Release with image digest |
| 9 | 📣 Notify | On any failure | Slack alert |

### Required GitHub Secrets

```
DOCKER_REGISTRY        # e.g. 123456789.dkr.ecr.ap-south-1.amazonaws.com
DOCKER_USERNAME        # Registry username
DOCKER_PASSWORD        # Registry password / token
AWS_ACCESS_KEY_ID      # IAM key with EKS + ECR + VPC permissions
AWS_SECRET_ACCESS_KEY  # IAM secret
SONAR_TOKEN            # SonarQube project token
SONAR_HOST_URL         # e.g. https://sonarqube.yourcompany.com
SLACK_WEBHOOK_URL      # Slack incoming webhook (optional)
```

---

## Kubernetes Deployment

```bash
# Point kubectl at your EKS cluster
aws eks update-kubeconfig --name india-tourism-prod-eks --region ap-south-1

# Apply all manifests
kubectl apply -f k8s/manifests.yaml

# Watch rollout
kubectl rollout status deployment/india-tourism-app -n india-tourism

# Check pods
kubectl get pods -n india-tourism

# View HPA (autoscaler)
kubectl get hpa -n india-tourism
```

### What gets deployed

- **Namespace** — `india-tourism`
- **Deployment** — 3 replicas, rolling update (zero downtime)
- **Service** — ClusterIP on port 80
- **Ingress** — HTTPS via cert-manager + Let's Encrypt
- **HPA** — Autoscale 2→10 pods based on CPU (70%) and Memory (80%)
- **PVC** — 1Gi persistent volume for user data
- **ConfigMap + Secret** — env vars injected at runtime

---

## Terraform — AWS Infrastructure

```bash
cd terraform

# First time setup
terraform init

# Preview changes
terraform plan

# Apply to AWS
terraform apply

# Get kubeconfig command
terraform output kubeconfig_cmd
```

**What Terraform provisions:**
- VPC with public + private subnets across 3 AZs
- EKS cluster (Kubernetes 1.29) with managed node group
- ECR repository for Docker images (with lifecycle policy)
- NAT Gateways (one per AZ for HA)
- IAM roles with IRSA for pods
- Security groups for ALB

---

## SonarQube Setup

1. Install SonarQube (Docker): `docker run -d -p 9000:9000 sonarqube:community`
2. Create project → get token
3. Add `SONAR_TOKEN` and `SONAR_HOST_URL` to GitHub Secrets
4. Pipeline runs analysis on every push; Quality Gate blocks merge on failure

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check — used by K8s probes |
| GET | `/home` | State explorer grid |
| GET | `/state/<name>` | State detail page |
| POST | `/api/favourite` | Toggle favourite (JSON) |
| POST | `/api/visited` | Toggle visited (JSON) |
| GET | `/api/random` | Random state (JSON) |
| GET | `/quiz` | Start quiz |
| POST | `/quiz/answer` | Submit answer |
| GET | `/profile` | User profile |

---

## Features

- ✅ Login / Signup with SHA-256 password hashing
- ✅ Guest mode (no account required)
- ✅ 25 states with Overview, Food, Places, Best Time, Fun Facts
- ✅ Favourites & Visited tracking (persisted per user)
- ✅ India Quiz (15 questions, scored + saved to profile)
- ✅ Random state discovery
- ✅ Responsive design with Playfair Display + DM Sans
- ✅ REST API with JSON responses
- ✅ `/health` endpoint for K8s liveness/readiness probes
- ✅ Multi-stage Docker build (builder → production)
- ✅ Non-root container user for security
- ✅ Gunicorn WSGI server (4 workers, 2 threads)