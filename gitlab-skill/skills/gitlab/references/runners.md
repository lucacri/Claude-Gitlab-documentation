# GitLab Runners Reference

## Overview

GitLab Runner is an application that works with GitLab CI/CD to run jobs in a pipeline. Runners can be installed on various platforms and execute jobs in different executors (Docker, Shell, Kubernetes, etc.).

## Runner Types

### Shared Runners
- Available to all projects in a GitLab instance
- Configured by administrators
- Uses shared resources

### Group Runners
- Available to all projects in a group
- Configured at the group level
- Shared within an organization

### Project Runners (Specific Runners)
- Dedicated to a single project
- Complete control over configuration
- Isolated execution environment

## Installation

### Linux

```bash
# Download the binary
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Give it permissions to execute
sudo chmod +x /usr/local/bin/gitlab-runner

# Create a GitLab CI user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and run as service
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

### macOS

```bash
# Download
sudo curl --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64

# Give permissions
sudo chmod +x /usr/local/bin/gitlab-runner

# Install
cd ~
gitlab-runner install
gitlab-runner start
```

### Windows

```powershell
# Create a folder
New-Item -Path 'C:\GitLab-Runner' -ItemType Directory

# Download binary
Invoke-WebRequest -Uri "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-amd64.exe" -OutFile "C:\GitLab-Runner\gitlab-runner.exe"

# Install as service
cd C:\GitLab-Runner
.\gitlab-runner.exe install
.\gitlab-runner.exe start
```

### Docker

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

### Kubernetes (Helm)

```bash
# Add GitLab Helm repository
helm repo add gitlab https://charts.gitlab.io

# Update
helm repo update

# Install
helm install --namespace gitlab-runner gitlab-runner \
  -f values.yaml \
  gitlab/gitlab-runner
```

## Registration

### Register Runner

```bash
gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "PROJECT_REGISTRATION_TOKEN" \
  --executor "docker" \
  --docker-image alpine:latest \
  --description "docker-runner" \
  --maintenance-note "Free-form maintainer notes about this runner" \
  --tag-list "docker,aws" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

### Interactive Registration

```bash
gitlab-runner register

# Follow prompts:
# 1. GitLab instance URL
# 2. Registration token
# 3. Description
# 4. Tags
# 5. Executor type
# 6. Executor-specific questions
```

### Get Registration Token

**Project Runners**:
- Navigate to: Settings > CI/CD > Runners
- Copy registration token

**Group Runners**:
- Navigate to: Group > Settings > CI/CD > Runners
- Copy registration token

**Instance Runners** (Admin):
- Navigate to: Admin Area > Overview > Runners
- Copy registration token

### Unregister Runner

```bash
# By URL and token
gitlab-runner unregister --url "https://gitlab.com/" --token "RUNNER_TOKEN"

# By name
gitlab-runner unregister --name "test-runner"

# Unregister all
gitlab-runner unregister --all-runners
```

## Executors

### Shell Executor

Runs jobs directly on the host machine.

```toml
[[runners]]
  name = "shell-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "shell"
```

**Pros**:
- Simple setup
- Fast execution
- Easy debugging

**Cons**:
- Less isolation
- Environment pollution
- Security concerns

### Docker Executor

Runs jobs in Docker containers (recommended).

```toml
[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
```

**Pros**:
- Clean environment per job
- Isolated execution
- Consistent builds
- Easy image management

**Cons**:
- Requires Docker
- Slight overhead

### Docker Machine Executor

Auto-scales runners using Docker Machine.

```toml
[[runners]]
  name = "docker-machine-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "docker+machine"
  [runners.docker]
    image = "alpine:latest"
  [runners.machine]
    IdleCount = 2
    IdleTime = 1800
    MaxBuilds = 100
    MachineDriver = "amazonec2"
    MachineName = "gitlab-runner-%s"
    MachineOptions = [
      "amazonec2-access-key=XXXX",
      "amazonec2-secret-key=XXXX",
      "amazonec2-region=us-east-1",
      "amazonec2-vpc-id=vpc-xxxxx",
      "amazonec2-subnet-id=subnet-xxxxx",
      "amazonec2-use-private-address=true",
      "amazonec2-instance-type=t2.micro"
    ]
```

### Kubernetes Executor

Runs jobs in Kubernetes pods.

```toml
[[runners]]
  name = "kubernetes-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "kubernetes"
  [runners.kubernetes]
    host = ""
    namespace = "gitlab-runner"
    privileged = false
    cpu_limit = "1"
    memory_limit = "1Gi"
    service_cpu_limit = "200m"
    service_memory_limit = "256Mi"
    helper_cpu_limit = "200m"
    helper_memory_limit = "256Mi"
    poll_interval = 3
    poll_timeout = 180
    [runners.kubernetes.pod_labels]
      "app" = "gitlab-runner"
```

### VirtualBox Executor

```toml
[[runners]]
  name = "virtualbox-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "virtualbox"
  [runners.virtualbox]
    base_name = "my-vm"
    base_snapshot = "my-snapshot"
```

### SSH Executor

```toml
[[runners]]
  name = "ssh-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "ssh"
  [runners.ssh]
    host = "example.com"
    port = "22"
    user = "gitlab-runner"
    password = "password"
    identity_file = "/home/user/.ssh/id_rsa"
```

## Configuration

### Configuration File Location

- **Linux**: `/etc/gitlab-runner/config.toml`
- **macOS**: `~/.gitlab-runner/config.toml`
- **Windows**: `C:\GitLab-Runner\config.toml`
- **Docker**: `/etc/gitlab-runner/config.toml` (inside container)

### Basic Configuration

```toml
concurrent = 4  # Max concurrent jobs
check_interval = 0  # Check for new jobs (0 = default 3s)

[session_server]
  session_timeout = 1800

[[runners]]
  name = "my-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "docker"
  [runners.custom_build_dir]
    enabled = true
  [runners.cache]
    Type = "s3"
    Path = "runner"
    Shared = true
    [runners.cache.s3]
      ServerAddress = "s3.amazonaws.com"
      AccessKey = "XXXX"
      SecretKey = "XXXX"
      BucketName = "runners-cache"
      BucketLocation = "us-east-1"
  [runners.docker]
    image = "alpine:latest"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
```

### Docker Executor Configuration

```toml
[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "docker"
  [runners.docker]
    # Default Docker image
    image = "alpine:latest"

    # Run containers in privileged mode
    privileged = false

    # Disable cache
    disable_cache = false

    # Volumes to mount
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]

    # Shared memory size
    shm_size = 0

    # Pull policy
    pull_policy = ["if-not-present"]

    # Network mode
    network_mode = "bridge"

    # DNS servers
    dns = ["8.8.8.8"]

    # CPU limit
    cpus = "2"

    # Memory limit
    memory = "2g"

    # Memory swap limit
    memory_swap = "4g"

    # OOM kill disable
    oom_kill_disable = false

    # Extra hosts
    extra_hosts = ["gitlab.example.com:192.168.1.1"]

    # Helper image
    helper_image = "gitlab/gitlab-runner-helper:latest"

    # Allow services
    allowed_images = ["ruby:*", "python:*"]
    allowed_services = ["postgres:*", "redis:*"]
```

## Runner Management

### Commands

```bash
# Start runner
gitlab-runner start

# Stop runner
gitlab-runner stop

# Restart runner
gitlab-runner restart

# Status
gitlab-runner status

# Verify configuration
gitlab-runner verify

# List runners
gitlab-runner list

# Run single job
gitlab-runner run-single

# View version
gitlab-runner --version
```

### Update Runner

```bash
# Linux
sudo gitlab-runner stop
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
sudo chmod +x /usr/local/bin/gitlab-runner
sudo gitlab-runner start

# Docker
docker pull gitlab/gitlab-runner:latest
docker stop gitlab-runner && docker rm gitlab-runner
# Re-create container with same configuration
```

### Runner API Management

**List runners**:
```bash
curl --header "PRIVATE-TOKEN: <token>" "https://gitlab.com/api/v4/runners"
```

**Get runner details**:
```bash
curl --header "PRIVATE-TOKEN: <token>" "https://gitlab.com/api/v4/runners/:id"
```

**Update runner**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/runners/:id" \
  --form "description=new-description" \
  --form "active=true" \
  --form "tag_list=docker,aws"
```

**Delete runner**:
```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/runners/:id"
```

## Caching

### Local Cache

```toml
[[runners]]
  executor = "docker"
  [runners.cache]
    Type = "local"
    Path = "/cache"
    Shared = true
    [runners.cache.local]
```

### S3 Cache

```toml
[[runners]]
  executor = "docker"
  [runners.cache]
    Type = "s3"
    Path = "runner"
    Shared = true
    [runners.cache.s3]
      ServerAddress = "s3.amazonaws.com"
      AccessKey = "AWS_ACCESS_KEY"
      SecretKey = "AWS_SECRET_KEY"
      BucketName = "runners-cache"
      BucketLocation = "us-east-1"
```

### Google Cloud Storage Cache

```toml
[[runners]]
  executor = "docker"
  [runners.cache]
    Type = "gcs"
    Path = "runner"
    Shared = true
    [runners.cache.gcs]
      AccessID = "cache-access-account@test-project-123456.iam.gserviceaccount.com"
      PrivateKey = "/path/to/key.json"
      BucketName = "runners-cache"
```

### Azure Cache

```toml
[[runners]]
  executor = "docker"
  [runners.cache]
    Type = "azure"
    Path = "runner"
    Shared = true
    [runners.cache.azure]
      AccountName = "account-name"
      AccountKey = "account-key"
      ContainerName = "runners-cache"
```

## Security

### Protected Runners

- Only run jobs on protected branches/tags
- Configure in project/group settings
- Prevents unauthorized job execution

```toml
[[runners]]
  # Runner configuration
  access_level = "ref_protected"
```

### Locked Runners

- Prevents runner from being enabled for other projects
- Useful for dedicated project runners

**Via UI**: Runner settings > Lock to current projects

**Via API**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/runners/:id" \
  --form "locked=true"
```

### Runner Tags

Control which runners execute which jobs:

**.gitlab-ci.yml**:
```yaml
job-name:
  tags:
    - docker
    - linux
  script:
    - echo "Running on tagged runner"
```

**Runner configuration**:
```toml
[[runners]]
  name = "docker-linux-runner"
  # Tags assigned during registration or via config
```

### Privileged Mode

Allows Docker-in-Docker but has security implications:

```toml
[[runners]]
  executor = "docker"
  [runners.docker]
    privileged = true  # Use with caution
```

**Security considerations**:
- Container escapes possible
- Host system access
- Use only when necessary
- Prefer rootless Docker

## Monitoring

### Prometheus Metrics

Enable metrics in config:

```toml
listen_address = ":9252"
```

**Metrics endpoint**: `http://runner-host:9252/metrics`

**Key metrics**:
- `gitlab_runner_jobs` - Running jobs
- `gitlab_runner_job_duration_seconds` - Job duration
- `gitlab_runner_errors_total` - Error count
- `gitlab_runner_api_request_statuses_total` - API request statuses

### Logging

```toml
# Logging level
log_level = "info"  # debug, info, warning, error, fatal, panic

# Log format
log_format = "text"  # text or json
```

**View logs**:
```bash
# Service logs (systemd)
sudo journalctl -u gitlab-runner -f

# Docker logs
docker logs -f gitlab-runner
```

## Troubleshooting

### Common Issues

**1. Runner not picking up jobs**
- Check runner is registered: `gitlab-runner verify`
- Verify runner is active in GitLab UI
- Check tags match job requirements
- Ensure runner has capacity (concurrent jobs)

**2. Docker executor issues**
- Verify Docker is running: `docker ps`
- Check Docker socket permissions
- Ensure Docker image is accessible
- Check network connectivity

**3. Permission errors**
- Check gitlab-runner user permissions
- Verify working directory permissions
- Check cache directory permissions

**4. Out of disk space**
- Clear old Docker images: `docker system prune -a`
- Check cache size
- Monitor disk usage

**5. SSL certificate errors**
- Add certificate to system: `gitlab-runner register --tls-ca-file=/path/to/ca.crt`
- Or disable verification (not recommended): `tls_verify = false`

### Debug Mode

```bash
# Run in debug mode
gitlab-runner --debug run

# Single job debug
gitlab-runner --debug run-single
```

### Health Check

```bash
# Verify runner can connect to GitLab
gitlab-runner verify

# Delete invalid runners
gitlab-runner verify --delete
```

## Best Practices

### 1. Runner Configuration

- Use Docker executor for isolation
- Configure appropriate concurrent jobs limit
- Set up distributed cache (S3, GCS)
- Use helper image from registry
- Configure resource limits

### 2. Security

- Use protected runners for sensitive jobs
- Lock runners to specific projects
- Avoid privileged mode when possible
- Rotate runner tokens regularly
- Use runner tags effectively

### 3. Performance

- Enable distributed caching
- Use local Docker image cache
- Configure appropriate timeout values
- Monitor runner metrics
- Scale runners based on load

### 4. Maintenance

- Keep runners updated
- Monitor disk space
- Clean up old Docker images
- Review runner logs regularly
- Set up alerts for runner failures

### 5. High Availability

- Deploy multiple runners
- Use auto-scaling (Docker Machine, Kubernetes)
- Configure health checks
- Implement monitoring
- Plan for failover

## Auto-Scaling

### Docker Machine Auto-Scaling

```toml
[[runners]]
  limit = 10  # Maximum runners
  executor = "docker+machine"

  [runners.machine]
    IdleCount = 2  # Idle machines to keep
    IdleTime = 600  # Seconds before removing idle machine
    MaxBuilds = 100  # Max builds per machine before removal
    MachineDriver = "amazonec2"
    MachineName = "gitlab-docker-machine-%s"
    MachineOptions = [
      "amazonec2-instance-type=t2.medium",
      "amazonec2-region=us-east-1",
      "amazonec2-vpc-id=vpc-xxxxx",
      "amazonec2-subnet-id=subnet-xxxxx",
      "amazonec2-zone=a",
      "amazonec2-security-group=gitlab-runner"
    ]

    # Periods when auto-scaling is active
    [[runners.machine.autoscaling]]
      Periods = ["* * 9-17 * * mon-fri *"]  # Business hours
      IdleCount = 5
      IdleTime = 600
      Timezone = "America/New_York"

    [[runners.machine.autoscaling]]
      Periods = ["* * * * * sat,sun *"]  # Weekends
      IdleCount = 1
      IdleTime = 300
```

### Kubernetes Auto-Scaling

Kubernetes handles scaling automatically based on resource requests.

## Additional Resources

- Official Runner Documentation: https://docs.gitlab.com/runner/
- Runner Executors: https://docs.gitlab.com/runner/executors/
- Advanced Configuration: https://docs.gitlab.com/runner/configuration/
- Auto-scaling: https://docs.gitlab.com/runner/configuration/autoscale.html
