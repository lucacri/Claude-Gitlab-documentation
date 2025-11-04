# GitLab Package Registry Reference

## Overview

GitLab Package Registry allows you to publish and share packages within your organization. Supports multiple package formats.

## Supported Package Types

- **Maven** (Java)
- **npm** (JavaScript/Node.js)
- **PyPI** (Python)
- **NuGet** (.NET)
- **Composer** (PHP)
- **Conan** (C/C++)
- **Go Modules**
- **Helm Charts** (Kubernetes)
- **Terraform Modules**
- **Generic packages**
- **RubyGems** (Ruby)
- **Debian** packages
- **RPM** packages

## npm Packages

### Publish npm Package

**.npmrc**:
```
@scope:registry=https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/
//gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=${CI_JOB_TOKEN}
```

**package.json**:
```json
{
  "name": "@scope/package-name",
  "version": "1.0.0",
  "publishConfig": {
    "@scope:registry": "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/"
  }
}
```

**.gitlab-ci.yml**:
```yaml
publish:
  image: node:18
  script:
    - echo "@scope:registry=https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/" > .npmrc
    - echo "//gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=${CI_JOB_TOKEN}" >> .npmrc
    - npm publish
  only:
    - tags
```

### Install npm Package

```bash
# Configure registry
npm config set @scope:registry https://gitlab.com/api/v4/projects/PROJECT_ID/packages/npm/

# With auth token
npm config set -- '//gitlab.com/api/v4/projects/PROJECT_ID/packages/npm/:_authToken' "TOKEN"

# Install
npm install @scope/package-name
```

## Maven Packages

### Publish Maven Package

**pom.xml**:
```xml
<repositories>
  <repository>
    <id>gitlab-maven</id>
    <url>https://gitlab.com/api/v4/projects/PROJECT_ID/packages/maven</url>
  </repository>
</repositories>

<distributionManagement>
  <repository>
    <id>gitlab-maven</id>
    <url>https://gitlab.com/api/v4/projects/PROJECT_ID/packages/maven</url>
  </repository>
  <snapshotRepository>
    <id>gitlab-maven</id>
    <url>https://gitlab.com/api/v4/projects/PROJECT_ID/packages/maven</url>
  </snapshotRepository>
</distributionManagement>
```

**settings.xml**:
```xml
<settings>
  <servers>
    <server>
      <id>gitlab-maven</id>
      <configuration>
        <httpHeaders>
          <property>
            <name>Job-Token</name>
            <value>${env.CI_JOB_TOKEN}</value>
          </property>
        </httpHeaders>
      </configuration>
    </server>
  </servers>
</settings>
```

**.gitlab-ci.yml**:
```yaml
deploy:
  image: maven:3-openjdk-11
  script:
    - mvn deploy -s ci_settings.xml
  only:
    - tags
```

## PyPI Packages

### Publish Python Package

**.pypirc**:
```ini
[distutils]
index-servers =
    gitlab

[gitlab]
repository = https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/pypi
username = gitlab-ci-token
password = ${CI_JOB_TOKEN}
```

**.gitlab-ci.yml**:
```yaml
publish:
  image: python:3.11
  script:
    - pip install twine build
    - python -m build
    - TWINE_PASSWORD=${CI_JOB_TOKEN} TWINE_USERNAME=gitlab-ci-token python -m twine upload --repository-url https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/pypi dist/*
  only:
    - tags
```

### Install Python Package

```bash
# pip install
pip install --index-url https://__token__:${PERSONAL_ACCESS_TOKEN}@gitlab.com/api/v4/projects/PROJECT_ID/packages/pypi/simple package-name

# requirements.txt
--index-url https://__token__:${PERSONAL_ACCESS_TOKEN}@gitlab.com/api/v4/projects/PROJECT_ID/packages/pypi/simple
package-name==1.0.0
```

## NuGet Packages

### Publish NuGet Package

**.gitlab-ci.yml**:
```yaml
publish:
  stage: deploy
  image: mcr.microsoft.com/dotnet/sdk:7.0
  script:
    - dotnet pack -c Release
    - dotnet nuget add source "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/nuget/index.json" --name gitlab --username gitlab-ci-token --password ${CI_JOB_TOKEN} --store-password-in-clear-text
    - dotnet nuget push "bin/Release/*.nupkg" --source gitlab
  only:
    - tags
```

### Install NuGet Package

```bash
# Add source
dotnet nuget add source "https://gitlab.com/api/v4/projects/PROJECT_ID/packages/nuget/index.json" --name gitlab --username USERNAME --password TOKEN

# Install
dotnet add package PackageName
```

## Composer Packages (PHP)

### Publish Composer Package

**composer.json**:
```json
{
  "name": "vendor/package-name",
  "version": "1.0.0",
  "type": "library",
  "repositories": [
    {
      "type": "composer",
      "url": "https://gitlab.com/api/v4/group/GROUP_ID/-/packages/composer/packages.json"
    }
  ]
}
```

**.gitlab-ci.yml**:
```yaml
publish:
  image: curlimages/curl:latest
  script:
    - 'curl --header "Job-Token: ${CI_JOB_TOKEN}" --data tag=${CI_COMMIT_TAG} "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/composer"'
  only:
    - tags
```

## Generic Packages

### Publish Generic Package

```bash
curl --header "JOB-TOKEN: ${CI_JOB_TOKEN}" \
  --upload-file path/to/file.zip \
  "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/generic/package-name/1.0.0/file.zip"
```

**.gitlab-ci.yml**:
```yaml
upload:
  stage: deploy
  script:
    - 'curl --header "JOB-TOKEN: ${CI_JOB_TOKEN}" --upload-file myfile.zip "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/mypackage/1.0.0/myfile.zip"'
```

### Download Generic Package

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/PROJECT_ID/packages/generic/package-name/1.0.0/file.zip" \
  -o file.zip
```

## Helm Charts

### Publish Helm Chart

**.gitlab-ci.yml**:
```yaml
publish:
  image: alpine/helm:latest
  script:
    - helm package chart/
    - 'curl --request POST --user gitlab-ci-token:${CI_JOB_TOKEN} --form "chart=@mychart-1.0.0.tgz" "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/helm/api/stable/charts"'
  only:
    - tags
```

### Install Helm Chart

```bash
# Add repo
helm repo add --username <username> --password <token> gitlab https://gitlab.com/api/v4/projects/PROJECT_ID/packages/helm/stable

# Install
helm install myrelease gitlab/mychart
```

## Terraform Modules

### Publish Terraform Module

**Project structure**:
```
terraform-module/
├── main.tf
├── variables.tf
├── outputs.tf
└── README.md
```

**.gitlab-ci.yml**:
```yaml
upload:
  image: curlimages/curl:latest
  script:
    - tar -czf module.tar.gz *
    - 'curl --header "JOB-TOKEN: ${CI_JOB_TOKEN}" --upload-file module.tar.gz "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/terraform/modules/my-module/my-system/1.0.0/file"'
  only:
    - tags
```

### Use Terraform Module

```hcl
module "example" {
  source = "gitlab.com/group/project//modules/my-module"
  version = "1.0.0"
}
```

## Conan Packages (C/C++)

### Publish Conan Package

**.gitlab-ci.yml**:
```yaml
publish:
  image: conanio/gcc11
  script:
    - conan remote add gitlab https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/conan
    - conan user ci_user -r gitlab -p ${CI_JOB_TOKEN}
    - conan create . mycompany/stable
    - conan upload "*" --all -r gitlab --confirm
  only:
    - tags
```

## Go Modules

### Publish Go Module

**Automatic**: Just create a Git tag

```bash
git tag v1.0.0
git push origin v1.0.0
```

**Use module**:
```go
import "gitlab.com/group/project/module"
```

```bash
go get gitlab.com/group/project/module@v1.0.0
```

**Configure private modules**:
```bash
git config --global url."https://oauth2:${GITLAB_TOKEN}@gitlab.com".insteadOf "https://gitlab.com"
export GOPRIVATE="gitlab.com/group/*"
```

## Package Management API

### List Packages

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/PROJECT_ID/packages"
```

### Get Package

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/PROJECT_ID/packages/PACKAGE_ID"
```

### Delete Package

```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/PROJECT_ID/packages/PACKAGE_ID"
```

### List Package Files

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/PROJECT_ID/packages/PACKAGE_ID/package_files"
```

## Package Cleanup Policies

### Configure Cleanup

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/PROJECT_ID" \
  --data "packages_cleanup_policy_attributes[keep_n_duplicated_package_files]=10" \
  --data "packages_cleanup_policy_attributes[older_than]=90d"
```

## Best Practices

### 1. Versioning

- Use semantic versioning (SemVer)
- Tag releases properly
- Maintain changelog
- Document breaking changes

### 2. Security

- Use deploy tokens for CI/CD
- Rotate tokens regularly
- Scan packages for vulnerabilities
- Sign packages when possible

### 3. Organization

- Use consistent naming
- Group related packages
- Document package usage
- Maintain package metadata

### 4. CI/CD Integration

```yaml
stages:
  - build
  - test
  - publish

build:
  stage: build
  script:
    - npm run build

test:
  stage: test
  script:
    - npm test

publish:
  stage: publish
  script:
    - npm publish
  only:
    - tags
  when: manual
```

## Additional Resources

- Package Registry Docs: https://docs.gitlab.com/ee/user/packages/package_registry/
- npm Registry: https://docs.gitlab.com/ee/user/packages/npm_registry/
- Maven Repository: https://docs.gitlab.com/ee/user/packages/maven_repository/
- PyPI Repository: https://docs.gitlab.com/ee/user/packages/pypi_repository/
