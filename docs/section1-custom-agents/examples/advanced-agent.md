---
name: "DevOps Assistant Agent"
description: "Agent avanzato per automazione DevOps, CI/CD e gestione infrastruttura"
version: "2.0.0"
author: "DevOps Team"
tags:
  - devops
  - ci-cd
  - automation
  - infrastructure
  - kubernetes
capabilities:
  - pipeline_analysis
  - infrastructure_as_code
  - deployment_assistance
  - monitoring_setup
  - security_scanning
tools:
  - kubectl
  - terraform
  - ansible
  - docker
integrations:
  - github-actions
  - jenkins
  - gitlab-ci
---

# DevOps Assistant Agent

## Descrizione Completa

Questo agent avanzato fornisce assistenza completa per operazioni DevOps, inclusa la gestione di pipeline CI/CD, infrastruttura as code, deployment automation e best practices di sicurezza.

## Capacità Principali

### 1. Analisi e Ottimizzazione Pipeline CI/CD

L'agent può:
- Analizzare configurazioni GitHub Actions, Jenkins, GitLab CI
- Suggerire ottimizzazioni per ridurre tempi di build
- Identificare problemi di sicurezza nelle pipeline
- Proporre strategie di caching efficaci

### 2. Infrastructure as Code (IaC)

Supporta:
- Terraform configurations
- AWS CloudFormation
- Ansible playbooks
- Kubernetes manifests

### 3. Deployment Strategies

Assiste con:
- Blue-Green deployments
- Canary releases
- Rolling updates
- Rollback procedures

### 4. Monitoring e Observability

Configura:
- Prometheus metrics
- Grafana dashboards
- Log aggregation
- Alert rules

## Configurazione Avanzata

### File di Configurazione JSON

```json
{
  "agent": {
    "name": "DevOps Assistant",
    "version": "2.0.0",
    "description": "Advanced DevOps automation agent"
  },
  "settings": {
    "temperature": 0.5,
    "max_tokens": 4000,
    "response_format": "markdown"
  },
  "tools": [
    {
      "name": "kubectl",
      "enabled": true,
      "config": {
        "default_namespace": "default",
        "context": "production"
      }
    },
    {
      "name": "terraform",
      "enabled": true,
      "config": {
        "backend": "s3",
        "state_lock": true
      }
    },
    {
      "name": "docker",
      "enabled": true,
      "config": {
        "registry": "ghcr.io",
        "auto_cleanup": true
      }
    }
  ],
  "knowledge_base": {
    "sources": [
      "knowledge/kubernetes-patterns.md",
      "knowledge/terraform-modules.md",
      "knowledge/security-policies.md",
      "knowledge/monitoring-best-practices.md"
    ],
    "update_frequency": "daily",
    "priority_topics": [
      "security",
      "performance",
      "reliability"
    ]
  },
  "integrations": {
    "github_actions": {
      "enabled": true,
      "workflows_path": ".github/workflows"
    },
    "vault": {
      "enabled": true,
      "url": "https://vault.example.com"
    }
  }
}
```

## Esempi Pratici

### Esempio 1: Ottimizzazione GitHub Actions Workflow

**Input:**
```yaml
# .github/workflows/ci.yml - PRIMA
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build
        run: npm install && npm run build
      - name: Test
        run: npm test
```

**Suggerimenti dell'Agent:**
- Aggiungi caching per node_modules
- Separa build e test in job paralleli
- Aggiungi condizioni per eseguire solo su branch specifici
- Implementa matrix strategy per test su multiple versioni

**Output Migliorato:**
```yaml
# .github/workflows/ci.yml - DOPO
name: CI
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-${{ matrix.node-version }}
          path: dist/
          
  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

### Esempio 2: Kubernetes Deployment con Best Practices

**Richiesta:** "Crea un deployment Kubernetes per un'applicazione web con alta disponibilità"

**Risposta dell'Agent:**

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: production
  labels:
    app: webapp
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
        version: v1
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - webapp
                topologyKey: kubernetes.io/hostname
      containers:
        - name: webapp
          image: myapp:v1.0.0
          ports:
            - containerPort: 8080
              name: http
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: url
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: production
spec:
  selector:
    app: webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

**Spiegazione delle Best Practices Applicate:**

1. **Alta Disponibilità:**
   - 3 replicas minime
   - Pod anti-affinity per distribuire i pod su nodi diversi
   - Rolling update strategy con zero downtime

2. **Resource Management:**
   - Resource requests e limits definiti
   - HorizontalPodAutoscaler per scalabilità automatica

3. **Health Checks:**
   - Liveness probe per riavviare container non sani
   - Readiness probe per traffic routing appropriato

4. **Sicurezza:**
   - Secrets per credenziali database
   - Namespace isolation

### Esempio 3: Terraform Module per AWS Infrastructure

**Richiesta:** "Crea un modulo Terraform per un'architettura web a 3 tier su AWS"

```hcl
# modules/three-tier-architecture/main.tf

# VPC Configuration
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

# Public Subnets
resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.environment}-public-subnet-${count.index + 1}"
    Tier = "public"
  }
}

# Application Load Balancer
resource "aws_lb" "web" {
  name               = "${var.environment}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id

  enable_deletion_protection = var.enable_deletion_protection

  tags = {
    Environment = var.environment
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "app" {
  name                = "${var.environment}-asg"
  vpc_zone_identifier = aws_subnet.private[*].id
  target_group_arns   = [aws_lb_target_group.app.arn]
  health_check_type   = "ELB"
  min_size            = var.min_instances
  max_size            = var.max_instances
  desired_capacity    = var.desired_instances

  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "${var.environment}-app-instance"
    propagate_at_launch = true
  }
}

# RDS Database
resource "aws_db_instance" "database" {
  identifier        = "${var.environment}-db"
  engine            = "postgres"
  engine_version    = "14.7"
  instance_class    = var.db_instance_class
  allocated_storage = var.db_allocated_storage
  storage_encrypted = true

  db_name  = var.db_name
  username = var.db_username
  password = var.db_password

  vpc_security_group_ids = [aws_security_group.database.id]
  db_subnet_group_name   = aws_db_subnet_group.database.name

  backup_retention_period = 7
  backup_window           = "03:00-04:00"
  maintenance_window      = "mon:04:00-mon:05:00"

  skip_final_snapshot = var.skip_final_snapshot

  tags = {
    Environment = var.environment
  }
}
```

## Workflow di Utilizzo

1. **Assessment Iniziale**: L'agent analizza la configurazione corrente
2. **Identificazione Problemi**: Rileva problemi di sicurezza, performance, reliability
3. **Suggerimenti**: Propone miglioramenti con spiegazioni dettagliate
4. **Implementazione**: Fornisce codice pronto all'uso
5. **Validazione**: Suggerisce test e verifiche

## Knowledge Base Integrata

L'agent ha accesso a:
- Pattern architetturali cloud-native
- Best practices di sicurezza DevOps
- Template Infrastructure as Code
- Procedure di troubleshooting
- Compliance e regulatory requirements

## Limitazioni e Considerazioni

- L'agent fornisce suggerimenti basati su best practices generali
- Sempre validare le configurazioni nel proprio ambiente
- Considerare i requisiti specifici di sicurezza e compliance
- Testare in ambiente non-produttivo prima del deployment

---

**Nota**: Questo è un esempio avanzato di Custom Agent. Per iniziare, vedi l'[esempio base](basic-agent.md).
