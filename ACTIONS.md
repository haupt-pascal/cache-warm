# 🔄 GitHub Actions Workflow Konzept

## 📋 Übersicht

Dieses Dokument beschreibt die CI/CD-Strategie für den Cache Warming Service unter Verwendung von GitHub Actions und Kubernetes. Die Workflows sind für eine Monorepo-Struktur optimiert und ermöglichen selektive Builds und Deployments basierend auf geänderten Dateien.

## 🏗️ Workflow-Architektur

```
.github/
└── workflows/
    ├── api-build-deploy.yml       # API Build & Deployment
    ├── worker-build-deploy.yml    # Worker Build & Deployment
    ├── dashboard-build-deploy.yml # Dashboard Build & Deployment
    ├── integration-tests.yml      # Integration Tests
    ├── code-quality.yml           # Linting & Type Checking
    └── release.yml                # Release-Prozess
```

## 🛠️ Workflow-Definitionen

### 1. API Build & Deployment

```yaml
# .github/workflows/api-build-deploy.yml
name: API Build & Deploy

on:
  push:
    branches: [main, develop]
    paths:
      - 'packages/api/**'
      - 'packages/shared/**'
      - 'k8s/api/**'
      - 'package.json'
      - 'yarn.lock'
  pull_request:
    paths:
      - 'packages/api/**'
      - 'packages/shared/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'yarn'
      
      - name: Install dependencies
        run: yarn install --frozen-lockfile
      
      - name: Lint
        run: yarn workspace @cache-warmer/api lint
      
      - name: Test
        run: yarn workspace @cache-warmer/api test
      
      - name: Build
        run: yarn workspace @cache-warmer/api build
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: ./packages/api
          push: ${{ github.event_name != 'pull_request' }}
          tags: ghcr.io/${{ github.repository }}/api:${{ github.sha }},ghcr.io/${{ github.repository }}/api:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    if: github.event_name != 'pull_request' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up kubeconfig
        uses: azure/k8s-set-context@v3
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
      
      - name: Update Kubernetes manifests
        run: |
          cd k8s/api
          kustomize edit set image ghcr.io/${{ github.repository }}/api:${{ github.sha }}
      
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -k k8s/api
          kubectl rollout status deployment/cache-warmer-api
```

### 2. Worker Build & Deployment

```yaml
# .github/workflows/worker-build-deploy.yml
name: Worker Build & Deploy

on:
  push:
    branches: [main, develop]
    paths:
      - 'packages/worker/**'
      - 'packages/shared/**'
      - 'k8s/worker/**'
      - 'package.json'
      - 'yarn.lock'
  pull_request:
    paths:
      - 'packages/worker/**'
      - 'packages/shared/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Bun
        uses: oven-sh/setup-bun@v1
        with:
          bun-version: latest
      
      - name: Install dependencies
        run: bun install
      
      - name: Lint
        run: bun run --cwd packages/worker lint
      
      - name: Test
        run: bun run --cwd packages/worker test
      
      - name: Build
        run: bun run --cwd packages/worker build
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: ./packages/worker
          push: ${{ github.event_name != 'pull_request' }}
          tags: ghcr.io/${{ github.repository }}/worker:${{ github.sha }},ghcr.io/${{ github.repository }}/worker:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    if: github.event_name != 'pull_request' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up kubeconfig
        uses: azure/k8s-set-context@v3
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
      
      - name: Update Kubernetes manifests
        run: |
          cd k8s/worker
          kustomize edit set image ghcr.io/${{ github.repository }}/worker:${{ github.sha }}
      
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -k k8s/worker
          kubectl rollout status deployment/cache-warmer-worker
```

### 3. Dashboard Build & Deployment

```yaml
# .github/workflows/dashboard-build-deploy.yml
name: Dashboard Build & Deploy

on:
  push:
    branches: [main, develop]
    paths:
      - 'packages/dashboard/**'
      - 'packages/shared/**'
      - 'k8s/dashboard/**'
      - 'package.json'
      - 'yarn.lock'
  pull_request:
    paths:
      - 'packages/dashboard/**'
      - 'packages/shared/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'yarn'
      
      - name: Install dependencies
        run: yarn install --frozen-lockfile
      
      - name: Lint
        run: yarn workspace @cache-warmer/dashboard lint
      
      - name: Test
        run: yarn workspace @cache-warmer/dashboard test
      
      - name: Build
        run: yarn workspace @cache-warmer/dashboard build
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: ./packages/dashboard
          push: ${{ github.event_name != 'pull_request' }}
          tags: ghcr.io/${{ github.repository }}/dashboard:${{ github.sha }},ghcr.io/${{ github.repository }}/dashboard:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    if: github.event_name != 'pull_request' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up kubeconfig
        uses: azure/k8s-set-context@v3
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
      
      - name: Update Kubernetes manifests
        run: |
          cd k8s/dashboard
          kustomize edit set image ghcr.io/${{ github.repository }}/dashboard:${{ github.sha }}
      
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -k k8s/dashboard
          kubectl rollout status deployment/cache-warmer-dashboard
```

### 4. Integration Tests

```yaml
# .github/workflows/integration-tests.yml
name: Integration Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_USER: postgres
          POSTGRES_DB: cache_warmer_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'yarn'
      
      - name: Install dependencies
        run: yarn install --frozen-lockfile
      
      - name: Setup integration environment
        run: |
          docker-compose -f docker-compose.test.yml up -d
          sleep 10  # Allow services to start
          
      - name: Run integration tests
        run: yarn test:integration
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/cache_warmer_test
          REDIS_URL: redis://localhost:6379
```

### 5. Code Quality

```yaml
# .github/workflows/code-quality.yml
name: Code Quality

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'yarn'
      
      - name: Install dependencies
        run: yarn install --frozen-lockfile
      
      - name: Lint
        run: yarn lint
  
  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'yarn'
      
      - name: Install dependencies
        run: yarn install --frozen-lockfile
      
      - name: Type check
        run: yarn type-check
```

### 6. Release Process

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build-all:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'yarn'
      
      - name: Install dependencies
        run: yarn install --frozen-lockfile
      
      - name: Build all packages
        run: yarn build
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Get version from tag
        id: get_version
        run: echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_ENV
      
      - name: Build and push API image
        uses: docker/build-push-action@v4
        with:
          context: ./packages/api
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/api:${{ env.VERSION }}
            ghcr.io/${{ github.repository }}/api:latest
      
      - name: Build and push Worker image
        uses: docker/build-push-action@v4
        with:
          context: ./packages/worker
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/worker:${{ env.VERSION }}
            ghcr.io/${{ github.repository }}/worker:latest
      
      - name: Build and push Dashboard image
        uses: docker/build-push-action@v4
        with:
          context: ./packages/dashboard
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/dashboard:${{ env.VERSION }}
            ghcr.io/${{ github.repository }}/dashboard:latest
      
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          name: Release ${{ env.VERSION }}
          draft: false
          prerelease: false
          generate_release_notes: true

  deploy-production:
    needs: build-all
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Get version from tag
        id: get_version
        run: echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_ENV
      
      - name: Set up kubeconfig
        uses: azure/k8s-set-context@v3
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG_PRODUCTION }}
      
      - name: Update Kubernetes manifests
        run: |
          cd k8s/overlays/production
          kustomize edit set image ghcr.io/${{ github.repository }}/api:${{ env.VERSION }}
          kustomize edit set image ghcr.io/${{ github.repository }}/worker:${{ env.VERSION }}
          kustomize edit set image ghcr.io/${{ github.repository }}/dashboard:${{ env.VERSION }}
      
      - name: Deploy to Production
        run: |
          kubectl apply -k k8s/overlays/production
          kubectl rollout status deployment/cache-warmer-api
          kubectl rollout status deployment/cache-warmer-worker
          kubectl rollout status deployment/cache-warmer-dashboard
```

## 🔄 Workflow-Trigger-Matrix

| Workflow               | File Changes                       | Branches        | Triggers                  |
|------------------------|-----------------------------------|-----------------|---------------------------|
| API Build & Deploy     | packages/api/**, packages/shared/**| main, develop   | Push, PR                  |
| Worker Build & Deploy  | packages/worker/**, packages/shared/** | main, develop | Push, PR                |
| Dashboard Build & Deploy | packages/dashboard/**, packages/shared/** | main, develop | Push, PR          |
| Integration Tests      | Any                               | main, develop   | Push, PR                  |
| Code Quality           | Any                               | main, develop   | Push, PR                  |
| Release                | Tag (v*)                          | any             | Tag Push                  |

## 💡 Best Practices

### 1. Secrets Management

- Verwende GitHub Secrets für sensitive Informationen
- Nutze OIDC für cloud-native Authentifizierung
- Betrachte die Integration mit einem externen Secret-Management-System

```yaml
# Beispiel für OIDC mit AWS
jobs:
  deploy:
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::123456789012:role/my-github-actions-role
          aws-region: eu-central-1
```

### 2. Caching-Strategien

- Cache Dependencies zwischen Workflow-Runs
- Cache Docker Build-Layers
- Cache Build-Artifacts für schnellere Deployments

```yaml
# Beispiel für Node.js Cache
steps:
  - uses: actions/setup-node@v3
    with:
      node-version: '18'
      cache: 'yarn'
```

### 3. Matrix-Builds für Tests

- Führe Tests in verschiedenen Umgebungen parallel aus
- Beschleunige die Test-Ausführung durch Parallelisierung

```yaml
# Beispiel für Matrix-Build
jobs:
  test:
    strategy:
      matrix:
        node-version: [16, 18, 20]
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
```

### 4. Auto-Scaling Worker Deployment

Automatisches Skalieren der Worker basierend auf der Queue-Länge:

```yaml
# Beispiel für HPA-Konfiguration in k8s/worker/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cache-worker-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cache-worker
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
```

## 🚀 Deployment-Strategien

### Umgebungs-Strategie

```
k8s/
├── base/                # Basis-Konfiguration
├── overlays/
│   ├── development/     # Entwicklungsumgebung
│   ├── staging/         # Testumgebung
│   └── production/      # Produktionsumgebung
```

### Branch-zu-Umgebungs-Mapping

| Branch   | Environment  | Deployment             |
|----------|-------------|------------------------|
| Feature  | -           | PR Preview (optional)  |
| develop  | Development | Automatisch nach Push  |
| release/* | Staging    | Automatisch nach Push  |
| main     | Production  | Automatisch nach Push  |
| v* Tags  | Production  | Automatisch nach Tag   |

### Rollback-Strategie

Bei einem fehlgeschlagenen Deployment:

1. Automatischer Rollback basierend auf Health-Checks
2. Manueller Rollback durch Redeploy eines früheren Tags
3. Disaster Recovery durch Infrastructure as Code

## 📝 Abschließende Gedanken

Diese GitHub Actions-Workflows bieten eine vollständige CI/CD-Pipeline für das Cache Warming Service Monorepo. Die selektiven Builds und Deployments stellen sicher, dass nur die geänderten Komponenten neu gebaut und deployed werden, was Zeit und Ressourcen spart.

Die Integration mit Kubernetes und die Verwendung von Kustomize ermöglichen flexible Deployments in verschiedenen Umgebungen. Die Workflows sind so konzipiert, dass sie skalierbar und wartbar sind, während sie gleichzeitig Best Practices für CI/CD folgen.
