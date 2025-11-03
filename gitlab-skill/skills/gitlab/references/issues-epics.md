# GitLab Issues and Epics Reference (Ultimate)

## Overview

Issues are the fundamental unit of work in GitLab. GitLab Ultimate adds Epics for managing groups of related issues across multiple projects.

## Issues

### Creating Issues

**Via UI**:
1. Navigate to project
2. Click Issues > New issue
3. Fill in details
4. Create issue

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues" \
  --data "title=Bug in authentication" \
  --data "description=Login fails when..." \
  --data "assignee_ids[]=123" \
  --data "labels=bug,priority::high" \
  --data "milestone_id=5" \
  --data "weight=3"
```

**Via CLI**:
```bash
glab issue create --title "Bug fix" --description "Details" --assignee @user
```

### Issue Fields

**Basic Fields**:
- Title (required)
- Description (Markdown)
- Assignees (multiple)
- Labels (multiple)
- Milestone
- Due date
- Weight (Ultimate)

**Advanced Fields (Ultimate)**:
- Epic
- Health status
- Iteration
- Time tracking (estimate/spent)

### Issue Types

**Standard Issue Types**:
- Issue (default)
- Incident
- Test Case (Ultimate)
- Requirement (Ultimate)

**Configure issue type**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues/:issue_iid" \
  --data "issue_type=incident"
```

### Issue Templates

Create in `.gitlab/issue_templates/`:

**Bug.md**:
```markdown
## Bug Description
<!-- What's broken? -->

## Steps to Reproduce
1.
2.
3.

## Expected Behavior
<!-- What should happen? -->

## Actual Behavior
<!-- What actually happens? -->

## Environment
- GitLab Version:
- Browser:
- OS:

## Screenshots
<!-- If applicable -->

## Additional Context
<!-- Any other info -->

/label ~bug
/assign @maintainer
```

**Feature.md**:
```markdown
## Feature Request

### Problem
<!-- What problem does this solve? -->

### Proposed Solution
<!-- How should we solve it? -->

### Alternatives Considered
<!-- What other solutions were considered? -->

### Additional Context

/label ~feature
/milestone %Next
```

### Issue Boards

**Create Board**:
1. Project > Issues > Boards
2. Create new board
3. Add lists (by label, assignee, milestone)

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/boards" \
  --data "name=Development Board"
```

**Board Lists**:
```bash
# Add list by label
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/boards/:board_id/lists" \
  --data "label_id=123"

# Add list by assignee
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/boards/:board_id/lists" \
  --data "assignee_id=456"
```

**Scoped Boards (Premium/Ultimate)**:
- Filter by milestone
- Filter by assignee
- Filter by labels
- Multiple boards per project

### Issue Weights (Ultimate)

Assign complexity points:

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues/:issue_iid" \
  --data "weight=5"
```

**Use in planning**:
- Sprint capacity planning
- Velocity tracking
- Burndown charts

### Health Status (Ultimate)

Track issue health:

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues/:issue_iid" \
  --data "health_status=on_track"
```

**Health Status Options**:
- `on_track` - Green
- `needs_attention` - Yellow
- `at_risk` - Red

### Time Tracking

**Estimate time**:
```
/estimate 2h
/estimate 1d 4h
/estimate 1w
```

**Log time spent**:
```
/spend 1h
/spend 30m
/spend 2d
```

**Via API**:
```bash
# Set estimate
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues/:issue_iid/time_estimate" \
  --data "duration=2h"

# Add time spent
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues/:issue_iid/add_spent_time" \
  --data "duration=1h"
```

## Epics (Ultimate)

### Overview

Epics track groups of related issues across multiple projects. Available at group level only.

### Creating Epics

**Via UI**:
1. Navigate to Group > Epics
2. New Epic
3. Fill in details

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/epics" \
  --data "title=Q4 Authentication Improvements" \
  --data "description=Improve authentication security" \
  --data "start_date=2025-10-01" \
  --data "end_date=2025-12-31" \
  --data "labels=security,q4"
```

### Epic Hierarchy

**Parent-Child Epics**:
```bash
# Add child epic
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/epics/:epic_id/epics" \
  --data "child_epic_id=456"

# Remove child epic
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/epics/:epic_id/epics/:child_epic_id"
```

**Epic Structure Example**:
```
Epic: Platform Modernization
├── Epic: Backend Migration
│   ├── Issue: Migrate auth service
│   ├── Issue: Migrate API gateway
│   └── Issue: Update dependencies
├── Epic: Frontend Rewrite
│   ├── Issue: Setup React
│   ├── Issue: Component library
│   └── Issue: Migration plan
└── Epic: Database Optimization
    ├── Issue: Add indexes
    ├── Issue: Query optimization
    └── Issue: Partitioning strategy
```

### Adding Issues to Epics

**Via UI**:
- Issue sidebar > Epic > Select epic

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:group_id/epics/:epic_id/issues/:issue_id"
```

**Via quick action**:
```
/epic &123
```

### Epic Boards (Ultimate)

Visualize epics across projects:

**Create Epic Board**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/epic_boards" \
  --data "name=Q4 Roadmap"
```

**Add lists**:
```bash
# By label
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/epic_boards/:board_id/lists" \
  --data "label_id=789"
```

### Roadmaps (Ultimate)

Visual timeline of epics:

**Access**: Group > Epics > Roadmap

**Features**:
- Gantt chart view
- Drag to adjust dates
- Filter by label, author, milestone
- Zoom levels (quarters, months, weeks)
- Progress tracking

**Example Use Cases**:
- Product roadmap planning
- Release planning
- Portfolio management
- Stakeholder communication

### Epic Relationships

**Link related epics**:
```bash
# Create epic link
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/epics/:epic_id/related_epics" \
  --data "target_epic_id=456" \
  --data "link_type=relates_to"
```

**Link types**:
- `relates_to` - Related
- `blocks` - This blocks target
- `is_blocked_by` - This is blocked by target

## Iterations (Ultimate)

Time-boxed periods for planning:

### Create Iteration

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/iterations" \
  --data "title=Sprint 42" \
  --data "description=Two week sprint" \
  --data "start_date=2025-11-01" \
  --data "due_date=2025-11-14"
```

### Iteration Cadences (Ultimate)

Automate iteration creation:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/iterations/cadences" \
  --data "title=Development Sprints" \
  --data "duration_in_weeks=2" \
  --data "iterations_in_advance=3" \
  --data "start_date=2025-01-01"
```

### Assign Issues to Iterations

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues/:issue_iid" \
  --data "iteration_id=123"
```

**Via quick action**:
```
/iteration *iteration:42
```

## Issue Analytics (Ultimate)

### Issue Statistics

**Group-level analytics**:
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/issues_statistics"
```

**Response**:
```json
{
  "statistics": {
    "counts": {
      "all": 150,
      "closed": 120,
      "opened": 30
    }
  }
}
```

### Burndown Charts (Ultimate)

Track issue completion over time:

**Milestone burndown**:
- View in milestone
- Shows ideal vs actual progress
- Scope changes highlighted

**Epic burndown**:
- Track epic progress
- Aggregate child issues
- Weight-based calculations

### Burnup Charts (Ultimate)

Track scope and progress:
- Total scope line
- Completed work line
- Scope change visualization

## Advanced Features (Ultimate)

### Requirements Management

Create and track requirements:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/requirements" \
  --data "title=System must support SSO" \
  --data "description=Single sign-on requirement"
```

**Requirement states**:
- `opened` - New requirement
- `satisfied` - Requirement met
- `failed` - Requirement not met
- `missing` - No test

### Test Cases

Manage test cases linked to requirements:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues" \
  --data "title=Test SSO login" \
  --data "issue_type=test_case" \
  --data "description=Test procedure..."
```

### Issue Links

**Link related issues**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues/:issue_iid/links" \
  --data "target_project_id=456" \
  --data "target_issue_iid=789" \
  --data "link_type=relates_to"
```

**Link types**:
- `relates_to` - Related to
- `blocks` - Blocks
- `is_blocked_by` - Is blocked by

**Via quick actions**:
```
/relate #123
/blocks #456
/blocked_by #789
```

### Issue Service Desk

Email-to-issue conversion:

**Enable Service Desk**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/services/service-desk" \
  --data "service_desk_enabled=true"
```

**Features**:
- Custom email address
- Auto-response templates
- Issue creation from emails
- Email notifications

### Issue Export/Import

**Export issues**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues/export"
```

**Import issues**:
- Via CSV upload
- Bulk import from file
- Preserve metadata

## Automation

### Issue Templates

**`.gitlab/issue_templates/`**:
- Create reusable templates
- Quick actions in templates
- Variable substitution

### Scheduled Issue Creation

```yaml
# .gitlab-ci.yml
create-weekly-report:
  script:
    - |
      curl --request POST --header "PRIVATE-TOKEN: $CI_JOB_TOKEN" \
        "$CI_API_V4_URL/projects/$CI_PROJECT_ID/issues" \
        --data "title=Weekly Report $(date +%Y-%m-%d)" \
        --data "description=Weekly status report" \
        --data "labels=report"
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
```

### Issue Webhooks

Trigger automation on issue events:

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def issue_webhook():
    data = request.json

    if data['object_kind'] == 'issue':
        action = data['object_attributes']['action']
        title = data['object_attributes']['title']

        if action == 'open':
            # Auto-assign based on labels
            if 'bug' in data['labels']:
                assign_to_oncall()
            elif 'security' in data['labels']:
                assign_to_security_team()
                notify_security_channel()

        elif action == 'close':
            # Update documentation
            update_changelog(title)

    return {'status': 'processed'}, 200
```

## Best Practices

### Issue Management

1. **Clear titles**: Descriptive, actionable titles
2. **Detailed descriptions**: Context, acceptance criteria
3. **Proper labeling**: Consistent label taxonomy
4. **Regular updates**: Keep stakeholders informed
5. **Close when done**: Don't leave stale issues

### Epic Planning

1. **Strategic alignment**: Epics align with goals
2. **Reasonable scope**: 3-6 months typical
3. **Clear outcomes**: Define success criteria
4. **Regular reviews**: Track progress weekly
5. **Communication**: Keep stakeholders updated

### Iteration Planning

1. **Consistent duration**: 1-2 week sprints
2. **Capacity planning**: Use weights
3. **Clear goals**: Define sprint objectives
4. **Daily updates**: Track progress
5. **Retrospectives**: Learn and improve

### Board Organization

1. **Clear workflow**: Define columns
2. **WIP limits**: Prevent bottlenecks
3. **Regular grooming**: Keep board clean
4. **Visualization**: Use labels effectively
5. **Automation**: Automate transitions

## API Examples

### Python

```python
import gitlab

gl = gitlab.Gitlab('https://gitlab.com', private_token='token')

# Create issue
project = gl.projects.get('group/project')
issue = project.issues.create({
    'title': 'Bug fix needed',
    'description': 'Details here',
    'labels': ['bug', 'priority::high'],
    'assignee_ids': [123]
})

# Create epic (Ultimate)
group = gl.groups.get('group-id')
epic = group.epics.create({
    'title': 'Q4 Initiative',
    'description': 'Description',
    'start_date': '2025-10-01',
    'end_date': '2025-12-31'
})

# Add issue to epic
epic.issues.create({'issue_id': issue.id})

# Create iteration (Ultimate)
iteration = group.iterations.create({
    'title': 'Sprint 42',
    'start_date': '2025-11-01',
    'due_date': '2025-11-14'
})
```

### JavaScript

```javascript
const { Gitlab } = require('@gitbeaker/node');
const api = new Gitlab({ token: 'token' });

// Create issue
const issue = await api.Issues.create('group/project', {
  title: 'Bug fix needed',
  description: 'Details',
  labels: 'bug,priority::high',
  assignee_ids: [123]
});

// Create epic (Ultimate)
const epic = await api.Epics.create('group-id', {
  title: 'Q4 Initiative',
  description: 'Description'
});

// Link issue to epic
await api.EpicIssues.create('group-id', epic.id, issue.id);
```

## Additional Resources

- Issues Documentation: https://docs.gitlab.com/ee/user/project/issues/
- Epics Documentation: https://docs.gitlab.com/ee/user/group/epics/
- Roadmaps: https://docs.gitlab.com/ee/user/group/roadmap/
- Issue Boards: https://docs.gitlab.com/ee/user/project/issue_board.html
- Iterations: https://docs.gitlab.com/ee/user/group/iterations/
