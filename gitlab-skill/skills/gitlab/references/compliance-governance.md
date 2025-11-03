# GitLab Compliance and Governance Reference (Ultimate)

## Compliance Frameworks

### Overview

Define and enforce compliance requirements across projects.

### Create Compliance Framework

**Via UI**: Group > Settings > General > Compliance frameworks

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/compliance_frameworks" \
  --data "name=SOC 2" \
  --data "description=SOC 2 compliance requirements" \
  --data "color=#1aaa55" \
  --data "pipeline_configuration_full_path=.gitlab/compliance/soc2.yml"
```

### Compliance Pipeline

**`.gitlab/compliance/soc2.yml`**:
```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/License-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml

compliance_scan:
  stage: test
  script:
    - echo "Running compliance checks"
    - ./scripts/compliance-check.sh
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### Apply Framework to Projects

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id" \
  --data "compliance_framework_id=123"
```

## Audit Events

### Overview

Track security and compliance-related actions.

### View Audit Events

**Via UI**: Group/Project > Security & Compliance > Audit Events

**Via API**:
```bash
# Group audit events
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/audit_events"

# Project audit events
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/audit_events"

# Instance audit events (admin)
curl --header "PRIVATE-TOKEN: <admin-token>" \
  "https://gitlab.com/api/v4/audit_events"
```

### Tracked Events

**User events**:
- Sign in / Sign out
- Failed login attempts
- Password changes
- Email changes
- 2FA enabled/disabled

**Group events**:
- Member added/removed
- Permission changes
- Group settings changed
- SAML SSO changes

**Project events**:
- Deploy key added/removed
- Protected branch changes
- Webhook created/modified
- Variables added/changed
- Access token created

**Security events**:
- Security policy changes
- Vulnerability state changes
- License compliance changes

### Streaming Audit Events

**Configure streaming** (Ultimate):

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/audit_events/streaming/destinations" \
  --data "destination_url=https://siem.example.com/events"
```

**Stream to**:
- Splunk
- ELK Stack
- Azure Sentinel
- Custom SIEM

### Audit Event Filters

```bash
# Filter by date
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/audit_events?created_after=2025-01-01&created_before=2025-03-31"

# Filter by author
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/audit_events?author_id=123"

# Filter by entity
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/audit_events?entity_type=Project&entity_id=456"
```

## Compliance Dashboard

**Access**: Group > Security & Compliance > Compliance

**Shows**:
- Framework adherence
- Merge request approvals
- Separation of duties
- License compliance
- Dependency vulnerabilities

## Merge Request Approvals (Compliance)

### Approval Rules

**Prevent author approval**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/approval_rules" \
  --data "name=Security Review" \
  --data "approvals_required=2" \
  --data "user_ids[]=security-team-lead-id" \
  --data "groups[]=security-team-group-id"
```

**Prevent committer approval**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id" \
  --data "merge_requests_author_approval=false" \
  --data "merge_requests_disable_committers_approval=true"
```

### Code Owner Approvals

**CODEOWNERS file**:
```
# Security team must approve auth changes
/lib/auth/ @security-team

# Legal must approve license changes
LICENSE @legal-team

# Multiple approvers
/docs/ @tech-writers @product-managers

# Pattern matching
*.yml @devops-team
```

**Require code owner approval**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id" \
  --data "require_code_owner_approval=true"
```

## Separation of Duties

### Protected Environments

**Configure protected environment**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/protected_environments" \
  --data "name=production" \
  --data "deploy_access_levels[][access_level]=40" \
  --data "approval_rules[][access_level]=40" \
  --data "approval_rules[][required_approvals]=2"
```

**Enforce separation**:
- Deployers ≠ Approvers
- Different teams for deploy vs approve
- Multi-level approval

## License Compliance

### License Policies

**Create license policy**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/managed_licenses" \
  --data "name=MIT" \
  --data "approval_status=approved"
```

**Approval statuses**:
- `approved` - Allowed
- `denied` - Blocked
- `blacklisted` - Blocked (legacy term)

**License detection**:
```yaml
# .gitlab-ci.yml
include:
  - template: Security/License-Scanning.gitlab-ci.yml
```

### View License Report

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/licenses"
```

## External Status Checks

Require external approval before merge.

**Configure external check**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/external_status_checks" \
  --data "name=Security Scan" \
  --data "external_url=https://security-scanner.example.com/check" \
  --data "protected_branches[]=main"
```

**External service responds**:
```json
{
  "status": "approved",
  "message": "All security checks passed"
}
```

## Compliance Reports

### Generate Compliance Report

```bash
# Merge request compliance
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/compliance/merge_requests"

# License compliance
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/licenses?per_page=100"
```

### Export Audit Logs

```bash
# Export to CSV
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/audit_events" \
  --header "Accept: text/csv" \
  -o audit-events.csv
```

## Credentials Inventory

Track personal access tokens and SSH keys.

**View credentials**:
```bash
curl --header "PRIVATE-TOKEN: <admin-token>" \
  "https://gitlab.com/api/v4/groups/:id/credentials"
```

**Shows**:
- Personal access tokens
- SSH keys
- Deploy tokens
- Group access tokens

## SCIM Provisioning

Automate user provisioning with SCIM.

**Enable SCIM**:
1. Group Settings > SAML SSO
2. Enable SCIM
3. Generate SCIM token
4. Configure IdP (Okta, Azure AD, etc.)

**SCIM operations**:
- Create users
- Update user attributes
- Deactivate users
- Sync group membership

## Best Practices

### 1. Compliance Frameworks

- Define clear requirements
- Document policies
- Automate enforcement
- Regular audits

### 2. Audit Events

- Enable streaming to SIEM
- Set up alerts
- Regular review
- Retention policies

### 3. Approvals

- Separation of duties
- Code owner reviews
- Multiple approvers for critical paths
- Document exceptions

### 4. License Management

- Maintain approved list
- Block denied licenses
- Regular scans
- Update policies

### 5. Access Control

- Principle of least privilege
- Regular access reviews
- Time-bound credentials
- Audit credential usage

## Additional Resources

- Compliance Documentation: https://docs.gitlab.com/ee/administration/compliance.html
- Audit Events: https://docs.gitlab.com/ee/administration/audit_events.html
- License Compliance: https://docs.gitlab.com/ee/user/compliance/license_compliance/
