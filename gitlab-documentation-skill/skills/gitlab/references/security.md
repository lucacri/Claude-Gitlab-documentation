# GitLab Security Features Reference

## Overview

GitLab provides comprehensive security scanning and vulnerability management built into the CI/CD pipeline.

## Security Scanning Types

### 1. SAST (Static Application Security Testing)

Analyzes source code for security vulnerabilities.

**.gitlab-ci.yml**:
```yaml
include:
  - template: Security/SAST.gitlab-ci.yml

variables:
  SAST_EXCLUDED_PATHS: "spec, test, tests, tmp"
```

**Supported languages**:
- JavaScript/TypeScript
- Python
- Ruby
- Java
- C/C++
- Go
- PHP
- C#/.NET
- Scala
- And more...

**Custom configuration**:
```yaml
sast:
  variables:
    SEARCH_MAX_DEPTH: 20
    SAST_ANALYZER_IMAGE_TAG: "latest"
    SAST_DISABLE_BABEL: "true"
```

### 2. DAST (Dynamic Application Security Testing)

Tests running applications for vulnerabilities.

```yaml
include:
  - template: Security/DAST.gitlab-ci.yml

variables:
  DAST_WEBSITE: https://example.com
  DAST_AUTH_URL: https://example.com/login
  DAST_USERNAME: testuser
  DAST_PASSWORD: $DAST_PASSWORD
  DAST_FULL_SCAN_ENABLED: "true"
```

**DAST Configuration**:
```yaml
dast:
  stage: test
  variables:
    DAST_API_SPECIFICATION: openapi.json
    DAST_API_HOST_OVERRIDE: https://api.example.com
  dast_configuration:
    site_profile: "Production Site"
    scanner_profile: "Full Scan"
```

**Browser-based DAST**:
```yaml
include:
  - template: DAST-On-Demand-Scan.gitlab-ci.yml

dast:
  variables:
    DAST_BROWSER_SCAN: "true"
    DAST_TARGET_AVAILABILITY_TIMEOUT: 120
```

### 3. Dependency Scanning

Checks dependencies for known vulnerabilities.

```yaml
include:
  - template: Security/Dependency-Scanning.gitlab-ci.yml

variables:
  DS_EXCLUDED_PATHS: "spec, test, tests, tmp"
  DS_JAVA_VERSION: 11
```

**Supported package managers**:
- npm/yarn (JavaScript)
- pip/pipenv (Python)
- bundler (Ruby)
- Maven/Gradle (Java)
- Go modules
- Composer (PHP)
- NuGet (.NET)
- CocoaPods (iOS)

**Custom analyzer**:
```yaml
dependency_scanning:
  variables:
    DS_ANALYZER_IMAGE: "registry.gitlab.com/security-products/gemnasium:latest"
    DS_ANALYZER_IMAGE_TAG: "2"
```

### 4. Container Scanning

Scans Docker images for vulnerabilities.

```yaml
include:
  - template: Security/Container-Scanning.gitlab-ci.yml

variables:
  CS_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  CS_DOCKERFILE_PATH: Dockerfile
```

**Scan custom registry**:
```yaml
container_scanning:
  variables:
    CS_REGISTRY_USER: $CI_REGISTRY_USER
    CS_REGISTRY_PASSWORD: $CI_REGISTRY_PASSWORD
    CS_IMAGE: registry.example.com/image:tag
```

### 5. Secret Detection

Prevents committing secrets to repository.

```yaml
include:
  - template: Security/Secret-Detection.gitlab-ci.yml

variables:
  SECRET_DETECTION_EXCLUDED_PATHS: "tests/, spec/"
```

**Detected secrets**:
- AWS credentials
- API keys
- OAuth tokens
- Private keys
- Passwords
- Database credentials
- And more...

**Custom rules**:
```yaml
secret_detection:
  variables:
    SECRET_DETECTION_HISTORIC_SCAN: "true"
    SECRET_DETECTION_LOG_OPTIONS: "--all --full-history"
```

### 6. License Compliance

Identifies licenses in dependencies.

```yaml
include:
  - template: Security/License-Scanning.gitlab-ci.yml

license_scanning:
  variables:
    LICENSE_FINDER_CLI_OPTS: '--aggregate-paths=. --decisions-file=.license_decisions.yml'
```

**License policies**:
```yaml
# .license_decisions.yml
allowed:
  - MIT
  - Apache-2.0
  - BSD-3-Clause
denied:
  - GPL-2.0
  - GPL-3.0
```

### 7. Coverage-Guided Fuzz Testing

Tests application with random inputs.

```yaml
include:
  - template: Security/Coverage-Fuzzing.gitlab-ci.yml

my-fuzz-target:
  extends: .fuzz_base
  script:
    - ./gitlab-cov-fuzz run -- ./fuzz-target
```

### 8. API Security Testing

Tests API endpoints for vulnerabilities.

```yaml
include:
  - template: Security/API-Security.gitlab-ci.yml

variables:
  DAST_API_SPECIFICATION: openapi.json
  DAST_API_TARGET_URL: https://api.example.com
```

## Security Dashboard

### Project Security Dashboard

View vulnerabilities at project level:
- Critical, High, Medium, Low severities
- Vulnerability trends
- Status (Detected, Confirmed, Dismissed, Resolved)
- Fix recommendations

### Group Security Dashboard

Aggregate view across projects (Ultimate):
- Cross-project vulnerabilities
- Priority vulnerabilities
- Compliance status
- Export capabilities

### Vulnerability Management

**Create vulnerability**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/vulnerabilities" \
  --data "title=SQL Injection" \
  --data "severity=critical" \
  --data "state=detected" \
  --data "description=Details..."
```

**Update vulnerability**:
```bash
curl --request PATCH --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/vulnerabilities/:id" \
  --data "state=confirmed"
```

**States**:
- `detected`: Newly detected
- `confirmed`: Verified as real
- `dismissed`: False positive/accepted risk
- `resolved`: Fixed

## Security Policies

### Scan Execution Policies

Enforce security scans on projects:

```yaml
# .gitlab/security-policies/policy.yml
scan_execution_policy:
  - name: Enforce SAST and Dependency Scanning
    description: Run security scans on all branches
    enabled: true
    rules:
      - type: pipeline
        branches:
          - main
          - develop
          - release/*
    actions:
      - scan: sast
      - scan: dependency_scanning
      - scan: secret_detection
```

### Scan Result Policies

Control merge based on scan results:

```yaml
scan_result_policy:
  - name: Block merge on critical vulnerabilities
    description: Prevent merging if critical vulnerabilities found
    enabled: true
    rules:
      - type: scan_finding
        branches:
          - main
        scanners:
          - sast
          - dependency_scanning
        severity_levels:
          - critical
        vulnerability_states:
          - newly_detected
    actions:
      - type: require_approval
        approvals_required: 2
        role_approvers:
          - security
```

## Vulnerability Reports

### Generate Reports

Security scanners generate JSON reports:

```yaml
sast:
  artifacts:
    reports:
      sast: gl-sast-report.json

dependency_scanning:
  artifacts:
    reports:
      dependency_scanning: gl-dependency-scanning-report.json

container_scanning:
  artifacts:
    reports:
      container_scanning: gl-container-scanning-report.json
```

### Report Format

```json
{
  "version": "15.0.0",
  "vulnerabilities": [
    {
      "id": "...",
      "category": "sast",
      "name": "SQL Injection",
      "message": "Potential SQL injection",
      "description": "...",
      "cve": "CVE-2021-12345",
      "severity": "Critical",
      "confidence": "High",
      "scanner": {
        "id": "semgrep",
        "name": "Semgrep"
      },
      "location": {
        "file": "app/controllers/users_controller.rb",
        "start_line": 42,
        "end_line": 45
      },
      "identifiers": [
        {
          "type": "cwe",
          "name": "CWE-89",
          "value": "89",
          "url": "https://cwe.mitre.org/data/definitions/89.html"
        }
      ],
      "links": [
        {
          "url": "https://owasp.org/www-community/attacks/SQL_Injection"
        }
      ],
      "solution": "Use parameterized queries"
    }
  ],
  "remediations": [],
  "dependency_files": []
}
```

## Compliance Features

### Compliance Framework

Define compliance requirements (Ultimate):

```yaml
compliance_framework:
  name: "SOC 2"
  description: "SOC 2 compliance requirements"
  color: "#1aaa55"
  default: false
  pipeline_configuration_full_path: ".gitlab/compliance/soc2.yml"
```

### Compliance Pipeline

```yaml
# .gitlab/compliance/soc2.yml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/License-Scanning.gitlab-ci.yml

compliance_audit:
  stage: test
  script:
    - audit-compliance.sh
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

### Audit Events

Track security-related activities:

```bash
# Get audit events
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/audit_events"

# Group audit events
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/audit_events"
```

**Tracked events**:
- Member additions/removals
- Permission changes
- Protected branch changes
- Security scan results
- Compliance violations
- And more...

## Security Best Practices

### 1. Enable All Scanners

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
  - template: Security/License-Scanning.gitlab-ci.yml
```

### 2. Block Merges on Critical Issues

Configure merge request approvals:
- Require approval from security team
- Block merges with critical vulnerabilities
- Require all security checks to pass

### 3. Regular Dependency Updates

```yaml
dependency_update:
  stage: maintain
  script:
    - bundle update
    - npm update
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
  only:
    variables:
      - $DEPENDENCY_UPDATE == "true"
```

### 4. Secret Management

Use CI/CD variables for secrets:
- Mark as protected
- Mark as masked
- Limit scope to specific environments
- Rotate regularly

```bash
# Add protected variable
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/variables" \
  --data "key=SECRET_KEY" \
  --data "value=secret_value" \
  --data "protected=true" \
  --data "masked=true" \
  --data "environment_scope=production"
```

### 5. Two-Factor Authentication

Enforce 2FA for all users:
- Group settings > General > Permissions
- Require 2FA for all group members
- Set grace period for enablement

### 6. IP Allowlisting

Restrict access by IP (Premium/Ultimate):

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id" \
  --data "ip_restriction_ranges[]=192.168.1.0/24" \
  --data "ip_restriction_ranges[]=10.0.0.0/8"
```

### 7. Security Training

GitLab provides security training:
- Secure coding practices
- OWASP Top 10
- Security testing
- Vulnerability remediation

## Security Integrations

### SIEM Integration

Export audit logs to SIEM:

**Splunk**:
```yaml
# .gitlab-ci.yml
export_to_splunk:
  script:
    - curl -k https://splunk.example.com:8088/services/collector \
        -H "Authorization: Splunk $SPLUNK_TOKEN" \
        -d '{"event": $AUDIT_DATA}'
```

**ELK Stack**:
```yaml
export_to_elk:
  script:
    - |
      curl -X POST "https://elasticsearch.example.com:9200/gitlab-audit/_doc" \
        -H "Content-Type: application/json" \
        -d "$AUDIT_DATA"
```

### Vulnerability Management Tools

Integrate with external tools:
- Jira for vulnerability tracking
- ServiceNow for incident management
- PagerDuty for security alerts

## Security API

### List Vulnerabilities

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/vulnerabilities?project_id=:id&severity=critical"
```

### Dismiss Vulnerability

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/vulnerabilities/:id/dismiss" \
  --data "dismissal_reason=acceptable_risk" \
  --data "comment=Risk accepted by security team"
```

### Resolve Vulnerability

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/vulnerabilities/:id/resolve"
```

## Security Hardening

### Runner Security

```toml
[[runners]]
  [runners.docker]
    privileged = false
    disable_cache = false
    volumes = ["/cache"]

    # Security settings
    security_opt = ["no-new-privileges"]
    cap_drop = ["ALL"]
    cap_add = ["NET_BIND_SERVICE"]
```

### Registry Security

```yaml
registry:
  storage_delete:
    enabled: true
  validation:
    manifests:
      urls:
        allow:
          - ^https://registry\.gitlab\.com/
```

### Git Security

```ruby
# /etc/gitlab/gitlab.rb
gitlab_rails['allowed_hosts'] = ['gitlab.example.com']
gitlab_rails['gitlab_shell_ssh_port'] = 2222
gitlab_rails['gitlab_shell_git_timeout'] = 800
```

## Incident Response

### Security Incident Template

```.markdown
# Security Incident: [TITLE]

## Severity
- [ ] Critical
- [ ] High
- [ ] Medium
- [ ] Low

## Detection
- Date/Time:
- Method:
- Reporter:

## Description
[Detailed description]

## Impact
[Affected systems/data]

## Response Actions
- [ ] Contain threat
- [ ] Assess damage
- [ ] Notify stakeholders
- [ ] Remediate vulnerability
- [ ] Document lessons learned

## Timeline
| Time | Action |
|------|--------|
|      |        |

## Root Cause

## Remediation

## Prevention
```

## Additional Resources

- Security Documentation: https://docs.gitlab.com/ee/user/application_security/
- Security Scanners: https://docs.gitlab.com/ee/user/application_security/security_scanner_integration/
- Vulnerability Management: https://docs.gitlab.com/ee/user/application_security/vulnerabilities/
- Compliance: https://docs.gitlab.com/ee/administration/compliance.html
