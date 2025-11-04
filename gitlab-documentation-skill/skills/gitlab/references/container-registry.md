# GitLab Container Registry Reference

## Overview

GitLab Container Registry is a secure Docker image registry built into GitLab, allowing you to store and manage Docker images.

## Registry URL Format

```
registry.gitlab.com/namespace/project
```

For self-hosted:
```
registry.your-domain.com/namespace/project
```

## Authentication

### Docker Login

```bash
# Using personal access token
docker login registry.gitlab.com -u <username> -p <token>

# Using deploy token
docker login registry.gitlab.com -u <deploy-token-username> -p <deploy-token>

# In CI/CD (automatic)
docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
```

### Token Scopes

Personal Access Token needs:
- `read_registry` - Pull images
- `write_registry` - Push images

Deploy Token needs:
- `read_registry` - Pull images
- `write_registry` - Push images

## Building and Pushing Images

### Basic Docker Build

```yaml
# .gitlab-ci.yml
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

### Multi-Stage Build

**Dockerfile**:
```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --production
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### Tagging Strategy

```yaml
build:
  script:
    # Commit SHA tag
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

    # Branch name tag
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG

    # Latest tag (main branch only)
    - |
      if [ "$CI_COMMIT_BRANCH" == "main" ]; then
        docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
        docker push $CI_REGISTRY_IMAGE:latest
      fi

    # Version tag (for tags)
    - |
      if [ -n "$CI_COMMIT_TAG" ]; then
        docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
        docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
      fi
```

## Pulling Images

### Pull from Registry

```bash
# Pull specific tag
docker pull registry.gitlab.com/group/project:tag

# Pull latest
docker pull registry.gitlab.com/group/project:latest

# Pull by SHA
docker pull registry.gitlab.com/group/project@sha256:abc123...
```

### Use in CI/CD

```yaml
test:
  image: $CI_REGISTRY_IMAGE:latest
  script:
    - npm test
```

### Use in Docker Compose

```yaml
version: '3.8'
services:
  app:
    image: registry.gitlab.com/group/project:latest
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
```

## Registry Management

### List Repository Tags

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/registry/repositories/:repository_id/tags"
```

### Get Tag Details

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/registry/repositories/:repository_id/tags/:tag_name"
```

### Delete Tag

```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/registry/repositories/:repository_id/tags/:tag_name"
```

### Delete Tags in Bulk

```bash
# Delete by regex
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/registry/repositories/:repository_id/tags" \
  --data "name_regex=.*-dev" \
  --data "keep_n=5" \
  --data "older_than=7d"
```

## Cleanup Policies

### Configure Cleanup Policy

**Via UI**:
1. Project Settings > Packages & Registries > Container Registry
2. Configure cleanup policy
3. Set rules

**Via API**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/registry/repositories/:repository_id" \
  --data "cadence=1d" \
  --data "keep_n=10" \
  --data "older_than=30d" \
  --data "name_regex=.*-dev" \
  --data "name_regex_keep=.*-stable"
```

**Policy options**:
- `cadence`: How often to run (1d, 7d, 14d, 1month, 3month)
- `keep_n`: Keep N most recent tags
- `older_than`: Delete tags older than specified time
- `name_regex`: Regex for tags to delete
- `name_regex_keep`: Regex for tags to keep

### Cleanup Example

```yaml
# Keep production tags forever
# Delete dev/feature tags after 7 days
# Keep last 5 tags per branch

cleanup_policy:
  enabled: true
  cadence: 1d
  keep_n: 5
  older_than: 7d
  name_regex: '^(?!main|prod|release).*'
  name_regex_keep: '^(main|prod|release-.*|v\d+\.\d+\.\d+)$'
```

## Registry Access Control

### Project-Level Access

Members inherit registry permissions from project role:
- Guest: No access
- Reporter: Pull images
- Developer: Pull and push images
- Maintainer: Full access
- Owner: Full access

### Deploy Tokens

Create deploy tokens for automation:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/deploy_tokens" \
  --data "name=Registry Token" \
  --data "scopes[]=read_registry" \
  --data "scopes[]=write_registry" \
  --data "expires_at=2025-12-31"
```

Use in CI/CD:
```yaml
variables:
  REGISTRY_TOKEN_USER: "deploy-token-user"
  REGISTRY_TOKEN_PASS: "deploy-token-password"

deploy:
  script:
    - docker login -u $REGISTRY_TOKEN_USER -p $REGISTRY_TOKEN_PASS $CI_REGISTRY
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

## Building with Kaniko

Alternative to Docker-in-Docker:

```yaml
build:
  stage: build
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    - mkdir -p /kaniko/.docker
    - echo "{\"auths\":{\"${CI_REGISTRY}\":{\"auth\":\"$(printf "%s:%s" "${CI_REGISTRY_USER}" "${CI_REGISTRY_PASSWORD}" | base64 | tr -d '\n')\"}}}" > /kaniko/.docker/config.json
    - /kaniko/executor
      --context "${CI_PROJECT_DIR}"
      --dockerfile "${CI_PROJECT_DIR}/Dockerfile"
      --destination "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
      --cache=true
      --cache-ttl=24h
```

**Advantages**:
- No Docker daemon needed
- More secure (no privileged mode)
- Better for Kubernetes
- Built-in caching

## Building with Buildah

Daemonless container builds:

```yaml
build:
  stage: build
  image: quay.io/buildah/stable
  script:
    - buildah login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - buildah bud -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - buildah push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

## Multi-Platform Images

### Build for Multiple Architectures

```yaml
build-multiarch:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
    - docker buildx create --use
  script:
    - docker buildx build
      --platform linux/amd64,linux/arm64,linux/arm/v7
      --tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      --tag $CI_REGISTRY_IMAGE:latest
      --push .
```

## Image Scanning

### Container Scanning

```yaml
include:
  - template: Security/Container-Scanning.gitlab-ci.yml

variables:
  CS_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

container_scanning:
  dependencies:
    - build
```

### Trivy Scanner

```yaml
trivy_scan:
  stage: test
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy image --exit-code 0 --no-progress $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - trivy image --exit-code 1 --severity CRITICAL --no-progress $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

## Image Signing

### Sign with Cosign

```yaml
sign:
  stage: sign
  image: gcr.io/projectsigstore/cosign:latest
  script:
    - echo "$COSIGN_KEY" > cosign.key
    - cosign sign --key cosign.key $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

### Verify Signature

```bash
cosign verify --key cosign.pub registry.gitlab.com/group/project:tag
```

## Registry Storage

### Configure Storage Backend

**Local storage**:
```ruby
# /etc/gitlab/gitlab.rb
registry['storage'] = {
  'filesystem' => {
    'rootdirectory' => '/var/opt/gitlab/gitlab-rails/shared/registry'
  }
}
```

**S3 storage**:
```ruby
registry['storage'] = {
  's3' => {
    'accesskey' => 'AKIAIOSFODNN7EXAMPLE',
    'secretkey' => 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
    'bucket' => 'gitlab-registry',
    'region' => 'us-east-1'
  }
}
```

**GCS storage**:
```ruby
registry['storage'] = {
  'gcs' => {
    'bucket' => 'gitlab-registry',
    'keyfile' => '/path/to/keyfile.json'
  }
}
```

**Azure storage**:
```ruby
registry['storage'] = {
  'azure' => {
    'accountname' => 'accountname',
    'accountkey' => 'base64encodedaccountkey',
    'container' => 'gitlab-registry'
  }
}
```

## Registry Mirroring

### Pull from External Registry

```yaml
variables:
  DOCKER_AUTH_CONFIG: |
    {
      "auths": {
        "docker.io": {
          "auth": "$(echo -n $DOCKERHUB_USER:$DOCKERHUB_PASSWORD | base64)"
        }
      }
    }

build:
  script:
    # Pull from Docker Hub
    - docker pull nginx:alpine

    # Tag for GitLab registry
    - docker tag nginx:alpine $CI_REGISTRY_IMAGE/nginx:alpine

    # Push to GitLab registry
    - docker push $CI_REGISTRY_IMAGE/nginx:alpine
```

## Troubleshooting

### Common Issues

**1. Authentication Failed**
```bash
# Clear Docker credentials
rm ~/.docker/config.json

# Re-authenticate
docker login registry.gitlab.com
```

**2. Push Denied**
```bash
# Check token permissions
# Ensure token has write_registry scope
# Verify project access level
```

**3. Image Not Found**
```bash
# Verify image path
docker pull registry.gitlab.com/namespace/project:tag

# Check tag exists
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/registry/repositories"
```

**4. Storage Issues**
```bash
# Check disk space
df -h /var/opt/gitlab

# Run garbage collection
gitlab-rake gitlab:cleanup:container_registry
```

### Enable Debug Logging

```ruby
# /etc/gitlab/gitlab.rb
registry['log_level'] = 'debug'
registry['log_formatter'] = 'json'
```

## Best Practices

### 1. Image Size Optimization

```dockerfile
# Use minimal base images
FROM alpine:latest

# Multi-stage builds
FROM node:18 AS builder
# ... build steps ...
FROM node:18-alpine
COPY --from=builder /app/dist ./dist

# Remove unnecessary files
RUN rm -rf /tmp/* /var/cache/apk/*

# Combine RUN commands
RUN apk add --no-cache curl && \
    curl -o file.tar.gz https://example.com/file.tar.gz && \
    tar -xzf file.tar.gz && \
    rm file.tar.gz
```

### 2. Security

- Scan all images
- Use specific tags (not latest)
- Sign images
- Regularly update base images
- Remove old images

### 3. Tagging

- Use semantic versioning for releases
- Include commit SHA for traceability
- Tag by environment (dev, staging, prod)
- Cleanup temporary tags

### 4. CI/CD Integration

```yaml
stages:
  - build
  - test
  - scan
  - deploy

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  script:
    - npm test

scan:
  stage: scan
  variables:
    CS_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  include:
    - template: Security/Container-Scanning.gitlab-ci.yml

deploy:
  stage: deploy
  script:
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
```

## Registry API Reference

### List Repositories

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/registry/repositories"
```

### Get Repository

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/registry/repositories/:repository_id"
```

### Delete Repository

```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/registry/repositories/:repository_id"
```

## Additional Resources

- Container Registry Docs: https://docs.gitlab.com/ee/user/packages/container_registry/
- Docker Documentation: https://docs.docker.com/
- Cleanup Policies: https://docs.gitlab.com/ee/user/packages/container_registry/reduce_container_registry_storage.html
