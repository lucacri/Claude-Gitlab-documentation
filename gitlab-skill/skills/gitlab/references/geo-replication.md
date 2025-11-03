# GitLab Geo Replication Reference (Ultimate, Self-Hosted)

## Overview

GitLab Geo provides read-only mirrors of your GitLab instance across multiple geographic locations for faster access and disaster recovery.

## Architecture

**Primary site**: Read-write GitLab instance
**Secondary sites**: Read-only replicas

**Replicated data**:
- Git repositories
- LFS objects
- Attachments
- Container registry
- Package registry
- Terraform state
- CI job artifacts
- Design repository

## Setup

### Prerequisites

- GitLab Ultimate license
- PostgreSQL replication
- SSH access between sites
- Same GitLab version on all sites

### Configure Primary Site

**`/etc/gitlab/gitlab.rb`**:

```ruby
##
## Enable Geo primary site
##

# External URL (must be HTTPS)
external_url 'https://primary.gitlab.example.com'

# Enable Geo
gitlab_rails['geo_node_name'] = 'Primary Site'

# Database configuration
gitlab_rails['db_host'] = 'primary-db.example.com'
gitlab_rails['db_password'] = 'secure-password'

# PostgreSQL settings for Geo
postgresql['md5_auth_cidr_addresses'] = ['10.0.0.0/8']
postgresql['listen_address'] = '0.0.0.0'
postgresql['sql_replication_user'] = 'gitlab_replicator'

##
## Geo-specific settings
##

# Configure tracking database
geo_postgresql['enable'] = true
geo_postgresql['listen_address'] = '0.0.0.0'
geo_postgresql['md5_auth_cidr_addresses'] = ['10.0.0.0/8']
```

**Apply configuration**:
```bash
sudo gitlab-ctl reconfigure
```

**Setup replication user**:
```bash
sudo gitlab-psql -c "CREATE USER gitlab_replicator REPLICATION ENCRYPTED PASSWORD 'secure-password';"
sudo gitlab-psql -c "GRANT CONNECT ON DATABASE gitlabhq_production TO gitlab_replicator;"
```

### Configure Secondary Site

**`/etc/gitlab/gitlab.rb`**:

```ruby
##
## Enable Geo secondary site
##

# External URL (must be HTTPS)
external_url 'https://secondary.gitlab.example.com'

# Enable Geo
gitlab_rails['geo_node_name'] = 'Secondary Site Europe'

# This is a secondary site
roles ['geo_secondary_role']

# Primary site URL
gitlab_rails['geo_primary_api_url'] = 'https://primary.gitlab.example.com/api/v4'

# Database replication
gitlab_rails['db_host'] = 'secondary-db.example.com'
gitlab_rails['db_password'] = 'secure-password'

# Configure read-only mode
gitlab_rails['db_database'] = 'gitlabhq_production'
postgresql['enable'] = false

##
## Geo tracking database
##

geo_postgresql['enable'] = true
geo_postgresql['listen_address'] = '0.0.0.0'
geo_postgresql['md5_auth_cidr_addresses'] = ['10.0.0.0/8']

##
## Object storage (optional but recommended)
##

gitlab_rails['object_store']['enabled'] = true
gitlab_rails['object_store']['connection'] = {
  'provider' => 'AWS',
  'region' => 'eu-west-1',
  'aws_access_key_id' => 'key',
  'aws_secret_access_key' => 'secret'
}
```

**Apply configuration**:
```bash
sudo gitlab-ctl reconfigure
```

### Setup PostgreSQL Replication

**On primary**:

```bash
# Create replication slot
sudo gitlab-psql -c "SELECT * FROM pg_create_physical_replication_slot('geo_slot');"

# Allow replication from secondary
# Edit pg_hba.conf
echo "host    replication gitlab_replicator 10.0.2.0/24 md5" >> /var/opt/gitlab/postgresql/data/pg_hba.conf

# Reload PostgreSQL
sudo gitlab-ctl restart postgresql
```

**On secondary**:

```bash
# Stop PostgreSQL
sudo gitlab-ctl stop postgresql

# Remove old data
sudo rm -rf /var/opt/gitlab/postgresql/data

# Replicate from primary
sudo gitlab-pg-basebackup -h primary-db.example.com -p 5432 -D /var/opt/gitlab/postgresql/data -U gitlab_replicator -Xs -P

# Create recovery configuration
cat > /var/opt/gitlab/postgresql/data/recovery.conf << EOF
standby_mode = 'on'
primary_conninfo = 'host=primary-db.example.com port=5432 user=gitlab_replicator password=secure-password sslmode=require'
primary_slot_name = 'geo_slot'
EOF

# Start PostgreSQL
sudo gitlab-ctl start postgresql
```

### Add Secondary to Primary

**Via UI**:
1. Admin Area > Geo > Nodes
2. Add Node
3. Enter secondary URL and internal URL
4. Save

**Via Rails Console on Primary**:

```ruby
sudo gitlab-rails console

# Add secondary node
GeoNode.create!(
  name: 'Secondary Site Europe',
  url: 'https://secondary.gitlab.example.com',
  internal_url: 'https://internal.secondary.gitlab.example.com',
  primary: false,
  enabled: true
)
```

### Generate OAuth Application

**On primary**:

```bash
sudo gitlab-rails runner "
  app = Doorkeeper::Application.create!(
    name: 'Geo Secondary',
    redirect_uri: 'https://secondary.gitlab.example.com/oauth/geo/callback',
    scopes: 'geo',
    trusted: true
  )
  puts \"Application ID: #{app.uid}\"
  puts \"Secret: #{app.secret}\"
"
```

**Add credentials to secondary `/etc/gitlab/gitlab.rb`**:

```ruby
gitlab_rails['geo_registry_replication_enabled'] = true
gitlab_rails['geo_registry_replication_primary_api_url'] = 'https://primary.gitlab.example.com/api/v4'

# OAuth credentials from primary
gitlab_rails['geo_registry_application_id'] = 'application-id'
gitlab_rails['geo_registry_secret'] = 'application-secret'
```

## Managing Geo Sites

### Check Sync Status

**Via UI**: Admin Area > Geo > Nodes

**Via API**:

```bash
# Get all Geo nodes
curl --header "PRIVATE-TOKEN: <admin-token>" \
  "https://primary.gitlab.example.com/api/v4/geo_nodes"

# Get specific node status
curl --header "PRIVATE-TOKEN: <admin-token>" \
  "https://primary.gitlab.example.com/api/v4/geo_nodes/:id/status"
```

**Via Rails Console**:

```ruby
# On secondary
Gitlab::Geo::HealthCheck.new.perform_checks

# Check sync status
Geo::ProjectRegistry.synced.count
Geo::ProjectRegistry.failed.count
```

### Resync Projects

**Resync specific project**:

```bash
# On secondary
sudo gitlab-rake geo:verification:repository:resync[project_id]
```

**Resync all projects**:

```bash
# On secondary
sudo gitlab-rake geo:verification:repository:resync:all
```

### Verify Data Integrity

```bash
# Verify repositories
sudo gitlab-rake geo:verification:repository:verify:all

# Verify wikis
sudo gitlab-rake geo:verification:wiki:verify:all
```

## Selective Sync

Replicate only specific groups/shards.

**Configure on secondary `/etc/gitlab/gitlab.rb`**:

```ruby
# Sync specific groups
gitlab_rails['geo_node_selective_sync_by_groups'] = [
  'group/subgroup',
  'another-group'
]

# Sync specific shards
gitlab_rails['geo_node_selective_sync_by_shards'] = [
  'default',
  'storage1'
]
```

## Object Storage Replication

**Configure unified object storage**:

```ruby
gitlab_rails['object_store']['enabled'] = true
gitlab_rails['object_store']['connection'] = {
  'provider' => 'AWS',
  'region' => 'us-east-1',
  'aws_access_key_id' => 'key',
  'aws_secret_access_key' => 'secret'
}

# Enable for each object type
gitlab_rails['object_store']['objects']['artifacts']['enabled'] = true
gitlab_rails['object_store']['objects']['lfs']['enabled'] = true
gitlab_rails['object_store']['objects']['uploads']['enabled'] = true
gitlab_rails['object_store']['objects']['packages']['enabled'] = true
```

## Failover and Disaster Recovery

### Planned Failover

**1. Verify secondary is in sync**:

```bash
# On secondary
sudo gitlab-rake geo:status
```

**2. Stop writes on primary**:

```bash
# On primary
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
```

**3. Wait for replication lag to be zero**

**4. Promote secondary to primary**:

```bash
# On secondary
sudo gitlab-ctl promote-to-primary-node
```

**5. Update DNS** to point to new primary

**6. Reconfigure old primary as secondary**

### Unplanned Failover

**If primary is down**:

```bash
# On secondary - force promotion
sudo gitlab-ctl promote-to-primary-node --force
```

**Then**:
1. Update DNS
2. Assess data loss
3. Rebuild old primary as secondary

## Monitoring Geo

### Prometheus Metrics

**Available metrics**:
- `geo_repositories`
- `geo_repositories_synced`
- `geo_repositories_failed`
- `geo_repositories_verification_total`
- `geo_wikis`
- `geo_lfs_objects`
- `geo_attachments`
- `geo_db_replication_lag_seconds`

**Example Prometheus query**:

```promql
# Replication lag
geo_db_replication_lag_seconds{url="https://secondary.gitlab.example.com"}

# Sync percentage
(geo_repositories_synced / geo_repositories) * 100
```

### Health Checks

```bash
# On secondary
sudo gitlab-rake gitlab:geo:check
```

**Checks**:
- Database replication
- Authentication
- HTTP/HTTPS connectivity
- Object storage
- Container registry

## Performance Tuning

### Concurrent Transfers

```ruby
# /etc/gitlab/gitlab.rb on secondary
gitlab_rails['geo_registry_replication_max_capacity'] = 25
gitlab_rails['geo_file_download_dispatch_worker_cron'] = "*/1 * * * *"
```

### Bandwidth Limits

```ruby
# Limit bandwidth usage
gitlab_rails['geo_transfer_bandwidth_limit'] = 100  # MB/s
```

### Batch Sizes

```ruby
gitlab_rails['geo_repository_verification_batch_size'] = 100
gitlab_rails['geo_file_download_worker_batch_size'] = 50
```

## Troubleshooting

### Replication Not Working

**Check database replication**:

```bash
# On primary
sudo gitlab-psql -c "SELECT * FROM pg_replication_slots;"

# On secondary
sudo gitlab-psql -c "SELECT * FROM pg_stat_wal_receiver;"
```

**Common issues**:
- Firewall blocking PostgreSQL port
- Wrong replication credentials
- Replication slot not created
- SSL/TLS certificate issues

### Slow Sync

**Check metrics**:

```bash
# On secondary
sudo gitlab-rake geo:status
```

**Optimize**:
- Increase concurrent transfers
- Use object storage
- Improve network bandwidth
- Add more Sidekiq workers

### Out of Sync

**Force resync**:

```bash
# On secondary
sudo gitlab-rake geo:verification:repository:reset
sudo gitlab-rake geo:verification:wiki:reset
```

## Best Practices

### 1. Network

- Use dedicated network link
- Configure QoS for replication
- Monitor bandwidth usage
- Plan for peak times

### 2. Object Storage

- Use shared object storage
- Enables faster failover
- Reduces sync overhead
- Consistent across sites

### 3. Monitoring

- Set up Prometheus alerts
- Monitor replication lag
- Track sync percentages
- Alert on failures

### 4. Testing

- Regular failover drills
- Test recovery procedures
- Document runbooks
- Train staff

### 5. Capacity Planning

- Plan for growth
- Monitor disk usage
- Scale secondary resources
- Review performance regularly

## Additional Resources

- Geo Documentation: https://docs.gitlab.com/ee/administration/geo/
- Geo Setup: https://docs.gitlab.com/ee/administration/geo/setup/
- Disaster Recovery: https://docs.gitlab.com/ee/administration/geo/disaster_recovery/
- Replication: https://docs.gitlab.com/ee/administration/geo/replication/
