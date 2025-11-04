# GitLab GraphQL API Reference

## Overview

GitLab's GraphQL API provides a more efficient alternative to REST API, allowing you to request exactly the data you need in a single query. The GraphQL API is the primary development focus for new features at GitLab.

## GraphQL Endpoint

```
https://gitlab.com/api/graphql
```

For self-hosted instances:
```
https://your-gitlab-instance.com/api/graphql
```

## Authentication

### Personal Access Token

```bash
curl --header "Authorization: Bearer <your_token>" \
  --header "Content-Type: application/json" \
  --request POST \
  --data '{"query": "{ currentUser { username } }"}' \
  https://gitlab.com/api/graphql
```

### In HTTP Headers

```
Authorization: Bearer <your_access_token>
```

## Interactive GraphQL Explorer (GraphiQL)

Access the interactive explorer:
```
https://gitlab.com/-/graphql-explorer
```

Features:
- Auto-completion
- Schema documentation
- Query validation
- Query history

## Basic Query Structure

```graphql
{
  currentUser {
    id
    username
    name
    email
  }
}
```

## Common Queries

### Get Current User

```graphql
{
  currentUser {
    id
    username
    name
    email
    avatarUrl
    webUrl
    status {
      message
      availability
    }
  }
}
```

### Get Project Details

```graphql
{
  project(fullPath: "group/project") {
    id
    name
    description
    visibility
    createdAt
    lastActivityAt
    webUrl
    repository {
      rootRef
      empty
    }
    statistics {
      commitCount
      storageSize
      repositorySize
      lfsObjectsSize
    }
  }
}
```

### List Projects

```graphql
{
  projects(first: 10, membership: true) {
    nodes {
      id
      name
      fullPath
      description
      visibility
      createdAt
      webUrl
      starCount
      forksCount
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### Get Merge Requests

```graphql
{
  project(fullPath: "group/project") {
    mergeRequests(first: 10, state: opened) {
      nodes {
        id
        iid
        title
        description
        state
        createdAt
        updatedAt
        author {
          username
          name
        }
        sourceBranch
        targetBranch
        webUrl
        draft
        workInProgress
        mergeStatus
        approved
        conflicts
        diffStats {
          additions
          deletions
        }
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
}
```

### Get Issues

```graphql
{
  project(fullPath: "group/project") {
    issues(first: 10, state: opened) {
      nodes {
        id
        iid
        title
        description
        state
        createdAt
        closedAt
        author {
          username
          name
        }
        assignees {
          nodes {
            username
            name
          }
        }
        labels {
          nodes {
            title
            color
          }
        }
        milestone {
          title
          dueDate
        }
        webUrl
        upvotes
        downvotes
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
}
```

### Get Pipeline Details

```graphql
{
  project(fullPath: "group/project") {
    pipeline(iid: "123") {
      id
      iid
      status
      ref
      createdAt
      updatedAt
      startedAt
      finishedAt
      duration
      coverage
      user {
        username
        name
      }
      jobs {
        nodes {
          id
          name
          stage
          status
          createdAt
          startedAt
          finishedAt
          duration
          webPath
        }
      }
    }
  }
}
```

### Get Commits

```graphql
{
  project(fullPath: "group/project") {
    repository {
      tree(ref: "main") {
        lastCommit {
          id
          sha
          title
          message
          authoredDate
          author {
            name
            email
          }
        }
      }
    }
  }
}
```

### Get Group Details

```graphql
{
  group(fullPath: "group-name") {
    id
    name
    fullPath
    description
    visibility
    webUrl
    projects(first: 10) {
      nodes {
        name
        fullPath
      }
    }
    members {
      nodes {
        user {
          username
          name
        }
        accessLevel {
          integerValue
          stringValue
        }
      }
    }
  }
}
```

## Mutations

### Create Issue

```graphql
mutation {
  createIssue(input: {
    projectPath: "group/project"
    title: "New issue title"
    description: "Issue description"
    assigneeIds: ["gid://gitlab/User/1"]
    labelIds: ["gid://gitlab/Label/1"]
  }) {
    issue {
      id
      iid
      title
      webUrl
    }
    errors
  }
}
```

### Update Issue

```graphql
mutation {
  updateIssue(input: {
    projectPath: "group/project"
    iid: "123"
    title: "Updated title"
    description: "Updated description"
    stateEvent: CLOSE
  }) {
    issue {
      id
      title
      state
    }
    errors
  }
}
```

### Create Merge Request

```graphql
mutation {
  mergeRequestCreate(input: {
    projectPath: "group/project"
    title: "New feature"
    description: "This MR adds..."
    sourceBranch: "feature-branch"
    targetBranch: "main"
  }) {
    mergeRequest {
      id
      iid
      title
      webUrl
    }
    errors
  }
}
```

### Update Merge Request

```graphql
mutation {
  mergeRequestUpdate(input: {
    projectPath: "group/project"
    iid: "123"
    title: "Updated title"
    description: "Updated description"
    assigneeIds: ["gid://gitlab/User/1"]
  }) {
    mergeRequest {
      id
      title
      state
    }
    errors
  }
}
```

### Merge Merge Request

```graphql
mutation {
  mergeRequestMerge(input: {
    projectPath: "group/project"
    iid: "123"
    squash: true
  }) {
    mergeRequest {
      id
      state
      mergedAt
    }
    errors
  }
}
```

### Create Note (Comment)

```graphql
mutation {
  createNote(input: {
    noteableId: "gid://gitlab/Issue/123"
    body: "This is a comment"
  }) {
    note {
      id
      body
      author {
        username
      }
    }
    errors
  }
}
```

### Update Project

```graphql
mutation {
  updateProject(input: {
    projectPath: "group/project"
    description: "New description"
    visibility: "public"
  }) {
    project {
      id
      description
      visibility
    }
    errors
  }
}
```

### Create Branch

```graphql
mutation {
  createBranch(input: {
    projectPath: "group/project"
    name: "new-branch"
    ref: "main"
  }) {
    branch {
      name
      commit {
        id
      }
    }
    errors
  }
}
```

### Create Snippet

```graphql
mutation {
  createSnippet(input: {
    projectPath: "group/project"
    title: "Code snippet"
    fileName: "example.rb"
    content: "puts 'Hello World'"
    visibility: "private"
  }) {
    snippet {
      id
      title
      webUrl
    }
    errors
  }
}
```

### Run Pipeline

```graphql
mutation {
  pipelineCreate(input: {
    projectPath: "group/project"
    ref: "main"
  }) {
    pipeline {
      id
      status
    }
    errors
  }
}
```

### Cancel Pipeline

```graphql
mutation {
  pipelineCancel(input: {
    id: "gid://gitlab/Ci::Pipeline/123"
  }) {
    pipeline {
      id
      status
    }
    errors
  }
}
```

## Variables and Fragments

### Using Variables

```graphql
query GetProject($fullPath: ID!) {
  project(fullPath: $fullPath) {
    id
    name
    description
  }
}
```

Variables:
```json
{
  "fullPath": "group/project"
}
```

### Using Fragments

```graphql
fragment UserInfo on User {
  id
  username
  name
  email
  avatarUrl
}

query {
  currentUser {
    ...UserInfo
  }
  project(fullPath: "group/project") {
    author {
      ...UserInfo
    }
  }
}
```

## Pagination

### Cursor-Based Pagination

```graphql
query {
  projects(first: 10, after: "cursor_value") {
    nodes {
      id
      name
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
      startCursor
      endCursor
    }
  }
}
```

### Complete Pagination Example

```python
import requests
import json

def fetch_all_projects(token):
    url = "https://gitlab.com/api/graphql"
    headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json"
    }

    has_next_page = True
    after_cursor = None
    all_projects = []

    while has_next_page:
        query = """
        query($after: String) {
          projects(first: 10, after: $after, membership: true) {
            nodes {
              id
              name
              fullPath
            }
            pageInfo {
              hasNextPage
              endCursor
            }
          }
        }
        """

        variables = {"after": after_cursor}

        response = requests.post(
            url,
            headers=headers,
            json={"query": query, "variables": variables}
        )

        data = response.json()["data"]
        projects = data["projects"]

        all_projects.extend(projects["nodes"])
        has_next_page = projects["pageInfo"]["hasNextPage"]
        after_cursor = projects["pageInfo"]["endCursor"]

    return all_projects
```

## Introspection

### Query Schema

```graphql
{
  __schema {
    types {
      name
      description
    }
  }
}
```

### Query Type Details

```graphql
{
  __type(name: "Project") {
    name
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

## Advanced Queries

### Nested Relationships

```graphql
{
  group(fullPath: "group-name") {
    projects(first: 5) {
      nodes {
        name
        mergeRequests(first: 3, state: opened) {
          nodes {
            title
            author {
              username
            }
            assignees {
              nodes {
                username
              }
            }
            approvedBy {
              nodes {
                username
              }
            }
          }
        }
      }
    }
  }
}
```

### Conditional Fields

```graphql
query GetProject($includeMergeRequests: Boolean!) {
  project(fullPath: "group/project") {
    id
    name
    mergeRequests(first: 10) @include(if: $includeMergeRequests) {
      nodes {
        title
      }
    }
  }
}
```

### Aliases

```graphql
{
  openIssues: project(fullPath: "group/project") {
    issues(state: opened) {
      count
    }
  }
  closedIssues: project(fullPath: "group/project") {
    issues(state: closed) {
      count
    }
  }
}
```

## Client Libraries

### Python (gql)

```python
from gql import gql, Client
from gql.transport.requests import RequestsHTTPTransport

transport = RequestsHTTPTransport(
    url="https://gitlab.com/api/graphql",
    headers={"Authorization": f"Bearer {token}"},
)

client = Client(transport=transport, fetch_schema_from_transport=True)

query = gql("""
    query {
        currentUser {
            username
            name
        }
    }
""")

result = client.execute(query)
print(result)
```

### JavaScript (Apollo Client)

```javascript
import { ApolloClient, InMemoryCache, gql, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';

const httpLink = createHttpLink({
  uri: 'https://gitlab.com/api/graphql',
});

const authLink = setContext((_, { headers }) => {
  return {
    headers: {
      ...headers,
      authorization: `Bearer ${token}`,
    }
  }
});

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache()
});

const GET_CURRENT_USER = gql`
  query {
    currentUser {
      username
      name
    }
  }
`;

client.query({ query: GET_CURRENT_USER })
  .then(result => console.log(result));
```

### Ruby (graphql-client)

```ruby
require "graphql/client"
require "graphql/client/http"

module GitLabAPI
  HTTP = GraphQL::Client::HTTP.new("https://gitlab.com/api/graphql") do
    def headers(context)
      { "Authorization" => "Bearer #{ENV['GITLAB_TOKEN']}" }
    end
  end

  Schema = GraphQL::Client.load_schema(HTTP)
  Client = GraphQL::Client.new(schema: Schema, execute: HTTP)
end

CurrentUserQuery = GitLabAPI::Client.parse <<-'GRAPHQL'
  query {
    currentUser {
      username
      name
    }
  }
GRAPHQL

result = GitLabAPI::Client.query(CurrentUserQuery)
puts result.data.current_user.username
```

## Rate Limiting

GraphQL queries count towards API rate limits based on query complexity.

**Check rate limit**:
```graphql
{
  currentUser {
    username
  }
}
```

Response headers:
- `RateLimit-Limit`: Total allowed
- `RateLimit-Remaining`: Remaining requests
- `RateLimit-Reset`: Reset timestamp

## Error Handling

### Error Response Format

```json
{
  "data": null,
  "errors": [
    {
      "message": "Error message",
      "path": ["field", "path"],
      "extensions": {
        "code": "ERROR_CODE"
      }
    }
  ]
}
```

### Common Error Types

- `NOT_FOUND`: Resource not found
- `UNAUTHORIZED`: Authentication required
- `FORBIDDEN`: Insufficient permissions
- `VALIDATION_ERROR`: Invalid input
- `INTERNAL_SERVER_ERROR`: Server error

### Error Handling Example

```python
def execute_query(query, variables=None):
    response = requests.post(
        "https://gitlab.com/api/graphql",
        headers={"Authorization": f"Bearer {token}"},
        json={"query": query, "variables": variables}
    )

    result = response.json()

    if "errors" in result:
        for error in result["errors"]:
            print(f"Error: {error['message']}")
            if "path" in error:
                print(f"Path: {error['path']}")
        return None

    return result["data"]
```

## Best Practices

### 1. Request Only Needed Fields

**Bad**:
```graphql
{
  project(fullPath: "group/project") {
    id
    name
    description
    # ... all fields
  }
}
```

**Good**:
```graphql
{
  project(fullPath: "group/project") {
    id
    name
  }
}
```

### 2. Use Fragments for Reusability

```graphql
fragment IssueFields on Issue {
  id
  iid
  title
  state
  author {
    username
  }
}

query {
  project(fullPath: "group/project") {
    openIssues: issues(state: opened) {
      nodes {
        ...IssueFields
      }
    }
    closedIssues: issues(state: closed) {
      nodes {
        ...IssueFields
      }
    }
  }
}
```

### 3. Implement Pagination

Always use pagination for large datasets:

```graphql
{
  projects(first: 100, after: $cursor) {
    nodes { ... }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### 4. Handle Errors Gracefully

```javascript
async function fetchData() {
  try {
    const result = await client.query({ query: MY_QUERY });
    return result.data;
  } catch (error) {
    if (error.graphQLErrors) {
      error.graphQLErrors.forEach(({ message, path }) => {
        console.error(`GraphQL error: ${message} at ${path}`);
      });
    }
    if (error.networkError) {
      console.error(`Network error: ${error.networkError}`);
    }
  }
}
```

### 5. Use Variables

```graphql
query GetProject($path: ID!, $mrCount: Int!) {
  project(fullPath: $path) {
    mergeRequests(first: $mrCount) {
      nodes { title }
    }
  }
}
```

### 6. Optimize Query Depth

Limit nested query depth to avoid performance issues:

```graphql
# Avoid deep nesting
{
  group {
    projects {
      mergeRequests {
        commits {
          notes {
            # Too deep!
          }
        }
      }
    }
  }
}
```

## Useful Queries for Common Tasks

### Search Across Projects

```graphql
{
  projects(search: "keyword", first: 10) {
    nodes {
      name
      fullPath
      description
    }
  }
}
```

### Get User Activity

```graphql
{
  user(username: "john.doe") {
    assignedMergeRequests(first: 10) {
      nodes {
        title
        project {
          name
        }
      }
    }
    authoredMergeRequests(first: 10) {
      nodes {
        title
        state
      }
    }
  }
}
```

### Get Deployment Status

```graphql
{
  project(fullPath: "group/project") {
    environments {
      nodes {
        name
        state
        lastDeployment {
          status
          createdAt
          deployable {
            ... on Job {
              name
            }
          }
        }
      }
    }
  }
}
```

### Get Security Vulnerabilities

```graphql
{
  project(fullPath: "group/project") {
    vulnerabilities {
      nodes {
        id
        title
        severity
        state
        reportType
        detectedAt
      }
    }
  }
}
```

## Migration from REST to GraphQL

### REST to GraphQL Comparison

**REST**: Get project and its merge requests
```bash
# 2 requests
curl "https://gitlab.com/api/v4/projects/123"
curl "https://gitlab.com/api/v4/projects/123/merge_requests"
```

**GraphQL**: Single request
```graphql
{
  project(fullPath: "group/project") {
    id
    name
    mergeRequests(first: 10) {
      nodes {
        title
      }
    }
  }
}
```

## Subscriptions (Real-time Updates)

GitLab supports GraphQL subscriptions for real-time updates:

```graphql
subscription {
  issuableUpdated {
    issue {
      id
      title
      state
    }
  }
}
```

## Additional Resources

- GraphQL API Documentation: https://docs.gitlab.com/ee/api/graphql/
- GraphQL Explorer: https://gitlab.com/-/graphql-explorer
- GraphQL Reference: https://docs.gitlab.com/ee/api/graphql/reference/
- Best Practices: https://docs.gitlab.com/ee/api/graphql/#best-practices
