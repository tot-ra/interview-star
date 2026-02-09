# Инфраструктура и DevOps - Ответы на вопросы интервью

## 📚 Теория

### 1. Docker Layers

**Вопрос:** Как работают слои в Docker? Как оптимизировать Dockerfile для кеширования?

**Ответ:**

**Docker Layers:**
```
Image Layers (read-only):
┌─────────────────────────────┐  Layer 5: CMD ["node", "app.js"]
│ Final Layer                 │  Layer 4: COPY . /app
├─────────────────────────────┤  Layer 3: RUN npm install
│ Layer 4                     │  Layer 2: WORKDIR /app
├─────────────────────────────┤  Layer 1: COPY package*.json /app
│ Layer 3                     │  Layer 0: FROM node:18-alpine
├─────────────────────────────┤
│ Layer 2                     │
├─────────────────────────────┤
│ Layer 1                     │
├─────────────────────────────┤
│ Base Image (Layer 0)        │
└─────────────────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ Container Layer (read-write)│
└─────────────────────────────┘
```

**Принципы:**
- Каждая инструкция создает новый слой
- Слои кешируются и reuse между builds
- Если слой меняется — все последующие пересобираются
- Union file system: слои накладываются друг на друга

**Оптимизация Dockerfile:**

```dockerfile
# ❌ ПЛОХО — меняется при каждом изменении кода
FROM node:18
COPY . /app
RUN npm install
RUN npm run build

# ✅ ХОРОШО — dependency кешируются отдельно
FROM node:18-alpine
WORKDIR /app

# Сначала копируем только package.json (меняется реже)
COPY package*.json ./
RUN npm ci --only=production

# Затем копируем код (меняется чаще)
COPY . .
RUN npm run build

CMD ["node", "dist/app.js"]
```

**Best Practices:**

```dockerfile
# 1. Multi-stage build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
CMD ["node", "dist/app.js"]

# 2. Использовать .dockerignore
# node_modules
# .git
# *.log
# .env

# 3. Минимальный базовый образ
FROM node:18-alpine  # 180MB
# вместо
FROM node:18         # 1GB

# 4. Объединять RUN команды (меньше слоев)
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# 5. Не запускать от root
RUN useradd -m myuser
USER myuser
```

**Уменьшение размера образа:**
```bash
# Использовать dive для анализа
dive myimage:latest

# Сжатие слоев (squash)
docker build --squash -t myimage .

# Alpine или Distroless
FROM gcr.io/distroless/nodejs18-debian11
```

---

### 2. Kubernetes Concepts

**Вопрос:** Объясните Pod, Deployment, Service, Ingress, ConfigMap, Secret.

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Kubernetes Architecture                     │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                      Ingress                          │  │
│  │  (HTTP routing, SSL termination, virtual hosting)     │  │
│  └──────────────────────┬────────────────────────────────┘  │
│                         │                                    │
│  ┌──────────────────────▼────────────────────────────────┐  │
│  │                     Service                           │  │
│  │  (ClusterIP, LoadBalancer, NodePort, ExternalName)    │  │
│  │  • Stable IP for Pod group                            │  │
│  │  • Load balancing across Pods                         │  │
│  │  • Service Discovery (DNS)                            │  │
│  └──────────────────────┬────────────────────────────────┘  │
│                         │                                    │
│  ┌──────────────────────▼────────────────────────────────┐  │
│  │                   Deployment                          │  │
│  │  • Manages ReplicaSet                                  │  │
│  │  • Rolling updates                                     │  │
│  │  • Rollback                                            │  │
│  └──────────────────────┬────────────────────────────────┘  │
│                         │                                    │
│  ┌──────────────────────▼────────────────────────────────┐  │
│  │                  ReplicaSet                           │  │
│  │  • Ensures desired number of Pods                      │  │
│  │  • Self-healing                                        │  │
│  └──────────────────────┬────────────────────────────────┘  │
│                         │                                    │
│  ┌──────────────────────▼────────────────────────────────┐  │
│  │                     Pod                               │  │
│  │  • Smallest deployable unit                            │  │
│  │  • One or more containers                              │  │
│  │  • Shared network namespace                            │  │
│  │  • Ephemeral (не сохраняет состояние)                  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ConfigMap & Secret:                                         │
│  ┌─────────────────┐  ┌─────────────────┐                    │
│  │   ConfigMap     │  │     Secret      │                    │
│  │  (plain text)   │  │  (base64 encode)│                    │
│  │  • Config files │  │  • Passwords    │                    │
│  │  • Env vars     │  │  • API keys     │                    │
│  │  • Command args │  │  • Certificates │                    │
│  └─────────────────┘  └─────────────────┘                    │
└─────────────────────────────────────────────────────────────┘
```

**Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 3000
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: db_host
```

**Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:1.0
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

**Service Types:**

| Type | Описание | Use Case |
|------|----------|----------|
| **ClusterIP** | Internal IP, доступен только внутри кластера | Internal communication |
| **NodePort** | Exposes service on each node's IP at static port | Development, bypassing LB |
| **LoadBalancer** | Exposes service externally using cloud provider's LB | Production external access |
| **ExternalName** | Maps service to external DNS name | External services |

**Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
```

---

### 3. Deployment Strategies

**Вопрос:** Сравните Rolling Update, Blue-Green и Canary deployments.

**Ответ:**

**Rolling Update (default in Kubernetes):**
```
Initial:  [v1] [v1] [v1]
Step 1:   [v1] [v1] [v2]  (one pod updated)
Step 2:   [v1] [v2] [v2]  (two pods updated)
Step 3:   [v2] [v2] [v2]  (all updated)

Pros:
• No extra resources needed
• Zero downtime
• Automatic rollback on failure

Cons:
• Mixed versions during update
• Hard to test new version with production traffic
```

**Blue-Green Deployment:**
```
Before:                 After:
┌─────────────┐        ┌─────────────┐
│   Blue      │        │   Blue      │ (idle)
│   (v1)      │        │   (v1)      │
│   ACTIVE    │        │             │
└──────┬──────┘        └─────────────┘
       │               ┌─────────────┐
       │               │   Green     │ (v2)
       │               │   (v2)      │
       │               │   ACTIVE    │
       ▼               └──────┬──────┘
    Users                    Users

Pros:
• Instant switch
• Easy rollback (switch back)
• Full testing before going live

Cons:
• 2x resources needed
• Database migrations tricky
• Cold start issues
```

**Canary Deployment:**
```
Phase 1:                      Phase 2:
┌──────────┐                 ┌──────────┐
│ v1  90%  │                 │ v1  50%  │
│ v2  10%  │                 │ v2  50%  │
└────┬─────┘                 └────┬─────┘
     │                            │
  Traffic with weighting      If metrics OK, increase
                              
Phase 3:                      Phase 4:
┌──────────┐                 ┌──────────┐
│ v1  10%  │                 │ v2  100% │
│ v2  90%  │                 │ v1  0%   │
└────┬─────┘                 └────┬─────┘
     │                            │
  Almost done                  Complete!

Pros:
• Test with real traffic
• Gradual rollout
• Minimize blast radius
• Metrics-driven

Cons:
• Complex setup
• Traffic splitting logic needed
• Monitoring critical
```

**Сравнение:**

| Стратегия | Сложность | Риск | Ресурсы | Rollback |
|-----------|-----------|------|---------|----------|
| Rolling | Низкая | Средний | Норм | Slow |
| Blue-Green | Средняя | Низкий | 2x | Instant |
| Canary | Высокая | Очень низкий | 1.1-1.5x | Fast |

**Service Mesh (Istio) Canary:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-service
spec:
  hosts:
  - my-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: my-service
        subset: v2
      weight: 100
  - route:
    - destination:
        host: my-service
        subset: v1
      weight: 90
    - destination:
        host: my-service
        subset: v2
      weight: 10
```

---

### 4. AWS Services

**Вопрос:** Сравните Lambda, ECS, EKS и EC2. Когда использовать каждый?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                  AWS Compute Options                         │
│                                                              │
│  Serverless (Lambda):                                       │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Event-driven functions                                ││
│  │ • Auto-scaling, pay per invocation                      ││
│  │ • 15 min max execution                                  ││
│  │ • Cold start latency                                    ││
│  │                                                         ││
│  │ Use: Webhooks, APIs, data processing, scheduled jobs    ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Containers (ECS/Fargate):                                  │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ ECS (Elastic Container Service):                        ││
│  │   • AWS native container orchestration                  ││
│  │   • Fargate: serverless containers                      ││
│  │   • EC2: manage your own capacity                       ││
│  │                                                         ││
│  │ EKS (Elastic Kubernetes Service):                       ││
│  │   • Managed Kubernetes                                  ││
│  │   • Full K8s API compatibility                          ││
│  │   • More complex, higher cost                           ││
│  │                                                         ││
│  │ Use: Long-running services, microservices               ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  VMs (EC2):                                                 │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Full control over environment                         ││
│  │ • Manage everything (OS, patching, scaling)             ││
│  │ • Predictable costs                                     ││
│  │                                                         ││
│  │ Use: Legacy apps, specific OS requirements, databases   ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

**Сравнение:**

| Аспект | Lambda | ECS Fargate | EKS | EC2 |
|--------|--------|-------------|-----|-----|
| **Мanagement** | Fully managed | Managed containers | Managed K8s | Self-managed |
| **Scaling** | Automatic | Automatic | Auto/manual | Manual/Auto Scaling |
| **Cold start** | 100ms-10s | None | None | None |
| **Max duration** | 15 min | Unlimited | Unlimited | Unlimited |
| **Cost model** | Per invocation | Per vCPU/memory | Per cluster + EC2 | Per hour |
| **Complexity** | Low | Medium | High | Medium |
| **Portability** | AWS only | AWS only | Cloud-agnostic | Cloud-agnostic |

**High Availability в AWS:**

```
┌─────────────────────────────────────────────────────────────┐
│              Multi-AZ Architecture                           │
│                                                              │
│         Availability Zone A         Availability Zone B     │
│         ┌─────────────┐            ┌─────────────┐          │
│         │   ALB       │◄──────────►│   ALB       │          │
│         │   (Active)  │            │   (Standby) │          │
│         └──────┬──────┘            └──────┬──────┘          │
│                │                          │                 │
│         ┌──────▼──────┐            ┌──────▼──────┐          │
│         │  ECS/EKS    │            │  ECS/EKS    │          │
│         │  (AZ A)     │            │  (AZ B)     │          │
│         └──────┬──────┘            └──────┬──────┘          │
│                │                          │                 │
│         ┌──────▼──────┐            ┌──────▼──────┐          │
│         │  RDS        │◄──────────►│  RDS        │          │
│         │  (Primary)  │  sync      │  (Replica)  │          │
│         └─────────────┘            └─────────────┘          │
│                                                              │
│  Auto Scaling Group distributes across AZs                  │
│  Multi-AZ RDS for database HA                               │
│  Route53 health checks for DNS failover                     │
└─────────────────────────────────────────────────────────────┘
```

---

### 5. CI/CD Pipeline

**Вопрос:** Объясните принципы CI/CD. Какие этапы должен содержать пайплайн?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                    CI/CD Pipeline                            │
│                                                              │
│  Developer Push ──► Build ──► Test ──► Security ──► Deploy  │
│                                                              │
│  ┌──────────────┐                                            │
│  │   BUILD      │                                            │
│  │ • Compile    │                                            │
│  │ • Install    │                                            │
│  │   deps       │                                            │
│  │ • Build      │                                            │
│  │   artifacts  │                                            │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │    TEST      │                                            │
│  │ • Unit tests │                                            │
│  │ • Linting    │                                            │
│  │ • Code       │                                            │
│  │   coverage   │                                            │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │  INTEGRATION │                                            │
│  │ • Integration│                                            │
│  │   tests      │                                            │
│  │ • API tests  │                                            │
│  │ • Contract   │                                            │
│  │   tests      │                                            │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │   SECURITY   │                                            │
│  │ • SAST       │                                            │
│  │ • DAST       │                                            │
│  │ • Dependency │                                            │
│  │   scanning   │                                            │
│  │ • Secret     │                                            │
│  │   detection  │                                            │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │   PACKAGE    │                                            │
│  │ • Build      │                                            │
│  │   container  │                                            │
│  │ • Push to    │                                            │
│  │   registry   │                                            │
│  │ • Tag image  │                                            │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │    DEPLOY    │                                            │
│  │ • Staging    │                                            │
│  │ • Production │                                            │
│  │ • Canary/    │                                            │
│  │   Blue-green │                                            │
│  └──────────────┘                                            │
└─────────────────────────────────────────────────────────────┘
```

**GitHub Actions пример:**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm test -- --coverage
    
    - name: Build
      run: npm run build
    
    - name: Security audit
      run: npm audit --audit-level=moderate

  docker:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: docker build -t myapp:${{ github.sha }} .
    
    - name: Push to registry
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push myapp:${{ github.sha }}

  deploy:
    needs: docker
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - name: Deploy to staging
      run: |
        kubectl set image deployment/app app=myapp:${{ github.sha }} -n staging
    
    - name: Run smoke tests
      run: ./scripts/smoke-tests.sh staging
    
    - name: Deploy to production
      run: |
        kubectl set image deployment/app app=myapp:${{ github.sha }} -n production
```

**Deployment Patterns:**

```yaml
# GitOps с ArgoCD
# Git = single source of truth
# ArgoCD следит за Git и синхронизирует кластер

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  project: default
  source:
    repoURL: https://github.com/org/gitops-repo
    targetRevision: HEAD
    path: apps/myapp/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

### 6. Infrastructure as Code

**Вопрос:** Сравните Terraform, CloudFormation и Pulumi.

**Ответ:**

| Аспект | Terraform | CloudFormation | Pulumi |
|--------|-----------|----------------|--------|
| **Language** | HCL (declarative) | JSON/YAML | TypeScript/Python/Go |
| **Clouds** | Multi-cloud | AWS only | Multi-cloud |
| **State** | Local/S3 backend | AWS-managed | Service/backend |
| **Modularity** | Modules | Nested stacks | Packages |
| **Testing** | Limited | Limited | Unit tests |
| **Community** | Very large | AWS-focused | Growing |

**Terraform пример:**
```hcl
# providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-west-2"
  }
}

provider "aws" {
  region = var.aws_region
}

# main.tf
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  
  name = "${var.project_name}-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  single_nat_gateway = false
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"
  
  cluster_name    = "${var.project_name}-cluster"
  cluster_version = "1.28"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  eks_managed_node_groups = {
    general = {
      min_size     = 2
      max_size     = 10
      desired_size = 3
      
      instance_types = ["m6i.large"]
      
      labels = {
        workload = "general"
      }
    }
  }
}
```

**Pulumi пример:**
```typescript
import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";

// VPC
const vpc = new awsx.ec2.Vpc("custom", {
  cidrBlock: "10.0.0.0/16",
  numberOfAvailabilityZones: 3,
});

// EKS Cluster
const cluster = new awsx.classic.ecs.Cluster("cluster", {
  vpc: vpc,
});

// Lambda
const lambda = new aws.lambda.Function("myFunction", {
  runtime: aws.lambda.Runtime.NodeJS18dX,
  handler: "index.handler",
  code: new pulumi.asset.FileArchive("./app"),
  environment: {
    variables: {
      TABLE_NAME: "myTable",
    },
  },
});

// Unit testable!
describe("infrastructure", () => {
  it("should create VPC with correct CIDR", () => {
    expect(vpc.vpc.cidrBlock).to.equal("10.0.0.0/16");
  });
});
```

**Выбор:**
- **Terraform:** Multi-cloud, mature ecosystem, team choice
- **CloudFormation:** AWS-only, deep integration, no extra tooling
- **Pulumi:** Developer-friendly, imperative approach, testing

---

### 7. Container Security

**Вопрос:** Какие best practices для безопасности контейнеров?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                Container Security Layers                     │
│                                                              │
│  1. Image Security:                                         │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Use minimal base images (Alpine, Distroless)         ││
│  │ • Scan for vulnerabilities (Trivy, Clair, Snyk)        ││
│  │ • Pin image versions (digest, not tag)                 ││
│  │ • Sign images (Docker Content Trust, Cosign)           ││
│  │ • Regular base image updates                           ││
│  │ • Multi-stage builds to reduce attack surface          ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  2. Runtime Security:                                       │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Don't run as root (USER directive)                   ││
│  │ • Read-only root filesystem                            ││
│  │ • Drop capabilities (CAP_SYS_ADMIN, etc.)              ││
│  │ • Resource limits (CPU, memory)                        ││
│  │ • Security contexts (Kubernetes)                       ││
│  │ • Network policies                                     ││
│  │ • Pod Security Standards                               ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  3. Secrets Management:                                     │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Never hardcode secrets in images                     ││
│  │ • Use secrets management (Vault, AWS Secrets Manager)  ││
│  │ • Kubernetes Secrets (with encryption at rest)         ││
│  │ • Environment variables for non-sensitive config       ││
│  │ • Rotate secrets regularly                             ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  4. Supply Chain:                                           │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • SBOM (Software Bill of Materials)                    ││
│  │ • Image provenance                                     ││
│  │ • Dependency scanning                                  │
│  │ • Artifact signing                                     ││
│  │ • Private registries                                   ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

**Secure Dockerfile:**
```dockerfile
# Use specific version
FROM node:18-alpine@sha256:1234567890abcdef...

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

WORKDIR /app

# Copy and install as root, then switch
COPY --chown=nodejs:nodejs package*.json ./
RUN npm ci --only=production

COPY --chown=nodejs:nodejs . .

USER nodejs

EXPOSE 3000

# Read-only filesystem
# docker run --read-only --tmpfs /tmp myimage

CMD ["node", "server.js"]
```

**Kubernetes Security:**

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: app
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
          requests:
            memory: "256Mi"
            cpu: "250m"
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        emptyDir: {}
```

**Security Scanning:**
```bash
# Trivy scan
trivy image myapp:latest

# In CI/CD
- name: Scan image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
```

---

### 8. Monitoring & Alerting

**Вопрос:** Какие метрики важны для backend-сервиса? Как настроить SLO/SLA?

**Ответ:**

**Метрики (RED method):**

```
Rate (RPS - Requests Per Second):
┌─────────────────────────────────────────────────────────────┐
│ • Текущая нагрузка                                          │
│ • Traffic patterns                                          │
│ • Scaling triggers                                          │
└─────────────────────────────────────────────────────────────┘

Errors:
┌─────────────────────────────────────────────────────────────┐
│ • Error rate (%)                                            │
│ • Error types (4xx, 5xx)                                    │
│ • Error budget                                              │
└─────────────────────────────────────────────────────────────┘

Duration (Latency):
┌─────────────────────────────────────────────────────────────┐
│ • p50, p95, p99 response times                              │
│ • Tail latency (p99.9)                                      │
│ • Latency distribution                                      │
└─────────────────────────────────────────────────────────────┘
```

**Дополнительные метрики:**

| Категория | Метрики |
|-----------|---------|
| **System** | CPU, Memory, Disk I/O, Network |
| **Application** | GC time, Event loop lag, Active connections |
| **Database** | Query time, Connection pool, Lock waits |
| **Business** | Orders/min, Revenue, Conversion rate |

**SLO/SLI/SLA:**

```
SLI (Service Level Indicator) — что измеряем
┌─────────────────────────────────────────────────────────────┐
│ • Request latency                                           │
│ • Error rate                                                │
│ • System throughput                                         │
│ • Availability                                              │
└─────────────────────────────────────────────────────────────┘

SLO (Service Level Objective) — целевое значение
┌─────────────────────────────────────────────────────────────┐
│ • 99% of requests complete in < 200ms                       │
│ • Error rate < 0.1%                                         │
│ • Availability = 99.9% ("three nines")                      │
│ • 99% of requests get valid response                        │
└─────────────────────────────────────────────────────────────┘

SLA (Service Level Agreement) — контракт с клиентом
┌─────────────────────────────────────────────────────────────┐
│ • Penalties for missing SLOs                                │
│ • Legal commitment                                          │
│ • Usually more conservative than internal SLOs              │
└─────────────────────────────────────────────────────────────┘

Error Budget:
┌─────────────────────────────────────────────────────────────┐
│ • If SLO is 99.9% availability                              │
│ • Error budget = 0.1% downtime                              │
│ • 43.8 minutes per month                                    │
│ • Team can "spend" this on risky deployments                │
│ • If budget exhausted → freeze on non-critical changes      │
└─────────────────────────────────────────────────────────────┘
```

**Prometheus + Grafana Setup:**

```yaml
# Prometheus metrics (Node.js with prom-client)
const client = require('prom-client');

const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 2, 5]
});

app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on('finish', () => {
    end({ 
      method: req.method, 
      route: req.route?.path || 'unknown',
      status_code: res.statusCode 
    });
  });
  next();
});

// Metrics endpoint
app.get('/metrics', (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(client.register.metrics());
});
```

**Alerting Rules:**

```yaml
# prometheus-alerts.yml
groups:
- name: service_alerts
  rules:
  - alert: HighErrorRate
    expr: |
      (
        sum(rate(http_requests_total{status=~"5.."}[5m]))
        /
        sum(rate(http_requests_total[5m]))
      ) > 0.01
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: High error rate detected
      
  - alert: HighLatency
    expr: |
      histogram_quantile(0.95, 
        sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
      ) > 0.5
    for: 10m
    labels:
      severity: warning
      
  - alert: ErrorBudgetBurn
    expr: |
      (
        sum(rate(http_requests_total{status=~"5.."}[1h]))
        /
        sum(rate(http_requests_total[1h]))
      ) > (14.4 * 0.001)  # 2% burn in 1h
    labels:
      severity: critical
```

---

### 9. Secrets Management

**Вопрос:** Как безопасно хранить и использовать секреты (Vault, AWS Secrets Manager)?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                Secrets Management                            │
│                                                              │
│  Anti-patterns (НЕ делать):                                 │
│  ❌ Hardcoded in code                                       │
│  ❌ Committed to Git                                        │
│  ❌ In environment variables in Docker image                │
│  ❌ In config files in container                            │
│                                                              │
│  Best Practices:                                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ 1. External Secret Store                                ││
│  │    • HashiCorp Vault                                    ││
│  │    • AWS Secrets Manager / Parameter Store              ││
│  │    • Azure Key Vault                                    ││
│  │    • GCP Secret Manager                                 ││
│  │                                                         ││
│  │ 2. Dynamic Secrets                                      ││
│  │    • Short-lived credentials                            │
│  │    • Automatic rotation                                 │
│  │    • Audit trail                                        ││
│  │                                                         ││
│  │ 3. Kubernetes Native                                    ││
│  │    • Sealed Secrets                                     │
│  │    • External Secrets Operator                          │
│  │    • Secrets encryption at rest                         ││
│  │                                                         ││
│  │ 4. Runtime Injection                                    ││
│  │    • Init containers fetch secrets                      │
│  │    • Sidecar injection                                  │
│  │    • Volume mounts with secrets                         ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

**Vault Integration:**

```javascript
const vault = require('node-vault')({
  apiVersion: 'v1',
  endpoint: 'http://vault:8200',
});

// AppRole authentication
const roleId = process.env.VAULT_ROLE_ID;
const secretId = process.env.VAULT_SECRET_ID;

const result = await vault.approleLogin({
  role_id: roleId,
  secret_id: secretId,
});

vault.token = result.auth.client_token;

// Read dynamic database credentials
const { data } = await vault.read('database/creds/my-role');
const { username, password } = data;

// Connect to DB with these credentials
const db = await createConnection({ username, password });

// Credentials auto-revoked after TTL
```

**AWS Secrets Manager:**

```javascript
const { SecretsManager } = require('@aws-sdk/client-secrets-manager');

const client = new SecretsManager({ region: 'us-west-2' });

async function getSecret() {
  const response = await client.getSecretValue({
    SecretId: 'prod/myapp/database',
  });
  
  const secret = JSON.parse(response.SecretString);
  return secret;
}

// In Kubernetes with IRSA
// Service Account → IAM Role → Secret access
```

**Kubernetes External Secrets:**

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    kind: ClusterSecretStore
    name: aws-secrets-manager
  target:
    name: database-credentials
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: prod/database
      property: username
  - secretKey: password
    remoteRef:
      key: prod/database
      property: password
```

---

### 10. High Availability Patterns

**Вопрос:** Как спроектировать высокодоступную архитектуру в AWS?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│              High Availability Architecture                  │
│                                                              │
│  Multi-AZ + Multi-Region Setup:                             │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    Global Layer                        │  │
│  │  ┌─────────────────┐    ┌─────────────────┐           │  │
│  │  │   Route 53      │    │   CloudFront    │           │  │
│  │  │  (DNS, Geo)     │    │  (CDN/Edge)     │           │  │
│  │  └────────┬────────┘    └─────────────────┘           │  │
│  │           │                                           │  │
│  │           ▼ Health Check                              │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌─────────────────────────────┬─────────────────────────┐  │
│  │       Region: us-east-1     │    Region: us-west-2    │  │
│  │                             │                         │  │
│  │  ┌───────────────────────┐  │  ┌─────────────────────┐│  │
│  │  │       ALB             │  │  │       ALB           ││  │
│  │  │  (Cross-AZ LB)        │  │  │  (Cross-AZ LB)      ││  │
│  │  └───────────┬───────────┘  │  └──────────┬──────────┘│  │
│  │              │               │             │           │  │
│  │  ┌───────────┴───────────┐   │  ┌─────────┴──────────┐│  │
│  │  │      EKS/ECS          │   │  │     EKS/ECS        ││  │
│  │  │  (Multi-AZ)           │   │  │  (Multi-AZ)        ││  │
│  │  │                       │   │  │                    ││  │
│  │  │  ┌─────┐  ┌─────┐    │   │  │ ┌─────┐  ┌─────┐   ││  │
│  │  │  │ Pod │  │ Pod │    │   │  │ │ Pod │  │ Pod │   ││  │
│  │  │  │ AZ-A│  │ AZ-B│    │   │  │ │ AZ-A│  │ AZ-B│   ││  │
│  │  │  └─────┘  └─────┘    │   │  │ └─────┘  └─────┘   ││  │
│  │  └───────────┬───────────┘   │  └─────────┬──────────┘│  │
│  │              │                │            │           │  │
│  │  ┌───────────▼───────────┐   │  ┌─────────▼──────────┐│  │
│  │  │    RDS Multi-AZ       │   │  │    RDS Multi-AZ    ││  │
│  │  │  (Primary + Standby)  │   │  │  (Read Replica)    ││  │
│  │  └───────────────────────┘   │  └────────────────────┘│  │
│  │                              │                         │  │
│  │  Active                      │  Standby/Read           │  │
│  └──────────────────────────────┴─────────────────────────┘  │
│                                                              │
│  RPO (Recovery Point Objective):                            │
│  • Sync replication: RPO = 0                                │
│  • Async replication: RPO = seconds to minutes              │
│                                                              │
│  RTO (Recovery Time Objective):                             │
│  • Automated failover: RTO = seconds to minutes             │
│  • Manual failover: RTO = minutes to hours                  │
└─────────────────────────────────────────────────────────────┘
```

**Disaster Recovery Patterns:**

| Pattern | RTO | RPO | Cost | Description |
|---------|-----|-----|------|-------------|
| **Backup & Restore** | Hours | 24h | $ | Regular backups, restore on disaster |
| **Pilot Light** | 10-30 min | Minutes | $$ | Core always running, scale up on DR |
| **Warm Standby** | Minutes | Seconds | $$$ | Scaled-down replica, quick promotion |
| **Hot Standby** | Seconds | 0 | $$$$ | Active-Active, full cost |

**Health Checks & Auto-recovery:**
```yaml
# Kubernetes liveness/readiness
livenessProbe:
  httpGet:
    path: /health/live
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /health/ready
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

## 💻 Практика / Hands-on Tasks

### 1. Dockerfile Optimization

**Задача:** Напишите оптимальный multi-stage Dockerfile для Node.js приложения.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│              Multi-stage Dockerfile Structure                │
│                                                              │
│  Stage 1: Dependencies (cacheable layer)                    │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ FROM node:18-alpine AS deps                             ││
│  │ WORKDIR /app                                            ││
│  │ COPY package*.json ./                                   ││
│  │ RUN npm ci --only=production                            ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Stage 2: Builder (compilation, tests)                      │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ FROM node:18-alpine AS builder                          ││
│  │ WORKDIR /app                                            ││
│  │ COPY package*.json ./                                   ││
│  │ RUN npm ci                                              ││
│  │ COPY . .                                                ││
│  │ RUN npm run build                                       ││
│  │ RUN npm run test:ci                                     ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Stage 3: Production (final image)                          │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ FROM node:18-alpine AS production                       ││
│  │ WORKDIR /app                                            ││
│  │                                                         ││
│  │ # Security: create non-root user                        ││
│  │ RUN addgroup -g 1001 -S nodejs                          ││
│  │ RUN adduser -S nodejs -u 1001                           ││
│  │                                                         ││
│  │ # Copy only necessary files from previous stages        ││
│  │ COPY --from=deps /app/node_modules ./node_modules       ││
│  │ COPY --from=builder /app/dist ./dist                    ││
│  │ COPY --from=builder /app/package*.json ./               ││
│  │                                                         ││
│  │ USER nodejs                                             ││
│  │ EXPOSE 3000                                             ││
│  │ CMD ["node", "dist/main.js"]                            ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Result:                                                     │
│  • ~50MB final image (vs 1GB+ with full node)               │
│  • No dev dependencies                                      │
│  • No source code, only compiled output                     │
│  • Non-root user                                            │
└─────────────────────────────────────────────────────────────┘
```

---

### 2. Kubernetes Manifests

**Задача:** Напишите манифесты для Deployment, Service и Ingress с health checks.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│              Complete K8s Application Manifest               │
│                                                              │
│  Namespace + ConfigMap + Secret + Deployment +              │
│  Service + Ingress + HPA + PDB + NetworkPolicy              │
│                                                              │
│  Structure:                                                  │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ 1. Namespace (isolation)                                ││
│  │ 2. ConfigMap (non-sensitive config)                     ││
│  │ 3. Secret (credentials)                                 ││
│  │ 4. Deployment (app with probes, resources)              ││
│  │ 5. Service (ClusterIP)                                  ││
│  │ 6. Ingress (external access, TLS)                       ││
│  │ 7. HPA (auto-scaling)                                   ││
│  │ 8. PDB (disruption budget)                              ││
│  │ 9. NetworkPolicy (security)                             ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Key features:                                               │
│  • Rolling update strategy                                  │
│  • Liveness + Readiness probes                              │
│  • Resource requests/limits                                 │
│  • Security context                                         │
│  • Pod anti-affinity for HA                                 │
│  • Graceful shutdown (preStop hook)                         │
└─────────────────────────────────────────────────────────────┘
```

---

### 3. CI Pipeline

**Задача:** Создайте GitHub Actions/GitLab CI pipeline для тестирования, сборки и деплоя.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Complete CI/CD Pipeline                     │
│                                                              │
│  Stages:                                                    │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ 1. Lint & Test                                          ││
│  │    • Code linting (ESLint, Prettier)                    ││
│  │    • Unit tests with coverage                           ││
│  │    • Security audit                                     ││
│  │    • SonarQube analysis                                 ││
│  │                                                         ││
│  │ 2. Build                                                ││
│  │    • Compile TypeScript                                 ││
│  │    • Build Docker image                                 ││
│  │    • Push to registry                                   ││
│  │    • Tag with commit SHA                                ││
│  │                                                         ││
│  │ 3. Security Scan                                        ││
│  │    • Container image scan (Trivy)                       ││
│  │    • Dependency vulnerability scan                      ││
│  │    • SAST/DAST if applicable                            ││
│  │                                                         ││
│  │ 4. Deploy to Staging                                    ││
│  │    • Update K8s manifests                               ││
│  │    • Apply with kubectl/ArgoCD                          ││
│  │    • Run smoke tests                                    ││
│  │    • Run integration tests                              ││
│  │                                                         ││
│  │ 5. Production Deployment (manual approval)              ││
│  │    • Canary deployment (10% → 50% → 100%)               ││
│  │    • Automated rollback on error rate                   ││
│  │    • Post-deployment verification                       ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Advanced features:                                          │
│  • Caching (node_modules, Docker layers)                    │
│  • Parallel jobs where possible                             │
│  • Matrix builds (Node versions)                            │
│  • Slack notifications                                      │
│  • Artifact retention                                       │
└─────────────────────────────────────────────────────────────┘
```

---

### 4. Terraform Configuration

**Задача:** Напишите Terraform конфигурацию для создания VPC, EC2 и RDS.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│              Terraform Infrastructure Stack                  │
│                                                              │
│  Components:                                                 │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ 1. VPC                                                  ││
│  │    • 3 AZs with public/private subnets                  │
│  │    • NAT Gateways for private subnets                   │
│  │    • Internet Gateway for public                        │
│  │    • VPC Flow Logs                                      │
│  │                                                         ││
│  │ 2. Security Groups                                      │
│  │    • ALB: 80/443 from internet                          ││
│  │    • EC2: 3000 from ALB                                 ││
│  │    • RDS: 5432 from EC2 security group                  │
│  │                                                         ││
│  │ 3. EC2 Auto Scaling Group                               ││
│  │    • Launch template with user data                     │
│  │    • Target tracking scaling                            │
│  │    • Application Load Balancer                          ││
│  │                                                         ││
│  │ 4. RDS Aurora                                           ││
│  │    • Multi-AZ cluster                                   │
│  │    • Encrypted storage                                  │
│  │    • Automated backups                                  │
│  │    • Performance Insights                               │
│  │                                                         ││
│  │ 5. Supporting Resources                                 │
│  │    • CloudWatch log groups                              │
│  │    • S3 bucket for backups                              │
│  │    • Route 53 records                                   │
│  │    • ACM certificate                                    │
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Structure:                                                  │
│  • modules/vpc/        - Reusable VPC module                │
│  • modules/compute/    - EC2/ASG module                     │
│  • modules/database/   - RDS module                         │
│  • environments/prod/  - Production configuration           │
│  • environments/dev/   - Development configuration          │
└─────────────────────────────────────────────────────────────┘
```

---

### 5. Helm Chart

**Задача:** Создайте Helm chart для развертывания приложения с конфигурируемыми параметрами.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Helm Chart Structure                        │
│                                                              │
│  myapp/                                                      │
│  ├── Chart.yaml          # Chart metadata                    │
│  ├── values.yaml         # Default values                    │
│  ├── values-production.yaml  # Environment override          │
│  ├── templates/                                              │
│  │   ├── _helpers.tpl    # Named templates                   │
│  │   ├── deployment.yaml # Main app deployment               │
│  │   ├── service.yaml    # Service                          │
│  │   ├── ingress.yaml    # Ingress rules                    │
│  │   ├── configmap.yaml  # ConfigMap                        │
│  │   ├── secret.yaml     # Secrets                          │
│  │   ├── hpa.yaml        # Horizontal Pod Autoscaler        │
│  │   ├── pdb.yaml        # Pod Disruption Budget            │
│  │   └── serviceaccount.yaml # RBAC                         │
│  └── charts/             # Subcharts                         │
│                                                              │
│  Key features:                                               │
│  • Templating with Go template syntax                       │
│  • Values override per environment                          │
│  • Conditional resources (if .Values.ingress.enabled)       │
│  • Global values accessible to subcharts                    │
│  • Hooks for pre/post install                               │
│                                                              │
│  Usage:                                                      │
│  helm install myapp ./myapp -f values-production.yaml       │
│  helm upgrade --install myapp ./myapp --set image.tag=1.2   │
└─────────────────────────────────────────────────────────────┘
```
