# GitLab Groups Administration Reference (Ultimate)

## Groups Overview

Groups allow you to manage multiple projects and users together.

## Creating Groups

**Via UI**:
1. Click "+" > New group
2. Choose "Create group"
3. Enter details
4. Create group

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups" \
  --data "name=Engineering" \
  --data "path=engineering" \
  --data "description=Engineering team" \
  --data "visibility=private"
```

## Subgroups

**Create subgroup**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups" \
  --data "name=Frontend" \
  --data "path=frontend" \
  --data "parent_id=123"
```

**Benefits**:
- Hierarchical organization
- Inherited permissions
- Shared resources
- Easier management

## SAML SSO (Ultimate)

### Overview

Single Sign-On for GitLab groups using SAML 2.0.

### Configure SAML SSO

**Group Settings > SAML SSO**:

1. Enable SAML SSO
2. Configure IdP details
3. Set up attribute mapping
4. Test configuration

**SAML settings**:
```ruby
# For self-hosted GitLab
# config/gitlab.yml or /etc/gitlab/gitlab.rb

gitlab_rails['omniauth_enabled'] = true
gitlab_rails['omniauth_allow_single_sign_on'] = ['saml']
gitlab_rails['omniauth_block_auto_created_users'] = false
gitlab_rails['omniauth_auto_link_saml_user'] = true

gitlab_rails['omniauth_providers'] = [
  {
    name: 'saml',
    args: {
      assertion_consumer_service_url: 'https://gitlab.example.com/users/auth/saml/callback',
      idp_cert_fingerprint: 'XX:XX:XX:XX...',
      idp_sso_target_url: 'https://idp.example.com/sso',
      issuer: 'https://gitlab.example.com',
      name_identifier_format: 'urn:oasis:names:tc:SAML:2.0:nameid-format:persistent',
      attribute_statements: {
        email: ['email'],
        first_name: ['first_name'],
        last_name: ['last_name'],
        name: ['name'],
        username: ['username']
      }
    },
    label: 'Company SSO'
  }
]
```

### SAML Group Sync

**Sync group membership** (Ultimate):

```ruby
gitlab_rails['omniauth_providers'] = [
  {
    name: 'saml',
    args: {
      # ... other settings ...
      attribute_statements: {
        groups: ['groups']
      }
    }
  }
]
```

**Group links**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/saml_group_links" \
  --data "saml_group_name=Developers" \
  --data "access_level=30"
```

### SCIM Provisioning

Automatic user provisioning:

**Enable SCIM**:
1. Group Settings > SAML SSO
2. Generate SCIM token
3. Configure in IdP

**SCIM operations**:
- Provision new users
- Update user attributes
- Deprovision users
- Sync group membership

**IdP Configuration** (Okta example):
```
SCIM URL: https://gitlab.com/api/scim/v2/groups/GROUP_ID
Authorization: Bearer SCIM_TOKEN
```

## Group Permissions

### Member Roles

**Guest (10)**:
- View group/projects
- Leave comments
- Create issues

**Reporter (20)**:
- Pull repositories
- Download artifacts
- View registry

**Developer (30)**:
- Push to repository
- Create MRs
- Manage issues
- Run pipelines

**Maintainer (40)**:
- Manage members
- Edit group settings
- Manage protected branches
- Deploy to environments

**Owner (50)**:
- Delete group
- Transfer projects
- Manage billing
- Configure SAML SSO

### Add Group Members

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/members" \
  --data "user_id=123" \
  --data "access_level=30" \
  --data "expires_at=2025-12-31"
```

### Invite by Email

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/invitations" \
  --data "email=user@example.com" \
  --data "access_level=30"
```

## Group Access Tokens (Ultimate)

**Create group token**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/access_tokens" \
  --data "name=CI Token" \
  --data "scopes[]=api" \
  --data "scopes[]=read_repository" \
  --data "access_level=40" \
  --data "expires_at=2025-12-31"
```

**Token applies to**:
- All group projects
- All subgroups
- Inherited by children

## Group Settings

### General

**Update group**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id" \
  --data "description=New description" \
  --data "visibility=internal" \
  --data "request_access_enabled=true"
```

### Shared Runners

**Enable for group**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id" \
  --data "shared_runners_enabled=true"
```

### CI/CD Variables

**Group-level variables**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/variables" \
  --data "key=DATABASE_URL" \
  --data "value=postgresql://..." \
  --data "protected=true" \
  --data "masked=true" \
  --data "environment_scope=production"
```

## IP Allowlist (Ultimate)

Restrict access by IP:

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id" \
  --data "ip_restriction_ranges[]=192.168.1.0/24" \
  --data "ip_restriction_ranges[]=10.0.0.0/8"
```

## Allowed Email Domains (Ultimate)

Restrict membership by email domain:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/allowed_email_domains" \
  --data "domain=example.com"
```

## Group-Level Features

### Epics (Ultimate)

Manage epics at group level:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/epics" \
  --data "title=Q4 Initiative"
```

### Roadmaps (Ultimate)

Visual planning across projects.

**Access**: Group > Epics > Roadmap

**Features**:
- Timeline view
- Drag-and-drop
- Filter by label
- Zoom levels

### Iterations (Ultimate)

Group-level iterations:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/iterations" \
  --data "title=Sprint 42" \
  --data "start_date=2025-11-01" \
  --data "due_date=2025-11-14"
```

### Milestones

Group milestones:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/milestones" \
  --data "title=v2.0" \
  --data "description=Version 2.0 release" \
  --data "due_date=2025-12-31"
```

## Compliance (Ultimate)

### Compliance Frameworks

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/compliance_frameworks" \
  --data "name=SOC 2" \
  --data "description=SOC 2 compliance" \
  --data "color=#1aaa55"
```

### Audit Events

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/audit_events?created_after=2025-01-01"
```

## Group Wikis (Ultimate)

**Enable group wiki**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/wikis" \
  --data "title=Home" \
  --data "content=# Welcome"
```

## Transfer Projects

**Transfer project to group**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:project_id/transfer" \
  --data "namespace=group-id"
```

## Group Transfer

**Transfer group to another namespace**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/transfer" \
  --data "group_id=parent-group-id"
```

## Group Deletion

**Delete group**:
```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id"
```

**Delayed deletion** (Ultimate):
- 7-day grace period
- Recoverable during period
- Permanent after period

**Restore deleted group**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/restore"
```

## Best Practices

### 1. Organization

- Clear group hierarchy
- Consistent naming
- Document structure
- Regular cleanup

### 2. Access Control

- Use group membership
- Leverage SAML SSO
- Regular access reviews
- Least privilege

### 3. Security

- Enable 2FA requirement
- IP allowlisting
- Email domain restrictions
- Audit event monitoring

### 4. Compliance

- Apply compliance frameworks
- Regular audits
- Document procedures
- Train members

### 5. Resource Management

- Group-level variables
- Shared runners
- Group tokens
- Resource limits

## Additional Resources

- Groups Documentation: https://docs.gitlab.com/ee/user/group/
- SAML SSO: https://docs.gitlab.com/ee/user/group/saml_sso/
- SCIM: https://docs.gitlab.com/ee/user/group/saml_sso/scim_setup.html
- Group Access Tokens: https://docs.gitlab.com/ee/user/group/settings/group_access_tokens.html
