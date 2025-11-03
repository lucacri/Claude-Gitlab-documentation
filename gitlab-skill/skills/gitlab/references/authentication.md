# GitLab Authentication Reference

## Overview

GitLab supports multiple authentication methods for accessing the API, Git repositories, and the web interface.

## Authentication Methods

### 1. Personal Access Tokens (PATs)

Most common method for API and Git authentication.

#### Creating a Personal Access Token

**Via UI**:
1. Navigate to User Settings > Access Tokens
2. Enter token name
3. Set expiration date (optional but recommended)
4. Select scopes
5. Click "Create personal access token"
6. Copy token (shown only once)

**Scopes**:
- `api` - Complete API access
- `read_api` - Read-only API access
- `read_user` - Read user information
- `read_repository` - Read repository (pull code)
- `write_repository` - Write repository (push code)
- `read_registry` - Read container registry
- `write_registry` - Write container registry
- `sudo` - Perform actions as any user (admin only)
- `admin_mode` - Admin mode access

#### Using Personal Access Tokens

**API requests**:
```bash
# Header method (preferred)
curl --header "PRIVATE-TOKEN: <your_access_token>" "https://gitlab.com/api/v4/projects"

# Query parameter method
curl "https://gitlab.com/api/v4/projects?private_token=<your_access_token>"
```

**Git operations**:
```bash
# Clone with token
git clone https://oauth2:<your_access_token>@gitlab.com/username/project.git

# Or configure credential helper
git config --global credential.helper store
# Then use token as password when prompted
```

**Python example**:
```python
import requests

token = "your_access_token"
headers = {"PRIVATE-TOKEN": token}

response = requests.get(
    "https://gitlab.com/api/v4/projects",
    headers=headers
)
print(response.json())
```

#### Token Best Practices

- Set expiration dates on all tokens
- Use minimal required scopes
- Store tokens securely (environment variables, secret managers)
- Rotate tokens regularly
- Revoke unused tokens
- Never commit tokens to repositories

### 2. OAuth 2.0

OAuth 2.0 for third-party application authorization.

#### OAuth Application Setup

**Create OAuth Application**:
1. Navigate to User Settings > Applications
2. Enter application name
3. Set redirect URI
4. Select scopes
5. Click "Save application"
6. Note Application ID and Secret

**Authorization Code Flow**:

```python
import requests
from flask import Flask, request, redirect

app = Flask(__name__)

CLIENT_ID = "your_client_id"
CLIENT_SECRET = "your_client_secret"
REDIRECT_URI = "http://localhost:5000/callback"
GITLAB_URL = "https://gitlab.com"

@app.route('/login')
def login():
    """Redirect user to GitLab authorization page"""
    auth_url = (
        f"{GITLAB_URL}/oauth/authorize?"
        f"client_id={CLIENT_ID}&"
        f"redirect_uri={REDIRECT_URI}&"
        f"response_type=code&"
        f"scope=read_user+api"
    )
    return redirect(auth_url)

@app.route('/callback')
def callback():
    """Handle OAuth callback"""
    code = request.args.get('code')

    # Exchange code for access token
    token_response = requests.post(
        f"{GITLAB_URL}/oauth/token",
        data={
            'client_id': CLIENT_ID,
            'client_secret': CLIENT_SECRET,
            'code': code,
            'grant_type': 'authorization_code',
            'redirect_uri': REDIRECT_URI
        }
    )

    token_data = token_response.json()
    access_token = token_data['access_token']
    refresh_token = token_data['refresh_token']

    # Use access token for API requests
    headers = {'Authorization': f'Bearer {access_token}'}
    user_response = requests.get(
        f"{GITLAB_URL}/api/v4/user",
        headers=headers
    )

    return user_response.json()

if __name__ == '__main__':
    app.run(port=5000)
```

**Refreshing Access Token**:
```python
def refresh_access_token(refresh_token):
    response = requests.post(
        f"{GITLAB_URL}/oauth/token",
        data={
            'client_id': CLIENT_ID,
            'client_secret': CLIENT_SECRET,
            'refresh_token': refresh_token,
            'grant_type': 'refresh_token',
            'redirect_uri': REDIRECT_URI
        }
    )
    return response.json()
```

#### OAuth Scopes

- `api` - Full API access
- `read_user` - Read user information
- `read_api` - Read-only API access
- `read_repository` - Read repositories
- `write_repository` - Write to repositories
- `read_registry` - Read container registry
- `write_registry` - Write container registry
- `sudo` - Admin impersonation (admin only)
- `openid` - OpenID Connect
- `profile` - User profile info
- `email` - User email

### 3. SSH Keys

SSH keys for Git operations.

#### Generating SSH Keys

```bash
# Generate new SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Or RSA (if ed25519 not supported)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519
```

#### Adding SSH Key to GitLab

**Via UI**:
1. Navigate to User Settings > SSH Keys
2. Paste public key content (`~/.ssh/id_ed25519.pub`)
3. Set title
4. Optional: Set expiration date
5. Click "Add key"

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --data "title=My SSH Key" \
  --data "key=$(cat ~/.ssh/id_ed25519.pub)" \
  "https://gitlab.com/api/v4/user/keys"
```

#### Using SSH Keys

```bash
# Clone with SSH
git clone git@gitlab.com:username/project.git

# Configure Git to use SSH
git remote set-url origin git@gitlab.com:username/project.git

# Test SSH connection
ssh -T git@gitlab.com
```

#### SSH Key Best Practices

- Use Ed25519 keys (more secure, faster)
- Set passphrase on private keys
- Use separate keys for different purposes
- Set expiration dates
- Store private keys securely
- Never share private keys

### 4. Deploy Keys

Read-only or read-write SSH keys for specific projects.

#### Creating Deploy Keys

**Via UI**:
1. Navigate to Project Settings > Repository > Deploy Keys
2. Enter title and key content
3. Check "Grant write permissions" if needed
4. Click "Add key"

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --data "title=Deploy Key" \
  --data "key=$(cat ~/.ssh/deploy_key.pub)" \
  --data "can_push=false" \
  "https://gitlab.com/api/v4/projects/:id/deploy_keys"
```

#### Deploy Key Use Cases

- CI/CD pipelines
- Deployment scripts
- Automated processes
- Read-only repository access

### 5. Deploy Tokens

Project or group-level tokens for registry and package access.

#### Creating Deploy Tokens

**Via UI**:
1. Navigate to Project/Group Settings > Repository > Deploy Tokens
2. Enter name
3. Set expiration date
4. Select scopes:
   - `read_repository` - Clone repositories
   - `read_registry` - Pull container images
   - `write_registry` - Push container images
   - `read_package_registry` - Pull packages
   - `write_package_registry` - Push packages
5. Click "Create deploy token"
6. Copy username and token

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "name": "My Deploy Token",
    "expires_at": "2025-12-31",
    "scopes": ["read_repository", "read_registry"]
  }' \
  "https://gitlab.com/api/v4/projects/:id/deploy_tokens"
```

#### Using Deploy Tokens

**Clone repository**:
```bash
git clone https://<deploy-token-username>:<deploy-token>@gitlab.com/group/project.git
```

**Pull Docker image**:
```bash
docker login -u <deploy-token-username> -p <deploy-token> registry.gitlab.com
docker pull registry.gitlab.com/group/project/image:tag
```

**CI/CD**:
```yaml
docker-build:
  script:
    - docker login -u $CI_DEPLOY_USER -p $CI_DEPLOY_PASSWORD $CI_REGISTRY
    - docker pull $CI_REGISTRY_IMAGE:latest
```

### 6. Job Tokens (CI/CD)

Temporary tokens for CI/CD job authentication.

#### Using CI_JOB_TOKEN

**API requests in CI**:
```yaml
test-api:
  script:
    - |
      curl --header "JOB-TOKEN: $CI_JOB_TOKEN" \
        "https://gitlab.com/api/v4/projects/$CI_PROJECT_ID"
```

**Clone other repositories**:
```yaml
build:
  script:
    - git clone https://gitlab-ci-token:${CI_JOB_TOKEN}@gitlab.com/group/project.git
```

**Pull Docker images**:
```yaml
build:
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker pull $CI_REGISTRY_IMAGE:latest
```

#### Job Token Scope

Configure which projects can be accessed:

1. Navigate to Settings > CI/CD > Token Access
2. Add allowed projects
3. Enable/disable token access

### 7. Project Access Tokens

Project-level tokens for automation.

#### Creating Project Access Tokens

**Via UI**:
1. Navigate to Project Settings > Access Tokens
2. Enter token name
3. Set expiration date
4. Select role (Guest, Reporter, Developer, Maintainer)
5. Select scopes
6. Click "Create project access token"

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "name": "Project Token",
    "scopes": ["api"],
    "access_level": 40,
    "expires_at": "2025-12-31"
  }' \
  "https://gitlab.com/api/v4/projects/:id/access_tokens"
```

**Access Levels**:
- 10: Guest
- 20: Reporter
- 30: Developer
- 40: Maintainer
- 50: Owner

### 8. Group Access Tokens

Group-level tokens for all group projects.

#### Creating Group Access Tokens

**Via UI**:
1. Navigate to Group Settings > Access Tokens
2. Configure token (similar to project tokens)
3. Token applies to all group projects

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "name": "Group Token",
    "scopes": ["api"],
    "access_level": 40
  }' \
  "https://gitlab.com/api/v4/groups/:id/access_tokens"
```

### 9. LDAP Authentication

Enterprise edition feature for LDAP/Active Directory integration.

#### LDAP Configuration

**`/etc/gitlab/gitlab.rb`**:
```ruby
gitlab_rails['ldap_enabled'] = true
gitlab_rails['ldap_servers'] = YAML.load <<-EOS
  main:
    label: 'LDAP'
    host: 'ldap.example.com'
    port: 636
    uid: 'sAMAccountName'
    encryption: 'simple_tls'
    verify_certificates: true
    bind_dn: 'CN=query user,OU=Users,DC=example,DC=com'
    password: 'password'
    active_directory: true
    base: 'OU=Users,DC=example,DC=com'
    user_filter: '(memberOf=CN=GitLab Users,OU=Groups,DC=example,DC=com)'
EOS
```

### 10. SAML Authentication

Enterprise edition SAML SSO support.

#### SAML Configuration

**`/etc/gitlab/gitlab.rb`**:
```ruby
gitlab_rails['omniauth_enabled'] = true
gitlab_rails['omniauth_allow_single_sign_on'] = ['saml']
gitlab_rails['omniauth_block_auto_created_users'] = false

gitlab_rails['omniauth_providers'] = [
  {
    name: 'saml',
    args: {
      assertion_consumer_service_url: 'https://gitlab.example.com/users/auth/saml/callback',
      idp_cert_fingerprint: 'XX:XX:XX...',
      idp_sso_target_url: 'https://idp.example.com/sso',
      issuer: 'https://gitlab.example.com',
      name_identifier_format: 'urn:oasis:names:tc:SAML:2.0:nameid-format:persistent'
    },
    label: 'Company SSO'
  }
]
```

## Security Best Practices

### Token Management

1. **Minimize Scope**: Use least privilege principle
2. **Set Expiration**: Always set token expiration dates
3. **Rotate Regularly**: Rotate tokens on schedule
4. **Secure Storage**: Use secret managers (Vault, AWS Secrets Manager)
5. **Monitor Usage**: Audit token usage regularly
6. **Revoke Unused**: Remove tokens no longer needed

### Secure Token Storage

**Environment Variables**:
```bash
export GITLAB_TOKEN="your_token"
```

**Git Credentials**:
```bash
# Store credentials securely
git config --global credential.helper 'cache --timeout=3600'
```

**Python example**:
```python
import os
from dotenv import load_dotenv

load_dotenv()
GITLAB_TOKEN = os.getenv('GITLAB_TOKEN')
```

**Docker Secrets**:
```bash
echo "your_token" | docker secret create gitlab_token -
```

### Two-Factor Authentication (2FA)

#### Enabling 2FA

1. Navigate to User Settings > Account > Two-Factor Authentication
2. Scan QR code with authenticator app
3. Enter verification code
4. Save recovery codes securely

#### Using 2FA with Git

When 2FA is enabled, use:
- Personal access tokens instead of passwords
- SSH keys for Git operations

```bash
# Use token as password
git clone https://oauth2:<your_token>@gitlab.com/username/project.git
```

### IP Allowlisting

Restrict API access by IP address (Premium/Ultimate).

**Group/Instance Settings**:
1. Navigate to Settings > General > Allowed IP addresses
2. Add IP ranges
3. Save changes

## API Authentication Examples

### Python (requests library)

```python
import requests

class GitLabClient:
    def __init__(self, token, base_url="https://gitlab.com"):
        self.token = token
        self.base_url = base_url
        self.headers = {"PRIVATE-TOKEN": token}

    def get(self, endpoint):
        url = f"{self.base_url}/api/v4/{endpoint}"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()

    def post(self, endpoint, data):
        url = f"{self.base_url}/api/v4/{endpoint}"
        response = requests.post(url, headers=self.headers, json=data)
        response.raise_for_status()
        return response.json()

# Usage
client = GitLabClient(token="your_token")
projects = client.get("projects")
```

### JavaScript (fetch API)

```javascript
class GitLabClient {
  constructor(token, baseURL = 'https://gitlab.com') {
    this.token = token;
    this.baseURL = baseURL;
  }

  async get(endpoint) {
    const response = await fetch(`${this.baseURL}/api/v4/${endpoint}`, {
      headers: {
        'PRIVATE-TOKEN': this.token
      }
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return await response.json();
  }

  async post(endpoint, data) {
    const response = await fetch(`${this.baseURL}/api/v4/${endpoint}`, {
      method: 'POST',
      headers: {
        'PRIVATE-TOKEN': this.token,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return await response.json();
  }
}

// Usage
const client = new GitLabClient('your_token');
const projects = await client.get('projects');
```

### Go

```go
package main

import (
    "fmt"
    "github.com/xanzy/go-gitlab"
)

func main() {
    git, err := gitlab.NewClient("your_token")
    if err != nil {
        panic(err)
    }

    // List projects
    projects, _, err := git.Projects.ListProjects(&gitlab.ListProjectsOptions{})
    if err != nil {
        panic(err)
    }

    for _, project := range projects {
        fmt.Printf("Project: %s\n", project.Name)
    }
}
```

### Ruby

```ruby
require 'gitlab'

# Initialize client
Gitlab.configure do |config|
  config.endpoint = 'https://gitlab.com/api/v4'
  config.private_token = 'your_token'
end

# List projects
projects = Gitlab.projects

projects.each do |project|
  puts "Project: #{project.name}"
end
```

## Troubleshooting

### Common Authentication Issues

**1. 401 Unauthorized**
- Verify token is valid and not expired
- Check token has required scopes
- Ensure token is included in request headers

**2. 403 Forbidden**
- Check user permissions on resource
- Verify token scope includes required access
- For protected resources, check branch protection rules

**3. SSH Connection Failed**
- Verify SSH key is added to GitLab
- Check SSH agent is running
- Test connection: `ssh -T git@gitlab.com`
- Verify SSH key permissions (600 for private key)

**4. Token Not Working with 2FA**
- Use personal access token instead of password
- Or use SSH keys for Git operations

**5. Deploy Token Issues**
- Verify token hasn't expired
- Check token scopes match operation
- Ensure token is enabled in project settings

## Additional Resources

- Personal Access Tokens: https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html
- OAuth 2.0: https://docs.gitlab.com/ee/api/oauth2.html
- SSH Keys: https://docs.gitlab.com/ee/user/ssh.html
- Deploy Keys: https://docs.gitlab.com/ee/user/project/deploy_keys/
- Deploy Tokens: https://docs.gitlab.com/ee/user/project/deploy_tokens/
