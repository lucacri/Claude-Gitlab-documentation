# GitLab Webhooks Reference

## Overview

GitLab webhooks allow external services to be notified when certain events happen in GitLab. When triggered, GitLab sends an HTTP POST request with event data to the configured URL.

## Webhook Configuration

### Creating a Webhook

**Via UI**: Project Settings > Webhooks

**Via API**:
```
POST /projects/:id/hooks
```

Parameters:
- `url` (required): The webhook URL
- `token`: Secret token for request verification
- `push_events`: Trigger on push events (default: true)
- `tag_push_events`: Trigger on tag push events
- `issues_events`: Trigger on issue events
- `confidential_issues_events`: Trigger on confidential issue events
- `merge_requests_events`: Trigger on merge request events
- `wiki_page_events`: Trigger on wiki page events
- `deployment_events`: Trigger on deployment events
- `job_events`: Trigger on job events
- `pipeline_events`: Trigger on pipeline events
- `releases_events`: Trigger on release events
- `enable_ssl_verification`: Verify SSL certificates (default: true)

### Webhook Security

**Secret Token**: Include a secret token to verify requests:

```python
import hmac
import hashlib

def verify_webhook(payload, signature, secret):
    expected = hmac.new(
        secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

**Request Headers**:
- `X-Gitlab-Token`: Contains the secret token (if configured)
- `X-Gitlab-Event`: Event type name
- `User-Agent`: GitLab/{version}

## Event Types

### Push Events

**Trigger**: Code pushed to repository

**Event Header**: `X-Gitlab-Event: Push Hook`

**Payload Structure**:
```json
{
  "object_kind": "push",
  "event_name": "push",
  "before": "95790bf891e76fee5e1747ab589903a6a1f80f22",
  "after": "da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
  "ref": "refs/heads/main",
  "checkout_sha": "da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
  "user_id": 4,
  "user_name": "John Doe",
  "user_username": "jdoe",
  "user_email": "john@example.com",
  "user_avatar": "https://gitlab.com/uploads/user/avatar/4/avatar.jpg",
  "project_id": 15,
  "project": {
    "id": 15,
    "name": "My Project",
    "description": "Project description",
    "web_url": "https://gitlab.com/namespace/project",
    "avatar_url": null,
    "git_ssh_url": "git@gitlab.com:namespace/project.git",
    "git_http_url": "https://gitlab.com/namespace/project.git",
    "namespace": "Namespace",
    "visibility_level": 0,
    "path_with_namespace": "namespace/project",
    "default_branch": "main"
  },
  "commits": [
    {
      "id": "da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
      "message": "Fix bug in authentication",
      "title": "Fix bug in authentication",
      "timestamp": "2025-01-15T12:30:00+00:00",
      "url": "https://gitlab.com/namespace/project/-/commit/da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
      "author": {
        "name": "John Doe",
        "email": "john@example.com"
      },
      "added": ["new-file.txt"],
      "modified": ["existing-file.txt"],
      "removed": ["old-file.txt"]
    }
  ],
  "total_commits_count": 1,
  "repository": {
    "name": "My Project",
    "url": "git@gitlab.com:namespace/project.git",
    "description": "Project description",
    "homepage": "https://gitlab.com/namespace/project"
  }
}
```

### Tag Push Events

**Trigger**: Tag created or deleted

**Event Header**: `X-Gitlab-Event: Tag Push Hook`

**Payload Structure**:
```json
{
  "object_kind": "tag_push",
  "event_name": "tag_push",
  "before": "0000000000000000000000000000000000000000",
  "after": "82b3d5ae55f7080f1e6022629cdb57bfae7cccc7",
  "ref": "refs/tags/v1.0.0",
  "checkout_sha": "82b3d5ae55f7080f1e6022629cdb57bfae7cccc7",
  "user_id": 4,
  "user_name": "John Doe",
  "user_avatar": "https://gitlab.com/uploads/user/avatar/4/avatar.jpg",
  "project_id": 15,
  "project": { /* project details */ },
  "commits": [ /* commit details */ ],
  "total_commits_count": 0,
  "repository": { /* repository details */ }
}
```

**Note**: `before` is all zeros for tag creation, `after` is all zeros for tag deletion.

### Issue Events

**Trigger**: Issue created, updated, closed, or reopened

**Event Header**: `X-Gitlab-Event: Issue Hook`

**Payload Structure**:
```json
{
  "object_kind": "issue",
  "event_type": "issue",
  "user": {
    "id": 4,
    "name": "John Doe",
    "username": "jdoe",
    "avatar_url": "https://gitlab.com/uploads/user/avatar/4/avatar.jpg",
    "email": "john@example.com"
  },
  "project": { /* project details */ },
  "object_attributes": {
    "id": 301,
    "title": "Bug in login feature",
    "assignee_ids": [5, 6],
    "assignee_id": 5,
    "author_id": 4,
    "project_id": 15,
    "created_at": "2025-01-15 12:30:00 UTC",
    "updated_at": "2025-01-15 13:00:00 UTC",
    "position": 0,
    "branch_name": null,
    "description": "Login page crashes when...",
    "milestone_id": 10,
    "state": "opened",
    "state_id": 1,
    "iid": 23,
    "url": "https://gitlab.com/namespace/project/-/issues/23",
    "action": "open",
    "labels": [
      {
        "id": 100,
        "title": "bug",
        "color": "#FF0000",
        "project_id": 15,
        "created_at": "2024-01-01 00:00:00 UTC",
        "updated_at": "2024-01-01 00:00:00 UTC"
      }
    ]
  },
  "assignees": [
    {
      "id": 5,
      "name": "Jane Smith",
      "username": "jsmith",
      "avatar_url": "https://gitlab.com/uploads/user/avatar/5/avatar.jpg"
    }
  ],
  "labels": [ /* label details */ ],
  "changes": {
    "updated_at": {
      "previous": "2025-01-15 12:30:00 UTC",
      "current": "2025-01-15 13:00:00 UTC"
    },
    "title": {
      "previous": "Old title",
      "current": "Bug in login feature"
    }
  }
}
```

**Action Values**: `open`, `update`, `close`, `reopen`

### Merge Request Events

**Trigger**: MR created, updated, merged, or closed

**Event Header**: `X-Gitlab-Event: Merge Request Hook`

**Payload Structure**:
```json
{
  "object_kind": "merge_request",
  "event_type": "merge_request",
  "user": { /* user details */ },
  "project": { /* project details */ },
  "object_attributes": {
    "id": 99,
    "iid": 1,
    "target_branch": "main",
    "source_branch": "feature-branch",
    "source_project_id": 15,
    "author_id": 4,
    "assignee_ids": [5],
    "assignee_id": 5,
    "reviewer_ids": [6, 7],
    "title": "Add new authentication feature",
    "created_at": "2025-01-15 12:00:00 UTC",
    "updated_at": "2025-01-15 14:00:00 UTC",
    "milestone_id": 10,
    "state": "opened",
    "state_id": 1,
    "merge_status": "can_be_merged",
    "target_project_id": 15,
    "description": "This MR adds...",
    "url": "https://gitlab.com/namespace/project/-/merge_requests/1",
    "source": { /* source project */ },
    "target": { /* target project */ },
    "last_commit": {
      "id": "da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
      "message": "Update authentication",
      "timestamp": "2025-01-15T13:30:00+00:00",
      "url": "https://gitlab.com/namespace/project/-/commit/da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
      "author": {
        "name": "John Doe",
        "email": "john@example.com"
      }
    },
    "work_in_progress": false,
    "draft": false,
    "action": "open",
    "assignee": { /* assignee details */ },
    "labels": [ /* label details */ ]
  },
  "labels": [ /* label details */ ],
  "changes": { /* changed attributes */ },
  "assignees": [ /* assignee details */ ],
  "reviewers": [ /* reviewer details */ ]
}
```

**Action Values**: `open`, `update`, `close`, `reopen`, `merge`, `approved`, `unapproved`

### Pipeline Events

**Trigger**: Pipeline status changes

**Event Header**: `X-Gitlab-Event: Pipeline Hook`

**Payload Structure**:
```json
{
  "object_kind": "pipeline",
  "object_attributes": {
    "id": 31,
    "ref": "main",
    "tag": false,
    "sha": "bcbb5ec396a2c0f828686f14fac9b80b780504f2",
    "before_sha": "bcbb5ec396a2c0f828686f14fac9b80b780504f2",
    "source": "push",
    "status": "success",
    "detailed_status": "passed",
    "stages": ["build", "test", "deploy"],
    "created_at": "2025-01-15 12:00:00 UTC",
    "finished_at": "2025-01-15 12:15:00 UTC",
    "duration": 900,
    "queued_duration": 10,
    "variables": [
      {
        "key": "ENVIRONMENT",
        "value": "production"
      }
    ]
  },
  "merge_request": {
    "id": 1,
    "iid": 1,
    "title": "Add feature",
    "source_branch": "feature-branch",
    "source_project_id": 15,
    "target_branch": "main",
    "target_project_id": 15,
    "state": "opened",
    "merge_status": "can_be_merged",
    "url": "https://gitlab.com/namespace/project/-/merge_requests/1"
  },
  "user": { /* user details */ },
  "project": { /* project details */ },
  "commit": {
    "id": "bcbb5ec396a2c0f828686f14fac9b80b780504f2",
    "message": "Add new feature",
    "title": "Add new feature",
    "timestamp": "2025-01-15T11:55:00+00:00",
    "url": "https://gitlab.com/namespace/project/-/commit/bcbb5ec396a2c0f828686f14fac9b80b780504f2",
    "author": {
      "name": "John Doe",
      "email": "john@example.com"
    }
  },
  "builds": [
    {
      "id": 380,
      "stage": "test",
      "name": "unit-tests",
      "status": "success",
      "created_at": "2025-01-15 12:00:00 UTC",
      "started_at": "2025-01-15 12:01:00 UTC",
      "finished_at": "2025-01-15 12:05:00 UTC",
      "duration": 240,
      "queued_duration": 60,
      "when": "on_success",
      "manual": false,
      "allow_failure": false,
      "user": { /* user details */ },
      "runner": {
        "id": 1,
        "description": "runner-1",
        "active": true,
        "runner_type": "project_type",
        "is_shared": false,
        "tags": ["docker", "linux"]
      },
      "artifacts_file": {
        "filename": "artifacts.zip",
        "size": 1024000
      },
      "environment": null
    }
  ]
}
```

**Status Values**: `pending`, `running`, `success`, `failed`, `canceled`, `skipped`

### Job Events

**Trigger**: Job status changes

**Event Header**: `X-Gitlab-Event: Job Hook`

**Payload Structure**:
```json
{
  "object_kind": "build",
  "ref": "main",
  "tag": false,
  "before_sha": "95790bf891e76fee5e1747ab589903a6a1f80f22",
  "sha": "da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
  "build_id": 380,
  "build_name": "unit-tests",
  "build_stage": "test",
  "build_status": "success",
  "build_created_at": "2025-01-15 12:00:00 UTC",
  "build_started_at": "2025-01-15 12:01:00 UTC",
  "build_finished_at": "2025-01-15 12:05:00 UTC",
  "build_duration": 240,
  "build_queued_duration": 60,
  "build_allow_failure": false,
  "build_failure_reason": "unknown_failure",
  "pipeline_id": 31,
  "runner": {
    "id": 1,
    "description": "runner-1",
    "runner_type": "project_type",
    "active": true,
    "is_shared": false,
    "tags": ["docker", "linux"]
  },
  "project_id": 15,
  "project_name": "My Project",
  "user": { /* user details */ },
  "commit": { /* commit details */ },
  "repository": { /* repository details */ },
  "environment": {
    "name": "production",
    "action": "start",
    "deployment_tier": "production"
  }
}
```

**Build Status Values**: `pending`, `running`, `success`, `failed`, `canceled`

### Deployment Events

**Trigger**: Deployment created or status changes

**Event Header**: `X-Gitlab-Event: Deployment Hook`

**Payload Structure**:
```json
{
  "object_kind": "deployment",
  "status": "success",
  "status_changed_at": "2025-01-15 12:20:00 UTC",
  "deployment_id": 15,
  "deployable_id": 380,
  "deployable_url": "https://gitlab.com/namespace/project/-/jobs/380",
  "environment": "production",
  "environment_tier": "production",
  "environment_slug": "production",
  "environment_external_url": "https://prod.example.com",
  "project": { /* project details */ },
  "short_sha": "da156088",
  "user": { /* user details */ },
  "user_url": "https://gitlab.com/jdoe",
  "commit_url": "https://gitlab.com/namespace/project/-/commit/da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
  "commit_title": "Deploy to production"
}
```

**Status Values**: `created`, `running`, `success`, `failed`, `canceled`

### Wiki Page Events

**Trigger**: Wiki page created, updated, or deleted

**Event Header**: `X-Gitlab-Event: Wiki Page Hook`

**Payload Structure**:
```json
{
  "object_kind": "wiki_page",
  "user": { /* user details */ },
  "project": { /* project details */ },
  "wiki": {
    "web_url": "https://gitlab.com/namespace/project/-/wikis/home",
    "git_ssh_url": "git@gitlab.com:namespace/project.wiki.git",
    "git_http_url": "https://gitlab.com/namespace/project.wiki.git",
    "path_with_namespace": "namespace/project",
    "default_branch": "main"
  },
  "object_attributes": {
    "title": "Home",
    "content": "# Welcome\n\nThis is the home page...",
    "format": "markdown",
    "message": "Update home page",
    "slug": "home",
    "url": "https://gitlab.com/namespace/project/-/wikis/home",
    "action": "update"
  }
}
```

**Action Values**: `create`, `update`, `delete`

### Release Events

**Trigger**: Release created, updated, or deleted

**Event Header**: `X-Gitlab-Event: Release Hook`

**Payload Structure**:
```json
{
  "id": 1,
  "created_at": "2025-01-15 12:00:00 UTC",
  "description": "Release notes for v1.0.0",
  "name": "Version 1.0.0",
  "released_at": "2025-01-15 12:00:00 UTC",
  "tag": "v1.0.0",
  "object_kind": "release",
  "project": { /* project details */ },
  "url": "https://gitlab.com/namespace/project/-/releases/v1.0.0",
  "action": "create",
  "assets": {
    "count": 2,
    "links": [
      {
        "id": 1,
        "external": true,
        "link_type": "other",
        "name": "Binary",
        "url": "https://example.com/binary"
      }
    ],
    "sources": [
      {
        "format": "zip",
        "url": "https://gitlab.com/namespace/project/-/archive/v1.0.0/project-v1.0.0.zip"
      },
      {
        "format": "tar.gz",
        "url": "https://gitlab.com/namespace/project/-/archive/v1.0.0/project-v1.0.0.tar.gz"
      }
    ]
  },
  "commit": { /* commit details */ }
}
```

**Action Values**: `create`, `update`, `delete`

## Testing Webhooks

### Via GitLab UI

1. Navigate to Project Settings > Webhooks
2. Find your webhook
3. Click "Test" dropdown
4. Select event type to test
5. View response in UI

### Via API

```bash
curl -X POST "https://gitlab.com/api/v4/projects/:id/hooks/:hook_id/test/push_events" \
  --header "PRIVATE-TOKEN: <token>"
```

### Local Testing

Use tools like:
- **ngrok**: Create public URL for local development
- **webhook.site**: Test webhook delivery
- **requestbin.com**: Inspect webhook payloads

```bash
# Using ngrok
ngrok http 3000

# Update webhook URL to ngrok URL
# Trigger event in GitLab
# View webhook payload in ngrok dashboard
```

## Webhook Best Practices

### 1. Security

- **Use HTTPS**: Always use HTTPS URLs for webhooks
- **Verify SSL**: Enable SSL verification in production
- **Secret Tokens**: Use secret tokens to verify webhook authenticity
- **Validate Payloads**: Verify request signatures
- **IP Allowlisting**: Restrict webhook sources to GitLab IPs

### 2. Reliability

- **Return Quickly**: Respond with 2xx status code quickly (< 10 seconds)
- **Async Processing**: Queue webhook payloads for async processing
- **Idempotency**: Handle duplicate webhook deliveries
- **Error Handling**: Handle errors gracefully
- **Retry Logic**: Implement retry logic for failed processing

### 3. Monitoring

- **Log Webhooks**: Log all webhook deliveries
- **Track Failures**: Monitor webhook failure rates
- **Alert on Issues**: Set up alerts for webhook failures
- **Recent Deliveries**: Check recent deliveries in GitLab UI

### 4. Performance

- **Rate Limiting**: Handle rate limits on external APIs
- **Batch Processing**: Batch similar webhook events
- **Caching**: Cache frequently accessed data
- **Timeouts**: Set appropriate timeouts

## Example Webhook Handler

### Python Flask Example

```python
from flask import Flask, request, jsonify
import hmac
import hashlib

app = Flask(__name__)
WEBHOOK_SECRET = "your-secret-token"

def verify_signature(payload, signature):
    """Verify webhook signature"""
    if not signature:
        return False

    expected = hmac.new(
        WEBHOOK_SECRET.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(expected, signature)

@app.route('/webhook', methods=['POST'])
def handle_webhook():
    # Get signature from header
    signature = request.headers.get('X-Gitlab-Token')

    # Verify signature
    if not verify_signature(request.data, signature):
        return jsonify({'error': 'Invalid signature'}), 401

    # Get event type
    event_type = request.headers.get('X-Gitlab-Event')

    # Parse payload
    payload = request.json

    # Handle different event types
    if event_type == 'Push Hook':
        handle_push_event(payload)
    elif event_type == 'Merge Request Hook':
        handle_merge_request_event(payload)
    elif event_type == 'Pipeline Hook':
        handle_pipeline_event(payload)

    return jsonify({'status': 'received'}), 200

def handle_push_event(payload):
    """Handle push events"""
    ref = payload['ref']
    commits = payload['commits']
    print(f"Received push to {ref} with {len(commits)} commits")

def handle_merge_request_event(payload):
    """Handle merge request events"""
    action = payload['object_attributes']['action']
    iid = payload['object_attributes']['iid']
    print(f"Merge request {iid} was {action}")

def handle_pipeline_event(payload):
    """Handle pipeline events"""
    status = payload['object_attributes']['status']
    ref = payload['object_attributes']['ref']
    print(f"Pipeline on {ref} status: {status}")

if __name__ == '__main__':
    app.run(port=5000)
```

### Node.js Express Example

```javascript
const express = require('express');
const crypto = require('crypto');
const bodyParser = require('body-parser');

const app = express();
const WEBHOOK_SECRET = 'your-secret-token';

// Verify webhook signature
function verifySignature(payload, signature) {
  if (!signature) return false;

  const expected = crypto
    .createHmac('sha256', WEBHOOK_SECRET)
    .update(payload)
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(expected),
    Buffer.from(signature)
  );
}

app.post('/webhook', bodyParser.raw({type: 'application/json'}), (req, res) => {
  const signature = req.headers['x-gitlab-token'];
  const eventType = req.headers['x-gitlab-event'];

  // Verify signature
  if (!verifySignature(req.body, signature)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const payload = JSON.parse(req.body.toString());

  // Handle different event types
  switch (eventType) {
    case 'Push Hook':
      handlePushEvent(payload);
      break;
    case 'Merge Request Hook':
      handleMergeRequestEvent(payload);
      break;
    case 'Pipeline Hook':
      handlePipelineEvent(payload);
      break;
  }

  res.json({ status: 'received' });
});

function handlePushEvent(payload) {
  console.log(`Push to ${payload.ref} with ${payload.commits.length} commits`);
}

function handleMergeRequestEvent(payload) {
  const { action, iid } = payload.object_attributes;
  console.log(`Merge request ${iid} was ${action}`);
}

function handlePipelineEvent(payload) {
  const { status, ref } = payload.object_attributes;
  console.log(`Pipeline on ${ref} status: ${status}`);
}

app.listen(5000, () => {
  console.log('Webhook server listening on port 5000');
});
```

## Troubleshooting

### Common Issues

1. **Webhook Not Triggering**
   - Verify event is enabled in webhook configuration
   - Check webhook URL is accessible from internet
   - Review Recent Deliveries in GitLab UI
   - Check firewall rules

2. **SSL Verification Failures**
   - Ensure SSL certificate is valid
   - Check certificate chain is complete
   - Temporarily disable SSL verification for testing (not recommended for production)

3. **Timeouts**
   - Reduce processing time in webhook handler
   - Return 2xx response quickly, process async
   - Check network connectivity

4. **Authentication Errors**
   - Verify secret token matches
   - Check signature verification logic
   - Ensure token is transmitted correctly

### Debugging Tips

1. Use webhook testing services (webhook.site, requestbin.com)
2. Check "Recent Deliveries" in GitLab webhook settings
3. Enable detailed logging in webhook handler
4. Test with curl/Postman using sample payloads
5. Verify IP addresses if using allowlisting
6. Check response status codes and headers

## Additional Resources

- GitLab Webhooks Documentation: https://docs.gitlab.com/ee/user/project/integrations/webhooks.html
- Webhook Events: https://docs.gitlab.com/ee/user/project/integrations/webhook_events.html
- System Hooks: https://docs.gitlab.com/ee/administration/system_hooks.html
