# GitLab Merge Requests Reference

## Overview

Merge Requests (MRs) are the primary mechanism for proposing changes, conducting code reviews, and merging code in GitLab.

## Creating Merge Requests

### Via UI

1. Navigate to project
2. Create/push to branch
3. Click "Create merge request"
4. Fill in details:
   - Title and description
   - Source and target branches
   - Assignees and reviewers
   - Labels and milestone
5. Click "Create merge request"

### Via API

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests" \
  --data "source_branch=feature" \
  --data "target_branch=main" \
  --data "title=Add new feature" \
  --data "description=This MR adds..." \
  --data "assignee_id=123" \
  --data "reviewer_ids[]=456"
```

### Via Git Push Options

```bash
# Create MR on push
git push -o merge_request.create \
  -o merge_request.target=main \
  -o merge_request.title="Add feature" \
  origin feature-branch

# Create and assign
git push -o merge_request.create \
  -o merge_request.assign="username" \
  origin feature-branch

# Create with labels
git push -o merge_request.create \
  -o merge_request.label="bug" \
  -o merge_request.label="priority::high" \
  origin feature-branch
```

### Via CLI

```bash
# Using glab
glab mr create --title "Add feature" \
  --description "Detailed description" \
  --source feature-branch \
  --target main \
  --assignee @username \
  --label bug,feature
```

## Merge Request Details

### Basic Information

```yaml
Title: Add user authentication
Description: |
  Implements JWT authentication for API endpoints.

  Changes:
  - Add JWT middleware
  - Add auth routes
  - Add user model
  - Add tests

  Closes #123
Source Branch: feature/auth
Target Branch: main
Author: @john
Assignee: @jane
Reviewers: @alice, @bob
Labels: feature, security
Milestone: v1.0
```

### Closing Issues

Link issues in description:
- `Closes #123`
- `Fixes #456`
- `Resolves #789`
- `Implements #101`

### Description Templates

Create `.gitlab/merge_request_templates/Default.md`:

```markdown
## What does this MR do?

<!-- Brief description -->

## Related issues

<!-- Link related issues -->
Closes #

## Author's checklist

- [ ] Tests added
- [ ] Documentation updated
- [ ] Changelog entry added
- [ ] Reviewed by peers

## Reviewer's checklist

- [ ] Code follows style guide
- [ ] Tests pass
- [ ] No security issues
- [ ] Performance acceptable
```

## Merge Request Workflow

### Draft/WIP Merge Requests

Mark as draft:
```bash
# In title
Draft: Add feature
WIP: Add feature

# Via API
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid" \
  --data "draft=true"
```

Remove draft status:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid" \
  --data "draft=false"
```

### Assignees and Reviewers

**Assignees**: Responsible for the MR
**Reviewers**: Review and approve the MR

```bash
# Set assignees
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid" \
  --data "assignee_ids[]=123" \
  --data "assignee_ids[]=456"

# Set reviewers
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid" \
  --data "reviewer_ids[]=789"
```

### Labels and Milestones

```bash
# Add labels
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid" \
  --data "labels=bug,feature,priority::high"

# Set milestone
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid" \
  --data "milestone_id=10"
```

## Code Review

### Comments and Discussions

**Add comment**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid/notes" \
  --data "body=Looks good!"
```

**Add line comment**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid/discussions" \
  --data "body=Consider refactoring this" \
  --data "position[base_sha]=abc123" \
  --data "position[start_sha]=def456" \
  --data "position[head_sha]=ghi789" \
  --data "position[new_path]=src/file.js" \
  --data "position[new_line]=42" \
  --data "position[position_type]=text"
```

### Resolvable Discussions

Mark discussions as resolvable:
- Reviewer creates resolvable discussion
- Author resolves after fixing
- Must resolve all before merging (if required)

```bash
# Resolve discussion
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid/discussions/:discussion_id" \
  --data "resolved=true"
```

### Suggestions

Apply code suggestions in reviews:

```markdown
```suggestion
const result = calculate(x, y);
return result * 2;
```
```

Apply suggestion:
- Click "Apply suggestion" button
- Or use API to apply

**Batch apply suggestions**:
1. Select multiple suggestions
2. Click "Apply X suggestions"
3. Commit with single commit

### Code Quality Reports

View code quality changes:
- New issues introduced
- Resolved issues
- Code quality score

Requires CI/CD job with code quality report:
```yaml
code_quality:
  image: docker:stable
  services:
    - docker:stable-dind
  script:
    - docker run --env SOURCE_CODE="$PWD" --volume "$PWD":/code \
        --volume /var/run/docker.sock:/var/run/docker.sock \
        registry.gitlab.com/gitlab-org/ci-cd/codequality:latest /code
  artifacts:
    reports:
      codequality: gl-code-quality-report.json
```

## Merge Request Approvals

### Configure Approval Rules

**Via UI**:
1. Project Settings > Merge requests > Approval rules
2. Add rule
3. Set required approvers
4. Set approval count

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/approval_rules" \
  --data "name=Security Review" \
  --data "approvals_required=2" \
  --data "user_ids[]=123" \
  --data "user_ids[]=456"
```

### Approve Merge Request

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid/approve"
```

### Unapprove

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid/unapprove"
```

### Approval Settings

**Require approvals**:
- Number of approvals needed
- Eligible approvers
- Approval groups
- Code owner approvals

**Prevent self-approval**:
- Author cannot approve own MR

**Prevent committer approval**:
- Users who committed cannot approve

**Require new approvals when pushed**:
- Reset approvals on new commits

## Merge Strategies

### Merge Commit

Creates merge commit with two parents:
```
     C---D feature
    /     \
A---B-------E main
```

### Fast-Forward Merge

No merge commit when possible:
```
A---B---C---D main
            (was feature)
```

### Squash and Merge

Combines all commits into one:
```
A---B---C main
        (squashed feature commits)
```

**Configure in MR**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid" \
  --data "squash=true"
```

### Semi-Linear Merge

Rebase then merge (no fast-forward):
```
     C'--D' feature
    /      \
A---B-------E main
```

## Merging

### Merge When Pipeline Succeeds

Automatically merge after pipeline passes:

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid/merge" \
  --data "merge_when_pipeline_succeeds=true" \
  --data "should_remove_source_branch=true"
```

### Manual Merge

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid/merge" \
  --data "squash=true" \
  --data "should_remove_source_branch=true" \
  --data "merge_commit_message=Custom merge message"
```

### Merge Trains

Queue merges to prevent breaking main branch:

**Enable merge trains**:
1. Project Settings > Merge requests
2. Enable "Merged results pipelines"
3. Enable "Merge trains"

Benefits:
- Serialized merging
- Pipeline runs against merged result
- Prevents breaking main branch
- Automatic rebase

## Merge Conflicts

### View Conflicts

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid/conflicts"
```

### Resolve Conflicts via UI

1. View MR with conflicts
2. Click "Resolve conflicts"
3. Choose conflict resolution
4. Commit resolution

### Resolve Conflicts Locally

```bash
# Fetch latest changes
git fetch origin

# Checkout source branch
git checkout feature-branch

# Merge or rebase
git merge origin/main
# or
git rebase origin/main

# Resolve conflicts in files
# Add resolved files
git add .

# Complete merge/rebase
git commit  # for merge
git rebase --continue  # for rebase

# Push
git push origin feature-branch
```

## Merge Request Diff

### View Diff

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid/changes"
```

### Diff Options

- **Inline diff**: Side-by-side comparison
- **Side-by-side diff**: Parallel view
- **File by file**: Navigate file by file
- **Ignore whitespace**: Skip whitespace changes

### Download Patch

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid.patch" \
  -o merge_request.patch
```

### Download Diff

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid.diff" \
  -o merge_request.diff
```

## Merge Request Pipelines

### Pipeline for Merged Results

Test merge result before merging:

**.gitlab-ci.yml**:
```yaml
workflow:
  rules:
    - if: $CI_MERGE_REQUEST_ID
      when: always
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: always
```

### Require Pipeline Success

Configure in project settings:
- Pipelines must succeed
- Required successful jobs

## Merge Request Dependencies

### Depend on Another MR

Use cross-references in description:
```markdown
Depends on !123
Blocked by !456
```

## Merge Request Metrics

### View Metrics

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests/:mr_iid"
```

**Metrics include**:
- Time to merge
- First comment time
- First approve time
- Review time
- Diff stats (additions/deletions)

### Merge Request Analytics

**Project level**:
- Average time to merge
- Throughput
- Open MRs over time

**Group level** (Premium/Ultimate):
- Cross-project metrics
- Team performance
- Bottleneck identification

## Merge Request Automation

### Quick Actions

Use in comments:
- `/approve` - Approve MR
- `/merge` - Merge MR
- `/close` - Close MR
- `/reopen` - Reopen MR
- `/assign @user` - Assign to user
- `/unassign` - Remove assignee
- `/label ~bug` - Add label
- `/unlabel ~bug` - Remove label
- `/milestone %v1.0` - Set milestone
- `/due 2025-12-31` - Set due date
- `/estimate 2h` - Set time estimate
- `/spend 1h` - Add time spent
- `/draft` - Mark as draft
- `/ready` - Remove draft status

### Merge Request Webhooks

Trigger automation on MR events:

```yaml
# Example CI/CD trigger
review-app:
  script:
    - deploy-review-app.sh
  only:
    - merge_requests
  environment:
    name: review/$CI_MERGE_REQUEST_IID
    url: https://$CI_MERGE_REQUEST_IID.review.example.com
    on_stop: stop-review-app
```

## Code Owners

### CODEOWNERS File

Create `.gitlab/CODEOWNERS`:

```
# Backend team owns all .rb files
*.rb @backend-team

# Frontend team owns React components
/src/components/ @frontend-team

# DevOps owns CI/CD config
/.gitlab-ci.yml @devops
/docker/ @devops

# Security team must approve sensitive files
/config/secrets/ @security-team
/lib/auth/ @security-team

# Everyone in group must approve
/docs/ @group/docs-team
```

**Syntax**:
- `path @user` - User approval
- `path @group` - Group approval
- `path @group/subgroup` - Subgroup approval

### Require Code Owner Approval

1. Project Settings > Merge requests
2. Enable "Require approval from code owners"
3. Code owners must approve changes to their files

## Merge Request Templates

### Create Templates

Create in `.gitlab/merge_request_templates/`:

**Feature.md**:
```markdown
## Feature Description

<!-- What feature does this add? -->

## Implementation Details

<!-- Technical details -->

## Testing

- [ ] Unit tests added
- [ ] Integration tests added
- [ ] Manual testing completed

## Screenshots

<!-- If applicable -->

## Related Issues

Closes #
```

**Bugfix.md**:
```markdown
## Bug Description

<!-- What bug does this fix? -->

## Root Cause

<!-- What caused the bug? -->

## Solution

<!-- How is it fixed? -->

## Testing

- [ ] Bug reproduced
- [ ] Fix verified
- [ ] Regression tests added

Fixes #
```

### Use Template

```bash
# Create MR with template
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/merge_requests" \
  --data "source_branch=feature" \
  --data "target_branch=main" \
  --data "title=Add feature" \
  --data "description=$(cat .gitlab/merge_request_templates/Feature.md)"
```

## Best Practices

### 1. Merge Request Size

- Keep MRs focused and small
- Aim for < 400 lines of changes
- Split large features into multiple MRs
- Easier to review and test

### 2. Title and Description

- Clear, descriptive title
- Explain "why" not just "what"
- Link related issues
- Include screenshots/videos
- Checklist for requirements

### 3. Commits

- Atomic commits
- Descriptive commit messages
- Consider squashing for cleaner history
- Sign commits for verification

### 4. Review Process

- Request specific reviewers
- Respond to all comments
- Resolve discussions
- Re-request review after changes
- Thank reviewers

### 5. Testing

- Add/update tests
- Ensure CI passes
- Manual testing in review app
- Performance testing if needed

### 6. Documentation

- Update relevant docs
- Add API documentation
- Update CHANGELOG
- Include inline comments

### 7. Communication

- Use discussions for questions
- Mark as draft while working
- Update status regularly
- Notify when ready for review

## CLI Examples

### Using glab

```bash
# View MRs
glab mr list
glab mr list --assignee @me
glab mr list --state merged

# View MR details
glab mr view 123

# Create MR
glab mr create

# Checkout MR
glab mr checkout 123

# Approve MR
glab mr approve 123

# Merge MR
glab mr merge 123

# Close MR
glab mr close 123

# View MR diff
glab mr diff 123
```

## API Examples

### Python

```python
import gitlab

gl = gitlab.Gitlab('https://gitlab.com', private_token='token')
project = gl.projects.get('group/project')

# Create MR
mr = project.mergerequests.create({
    'source_branch': 'feature',
    'target_branch': 'main',
    'title': 'Add feature',
    'description': 'Description',
    'assignee_id': 123
})

# Get MR
mr = project.mergerequests.get(1)

# Add comment
mr.notes.create({'body': 'Looks good!'})

# Approve
mr.approve()

# Merge
mr.merge()
```

### JavaScript

```javascript
const { Gitlab } = require('@gitbeaker/node');
const api = new Gitlab({ token: 'token' });

// Create MR
const mr = await api.MergeRequests.create('group/project', 'feature', 'main', 'Add feature');

// Get MR
const mr = await api.MergeRequests.show('group/project', 1);

// Add comment
await api.MergeRequestNotes.create('group/project', 1, 'Looks good!');

// Approve
await api.MergeRequests.approve('group/project', 1);

// Merge
await api.MergeRequests.accept('group/project', 1, {
  should_remove_source_branch: true,
  squash: true
});
```

## Additional Resources

- Merge Requests Documentation: https://docs.gitlab.com/ee/user/project/merge_requests/
- Code Review Guidelines: https://docs.gitlab.com/ee/development/code_review.html
- Approval Rules: https://docs.gitlab.com/ee/user/project/merge_requests/approvals/
- Code Owners: https://docs.gitlab.com/ee/user/project/code_owners.html
