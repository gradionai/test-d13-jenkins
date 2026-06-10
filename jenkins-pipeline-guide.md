# Jenkins Pipeline Setup Guide for Demo App

## Prerequisites

- Jenkins instance with the **D13 shared library** configured
- GitHub repo containing this project
- AWS ECR access for Docker image storage
- Kubernetes cluster for deployment

## Step 1: Create a Jenkins Pipeline Job

1. Go to your Jenkins dashboard → **New Item**
2. Enter name: `demo-app` (or `sample-github-demo`)
3. Select **Pipeline** → OK
4. Under **Pipeline** section:
   - **Definition**: `Pipeline script from SCM`
   - **SCM**: `Git`
   - **Repository URL**: your GitHub repo URL (e.g. `https://github.com/<org>/sample-github-demo.git`)
   - **Credentials**: select or add your GitHub credentials
   - **Branch**: `*/develop`
   - **Script Path**: `Jenkinsfile`
5. Save

## Step 2: Ensure the D13 Shared Library is Available

The `Jenkinsfile` uses `@Library('d13') _` — this must be configured in Jenkins:

1. Go to **Manage Jenkins** → **System** → **Global Pipeline Libraries**
2. Verify a library named `d13` exists and points to the D13 shared library repo
3. This library provides `d13Build()` and `d13Deployment()` functions

## Step 3: Webhook (Optional but Recommended)

To trigger builds automatically on push:

1. In your GitHub repo → **Settings** → **Webhooks** → **Add webhook**
2. Payload URL: `https://<your-jenkins>/github-webhook/`
3. Content type: `application/json`
4. Events: **Just the push event**
5. In Jenkins job config, check **GitHub hook trigger for GITScm polling**

## Step 4: Required Credentials

Make sure Jenkins has access to:

| Credential | Purpose |
|------------|---------|
| GitHub | Pull the repo source code |
| AWS ECR | Docker image push (handled by `d13Build()`) |
| Kubernetes | Deployment (handled by `d13Deployment()`) |

## What Happens When the Pipeline Runs

```
d13Build()      → Reads .d13.yaml → builds demo-app/Dockerfile → pushes image to ECR
d13Deployment() → Reads .d13.yaml + .d13.develop.yaml → generates Helm values → deploys to K8s
```

## Local Testing

Run the app locally to verify before deploying:

```bash
cd demo-app
go run main.go
# Then test:
# curl http://localhost:8080/       → "Hello from D13 demo app"
# curl http://localhost:8080/healthz → "ok"
```

Or via Docker:

```bash
docker build -t demo-app ./demo-app
docker run -p 8080:8080 demo-app
```
