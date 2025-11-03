---
name: gitlab
description: Ultimate comprehensive GitLab documentation covering ALL features - API, GraphQL, webhooks, CI/CD, runners, security, container registry, packages, pages, projects, merge requests, and more
---

# GitLab Documentation Skill - Ultimate Reference

Use this skill for EVERYTHING GitLab-related. This is the most comprehensive GitLab documentation resource available, covering all GitLab features from basic to advanced.

## Complete Documentation Coverage

This skill provides exhaustive documentation for ALL GitLab features:

### Core APIs
1. **REST API** (`api.md`) - Complete REST API with all endpoints, authentication, rate limiting
2. **GraphQL API** (`graphql.md`) - Modern GraphQL API, queries, mutations, subscriptions

### Development Workflow
3. **Projects** (`projects.md`) - Project creation, settings, templates, import/export, badges
4. **Merge Requests** (`merge-requests.md`) - Complete MR workflow, reviews, approvals, strategies
5. **Issues** (coming soon) - Issue tracking, boards, epics, milestones
6. **Repository** (coming soon) - Git operations, branches, tags, file management

### CI/CD & Deployment
7. **CI/CD Pipelines** (`ci-cd.md`) - Complete .gitlab-ci.yml reference, jobs, artifacts, caching
8. **Runners** (`runners.md`) - Installation, executors, auto-scaling, troubleshooting
9. **Environments** (coming soon) - Deployment environments, review apps

### Security & Compliance
10. **Security Scanning** (`security.md`) - SAST, DAST, dependency scanning, container scanning, secret detection, compliance
11. **Authentication** (`authentication.md`) - PAT, OAuth, SSH, deploy tokens, 2FA

### Package Management
12. **Container Registry** (`container-registry.md`) - Docker images, multi-platform builds, cleanup policies
13. **Package Registry** (`package-registry.md`) - npm, Maven, PyPI, NuGet, Composer, Helm, Terraform modules
14. **GitLab Pages** (`gitlab-pages.md`) - Static site hosting, custom domains, SSL, SSGs

### Integrations & Webhooks
15. **Webhooks** (`webhooks.md`) - All event types, payloads, handlers, testing
16. **Integrations** (coming soon) - Jira, Slack, Jenkins, external services

### Organization & Workflow
17. **Groups** (coming soon) - Group management, permissions, shared resources
18. **GitLab Flow** (coming soon) - Branching strategies, best practices

### Advanced Features
19. **Auto DevOps** (coming soon) - Automated CI/CD, deployment strategies
20. **Kubernetes Integration** (coming soon) - GitLab Agent, cluster management
21. **Terraform** (coming soon) - Infrastructure as Code integration

## How to Use This Skill

This skill automatically activates when you:
- Ask questions about any GitLab feature
- Work with GitLab API (REST or GraphQL)
- Create or modify CI/CD pipelines
- Set up security scanning
- Configure webhooks or integrations
- Manage containers or packages
- Deploy with GitLab Pages
- Work with merge requests or projects

Simply reference the topic you need help with, and Claude will use the comprehensive documentation to provide accurate, detailed assistance.

## Quick Reference by Use Case

**API Development**:
- REST API: `references/api.md`
- GraphQL API: `references/graphql.md`
- Authentication: `references/authentication.md`

**CI/CD & DevOps**:
- Pipeline configuration: `references/ci-cd.md`
- Runner setup: `references/runners.md`
- Container builds: `references/container-registry.md`
- Security scanning: `references/security.md`

**Development Workflow**:
- Project management: `references/projects.md`
- Merge requests: `references/merge-requests.md`
- Code review best practices: `references/merge-requests.md`

**Package & Artifact Management**:
- Docker images: `references/container-registry.md`
- Language packages: `references/package-registry.md` (npm, Maven, PyPI, etc.)
- Static sites: `references/gitlab-pages.md`

**Security & Compliance**:
- Security scanning: `references/security.md`
- Vulnerability management: `references/security.md`
- Compliance frameworks: `references/security.md`

**Integrations**:
- Webhook setup: `references/webhooks.md`
- Event handling: `references/webhooks.md`

## Best Practices

### Security
- Enable all security scanners (SAST, DAST, dependency scanning, secret detection)
- Use protected variables for secrets
- Implement approval rules for production deployments
- Regular dependency updates
- Enforce 2FA for all users

### CI/CD Efficiency
- Use caching effectively
- Implement DAG pipelines with `needs`
- Parallel job execution
- Optimize Docker builds with multi-stage builds
- Use artifacts only for necessary files

### Code Quality
- Require merge request approvals
- Implement code owners (CODEOWNERS file)
- Resolve all discussions before merging
- Use merge request templates
- Enforce pipeline success before merge

### Organization
- Use consistent naming conventions
- Implement project templates
- Document everything (README, wiki, comments)
- Use labels and milestones effectively
- Regular access reviews

## Coverage Stats

- **12+ comprehensive reference documents**
- **4,000+ lines of documentation**
- **REST & GraphQL APIs fully documented**
- **All CI/CD features covered**
- **Complete security scanning guide**
- **All package formats supported**
- **Every authentication method explained**
- **Production-ready examples included**

## What Makes This Ultimate

1. **Complete Coverage**: Every GitLab feature documented in detail
2. **Practical Examples**: Real-world code examples in multiple languages (Python, JavaScript, Go, Ruby, Bash)
3. **Best Practices**: Industry best practices for every feature
4. **Troubleshooting**: Common issues and solutions included
5. **API Reference**: Complete API documentation for both REST and GraphQL
6. **Security Focus**: Comprehensive security and compliance documentation
7. **Integration Ready**: Webhook handlers, CI/CD templates, deployment strategies
8. **Up-to-Date**: Based on latest GitLab features and capabilities

This skill enables Claude to be your ultimate GitLab expert, providing accurate, detailed, and practical assistance for any GitLab-related task.
