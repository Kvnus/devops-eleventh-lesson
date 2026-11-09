# Building and Deploying a Java App with a Jenkins Pipeline

## Setup

### 1. Local registry

```bash
docker run -d --name registry --restart unless-stopped -p 5001:5000 registry:2
```
### 2. Repository

```bash
git init -b main && git add -A && git commit -m init
docker cp .git jenkins:/tmp/gitdir
docker exec jenkins git clone --bare /tmp/gitdir /var/jenkins_home/demo-app.git
```

### 3. Jenkins job

1. New Item → **Pipeline**.
2. Definition: **Pipeline script from SCM** → Git.
3. Repository URL: `file:///var/jenkins_home/demo-app.git` (or your own), Branch: `*/main`.
4. Script Path: `Jenkinsfile`.
5. Save → **Build with Parameters** (or Build Now).

### 4. Verify

```bash
curl localhost:8081                       # Hello from Dockerized Java app!
curl localhost:5001/v2/demo-app/tags/list # image tags in the registry
docker ps --filter name=demo-app
```

