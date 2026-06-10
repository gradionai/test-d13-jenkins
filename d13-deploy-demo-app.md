# Deploying a Demo App with D13

## How it works

D13 doesn't need the app code to live in this repo. You define your app as a **private service** in `.d13.yaml`, and D13 handles:

1. Building Docker image -> pushing to AWS ECR
2. Generating Helm values -> deploying to Kubernetes

## What you need to change

### 1. Add your app to `.d13.yaml` under `services.private`

```yaml
services:
  private:
    - name: my-demo-app
      enable: true
      d13Version: 1.0.0
      branch:
        - develop
      gitSecret: false
      dockerBuild:
        dockerFile: Dockerfile
      details:
        hostname:
          - host: my-demo-app.devops.nfq.asia
            paths:
              - /
        port: 8080
        replicas: 1
        resources:
          requests:
            cpu: 50m
            memory: 50Mi
          limits:
            cpu: 100m
            memory: 256Mi
```

### 2. Your app repo needs a Dockerfile

### 3. The Jenkinsfile does NOT need changes

It already calls `d13Build()` and `d13Deployment()` which read from `.d13.yaml`.

## If your app lives in a separate GitHub repo

D13's build process (`d13Build`) builds Docker images from **this repo's context** -- it runs `docker build` using the Dockerfile path relative to the project root. If your app code lives in a separate GitHub repo, you have two options:

- **Option A**: Add your app as a git submodule or copy it into this repo.
- **Option B**: Pre-build your Docker image in your own repo's CI, push it to ECR, and configure D13 to just deploy (skip the build step).

## Key configuration files

| File | Purpose |
|------|---------|
| `.d13.yaml` | Master service configuration |
| `.d13.develop.yaml` | Environment overrides (develop) |
| `.d13.production.yaml` | Environment overrides (production) |
| `app/configs/configs.yaml` | D13 system config with Helm charts & field mappings |
| `app/templates/private/1.0.0.yaml.tpl` | Helm template for private services |
| `Jenkinsfile` | Jenkins pipeline definition |

## Available service configuration options

### Autoscaling

```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
```

### Health checks

```yaml
readinessProbe:
  httpGet:
    path: /healthz
    port: 8080
```

### Persistent volumes

```yaml
persistentVolume:
  enabled: true
  size: 2Gi
  mountPath: /data
```

### Cronjobs

```yaml
cronjobs:
  - name: backup-job
    schedule: "0 3 * * *"
    command: "cd /var/www && php artisan backup"
    resources:
      limits:
        cpu: 200m
        memory: 256Mi
```

### Workers

```yaml
workers:
  - name: queue-worker
    command: "cd /var/www && php artisan queue:work"
    resources: {}
```

### Mount files (ConfigMaps)

```yaml
mountFiles:
  - name: nginx-config
    filePath: /etc/nginx/nginx.conf
```
