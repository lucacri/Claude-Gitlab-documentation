# GitLab ULTIMATE Edition Documentation Skill for Claude Code

**Complete documentation for GitLab ULTIMATE Edition (Self-Hosted)** - Every feature, API, and configuration for GitLab Ultimate tier, with special focus on self-hosted enterprise installations.

## Overview

This plugin provides **complete documentation for GitLab ULTIMATE Edition**, the top-tier GitLab subscription with ALL features unlocked. Specifically designed for self-hosted GitLab Ultimate instances, this skill covers every Ultimate-exclusive feature including Epics, Geo replication, advanced analytics, compliance frameworks, and enterprise integrations.

**Perfect for**: Self-hosted GitLab Ultimate users, enterprise teams, DevOps engineers working with the full GitLab platform.

## Complete GitLab ULTIMATE Feature Coverage

### Core APIs & Development (2500+ lines)
- **REST API** - Complete API reference
- **GraphQL API** - Full GraphQL implementation
- **Projects** - Complete project management
- **Merge Requests** - Full MR workflow with approvals
- **Webhooks** - All event types with handlers

### **ULTIMATE: Issues & Planning** (600+ lines)
- **Epics** ⭐ - Multi-project issue tracking
- **Epic Boards** ⭐ - Visualize epic workflows
- **Roadmaps** ⭐ - Timeline planning with Gantt charts
- **Iterations** ⭐ - Sprint planning and cadences
- **Issue Weights** ⭐ - Capacity tracking
- **Health Status** ⭐ - Issue health monitoring
- **Requirements** ⭐ - Requirements management
- **Test Cases** ⭐ - Test case tracking

### CI/CD & Deployment (2000+ lines)
- **CI/CD Pipelines** - Complete `.gitlab-ci.yml` reference
- **Runners** - All executors, auto-scaling
- **GitLab Pages** - Static site hosting

### **ULTIMATE: Security & Compliance** (1500+ lines)
- **Security Scanning** - All scanners (SAST, DAST, etc.)
- **Compliance Frameworks** ⭐ - SOC 2, PCI-DSS enforcement
- **Audit Events** ⭐ - Comprehensive audit logging
- **Audit Event Streaming** ⭐ - Stream to SIEM
- **Security Policies** ⭐ - Scan execution policies
- **License Compliance** ⭐ - Dependency license management
- **Credentials Inventory** ⭐ - Token tracking
- **External Status Checks** ⭐ - External approvals

### Package & Container Management (1200+ lines)
- **Container Registry** - Docker builds, multi-platform
- **Package Registry** - All package formats

### **ULTIMATE: Analytics & Insights** (500+ lines)
- **Value Stream Analytics** ⭐ - Custom value streams
- **DORA Metrics** ⭐ - Four key DevOps metrics
- **Productivity Analytics** ⭐ - Team insights
- **Code Review Analytics** ⭐ - Review efficiency
- **Contribution Analytics** ⭐ - Member tracking
- **DevOps Adoption** ⭐ - Feature adoption metrics
- **Custom Insights** ⭐ - YAML dashboards

### **ULTIMATE: Operations & Incidents** (600+ lines)
- **Incident Management** ⭐ - Incident tracking
- **On-Call Schedules** ⭐ - Rotation management
- **Escalation Policies** ⭐ - Multi-level escalation
- **Alert Management** ⭐ - Integrated alerting
- **Status Pages** ⭐ - Public status pages
- **Error Tracking** ⭐ - Sentry integration

### **ULTIMATE: Enterprise & Geo** (800+ lines - Self-Hosted)
- **Geo Replication** ⭐ - Multi-site replication (CRITICAL!)
- **SAML SSO** ⭐ - Enterprise single sign-on
- **SCIM Provisioning** ⭐ - Automated user provisioning
- **Group Access Tokens** ⭐ - Group-level automation
- **IP Allowlisting** ⭐ - IP-based access control
- **Email Domain Restrictions** ⭐ - Domain-based membership

⭐ = **GitLab ULTIMATE Exclusive Feature**

## Documentation Stats

- **18 comprehensive reference documents**
- **7,000+ lines of detailed documentation**
- **150+ production-ready code examples**
- **Complete ULTIMATE tier coverage**
- **Self-hosted specific features** (Geo, LDAP, advanced auth)
- **Enterprise features** (SAML SSO, SCIM, compliance)
- **All Ultimate-exclusive features** documented
- **Multiple programming languages** (Python, JavaScript, Go, Ruby, Bash)

## Installation

### Quick Install

```bash
# In Claude Code, use the /plugin command:
/plugin https://github.com/lucacri/Claude-Gitlab-documentation
```

### Manual Installation

```bash
# Clone the repository
git clone https://github.com/lucacri/Claude-Gitlab-documentation.git

# In Claude Code
/plugin /path/to/Claude-Gitlab-documentation
```

## Usage

The skill automatically activates when you:

- Ask questions about ANY GitLab feature
- Work with GitLab API (REST or GraphQL)
- Create or modify CI/CD pipelines
- Set up security scanning
- Configure webhooks or integrations
- Manage containers or packages
- Deploy with GitLab Pages
- Work with merge requests, projects, or any GitLab feature

### Example Interactions

**GraphQL API Query**:
```
You: "Show me how to fetch merge requests using GraphQL with pagination"
Claude: *Provides complete GraphQL query with pagination using gitlab skill*
```

**Security Scanning Setup**:
```
You: "Set up SAST, dependency scanning, and container scanning in my pipeline"
Claude: *Creates complete .gitlab-ci.yml with all security scanners configured*
```

**Multi-Platform Docker Build**:
```
You: "Build Docker image for AMD64, ARM64, and ARMv7"
Claude: *Provides complete pipeline with buildx configuration*
```

**Package Publishing**:
```
You: "Publish npm package to GitLab Package Registry"
Claude: *Complete setup with .npmrc, authentication, and CI/CD pipeline*
```

## Complete Documentation Reference

### Core APIs & Development
- **`api.md`** (300 lines) - REST API reference
- **`graphql.md`** (500 lines) - GraphQL API complete
- **`projects.md`** (450 lines) - Project management
- **`merge-requests.md`** (600 lines) - MR workflows
- **`webhooks.md`** (450 lines) - All webhook events

### ULTIMATE: Planning & Portfolio
- **`issues-epics.md`** (600 lines) - **Epics, roadmaps, iterations, requirements, test cases**

### CI/CD & Deployment
- **`ci-cd.md`** (600 lines) - Complete pipeline reference
- **`runners.md`** (500 lines) - All executors and configs
- **`gitlab-pages.md`** (400 lines) - Static site hosting

### ULTIMATE: Security & Compliance
- **`security.md`** (500 lines) - All security scanners, policies
- **`compliance-governance.md`** (400 lines) - **Compliance frameworks, audit events, streaming**
- **`authentication.md`** (450 lines) - All auth methods including LDAP, SAML

### ULTIMATE: Analytics
- **`analytics.md`** (500 lines) - **Value Stream Analytics, DORA metrics, productivity, insights**

### ULTIMATE: Operations
- **`operations-incidents.md`** (600 lines) - **Incident management, on-call, alerts, status pages**

### ULTIMATE: Enterprise (Self-Hosted)
- **`geo-replication.md`** (500 lines) - **Geo replication, disaster recovery, failover**
- **`groups-administration.md`** (400 lines) - **SAML SSO, SCIM, group tokens, IP allowlisting**

### Package Management
- **`container-registry.md`** (400 lines) - Docker registry
- **`package-registry.md`** (350 lines) - All package formats

## Why This is Ultimate

1. **Exhaustive Coverage** - Every GitLab feature documented in detail
2. **Production-Ready Examples** - Real-world code in multiple languages
3. **Best Practices Built-In** - Industry standards for every feature
4. **Troubleshooting Included** - Common issues and solutions
5. **Complete API Reference** - Both REST and GraphQL fully documented
6. **Security-First** - Comprehensive security and compliance docs
7. **Integration Ready** - Webhook handlers, CI/CD templates, deployments
8. **Always Current** - Based on latest GitLab capabilities

## Use Cases

Perfect for:

- **API Development** - Building integrations with GitLab
- **CI/CD Engineering** - Creating sophisticated pipelines
- **DevSecOps** - Implementing security scanning and compliance
- **Container Management** - Building and managing Docker images
- **Package Publishing** - Publishing packages in any format
- **Static Site Hosting** - Deploying documentation and websites
- **GitLab Administration** - Managing GitLab instances
- **Learning GitLab** - Comprehensive reference for all features

## What Makes This Different

Unlike basic GitLab documentation, this skill provides:

- **Contextual Help** - Claude understands your needs and references the right docs
- **Code Generation** - Get complete, working examples instantly
- **Best Practices** - Not just "how" but "how best"
- **Multiple Languages** - Examples in Python, JS, Go, Ruby, Bash
- **Complete Coverage** - Nothing left out, everything documented
- **Real-World Focus** - Production-ready solutions, not just tutorials

## Contributing

Contributions welcome! To add or improve documentation:

1. Fork the repository
2. Create a feature branch
3. Add/update documentation in `gitlab-skill/skills/gitlab/references/`
4. Update `SKILL.md` if adding new sections
5. Submit a pull request

## Project Structure

```
Claude-Gitlab-documentation/
├── .claude-plugin/
│   └── marketplace.json          # Plugin marketplace config
├── gitlab-skill/
│   ├── .claude-plugin/
│   │   └── plugin.json           # Plugin metadata
│   └── skills/
│       └── gitlab/
│           ├── SKILL.md          # Skill definition (updated with all features)
│           └── references/       # 12+ comprehensive docs (5000+ lines)
│               ├── api.md
│               ├── graphql.md
│               ├── projects.md
│               ├── merge-requests.md
│               ├── webhooks.md
│               ├── ci-cd.md
│               ├── runners.md
│               ├── gitlab-pages.md
│               ├── security.md
│               ├── authentication.md
│               ├── container-registry.md
│               └── package-registry.md
├── README.md                     # This file
└── LICENSE
```

## Requirements

- Claude Code (latest version)
- GitLab account (for testing features)

## Version History

### v3.0.0 - GitLab ULTIMATE Edition (2025-01-15)
- **18 comprehensive reference documents** (7000+ lines)
- **Complete GitLab Ultimate tier coverage**
- Added **Epics, Roadmaps, Iterations** (issues-epics.md)
- Added **Value Stream Analytics & DORA Metrics** (analytics.md)
- Added **Compliance Frameworks & Audit Streaming** (compliance-governance.md)
- Added **Incident Management & On-Call** (operations-incidents.md)
- Added **Geo Replication for Self-Hosted** (geo-replication.md)
- Added **SAML SSO & SCIM Provisioning** (groups-administration.md)
- All Ultimate-exclusive features documented
- Self-hosted specific configurations
- 150+ production-ready examples

### v2.0.0 - Comprehensive Edition (2025-01-15)
- Initial comprehensive documentation
- 12 reference documents
- Core features covered

### v1.0.0 (2025-01-15)
- Initial release with core documentation

## License

MIT License - See [LICENSE](LICENSE) file for details.

## Support

- **Issues**: https://github.com/lucacri/Claude-Gitlab-documentation/issues
- **Official GitLab Docs**: https://docs.gitlab.com/
- **GitLab API**: https://docs.gitlab.com/ee/api/
- **Claude Code Docs**: https://docs.claude.com/en/docs/claude-code

## Author

Created for the GitLab and Claude Code community

---

**Make Claude Code your ultimate GitLab expert!** 🚀
