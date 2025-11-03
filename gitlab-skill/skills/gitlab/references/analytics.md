# GitLab Analytics Reference (Ultimate)

## Overview

GitLab Ultimate provides comprehensive analytics for measuring team performance, project health, and delivery metrics.

## Value Stream Analytics

### Overview

Measure time from idea to production across multiple stages.

**Access**: Group > Analytics > Value Stream Analytics

### Default Stages

1. **Issue** - Time in backlog
2. **Plan** - Time to first commit
3. **Code** - Time writing code
4. **Test** - Time running CI
5. **Review** - Time in code review
6. **Staging** - Time in staging
7. **Production** - Time to production

### Custom Value Streams (Ultimate)

Create custom measurement workflows:

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/analytics/value_stream_analytics/value_streams" \
  --data "name=Feature Development" \
  --data '{"stages": [
    {
      "name": "Planning",
      "start_event_identifier": "issue_created",
      "end_event_identifier": "issue_first_mentioned_in_commit"
    },
    {
      "name": "Development",
      "start_event_identifier": "merge_request_created",
      "end_event_identifier": "merge_request_merged"
    }
  ]}'
```

### Available Events

**Start Events**:
- `issue_created`
- `issue_first_mentioned_in_commit`
- `merge_request_created`
- `merge_request_first_deployed_to_production`
- `merge_request_label_added`
- `merge_request_first_commit_at`

**End Events**:
- `issue_closed`
- `issue_first_mentioned_in_commit`
- `merge_request_merged`
- `merge_request_closed`
- `merge_request_first_deployed_to_production`
- `merge_request_label_removed`

### VSA Metrics

```bash
# Get VSA metrics
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/analytics/value_stream_analytics/summary" \
  --data "from=2025-01-01" \
  --data "to=2025-03-31"
```

**Response includes**:
- Lead time
- Cycle time
- Deployment frequency
- Time to restore service

## DORA Metrics (Ultimate)

Four key DevOps Research and Assessment metrics.

### 1. Deployment Frequency

How often code is deployed to production:

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/dora/metrics?metric=deployment_frequency&start_date=2025-01-01&end_date=2025-03-31"
```

**Tracked via**:
- Deployments to production environment
- Merge requests merged to default branch

### 2. Lead Time for Changes

Time from commit to production:

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/dora/metrics?metric=lead_time_for_changes"
```

**Calculation**:
- Median time from commit to production deployment

### 3. Time to Restore Service

Recovery time from production incidents:

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/dora/metrics?metric=time_to_restore_service"
```

**Tracked via**:
- Incident issues (issue type: incident)
- Time from creation to closure

### 4. Change Failure Rate

Percentage of deployments causing incidents:

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/dora/metrics?metric=change_failure_rate"
```

**Calculation**:
- Incidents / Total deployments

### DORA Dashboard

**View metrics**:
1. Project > Analytics > CI/CD Analytics
2. DORA Metrics tab
3. Filter by date range
4. Compare across projects/groups

**Performance Levels** (Google DORA research):

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| Deployment Frequency | Multiple times per day | Weekly to monthly | Monthly to every 6 months | Less than every 6 months |
| Lead Time | Less than 1 hour | 1 day to 1 week | 1 month to 6 months | More than 6 months |
| Time to Restore | Less than 1 hour | Less than 1 day | 1 day to 1 week | More than 6 months |
| Change Failure Rate | 0-15% | 16-30% | 16-30% | 16-30% |

## Code Review Analytics (Ultimate)

Measure code review efficiency.

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/analytics/merge_request_analytics?group_id=:id"
```

**Metrics include**:
- Time to first review
- Time to merge
- Review thoroughness
- Number of iterations
- Reviewer workload

## Repository Analytics

Track repository activity and health.

**Access**: Project > Analytics > Repository

**Metrics**:
- Commits over time
- Contributors
- Commit activity by time of day
- Contributor activity

```bash
# Get contributor statistics
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/repository/contributors"
```

## Issue Analytics (Ultimate)

Analyze issue patterns and trends.

**Access**: Group > Analytics > Issue Analytics

**Visualizations**:
- Issues created vs closed
- Issues by type
- Issues by label
- Average time to close
- Issue age distribution

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/issues_statistics?created_after=2025-01-01"
```

## Productivity Analytics (Ultimate)

Measure team productivity.

**Access**: Group > Analytics > Productivity

**Metrics tracked**:
- Merge requests created
- Merge requests merged
- Issues created
- Issues closed
- Commits pushed
- Pipelines run

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/analytics/productivity_analytics" \
  --data "merged_after=2025-01-01" \
  --data "merged_before=2025-03-31"
```

## Merge Request Analytics (Ultimate)

Detailed MR metrics.

**Metrics**:
- Open MRs
- MR throughput
- Time to merge distribution
- Review time
- Lines of code changed

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/analytics/merge_request_analytics"
```

## CI/CD Analytics

Pipeline performance metrics.

**Access**: Project > Analytics > CI/CD Analytics

**Metrics**:
- Pipeline success rate
- Pipeline duration
- Job success rate
- Runner usage
- Pipeline trends

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/pipelines?per_page=100"
```

## Insights (Ultimate)

Custom analytics dashboards with YAML configuration.

**Create `.gitlab/insights.yml`**:

```yaml
insights:
  bugsboard:
    title: "Bugs Dashboard"
    charts:
      - title: "Monthly bugs created"
        description: "Open bugs created per month"
        type: bar
        query:
          issuable_type: issue
          issuable_state: opened
          filter_labels:
            - bug
          group_by: month
          period_limit: 12
      
      - title: "Bug priority breakdown"
        type: pie
        query:
          issuable_type: issue
          filter_labels:
            - bug
          collection_labels:
            - priority::high
            - priority::medium
            - priority::low
      
      - title: "Bug resolution time"
        type: line
        query:
          issuable_type: issue
          filter_labels:
            - bug
          group_by: month
          issuable_state: closed
```

**Chart Types**:
- `bar` - Bar chart
- `line` - Line chart
- `stacked-bar` - Stacked bar
- `pie` - Pie chart

## Contribution Analytics (Ultimate)

Track member contributions.

**Access**: Group > Analytics > Contribution Analytics

**Tracks per member**:
- Issues created
- Merge requests created
- Merge requests merged
- Push events
- Total contributions

## Group Activity Analytics (Ultimate)

Group-level activity overview.

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/analytics/group_activity"
```

**Includes**:
- Recent issues
- Recent merge requests
- New members
- Total projects

## DevOps Adoption (Ultimate)

Track DevOps adoption across groups.

**Access**: Group > Analytics > DevOps Adoption

**Tracks adoption of**:
- Approvals
- Code owners
- Dependency scanning
- DAST
- SAST
- Merge requests
- Pipelines
- Runners

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/analytics/devops_adoption"
```

## Custom Reports

### Dashboards with API

**Example: Custom dashboard**:

```python
import requests
import pandas as pd
import plotly.express as px

class GitLabDashboard:
    def __init__(self, token, group_id):
        self.token = token
        self.group_id = group_id
        self.base_url = "https://gitlab.com/api/v4"
        
    def get_issue_metrics(self):
        """Get issue metrics"""
        response = requests.get(
            f"{self.base_url}/groups/{self.group_id}/issues_statistics",
            headers={"PRIVATE-TOKEN": self.token}
        )
        return response.json()
    
    def get_merge_request_metrics(self):
        """Get MR metrics"""
        response = requests.get(
            f"{self.base_url}/groups/{self.group_id}/merge_requests",
            params={"state": "merged", "per_page": 100},
            headers={"PRIVATE-TOKEN": self.token}
        )
        mrs = response.json()
        
        # Calculate time to merge
        times = []
        for mr in mrs:
            created = pd.to_datetime(mr['created_at'])
            merged = pd.to_datetime(mr['merged_at'])
            times.append((merged - created).total_seconds() / 3600)  # hours
            
        return {
            "count": len(mrs),
            "avg_time_to_merge": sum(times) / len(times) if times else 0
        }
    
    def get_dora_metrics(self, project_id):
        """Get DORA metrics"""
        response = requests.get(
            f"{self.base_url}/projects/{project_id}/dora/metrics",
            params={"metric": "deployment_frequency"},
            headers={"PRIVATE-TOKEN": self.token}
        )
        return response.json()
    
    def create_dashboard(self):
        """Create comprehensive dashboard"""
        issue_data = self.get_issue_metrics()
        mr_data = self.get_merge_request_metrics()
        
        print(f"Issues Open: {issue_data['statistics']['counts']['opened']}")
        print(f"Issues Closed: {issue_data['statistics']['counts']['closed']}")
        print(f"MRs Merged: {mr_data['count']}")
        print(f"Avg Time to Merge: {mr_data['avg_time_to_merge']:.1f} hours")

# Usage
dashboard = GitLabDashboard(token="your-token", group_id="group-id")
dashboard.create_dashboard()
```

## Export Analytics

### Export to CSV

Most analytics views support CSV export:

```bash
# Export issue statistics
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/groups/:id/issues?per_page=100" \
  -H "Accept: text/csv" \
  -o issues.csv
```

### Export to BI Tools

**Connect to tools like**:
- Tableau
- Power BI
- Looker
- Grafana

**Using**:
- GitLab API
- PostgreSQL direct connection (self-hosted)
- Custom ETL pipelines

## Best Practices

### 1. Regular Review

- Review metrics weekly
- Track trends over time
- Set improvement goals
- Share with team

### 2. Actionable Insights

- Focus on actionable metrics
- Identify bottlenecks
- Address root causes
- Experiment and iterate

### 3. Balanced Metrics

- Don't optimize single metric
- Consider trade-offs
- Focus on outcomes
- Align with business goals

### 4. Team Context

- Consider team size
- Account for project complexity
- Compare like with like
- Avoid gaming metrics

## Additional Resources

- Analytics Documentation: https://docs.gitlab.com/ee/user/analytics/
- Value Stream Analytics: https://docs.gitlab.com/ee/user/group/value_stream_analytics/
- DORA Metrics: https://docs.gitlab.com/ee/user/analytics/dora_metrics.html
- Insights: https://docs.gitlab.com/ee/user/project/insights/
