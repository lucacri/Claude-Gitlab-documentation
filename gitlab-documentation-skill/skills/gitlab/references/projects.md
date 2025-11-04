# GitLab Projects Reference

## Overview

Projects in GitLab contain your source code, issues, merge requests, and CI/CD pipelines. They are the central hub for all development activity.

## Project Creation

### Via UI

1. Click "+" dropdown > "New project/repository"
2. Choose creation method:
   - Create blank project
   - Create from template
   - Import project
   - Run CI/CD for external repository
3. Configure project settings
4. Click "Create project"

### Via API

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "name": "My Project",
    "description": "Project description",
    "visibility": "private",
    "initialize_with_readme": true,
    "namespace_id": 123
  }' \
  "https://gitlab.com/api/v4/projects"
```

### Via CLI

```bash
# Using glab CLI
glab repo create my-project --description "My project" --private
```

## Project Settings

### General Settings

**Project Name and Description**:
- Name: Display name
- Description: Project overview
- Project avatar: Custom image
- Topics: Searchable tags

**Visibility**:
- Private: Only project members
- Internal: Any authenticated user
- Public: Everyone (including anonymous)

**Project URL**:
- Namespace: Group or user
- Project slug: URL-friendly name

### Merge Request Settings

**Merge method**:
- Merge commit: Traditional merge
- Merge commit with semi-linear history: Rebase then merge
- Fast-forward merge: Only if fast-forward possible

**Merge options**:
- Enable "Delete source branch" by default
- Enable merge request approvals
- Squash commits when merging
- Require merge request for push
- Enable merge trains

**Merge checks**:
- Pipelines must succeed
- All threads must be resolved
- Status checks must pass

### Feature Visibility

Control feature availability:
- Issues
- Repository
- Merge requests
- Forks
- CI/CD
- Container Registry
- Analytics
- Requirements
- Wiki
- Snippets
- Pages
- Operations

### Protected Branches

```bash
# Via API
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/protected_branches" \
  --data "name=main" \
  --data "push_access_level=40" \
  --data "merge_access_level=40"
```

**Access levels**:
- 0: No access
- 30: Developer
- 40: Maintainer
- 60: Admin

**Wildcard protection**:
```
release/*
stable-*
```

### Protected Tags

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/protected_tags" \
  --data "name=v*" \
  --data "create_access_level=40"
```

## Project Templates

### Built-in Templates

GitLab provides templates for:
- **Ruby on Rails**
- **Spring**
- **Express**
- **Hugo**
- **Jekyll**
- **Hexo**
- **GitBook**
- **Gatsby**
- **Nfancy**
- **Android**
- **iOS (Swift)**
- **TYPO3**
- **Laravel**
- **Salesforce**
- And more...

### Custom Project Templates

Create custom templates for your organization:

1. Create a project with desired structure
2. Add template files and configuration
3. Mark as template (Premium/Ultimate)
4. Use in project creation

### Project Template API

```bash
# List templates
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/templates/dockerfiles"

# Get specific template
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/templates/dockerfiles/Ruby"
```

Available template types:
- `dockerfiles`
- `gitignores`
- `gitlab_ci_ymls`
- `licenses`

## Project Import/Export

### Export Project

**Via UI**:
1. Settings > General > Advanced
2. Export project
3. Download export file

**Via API**:
```bash
# Start export
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/export"

# Check status
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/export"

# Download
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/export/download" \
  --output project-export.tar.gz
```

### Import Project

**Via UI**:
1. New project > Import project
2. Upload export file
3. Configure import settings
4. Import

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --form "file=@project-export.tar.gz" \
  --form "path=imported-project" \
  "https://gitlab.com/api/v4/projects/import"
```

### Import from Other Sources

**GitHub**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --data "github_personal_access_token=<github_token>" \
  --data "target_namespace=my-group" \
  --data "repo_id=123456" \
  "https://gitlab.com/api/v4/import/github"
```

**Bitbucket**:
- Via UI: New project > Import from Bitbucket
- Authenticate with Bitbucket
- Select repositories

**GitLab.com to self-hosted**:
- Use project export/import
- Or remote mirrors

## Project Members and Permissions

### Member Roles

**Guest (10)**:
- View issues, merge requests
- Leave comments
- No repository access

**Reporter (20)**:
- Pull repository
- Download artifacts
- View CI/CD analytics

**Developer (30)**:
- Push to repository
- Create merge requests
- Manage issues and MRs
- Run CI/CD pipelines

**Maintainer (40)**:
- Manage project settings
- Manage team members
- Manage protected branches
- Deploy to production

**Owner (50)**:
- Delete project
- Transfer project
- Change project visibility

### Add Members

**Via UI**:
1. Project > Members
2. Invite members
3. Select role and expiration
4. Send invitation

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/members" \
  --data "user_id=123" \
  --data "access_level=30"
```

### Share Project with Group

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/share" \
  --data "group_id=456" \
  --data "group_access=30" \
  --data "expires_at=2025-12-31"
```

## Project Badges

Add badges to display project information:

```bash
# Create badge
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/badges" \
  --data "link_url=https://example.com" \
  --data "image_url=https://example.com/badge.svg" \
  --data "name=Pipeline"
```

**Badge placeholders**:
- `%{project_path}`: Project path
- `%{project_id}`: Project ID
- `%{default_branch}`: Default branch
- `%{commit_sha}`: Latest commit SHA

**Example badges**:
```
Pipeline: https://gitlab.com/%{project_path}/badges/%{default_branch}/pipeline.svg
Coverage: https://gitlab.com/%{project_path}/badges/%{default_branch}/coverage.svg
```

## Project Milestones

### Create Milestone

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/milestones" \
  --data "title=v1.0" \
  --data "description=First release" \
  --data "due_date=2025-12-31" \
  --data "start_date=2025-01-01"
```

### Milestone Burndown Chart

Track progress with burndown charts (Premium/Ultimate):
- Issues opened vs. closed
- Weight completion
- Time remaining

## Project Labels

### Create Labels

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/labels" \
  --data "name=bug" \
  --data "color=#FF0000" \
  --data "description=Bug report" \
  --data "priority=1"
```

### Scoped Labels

Create hierarchical labels:
- `status::open`
- `status::in-progress`
- `status::done`
- `priority::high`
- `priority::low`

### Label Templates

Use label templates for consistency:
```yaml
# .gitlab/labels.yml
- name: "bug"
  color: "#FF0000"
  description: "Bug report"
- name: "feature"
  color: "#00FF00"
  description: "Feature request"
```

## Project Webhooks

Configure webhooks for external integrations:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/hooks" \
  --data "url=https://example.com/webhook" \
  --data "push_events=true" \
  --data "merge_requests_events=true" \
  --data "token=secret-token"
```

See `webhooks.md` for detailed webhook documentation.

## Project Integrations

### Built-in Integrations

**Jira**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/services/jira" \
  --data "url=https://jira.example.com" \
  --data "username=user" \
  --data "password=pass" \
  --data "project_key=PROJECT"
```

**Slack**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/services/slack" \
  --data "webhook=https://hooks.slack.com/services/..." \
  --data "username=GitLab" \
  --data "channel=#general"
```

**Other integrations**:
- Asana
- Microsoft Teams
- Discord
- Mattermost
- Jenkins
- Bamboo
- Datadog
- Prometheus

## Project Statistics

### Get Statistics

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id?statistics=true"
```

**Available statistics**:
- Commit count
- Storage size
- Repository size
- Wiki size
- LFS objects size
- Job artifacts size
- Packages size

### Contribution Analytics

View contributor statistics:
- Commits per contributor
- Additions/deletions
- Date range filtering
- Export to CSV

## Project Snippets

### Create Snippet

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/snippets" \
  --data "title=Example" \
  --data "file_name=example.rb" \
  --data "content=puts 'Hello'" \
  --data "visibility=private"
```

### Snippet Features

- Multiple files per snippet
- Version history
- Comments
- Cloning with git
- Embedding in web pages

## Project Wiki

### Enable Wiki

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id" \
  --data "wiki_enabled=true"
```

### Create Wiki Page

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/wikis" \
  --data "title=Home" \
  --data "content=# Welcome\n\nThis is the home page" \
  --data "format=markdown"
```

**Supported formats**:
- Markdown
- RDoc
- AsciiDoc
- Org

### Clone Wiki

```bash
git clone https://gitlab.com/group/project.wiki.git
```

## Project Access Tokens

Create tokens for project automation:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/access_tokens" \
  --data "name=Deploy Token" \
  --data "scopes[]=api" \
  --data "expires_at=2025-12-31" \
  --data "access_level=40"
```

## Project Deploy Tokens

Special tokens for deployment:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/deploy_tokens" \
  --data "name=Deploy" \
  --data "scopes[]=read_repository" \
  --data "scopes[]=read_registry" \
  --data "expires_at=2025-12-31"
```

## Project Mirror

### Push Mirror

Automatically push changes to remote repository:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/remote_mirrors" \
  --data "url=https://github.com/user/repo.git" \
  --data "enabled=true" \
  --data "only_protected_branches=false"
```

### Pull Mirror

Automatically pull changes from remote repository:

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id" \
  --data "import_url=https://github.com/user/repo.git" \
  --data "mirror=true" \
  --data "mirror_trigger_builds=true"
```

## Project Transfer

### Transfer to Another Namespace

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/transfer" \
  --data "namespace=new-namespace"
```

**Considerations**:
- Updates all URLs
- Maintains git remotes
- Preserves history
- May affect integrations

## Project Archiving

### Archive Project

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/archive"
```

**Effects**:
- Read-only mode
- No new commits/issues/MRs
- Maintains visibility
- Can be unarchived

### Unarchive Project

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/unarchive"
```

## Project Deletion

### Delete Project

```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id"
```

**Delayed deletion** (Premium/Ultimate):
- 7-day grace period
- Recoverable within period
- Permanent after period

### Restore Deleted Project

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/restore"
```

## Project Best Practices

### 1. Organization

- Use descriptive names
- Add comprehensive README
- Document setup process
- Maintain CHANGELOG
- Include LICENSE file

### 2. Branch Protection

- Protect main/master branch
- Require merge requests
- Enable status checks
- Require approvals
- Prevent force push

### 3. Access Control

- Use least privilege
- Regular access reviews
- Remove inactive users
- Use deploy tokens for CI/CD
- Enable 2FA requirement

### 4. Documentation

- Keep README updated
- Document API endpoints
- Include examples
- Maintain wiki
- Use issue templates

### 5. Automation

- Set up CI/CD pipelines
- Automate testing
- Configure automatic merges
- Use scheduled pipelines
- Implement code quality checks

### 6. Security

- Enable security scanning
- Review dependencies
- Configure secret detection
- Use protected variables
- Regular security audits

### 7. Performance

- Archive old projects
- Clean up old branches
- Manage repository size
- Optimize CI/CD pipelines
- Use caching effectively

## Project CLI Management

### Using glab

```bash
# View project
glab repo view group/project

# Clone project
glab repo clone group/project

# Fork project
glab repo fork group/project

# Archive project
glab repo archive group/project

# View project issues
glab issue list

# View merge requests
glab mr list
```

## Project API Examples

### Python

```python
import gitlab

gl = gitlab.Gitlab('https://gitlab.com', private_token='token')

# Get project
project = gl.projects.get('group/project')

# Update project
project.description = 'New description'
project.save()

# Add member
project.members.create({
    'user_id': 123,
    'access_level': gitlab.const.DEVELOPER_ACCESS
})

# Create label
project.labels.create({
    'name': 'bug',
    'color': '#FF0000'
})
```

### JavaScript

```javascript
const { Gitlab } = require('@gitbeaker/node');

const api = new Gitlab({
  token: 'personal-access-token',
});

// Get project
const project = await api.Projects.show('group/project');

// Update project
await api.Projects.edit('group/project', {
  description: 'New description',
});

// Add member
await api.ProjectMembers.add('group/project', 123, 30);
```

## Additional Resources

- Project Settings: https://docs.gitlab.com/ee/user/project/settings/
- Project Import/Export: https://docs.gitlab.com/ee/user/project/settings/import_export.html
- Project Templates: https://docs.gitlab.com/ee/user/project/working_with_projects.html#project-templates
- Protected Branches: https://docs.gitlab.com/ee/user/project/protected_branches.html
