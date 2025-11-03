# GitLab Pages Reference

## Overview

GitLab Pages allows you to host static websites directly from your GitLab repository. Perfect for documentation, portfolios, blogs, and landing pages.

## Pages URL Format

**GitLab.com**:
```
https://username.gitlab.io/project-name
```

**Custom domain**:
```
https://www.example.com
```

**Group pages**:
```
https://groupname.gitlab.io
```

## Creating a Pages Site

### Basic Setup

**.gitlab-ci.yml**:
```yaml
pages:
  stage: deploy
  script:
    - mkdir .public
    - cp -r * .public
    - mv .public public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

**Requirements**:
1. Job must be named `pages`
2. Must create `public/` directory
3. Must have artifact with `public/` path
4. Runs on default branch (usually main)

### Static HTML

**index.html**:
```html
<!DOCTYPE html>
<html>
<head>
    <title>My GitLab Page</title>
</head>
<body>
    <h1>Welcome!</h1>
    <p>This is hosted on GitLab Pages</p>
</body>
</html>
```

**.gitlab-ci.yml**:
```yaml
pages:
  stage: deploy
  script:
    - mkdir public
    - cp index.html public/
  artifacts:
    paths:
      - public
  only:
    - main
```

## Static Site Generators

### Hugo

```yaml
image: registry.gitlab.com/pages/hugo:latest

variables:
  GIT_SUBMODULE_STRATEGY: recursive

pages:
  script:
    - hugo
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### Jekyll

```yaml
image: ruby:2.7

before_script:
  - bundle install

pages:
  script:
    - bundle exec jekyll build -d public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### Gatsby

```yaml
image: node:18

cache:
  paths:
    - node_modules/
    - .cache/

pages:
  script:
    - npm install
    - npm run build
    - mv public public-gatsby
    - mv public-gatsby public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### Next.js

```yaml
image: node:18

cache:
  paths:
    - node_modules/
    - .next/cache/

pages:
  script:
    - npm install
    - npm run build
    - npm run export
    - mv out public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### VuePress

```yaml
image: node:18

pages:
  script:
    - npm install
    - npm run docs:build
    - mv docs/.vuepress/dist public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### MkDocs

```yaml
image: python:3.11

pages:
  script:
    - pip install mkdocs mkdocs-material
    - mkdocs build
    - mv site public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### Docusaurus

```yaml
image: node:18

pages:
  script:
    - npm install
    - npm run build
    - mv build public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### Hexo

```yaml
image: node:18

pages:
  script:
    - npm install hexo-cli -g
    - npm install
    - hexo generate
    - mv public public-hexo
    - mv public-hexo public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

## Custom Domain Setup

### Add Custom Domain

**Via UI**:
1. Settings > Pages
2. New Domain
3. Enter domain name
4. Save

**Via API**:
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/pages/domains" \
  --data "domain=www.example.com"
```

### DNS Configuration

**For root domain (example.com)**:
```
Type: A
Name: @
Value: 35.185.44.232
```

**For subdomain (www.example.com)**:
```
Type: CNAME
Name: www
Value: username.gitlab.io
```

**Verify DNS**:
```bash
dig www.example.com
nslookup www.example.com
```

### SSL/TLS Certificate

**Automatic Let's Encrypt**:
1. Add custom domain
2. Configure DNS
3. Enable "Automatic certificate management using Let's Encrypt"

**Manual certificate**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/pages/domains/www.example.com" \
  --form "certificate=@/path/to/cert.pem" \
  --form "key=@/path/to/key.pem"
```

## Access Control

### Private Pages

Require authentication to access pages (Premium/Ultimate):

**Enable private pages**:
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id" \
  --data "pages_access_level=private"
```

**Access levels**:
- `public` - Anyone can access
- `private` - Only project members
- `enabled` - Controlled by project visibility
- `disabled` - Pages disabled

### Authentication

When pages are private:
- Users must log in to GitLab
- Project members can access
- Access controlled by member permissions

## Preview Environments

Create preview for merge requests:

```yaml
pages:
  stage: deploy
  script:
    - npm run build
    - mv dist public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

pages:preview:
  stage: deploy
  script:
    - npm run build
    - mv dist public
  artifacts:
    paths:
      - public
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_PROJECT_ROOT_NAMESPACE.gitlab.io/-/$CI_PROJECT_NAME/-/jobs/$CI_JOB_ID/artifacts/public/index.html
  rules:
    - if: $CI_MERGE_REQUEST_ID
```

## Advanced Configuration

### Custom 404 Page

Create `public/404.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Page Not Found</title>
</head>
<body>
    <h1>404 - Page Not Found</h1>
    <p>The page you're looking for doesn't exist.</p>
    <a href="/">Go Home</a>
</body>
</html>
```

### Redirects

Create `public/_redirects`:
```
# Redirect /old-page to /new-page
/old-page /new-page 301

# Redirect with wildcards
/blog/* /news/:splat 301

# Conditional redirects
/api/* https://api.example.com/:splat 200

# Country-based redirects
/  /us  302  Country=us
/  /uk  302  Country=uk
```

### Headers

Create `public/_headers`:
```
# Security headers
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  X-XSS-Protection: 1; mode=block
  Referrer-Policy: no-referrer-when-downgrade

# Cache control
/static/*
  Cache-Control: public, max-age=31536000, immutable

# CORS
/api/*
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Methods: GET, POST, OPTIONS
```

### Base URL Configuration

For subdirectory deployment:

**Hugo**:
```toml
# config.toml
baseURL = "https://username.gitlab.io/project-name/"
```

**Jekyll**:
```yaml
# _config.yml
baseurl: "/project-name"
url: "https://username.gitlab.io"
```

**VuePress**:
```js
// .vuepress/config.js
module.exports = {
  base: '/project-name/',
}
```

## Optimization

### Image Optimization

```yaml
pages:
  before_script:
    - npm install -g sharp-cli
  script:
    - |
      find public -name '*.jpg' -o -name '*.png' | while read img; do
        sharp -i "$img" -o "$img" resize 1920
      done
```

### Minification

```yaml
pages:
  script:
    - npm run build
    - npm install -g html-minifier clean-css-cli uglify-js
    - |
      find public -name '*.html' -exec html-minifier --collapse-whitespace --remove-comments --minify-css --minify-js {} -o {} \;
      find public -name '*.css' -exec cleancss -o {} {} \;
      find public -name '*.js' -exec uglifyjs {} -c -m -o {} \;
```

### Compression

```yaml
pages:
  script:
    - npm run build
    - mv dist public
    - find public -type f -regex '.*\.\(htm\|html\|txt\|text\|js\|css\|svg\|json\)$' -exec gzip -f -k {} \;
    - find public -type f -regex '.*\.\(htm\|html\|txt\|text\|js\|css\|svg\|json\)$' -exec brotli -f -k {} \;
```

## CI/CD Examples

### Multi-Branch Deployment

```yaml
pages:
  stage: deploy
  script:
    - npm run build
    - mv dist public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

pages:staging:
  stage: deploy
  script:
    - npm run build
    - mv dist public
  artifacts:
    paths:
      - public
  environment:
    name: staging
    url: https://staging.example.com
  rules:
    - if: $CI_COMMIT_BRANCH == "staging"
```

### Conditional Deployment

```yaml
pages:
  stage: deploy
  script:
    - |
      if [ "$CI_COMMIT_BRANCH" == "main" ]; then
        export NODE_ENV=production
      else
        export NODE_ENV=development
      fi
    - npm run build
    - mv dist public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH
```

### Scheduled Rebuild

```yaml
pages:
  script:
    - npm run build
    - mv dist public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_PIPELINE_SOURCE == "schedule"
```

## Monitoring

### Analytics

Add Google Analytics:
```html
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>
```

### Status Checks

```yaml
test:pages:
  stage: test
  script:
    - npm run build
    - npm run test
    - |
      # Check for broken links
      npm install -g broken-link-checker
      blc http://localhost:8000 -ro
```

## Troubleshooting

### Pages Not Updating

1. Check pipeline succeeded
2. Verify `pages` job ran
3. Check artifacts were created
4. Clear browser cache
5. Wait for CDN propagation (up to 30 minutes)

### 404 Errors

1. Verify file exists in `public/` directory
2. Check file paths (case-sensitive)
3. Verify artifact includes files
4. Check baseURL configuration

### Custom Domain Issues

1. Verify DNS configuration: `dig www.example.com`
2. Check SSL certificate status
3. Ensure domain verified in GitLab
4. Check domain registrar settings
5. Wait for DNS propagation (up to 48 hours)

### Build Failures

```yaml
pages:
  script:
    - set -e  # Exit on error
    - npm install
    - npm run build || (echo "Build failed" && exit 1)
    - mv dist public
  artifacts:
    paths:
      - public
    when: on_success
```

## Best Practices

### 1. Performance

- Optimize images
- Minify CSS/JS
- Enable compression
- Use CDN
- Implement caching

### 2. Security

- Use HTTPS (automatic with Let's Encrypt)
- Set security headers
- Validate user input (if using forms)
- Keep dependencies updated

### 3. SEO

```html
<meta name="description" content="Site description">
<meta name="keywords" content="keywords, here">
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Description">
<meta property="og:image" content="https://example.com/image.jpg">
<meta name="twitter:card" content="summary_large_image">

<!-- Robots -->
<meta name="robots" content="index, follow">

<!-- Canonical URL -->
<link rel="canonical" href="https://example.com/page">
```

### 4. Accessibility

- Use semantic HTML
- Add alt text to images
- Ensure keyboard navigation
- Maintain color contrast
- Test with screen readers

### 5. Version Control

```yaml
pages:
  script:
    - echo "Build $CI_COMMIT_SHA on $(date)" > public/version.txt
    - npm run build
```

## API Reference

### Get Pages

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/pages"
```

### Delete Pages

```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/pages"
```

### List Domains

```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/pages/domains"
```

### Add Domain

```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/pages/domains" \
  --data "domain=www.example.com" \
  --data "auto_ssl_enabled=true"
```

### Update Domain

```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/pages/domains/www.example.com" \
  --data "auto_ssl_enabled=true"
```

### Delete Domain

```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/:id/pages/domains/www.example.com"
```

## Examples Repository

GitLab provides example pages projects:
- https://gitlab.com/pages
- Hugo: https://gitlab.com/pages/hugo
- Jekyll: https://gitlab.com/pages/jekyll
- Gatsby: https://gitlab.com/pages/gatsby
- Next.js: https://gitlab.com/pages/nextjs

## Additional Resources

- Pages Documentation: https://docs.gitlab.com/ee/user/project/pages/
- Pages Examples: https://gitlab.com/pages
- Custom Domains: https://docs.gitlab.com/ee/user/project/pages/custom_domains_ssl_tls_certification/
- Let's Encrypt: https://letsencrypt.org/
