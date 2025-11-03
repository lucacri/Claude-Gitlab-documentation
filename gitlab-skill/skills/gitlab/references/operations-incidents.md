# GitLab Operations and Incident Management Reference (Ultimate)

## Incidents

### Creating Incidents

Incidents are special issue types for operational problems.

**Via UI**:
1. Project > Monitor > Incidents
2. New incident
3. Fill in details

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues" \
  --data "title=Production API down" \
  --data "issue_type=incident" \
  --data "severity=critical" \
  --data "description=API returning 500 errors"
```

**Via integrations**:
- PagerDuty
- Prometheus AlertManager
- Generic webhook

### Incident Severity

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/issues/:incident_iid" \
  --data "severity=critical"
```

**Severity levels**:
- `critical` - Complete outage
- `high` - Major functionality impaired
- `medium` - Partial functionality affected
- `low` - Minor issue
- `unknown` - To be determined

### Incident Timeline

**Add timeline event**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/incidents/:incident_iid/timeline_events" \
  --data "note=Database failover completed" \
  --data "occurred_at=2025-01-15T14:30:00Z"
```

**Timeline shows**:
- When incident started
- Actions taken
- Status changes
- Resolution time

## On-Call Schedules

### Creating Schedules

**Via UI**: Project > Monitor > On-call Schedules

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/incident_management/oncall_schedules" \
  --data "name=Primary On-Call" \
  --data "description=24/7 on-call rotation" \
  --data "timezone=America/New_York"
```

### Rotation Configuration

```bash
# Add rotation
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/incident_management/oncall_schedules/:schedule_id/rotations" \
  --data "name=Weekly Rotation" \
  --data "starts_at=2025-01-01T09:00:00Z" \
  --data "rotation_length=1" \
  --data "rotation_length_unit=weeks" \
  --data "participants[][user_id]=123" \
  --data "participants[][user_id]=456"
```

### Escalation Policies

**Create policy**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/incident_management/escalation_policies" \
  --data "name=Standard Escalation" \
  --data "description=Escalate after 15 minutes" \
  --data "rules[][oncall_schedule_id]=schedule-1" \
  --data "rules[][elapsed_time_seconds]=900"
```

**Escalation rules**:
```json
{
  "rules": [
    {
      "oncall_schedule_id": 1,
      "elapsed_time_seconds": 0  // Immediately
    },
    {
      "oncall_schedule_id": 2,
      "elapsed_time_seconds": 900  // After 15 minutes
    },
    {
      "user_id": 123,
      "elapsed_time_seconds": 1800  // After 30 minutes
    }
  ]
}
```

## Alert Management

### Alert Integration

**Create integration**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/alert_management/http_integrations" \
  --data "name=Prometheus Alerts" \
  --data "active=true"
```

**Integration types**:
- HTTP endpoint (generic)
- Prometheus
- PagerDuty
- Datadog

### Incoming Alerts

**Webhook payload**:
```json
{
  "title": "High CPU usage",
  "description": "CPU usage > 90%",
  "severity": "high",
  "monitoring_tool": "Prometheus",
  "service": "web-api",
  "hosts": ["web-1.example.com"],
  "start_time": "2025-01-15T14:00:00Z"
}
```

**Create alert via API**:
```bash
curl --request POST \
  "https://gitlab.com/api/v4/projects/:id/alert_management/alerts" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer <integration-token>" \
  --data '{
    "title": "High CPU usage",
    "severity": "high",
    "description": "CPU > 90% for 5 minutes"
  }'
```

### Alert Status

```bash
# Update alert status
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/alert_management/alerts/:alert_iid" \
  --data "status=acknowledged"
```

**Status options**:
- `triggered` - New alert
- `acknowledged` - Someone is looking
- `resolved` - Problem fixed
- `ignored` - False alarm

## Error Tracking

### Sentry Integration

**Configure Sentry**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/services/sentry" \
  --data "api_url=https://sentry.io/api/0/projects/org/project/" \
  --data "auth_token=sentry-auth-token" \
  --data "active=true"
```

**Error tracking shows**:
- Error frequency
- First seen / Last seen
- Stack traces
- Affected users
- Related issues

### GitLab Error Tracking

Built-in error tracking (self-hosted):

```ruby
# config/gitlab.yml
production:
  error_tracking:
    enabled: true
    api_url: https://gitlab.example.com/api/v4/error_tracking
```

## Status Page

### Public Status Page

**Enable status page**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id" \
  --data "operations_access_level=enabled"
```

**Configure via `.gitlab/status-page.yml`**:
```yaml
services:
  - name: API
    url: https://api.example.com
  - name: Web Application
    url: https://app.example.com
  - name: Database
    url: postgres://db.example.com

incidents:
  publish: true
  limit: 100
```

**Status page features**:
- Current status
- Incident history
- Maintenance schedules
- Subscribe to updates
- RSS feed

## Incident Metrics

### MTTR (Mean Time to Recovery)

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/incidents?closed_after=2025-01-01"
```

**Calculate MTTR**:
```python
import requests
from datetime import datetime

def calculate_mttr(project_id, token):
    response = requests.get(
        f"https://gitlab.com/api/v4/projects/{project_id}/incidents",
        headers={"PRIVATE-TOKEN": token},
        params={"state": "closed", "per_page": 100}
    )
    
    incidents = response.json()
    recovery_times = []
    
    for incident in incidents:
        created = datetime.fromisoformat(incident['created_at'].replace('Z', '+00:00'))
        closed = datetime.fromisoformat(incident['closed_at'].replace('Z', '+00:00'))
        recovery_time = (closed - created).total_seconds() / 3600  # hours
        recovery_times.append(recovery_time)
    
    mttr = sum(recovery_times) / len(recovery_times) if recovery_times else 0
    return mttr

print(f"MTTR: {calculate_mttr(123, 'token'):.2f} hours")
```

### Incident Frequency

Track incidents over time:
- By severity
- By service
- By time of day
- By day of week

## Runbooks

### Incident Response Runbook

Create in project wiki or repository:

**`runbooks/api-outage.md`**:
```markdown
# API Outage Response

## Severity: Critical

## Symptoms
- API returning 500 errors
- High error rate in monitoring
- Customer reports of service unavailability

## Initial Response
1. Acknowledge incident in GitLab
2. Page on-call engineer
3. Create war room channel
4. Start incident timeline

## Investigation Steps
1. Check application logs
   ```
   kubectl logs -l app=api --tail=100
   ```

2. Check database connectivity
   ```
   psql -h db.example.com -U gitlab -c "SELECT 1"
   ```

3. Check infrastructure metrics
   - CPU usage
   - Memory usage
   - Network connectivity

## Common Causes
- Database connection pool exhausted
- Memory leak in application
- DDoS attack
- Infrastructure failure

## Resolution Steps
### Database Connection Issue
```bash
# Restart application pods
kubectl rollout restart deployment/api

# Scale database connections
kubectl edit configmap api-config
# Increase max_connections
```

### Memory Leak
```bash
# Rolling restart
kubectl rollout restart deployment/api --watch
```

## Post-Incident
1. Update incident timeline
2. Close incident
3. Schedule post-mortem
4. Document lessons learned
5. Create follow-up issues
```

### Link Runbook to Incident

```markdown
<!-- In incident description -->
See runbook: [API Outage](./runbooks/api-outage.md)
```

## Integrations

### PagerDuty

**Configure integration**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/services/pagerduty" \
  --data "api_url=https://api.pagerduty.com" \
  --data "api_key=pagerduty-api-key" \
  --data "active=true"
```

**Features**:
- Auto-create incidents
- Sync incident status
- Link to PagerDuty incidents
- Trigger on-call pages

### Slack

**Incident notifications**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/services/slack" \
  --data "webhook=https://hooks.slack.com/services/..." \
  --data "incidents_channel=#incidents" \
  --data "notify_only_broken_pipelines=false"
```

## Best Practices

### 1. Incident Response

- Clear severity definitions
- Documented runbooks
- Regular on-call training
- Post-mortem culture

### 2. On-Call Management

- Fair rotation schedule
- Backup coverage
- Clear escalation paths
- On-call compensation

### 3. Alert Management

- Actionable alerts only
- Proper alert routing
- Alert aggregation
- Regular alert review

### 4. Metrics

- Track MTTR
- Incident frequency
- False positive rate
- Time to acknowledge

### 5. Communication

- Stakeholder updates
- Status page maintenance
- Internal notifications
- Post-incident reports

## Additional Resources

- Incident Management: https://docs.gitlab.com/ee/operations/incident_management/
- On-Call Schedules: https://docs.gitlab.com/ee/operations/incident_management/oncall_schedules.html
- Alert Management: https://docs.gitlab.com/ee/operations/incident_management/alerts.html
