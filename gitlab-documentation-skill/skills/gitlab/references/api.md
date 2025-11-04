# GitLab REST API Reference

## Overview

The GitLab REST API provides programmatic access to GitLab resources. All API requests require authentication and return JSON responses.

## Base URL

```
https://gitlab.com/api/v4
```

For self-hosted GitLab instances:
```
https://your-gitlab-instance.com/api/v4
```

## Authentication

### Personal Access Tokens (Recommended)

Include in request header:
```
PRIVATE-TOKEN: <your_access_token>
```

Or as query parameter:
```
?private_token=<your_access_token>
```

### OAuth 2.0 Token

Include in request header:
```
Authorization: Bearer <oauth_token>
```

### Job Token (CI/CD)

Use within GitLab CI/CD jobs:
```
JOB-TOKEN: <CI_JOB_TOKEN>
```

## Rate Limiting

- Default: 2000 requests per minute per user
- Authenticated requests: counted per user
- Unauthenticated requests: counted per IP
- Response headers include rate limit info:
  - `RateLimit-Limit`: Request limit
  - `RateLimit-Remaining`: Remaining requests
  - `RateLimit-Reset`: Reset time (Unix timestamp)

## Common Parameters

### Pagination

- `page`: Page number (default: 1)
- `per_page`: Items per page (default: 20, max: 100)

Headers in response:
- `X-Total`: Total number of items
- `X-Total-Pages`: Total number of pages
- `X-Per-Page`: Items per page
- `X-Page`: Current page
- `X-Next-Page`: Next page number
- `X-Prev-Page`: Previous page number

### Filtering

- `search`: Search by specific fields
- `order_by`: Order results (e.g., `created_at`, `updated_at`, `name`)
- `sort`: Sort direction (`asc` or `desc`)

## Core API Resources

### Projects

**List all projects**
```
GET /projects
```

**Get project by ID**
```
GET /projects/:id
```

**Create project**
```
POST /projects
```

Parameters:
- `name` (required): Project name
- `path`: Repository path (defaults to name)
- `namespace_id`: Namespace for project
- `description`: Project description
- `visibility`: `private`, `internal`, or `public`
- `initialize_with_readme`: Boolean

**Update project**
```
PUT /projects/:id
```

**Delete project**
```
DELETE /projects/:id
```

**Fork project**
```
POST /projects/:id/fork
```

**List project members**
```
GET /projects/:id/members
```

**Add project member**
```
POST /projects/:id/members
```

Parameters:
- `user_id` (required): User ID
- `access_level` (required): 10 (Guest), 20 (Reporter), 30 (Developer), 40 (Maintainer), 50 (Owner)

### Repository

**List repository files**
```
GET /projects/:id/repository/tree
```

Parameters:
- `path`: Directory path
- `ref`: Branch/tag name
- `recursive`: Boolean for recursive listing

**Get file content**
```
GET /projects/:id/repository/files/:file_path
```

Parameters:
- `ref` (required): Branch/tag/commit

**Create file**
```
POST /projects/:id/repository/files/:file_path
```

Parameters:
- `branch` (required): Branch name
- `content` (required): File content
- `commit_message` (required): Commit message
- `encoding`: `text` or `base64`

**Update file**
```
PUT /projects/:id/repository/files/:file_path
```

**Delete file**
```
DELETE /projects/:id/repository/files/:file_path
```

**Get repository archive**
```
GET /projects/:id/repository/archive
```

### Commits

**List commits**
```
GET /projects/:id/repository/commits
```

Parameters:
- `ref_name`: Branch/tag name
- `since`: Start date (ISO 8601 format)
- `until`: End date
- `path`: File path to filter commits

**Get single commit**
```
GET /projects/:id/repository/commits/:sha
```

**Create commit**
```
POST /projects/:id/repository/commits
```

Parameters:
- `branch` (required): Branch name
- `commit_message` (required): Commit message
- `actions` (required): Array of actions

Action types:
- `create`: Create file
- `delete`: Delete file
- `move`: Move file
- `update`: Update file
- `chmod`: Change permissions

**Get commit diff**
```
GET /projects/:id/repository/commits/:sha/diff
```

**Get commit comments**
```
GET /projects/:id/repository/commits/:sha/comments
```

### Branches

**List branches**
```
GET /projects/:id/repository/branches
```

**Get single branch**
```
GET /projects/:id/repository/branches/:branch
```

**Create branch**
```
POST /projects/:id/repository/branches
```

Parameters:
- `branch` (required): Branch name
- `ref` (required): Source branch/tag/commit

**Delete branch**
```
DELETE /projects/:id/repository/branches/:branch
```

**Protect branch**
```
POST /projects/:id/protected_branches
```

Parameters:
- `name` (required): Branch name or wildcard
- `push_access_level`: 0 (No access), 30 (Developer), 40 (Maintainer)
- `merge_access_level`: Similar access levels
- `allow_force_push`: Boolean

### Merge Requests

**List merge requests**
```
GET /projects/:id/merge_requests
```

Parameters:
- `state`: `opened`, `closed`, `merged`, `all`
- `scope`: `created_by_me`, `assigned_to_me`, `all`
- `source_branch`: Filter by source branch
- `target_branch`: Filter by target branch

**Get merge request**
```
GET /projects/:id/merge_requests/:merge_request_iid
```

**Create merge request**
```
POST /projects/:id/merge_requests
```

Parameters:
- `source_branch` (required): Source branch
- `target_branch` (required): Target branch
- `title` (required): MR title
- `description`: MR description
- `assignee_id`: Assignee user ID
- `reviewer_ids`: Array of reviewer user IDs
- `labels`: Comma-separated label names
- `remove_source_branch`: Boolean

**Update merge request**
```
PUT /projects/:id/merge_requests/:merge_request_iid
```

**Accept/Merge MR**
```
PUT /projects/:id/merge_requests/:merge_request_iid/merge
```

Parameters:
- `merge_commit_message`: Custom merge message
- `squash_commit_message`: Squash commit message
- `squash`: Boolean
- `should_remove_source_branch`: Boolean

**Get MR changes**
```
GET /projects/:id/merge_requests/:merge_request_iid/changes
```

**Get MR commits**
```
GET /projects/:id/merge_requests/:merge_request_iid/commits
```

**Get MR approvals**
```
GET /projects/:id/merge_requests/:merge_request_iid/approvals
```

**Approve MR**
```
POST /projects/:id/merge_requests/:merge_request_iid/approve
```

### Issues

**List issues**
```
GET /projects/:id/issues
```

Parameters:
- `state`: `opened`, `closed`, `all`
- `labels`: Comma-separated label names
- `milestone`: Milestone title
- `assignee_id`: Assignee user ID
- `author_id`: Author user ID

**Get issue**
```
GET /projects/:id/issues/:issue_iid
```

**Create issue**
```
POST /projects/:id/issues
```

Parameters:
- `title` (required): Issue title
- `description`: Issue description
- `assignee_ids`: Array of assignee IDs
- `labels`: Comma-separated labels
- `milestone_id`: Milestone ID
- `due_date`: Due date (YYYY-MM-DD)
- `weight`: Issue weight

**Update issue**
```
PUT /projects/:id/issues/:issue_iid
```

**Close issue**
```
PUT /projects/:id/issues/:issue_iid
```

Parameters:
- `state_event`: `close`

**Delete issue**
```
DELETE /projects/:id/issues/:issue_iid
```

### Pipelines

**List pipelines**
```
GET /projects/:id/pipelines
```

Parameters:
- `ref`: Branch/tag name
- `status`: `running`, `pending`, `success`, `failed`, `canceled`, `skipped`
- `scope`: `running`, `pending`, `finished`, `branches`, `tags`

**Get pipeline**
```
GET /projects/:id/pipelines/:pipeline_id
```

**Create pipeline**
```
POST /projects/:id/pipeline
```

Parameters:
- `ref` (required): Branch/tag name
- `variables`: Array of pipeline variables

**Retry pipeline**
```
POST /projects/:id/pipelines/:pipeline_id/retry
```

**Cancel pipeline**
```
POST /projects/:id/pipelines/:pipeline_id/cancel
```

**Delete pipeline**
```
DELETE /projects/:id/pipelines/:pipeline_id
```

**Get pipeline variables**
```
GET /projects/:id/pipelines/:pipeline_id/variables
```

### Jobs

**List project jobs**
```
GET /projects/:id/jobs
```

Parameters:
- `scope`: `created`, `pending`, `running`, `failed`, `success`, `canceled`, `skipped`, `manual`

**Get pipeline jobs**
```
GET /projects/:id/pipelines/:pipeline_id/jobs
```

**Get job**
```
GET /projects/:id/jobs/:job_id
```

**Get job artifacts**
```
GET /projects/:id/jobs/:job_id/artifacts
```

**Download artifact file**
```
GET /projects/:id/jobs/:job_id/artifacts/:artifact_path
```

**Get job trace**
```
GET /projects/:id/jobs/:job_id/trace
```

**Retry job**
```
POST /projects/:id/jobs/:job_id/retry
```

**Cancel job**
```
POST /projects/:id/jobs/:job_id/cancel
```

**Erase job**
```
POST /projects/:id/jobs/:job_id/erase
```

**Play manual job**
```
POST /projects/:id/jobs/:job_id/play
```

### Users

**List users**
```
GET /users
```

**Get user**
```
GET /users/:id
```

**Get current user**
```
GET /user
```

**Create user** (Admin only)
```
POST /users
```

**Update user**
```
PUT /users/:id
```

**Delete user** (Admin only)
```
DELETE /users/:id
```

### Groups

**List groups**
```
GET /groups
```

**Get group**
```
GET /groups/:id
```

**Create group**
```
POST /groups
```

Parameters:
- `name` (required): Group name
- `path` (required): Group path
- `description`: Group description
- `visibility`: `private`, `internal`, `public`

**Update group**
```
PUT /groups/:id
```

**Delete group**
```
DELETE /groups/:id
```

**List group members**
```
GET /groups/:id/members
```

**Add group member**
```
POST /groups/:id/members
```

### Runners

**List runners**
```
GET /runners
```

**Get runner**
```
GET /runners/:id
```

**Update runner**
```
PUT /runners/:id
```

Parameters:
- `description`: Runner description
- `active`: Boolean
- `tag_list`: Array of tags
- `run_untagged`: Boolean
- `locked`: Boolean
- `access_level`: `not_protected`, `ref_protected`

**Delete runner**
```
DELETE /runners/:id
```

**List project runners**
```
GET /projects/:id/runners
```

**Enable runner for project**
```
POST /projects/:id/runners
```

**Disable runner for project**
```
DELETE /projects/:id/runners/:runner_id
```

### Variables

**List project variables**
```
GET /projects/:id/variables
```

**Get variable**
```
GET /projects/:id/variables/:key
```

**Create variable**
```
POST /projects/:id/variables
```

Parameters:
- `key` (required): Variable key
- `value` (required): Variable value
- `variable_type`: `env_var` or `file`
- `protected`: Boolean
- `masked`: Boolean
- `environment_scope`: Environment scope (default: `*`)

**Update variable**
```
PUT /projects/:id/variables/:key
```

**Delete variable**
```
DELETE /projects/:id/variables/:key
```

### Webhooks

**List project webhooks**
```
GET /projects/:id/hooks
```

**Get webhook**
```
GET /projects/:id/hooks/:hook_id
```

**Create webhook**
```
POST /projects/:id/hooks
```

Parameters:
- `url` (required): Webhook URL
- `token`: Secret token
- `push_events`: Boolean
- `issues_events`: Boolean
- `merge_requests_events`: Boolean
- `wiki_page_events`: Boolean
- `pipeline_events`: Boolean
- `job_events`: Boolean
- `deployment_events`: Boolean
- `enable_ssl_verification`: Boolean

**Update webhook**
```
PUT /projects/:id/hooks/:hook_id
```

**Delete webhook**
```
DELETE /projects/:id/hooks/:hook_id
```

**Test webhook**
```
POST /projects/:id/hooks/:hook_id/test/:trigger
```

Triggers: `push_events`, `issues_events`, `merge_requests_events`, etc.

### Tags

**List tags**
```
GET /projects/:id/repository/tags
```

**Get tag**
```
GET /projects/:id/repository/tags/:tag_name
```

**Create tag**
```
POST /projects/:id/repository/tags
```

Parameters:
- `tag_name` (required): Tag name
- `ref` (required): Source commit SHA
- `message`: Tag message
- `release_description`: Release description

**Delete tag**
```
DELETE /projects/:id/repository/tags/:tag_name
```

### Releases

**List releases**
```
GET /projects/:id/releases
```

**Get release**
```
GET /projects/:id/releases/:tag_name
```

**Create release**
```
POST /projects/:id/releases
```

Parameters:
- `name` (required): Release name
- `tag_name` (required): Tag name
- `description`: Release description
- `ref`: Commit SHA, branch, or tag (required if tag doesn't exist)
- `milestones`: Array of milestone titles
- `assets`: Release assets

**Update release**
```
PUT /projects/:id/releases/:tag_name
```

**Delete release**
```
DELETE /projects/:id/releases/:tag_name
```

## Error Handling

### HTTP Status Codes

- `200 OK`: Request successful
- `201 Created`: Resource created
- `204 No Content`: Successful deletion
- `400 Bad Request`: Invalid parameters
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `409 Conflict`: Resource conflict
- `422 Unprocessable Entity`: Validation error
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

### Error Response Format

```json
{
  "message": "Error message",
  "error": "error_type"
}
```

Validation errors:
```json
{
  "message": {
    "field": ["error message"]
  }
}
```

## Best Practices

1. **Use Personal Access Tokens**: More secure than OAuth for scripts and automation
2. **Handle Rate Limits**: Implement exponential backoff for rate limit errors
3. **Use Pagination**: For large datasets, implement proper pagination
4. **Cache Responses**: Cache API responses when appropriate
5. **Error Handling**: Always handle error responses gracefully
6. **Project IDs**: Use URL-encoded project paths (e.g., `namespace%2Fproject`)
7. **Webhooks**: Use webhook tokens and verify SSL certificates
8. **Minimize Requests**: Batch operations when possible
9. **Use Specific Scopes**: Request only necessary API scopes
10. **Monitor Usage**: Track API usage through rate limit headers

## Additional Resources

- Official GitLab API Documentation: https://docs.gitlab.com/ee/api/
- API Libraries: Available for Python, Ruby, JavaScript, Go, etc.
- GraphQL API: Available at `/api/graphql` for more efficient queries
