# GitLab CI/CD Reference

## Overview

GitLab CI/CD is a built-in continuous integration and deployment tool. Pipelines are defined using `.gitlab-ci.yml` files in the repository root.

## .gitlab-ci.yml Structure

### Basic Example

```yaml
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  script:
    - echo "Building the application..."
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/

test-job:
  stage: test
  script:
    - echo "Running tests..."
    - npm test

deploy-job:
  stage: deploy
  script:
    - echo "Deploying application..."
    - ./deploy.sh
  environment:
    name: production
  only:
    - main
```

## Pipeline Configuration

### Stages

Define pipeline stages (executed in order):

```yaml
stages:
  - build
  - test
  - deploy
  - release
```

**Default stages** (if not specified):
```yaml
stages:
  - .pre
  - build
  - test
  - deploy
  - .post
```

### Jobs

Jobs define what to execute:

```yaml
job-name:
  stage: test
  script:
    - echo "Running job"
  tags:
    - docker
  only:
    - main
```

### Script

Commands to execute:

```yaml
script:
  - echo "Single command"

# Or multi-line
script:
  - echo "First command"
  - echo "Second command"
  - |
    echo "Multi-line script"
    for i in {1..5}; do
      echo "Line $i"
    done
```

### before_script and after_script

```yaml
before_script:
  - echo "Executed before every job"

after_script:
  - echo "Executed after every job"

job-name:
  before_script:
    - echo "Job-specific before script"
  script:
    - echo "Main script"
  after_script:
    - echo "Job-specific after script"
```

## Images and Services

### Docker Image

```yaml
image: node:18

# Job-specific image
job-name:
  image: python:3.11
  script:
    - python --version
```

### Services (Docker-in-Docker, databases, etc.)

```yaml
services:
  - docker:dind
  - postgres:14

variables:
  POSTGRES_DB: test_db
  POSTGRES_USER: user
  POSTGRES_PASSWORD: password
```

## Variables

### Global Variables

```yaml
variables:
  ENVIRONMENT: "production"
  API_URL: "https://api.example.com"
```

### Job Variables

```yaml
job-name:
  variables:
    JOB_VARIABLE: "value"
  script:
    - echo $JOB_VARIABLE
```

### Predefined Variables

Common CI/CD variables:

- `CI`: Always `true` in CI
- `CI_COMMIT_SHA`: Commit SHA
- `CI_COMMIT_REF_NAME`: Branch or tag name
- `CI_COMMIT_BRANCH`: Branch name
- `CI_COMMIT_TAG`: Tag name
- `CI_PROJECT_ID`: Project ID
- `CI_PROJECT_NAME`: Project name
- `CI_PROJECT_PATH`: Project path
- `CI_PIPELINE_ID`: Pipeline ID
- `CI_PIPELINE_IID`: Pipeline IID
- `CI_JOB_ID`: Job ID
- `CI_JOB_NAME`: Job name
- `CI_JOB_STAGE`: Job stage
- `CI_REGISTRY`: GitLab container registry
- `CI_REGISTRY_IMAGE`: Full registry path
- `CI_REGISTRY_USER`: Registry username
- `CI_REGISTRY_PASSWORD`: Registry password
- `GITLAB_USER_EMAIL`: User email
- `GITLAB_USER_LOGIN`: User username

### Variable Precedence

(Highest to lowest priority)
1. Trigger variables, scheduled pipeline variables, manual pipeline run variables
2. Project variables
3. Group variables
4. Instance variables
5. Variables in .gitlab-ci.yml
6. Predefined variables

## Artifacts

### Basic Artifacts

```yaml
job-name:
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
      - build/
    expire_in: 1 week
```

### Artifact Options

```yaml
artifacts:
  # Files to include
  paths:
    - dist/
    - "*.log"

  # Files to exclude
  exclude:
    - dist/*.md

  # Expiration time
  expire_in: 30 days  # default: 30 days
  # Options: 1 day, 1 week, 1 month, never

  # When to upload artifacts
  when: always  # on_success (default), on_failure, always

  # Artifact name
  name: "$CI_JOB_NAME-$CI_COMMIT_REF_NAME"

  # Make artifacts public
  public: true

  # Untracked files
  untracked: false

  # Reports
  reports:
    junit: test-results.xml
    coverage_report:
      coverage_format: cobertura
      path: coverage.xml
```

### Artifact Reports

```yaml
test-job:
  script:
    - npm test
  artifacts:
    reports:
      junit: test-results.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
      dotenv: build.env
```

### Dependencies

```yaml
build-job:
  script:
    - npm run build
  artifacts:
    paths:
      - dist/

deploy-job:
  dependencies:
    - build-job
  script:
    - ls -la dist/
```

### Needs (DAG Pipelines)

```yaml
build-job:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/

test-job:
  stage: test
  needs: [build-job]
  script:
    - npm test

deploy-job:
  stage: deploy
  needs: [test-job]
  script:
    - ./deploy.sh
```

## Caching

### Cache Configuration

```yaml
cache:
  paths:
    - node_modules/
    - .npm/

# Job-specific cache
job-name:
  cache:
    key: "$CI_COMMIT_REF_NAME"
    paths:
      - vendor/
    policy: pull-push  # pull-push (default), pull, push
```

### Cache Keys

```yaml
# Static key
cache:
  key: my-cache

# Dynamic key based on branch
cache:
  key: "$CI_COMMIT_REF_NAME"

# Key with files
cache:
  key:
    files:
      - package-lock.json
    prefix: npm-cache

# Multiple caches
cache:
  - key: npm-cache
    paths:
      - node_modules/
  - key: build-cache
    paths:
      - dist/
```

### Cache vs Artifacts

**Use Cache for**:
- Dependencies (node_modules, vendor)
- Compiled libraries
- Downloaded packages

**Use Artifacts for**:
- Build output
- Test results
- Files needed in later stages

## Job Control

### Rules

```yaml
job-name:
  script:
    - echo "Running job"
  rules:
    # Run on main branch
    - if: $CI_COMMIT_BRANCH == "main"
      when: always

    # Run on merge requests
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

    # Run if file exists
    - exists:
        - Dockerfile
      when: manual

    # Run if files changed
    - changes:
        - src/**/*.js
        - package.json

    # Default
    - when: never
```

### Only/Except (Legacy)

```yaml
job-name:
  only:
    - main
    - tags
    - merge_requests
  except:
    - schedules
```

### When

```yaml
job-name:
  when: manual  # on_success (default), on_failure, always, manual, delayed, never

# Delayed job
job-delayed:
  when: delayed
  start_in: 30 minutes
```

### Allow Failure

```yaml
job-name:
  allow_failure: true
  script:
    - ./optional-script.sh

# Conditional failure
job-conditional:
  allow_failure:
    exit_codes:
      - 137  # Specific exit code
      - 139
```

## Environments

### Basic Environment

```yaml
deploy-production:
  stage: deploy
  script:
    - ./deploy.sh
  environment:
    name: production
    url: https://prod.example.com
```

### Dynamic Environments

```yaml
deploy-review:
  stage: deploy
  script:
    - ./deploy-review.sh
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_COMMIT_REF_SLUG.review.example.com
    on_stop: stop-review
  only:
    - branches
  except:
    - main

stop-review:
  stage: deploy
  script:
    - ./stop-review.sh
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop
  when: manual
```

### Environment Tiers

```yaml
environment:
  name: production
  deployment_tier: production  # production, staging, testing, development, other
```

## Includes

### Include External Files

```yaml
include:
  # Include from same project
  - local: '/templates/.gitlab-ci-template.yml'

  # Include from another project
  - project: 'group/project'
    ref: main
    file: '/templates/.gitlab-ci.yml'

  # Include from URL
  - remote: 'https://example.com/.gitlab-ci.yml'

  # Include template
  - template: Auto-DevOps.gitlab-ci.yml
```

### Include with Rules

```yaml
include:
  - local: '/templates/docker.yml'
    rules:
      - if: $DOCKER_ENABLED == "true"
```

## Extends

### Template Jobs

```yaml
.deploy-template:
  script:
    - ./deploy.sh
  only:
    - main

deploy-staging:
  extends: .deploy-template
  variables:
    ENVIRONMENT: staging

deploy-production:
  extends: .deploy-template
  variables:
    ENVIRONMENT: production
  when: manual
```

## Parallel Jobs

### Matrix

```yaml
test:
  parallel:
    matrix:
      - NODE_VERSION: ["14", "16", "18"]
        OS: ["linux", "windows"]
  script:
    - node --version
    - echo "Testing on $OS with Node $NODE_VERSION"
```

### Simple Parallel

```yaml
test:
  parallel: 5
  script:
    - echo "Running test $CI_NODE_INDEX of $CI_NODE_TOTAL"
```

## Triggers

### Multi-Project Pipelines

```yaml
trigger-downstream:
  trigger:
    project: group/downstream-project
    branch: main
    strategy: depend  # Wait for downstream pipeline
```

### Parent-Child Pipelines

```yaml
trigger-child:
  trigger:
    include: path/to/child-pipeline.yml
    strategy: depend
```

## Docker

### Building Docker Images

```yaml
build-image:
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

### GitLab Container Registry

```yaml
build-and-push:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
    - docker push $CI_REGISTRY_IMAGE:latest
```

## Security Scanning

### SAST (Static Application Security Testing)

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
```

### Dependency Scanning

```yaml
include:
  - template: Security/Dependency-Scanning.gitlab-ci.yml
```

### Container Scanning

```yaml
include:
  - template: Security/Container-Scanning.gitlab-ci.yml

container_scanning:
  variables:
    CI_APPLICATION_REPOSITORY: $CI_REGISTRY_IMAGE
    CI_APPLICATION_TAG: $CI_COMMIT_SHA
```

### Secret Detection

```yaml
include:
  - template: Security/Secret-Detection.gitlab-ci.yml
```

## Advanced Features

### Retry

```yaml
job-name:
  retry: 2  # Retry up to 2 times

# Conditional retry
job-conditional:
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
```

### Timeout

```yaml
job-name:
  timeout: 3 hours  # Default: 1 hour, Max: configured by admin
```

### Resource Group

```yaml
deploy-production:
  resource_group: production
  script:
    - ./deploy.sh
```

### Coverage

```yaml
test-job:
  script:
    - npm test
  coverage: '/Coverage: \d+\.\d+%/'
```

## Pipeline Types

### Merge Request Pipelines

```yaml
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
```

### Scheduled Pipelines

```yaml
job-name:
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
  script:
    - ./nightly-build.sh
```

### Multi-Branch Pipelines

```yaml
workflow:
  rules:
    - if: $CI_COMMIT_BRANCH

build-job:
  script:
    - echo "Building for branch: $CI_COMMIT_BRANCH"
```

## Complete Example

```yaml
# Define Docker image
image: node:18

# Define stages
stages:
  - build
  - test
  - security
  - deploy

# Global variables
variables:
  NODE_ENV: production
  CACHE_KEY: "$CI_COMMIT_REF_SLUG"

# Global cache
cache:
  key: $CACHE_KEY
  paths:
    - node_modules/
    - .npm/

# Global before script
before_script:
  - npm ci --cache .npm --prefer-offline

# Build job
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  rules:
    - if: $CI_COMMIT_BRANCH

# Unit tests
unit-tests:
  stage: test
  script:
    - npm run test:unit
  coverage: '/Lines\s+:\s+(\d+\.\d+)%/'
  artifacts:
    reports:
      junit: test-results.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

# Integration tests
integration-tests:
  stage: test
  services:
    - postgres:14
  variables:
    POSTGRES_DB: test_db
    POSTGRES_USER: test_user
    POSTGRES_PASSWORD: test_password
  script:
    - npm run test:integration

# SAST scanning
sast:
  stage: security
  include:
    - template: Security/SAST.gitlab-ci.yml

# Dependency scanning
dependency_scanning:
  stage: security
  include:
    - template: Security/Dependency-Scanning.gitlab-ci.yml

# Build Docker image
docker-build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# Deploy to staging
deploy-staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
    deployment_tier: staging
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# Deploy to production
deploy-production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com
    deployment_tier: production
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

## Best Practices

### 1. Pipeline Efficiency

- **Use caching**: Cache dependencies to speed up builds
- **Parallel jobs**: Run independent jobs in parallel
- **Artifacts**: Only include necessary files
- **DAG pipelines**: Use `needs` to avoid waiting for entire stages

### 2. Security

- **Protected variables**: Use protected variables for secrets
- **Masked variables**: Mask sensitive values in logs
- **Security scanning**: Include SAST, dependency scanning, etc.
- **Least privilege**: Give jobs minimum necessary permissions

### 3. Reliability

- **Retry failed jobs**: Configure retry for flaky tests
- **Timeouts**: Set appropriate timeouts
- **Failure handling**: Use `allow_failure` for optional jobs
- **Health checks**: Test deployments after deployment

### 4. Maintainability

- **DRY principle**: Use extends and templates
- **Includes**: Separate common configuration
- **Documentation**: Comment complex configurations
- **Consistent naming**: Use clear, consistent job names

### 5. Testing

- **Test early**: Run fast tests first
- **Parallel testing**: Split test suites
- **Test coverage**: Track and enforce coverage
- **Multiple environments**: Test on different OS/versions

## Troubleshooting

### Common Issues

1. **Job stuck in pending**
   - Check runner availability
   - Verify runner tags match job tags
   - Check runner capacity

2. **Cache not working**
   - Verify cache key
   - Check cache path
   - Ensure runner has cache configured

3. **Artifacts not available**
   - Check artifact expiration
   - Verify artifact paths
   - Ensure job completed successfully

4. **Variables not expanding**
   - Check variable syntax ($VARIABLE or ${VARIABLE})
   - Verify variable scope (global, job, protected)
   - Check variable precedence

5. **Docker issues**
   - Verify docker:dind service is running
   - Check Docker TLS configuration
   - Ensure sufficient disk space

### Debugging

```yaml
debug-job:
  script:
    # Print all environment variables
    - env | sort

    # Print specific variables
    - echo $CI_COMMIT_SHA
    - echo $CI_PIPELINE_ID

    # Print working directory
    - pwd
    - ls -la

    # Check cache
    - ls -la node_modules/ || echo "Cache not present"
```

## Additional Resources

- Official Documentation: https://docs.gitlab.com/ee/ci/
- CI/CD Examples: https://docs.gitlab.com/ee/ci/examples/
- Pipeline Configuration Reference: https://docs.gitlab.com/ee/ci/yaml/
- GitLab CI/CD Templates: https://gitlab.com/gitlab-org/gitlab-foss/-/tree/master/lib/gitlab/ci/templates
