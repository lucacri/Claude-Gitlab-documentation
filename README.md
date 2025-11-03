# GitLab Documentation Skill for Claude Code

A comprehensive GitLab documentation skill for Claude Code that provides instant access to GitLab API, webhooks, CI/CD pipelines, runners, authentication methods, and more.

## Overview

This plugin adds a GitLab skill to Claude Code that automatically activates when you're working with GitLab-related tasks. It provides comprehensive, searchable documentation covering all aspects of GitLab, enabling Claude to assist you more effectively with GitLab projects.

## Features

The GitLab skill includes comprehensive documentation for:

- **GitLab REST API** - Complete API reference with all endpoints, parameters, and examples
- **Webhooks** - Event types, payload structures, and webhook handler examples
- **CI/CD Pipelines** - `.gitlab-ci.yml` syntax, jobs, stages, artifacts, caching, and more
- **GitLab Runners** - Installation, configuration, executors, and troubleshooting
- **Authentication** - Personal access tokens, OAuth, SSH keys, deploy tokens, and security best practices

## Installation

### Using Claude Code

1. Install the plugin using the `/plugin` command:
   ```
   /plugin https://github.com/lucacri/Claude-Gitlab-documentation
   ```

2. The skill will be automatically enabled and ready to use.

### Manual Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/lucacri/Claude-Gitlab-documentation.git
   ```

2. In Claude Code, run:
   ```
   /plugin /path/to/Claude-Gitlab-documentation
   ```

## Usage

Once installed, the GitLab skill automatically activates when you:

- Ask questions about GitLab features
- Work with GitLab API
- Create or modify `.gitlab-ci.yml` files
- Set up webhooks
- Configure runners
- Need authentication information

### Example Interactions

**API Usage**:
```
You: "How do I list all projects using the GitLab API?"
Claude: *Uses GitLab skill to provide API endpoint, authentication, and example code*
```

**CI/CD Configuration**:
```
You: "Create a GitLab CI pipeline that builds a Docker image and deploys to production"
Claude: *References CI/CD documentation to create a complete .gitlab-ci.yml*
```

**Webhook Setup**:
```
You: "Show me how to set up a webhook for merge request events"
Claude: *Provides webhook configuration and handler code examples*
```

**Runner Configuration**:
```
You: "Help me set up a GitLab Runner with Docker executor"
Claude: *Uses runner documentation to guide installation and configuration*
```

## Documentation Structure

```
gitlab-skill/
├── skills/
│   └── gitlab/
│       ├── SKILL.md                    # Skill definition and instructions
│       └── references/                 # Reference documentation
│           ├── api.md                  # GitLab REST API
│           ├── webhooks.md             # Webhooks and event handlers
│           ├── ci-cd.md                # CI/CD pipeline configuration
│           ├── runners.md              # GitLab Runner setup and management
│           └── authentication.md       # Authentication methods
```

## Documentation Contents

### API Reference (`api.md`)
- Authentication methods (PAT, OAuth, Job tokens)
- Rate limiting
- Core API resources:
  - Projects, Repositories, Commits, Branches
  - Merge Requests, Issues
  - Pipelines, Jobs, Artifacts
  - Users, Groups, Runners
  - Variables, Webhooks, Tags, Releases
- Error handling
- Best practices

### Webhooks (`webhooks.md`)
- Webhook configuration and security
- Event types and payloads:
  - Push, Tag Push, Issue, Merge Request
  - Pipeline, Job, Deployment
  - Wiki Page, Release
- Testing and debugging
- Example webhook handlers (Python, Node.js)
- Best practices and troubleshooting

### CI/CD (`ci-cd.md`)
- `.gitlab-ci.yml` structure and syntax
- Pipeline configuration:
  - Stages, Jobs, Scripts
  - Images and Services
  - Variables (predefined and custom)
- Artifacts and caching
- Job control (rules, only/except, when)
- Environments and deployments
- Advanced features:
  - Includes, Extends, Parallel jobs
  - Triggers, Docker builds
  - Security scanning
- Complete examples and best practices

### Runners (`runners.md`)
- Runner types (Shared, Group, Project)
- Installation on various platforms
- Executors:
  - Shell, Docker, Kubernetes
  - Docker Machine, VirtualBox, SSH
- Configuration and management
- Caching strategies (Local, S3, GCS, Azure)
- Security (Protected runners, Tags, Privileged mode)
- Monitoring and troubleshooting
- Auto-scaling

### Authentication (`authentication.md`)
- Personal Access Tokens
- OAuth 2.0
- SSH Keys
- Deploy Keys and Tokens
- Job Tokens (CI/CD)
- Project and Group Access Tokens
- LDAP and SAML (Enterprise)
- Security best practices
- Two-Factor Authentication
- API client examples (Python, JavaScript, Go, Ruby)

## Contributing

Contributions are welcome! If you'd like to improve the documentation or add new sections:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

### Adding New Documentation

To add new reference documentation:

1. Create a new `.md` file in `gitlab-skill/skills/gitlab/references/`
2. Update `SKILL.md` to reference the new documentation
3. Follow the existing documentation format and style

## Use Cases

This skill is particularly useful for:

- **API Integration**: Building tools and integrations with GitLab
- **CI/CD Setup**: Creating and optimizing GitLab CI/CD pipelines
- **Automation**: Automating GitLab workflows with scripts
- **DevOps**: Setting up and managing GitLab infrastructure
- **Learning**: Understanding GitLab features and best practices

## Requirements

- Claude Code (latest version)
- GitLab account (for testing and implementation)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For issues or questions:

- Open an issue on GitHub
- Check the documentation in the `references/` directory
- Refer to official GitLab documentation at https://docs.gitlab.com

## Acknowledgments

- Documentation based on GitLab official documentation
- Community contributions and feedback
- Claude Code plugin system by Anthropic

## Version History

### v1.0.0 (2025-01-15)
- Initial release
- Comprehensive API documentation
- Webhooks reference with examples
- Complete CI/CD pipeline documentation
- Runner setup and management guide
- Authentication methods and security
- Best practices and troubleshooting

## Roadmap

Future enhancements may include:

- GitLab GraphQL API documentation
- Additional language examples (Java, PHP, etc.)
- GitLab Pages documentation
- Package registry documentation
- GitLab Kubernetes Agent documentation
- Advanced security features (SAST, DAST, etc.)
- GitLab Terraform integration

## Author

Created for the GitLab and Claude Code community

## Related Resources

- [GitLab Official Documentation](https://docs.gitlab.com/)
- [GitLab API Documentation](https://docs.gitlab.com/ee/api/)
- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [Claude Skills](https://www.anthropic.com/news/skills)
