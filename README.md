# Obot MCP Catalog

This repository contains the official catalog of MCP (Model Context Protocol) servers for the [Obot platform](https://obot.ai). Each server definition enables AI assistants to interact with external services, APIs, and tools.

## Overview

The catalog currently includes **77 MCP servers** across various categories:

- **Cloud & Infrastructure**: AWS, Azure, GCP, DigitalOcean, Cloudflare
- **Databases**: PostgreSQL, MongoDB, MySQL, Redis, Elasticsearch, Snowflake
- **Developer Tools**: GitHub, GitLab, Playwright, Terraform
- **Communication**: Slack, Gmail, Outlook, Microsoft Teams
- **Business & Productivity**: Salesforce, HubSpot, Notion, Asana, Linear
- **Search & Retrieval**: Brave Search, Exa, Tavily, DuckDuckGo

## Server Types

Obot supports two runtime types for MCP servers:

### Containerized Servers (Single-User)

Each user gets their own isolated server instance running in a container. Users provide their own credentials. This is the most common type for databases and services requiring user-specific authentication.

### Remote Servers

Hosted MCP servers accessible via HTTP. These connect to external services that handle their own authentication or use API keys passed via headers.

## Adding a New MCP Server

### Step 1: Create the YAML Definition

Create a new YAML file in the repository root with the service name (e.g., `myservice.yaml`).

### Step 2: Define Basic Metadata

```yaml
name: My Service
shortDescription: Brief one-line description of what this server does
description: |
  Detailed markdown description including:

  ## Features
  - Feature 1
  - Feature 2

  ## What you'll need to connect

  **Required:**
  - **API Key**: Description of how to obtain it

  **Optional:**
  - **Region**: Service region (default: us-east-1)

metadata:
  categories: Category1, Category2
icon: https://example.com/icon.png
repoURL: https://github.com/org/repo
```

#### Available Categories

- `Automation & Browsers`
- `Business`
- `Cloud & Infrastructure`
- `Communication`
- `Databases`
- `Developer Tools`
- `Documentation`
- `Retrieval & Search`
- `Finance`
- `Productivity`

### Step 3: Configure the Runtime

#### Option A: Containerized Server

For servers that run in containers with user-provided credentials:

```yaml
runtime: containerized
containerizedConfig:
  image: ghcr.io/obot-platform/mcp-images/myservice:1.0.0
  port: 8099
  path: /
  args:
    - myservice-executable
    - --optional-flag
```

**Container Requirements:**
- Must expose an HTTP endpoint for MCP communication
- Standard port is `8099`
- Path is typically `/` or `/mcp`
- Images hosted at `ghcr.io/obot-platform/mcp-images/`

#### Option B: Remote Server

For hosted MCP services:

```yaml
runtime: remote
remoteConfig:
  hostname: api.myservice.com
  # OR use fixedURL for complete URL:
  # fixedURL: https://api.myservice.com/mcp
  headers:
    - name: API Key
      description: Your API key from myservice.com
      key: X-Api-Key
      required: true
      sensitive: true
```

### Step 4: Define Environment Variables (Containerized Only)

```yaml
env:
  - key: API_KEY
    name: API Key
    required: true
    sensitive: true
    description: Your API key for authentication
  - key: REGION
    name: Region
    required: false
    sensitive: false
    description: Service region (default: us-east-1)
```

**Field Reference:**
| Field | Description |
|-------|-------------|
| `key` | Environment variable name passed to the container |
| `name` | Display name shown in the UI |
| `required` | Whether the field is mandatory |
| `sensitive` | If `true`, value is masked in the UI |
| `description` | Help text for users |

### Step 5: Define Tool Preview

Document the tools your MCP server provides:

```yaml
toolPreview:
  - name: list_items
    description: List all items in the account
    params: {}
  - name: get_item
    description: Get details about a specific item
    params:
      item_id: The unique identifier of the item
  - name: create_item
    description: Create a new item
    params:
      name: Name of the item to create
      type:
        type: string
        description: Type of item (optional)
```

## Complete Examples

### Containerized Server Example

```yaml
name: PostgreSQL
shortDescription: Analyze performance, optimize queries, and tune PostgreSQL databases
description: |
  A Model Context Protocol server for PostgreSQL database management.

  ## Features
  - Database health analysis
  - Query performance optimization
  - Schema intelligence

  ## What you'll need to connect

  **Required:**
  - **Database URI**: PostgreSQL connection string

metadata:
  categories: Databases
icon: https://avatars.githubusercontent.com/u/177543?s=48&v=4
repoURL: https://github.com/crystaldba/postgres-mcp

runtime: containerized
containerizedConfig:
  image: ghcr.io/obot-platform/mcp-images/postgresql:0.3.0
  port: 8099
  path: /
  args:
    - postgres-mcp
    - --access-mode=unrestricted

env:
  - key: DATABASE_URI
    name: Database URI
    required: true
    sensitive: false
    description: Connection string (e.g., postgresql://user:pass@host:5432/db)

toolPreview:
  - name: list_schemas
    description: Lists all database schemas
  - name: execute_sql
    description: Executes SQL statements
    params:
      query: SQL query to execute
  - name: analyze_db_health
    description: Performs comprehensive health checks
```

### Remote Server Example

```yaml
name: GitHub
shortDescription: Manage GitHub repositories, issues, PRs, and workflows
description: |
  A hosted MCP server for GitHub integration.

  ## Features
  - Repository management
  - Issue and PR automation
  - CI/CD workflow monitoring

  ## What you'll need to connect

  **Required:**
  - **Personal Access Token**: GitHub PAT with repository permissions

metadata:
  categories: Developer Tools
icon: https://avatars.githubusercontent.com/u/9919?v=4
repoURL: https://github.com/github/github-mcp-server

runtime: remote
remoteConfig:
  hostname: api.githubcopilot.com
  headers:
    - name: Personal Access Token
      description: GitHub PAT
      key: Authorization
      required: true
      sensitive: true

toolPreview:
  - name: create_issue
    description: Create a new issue in a repository
    params:
      owner: Repository owner
      repo: Repository name
      title: Issue title
      body: Issue body content (optional)
  - name: list_pull_requests
    description: List pull requests for a repository
    params:
      owner: Repository owner
      repo: Repository name
```

### Remote Server Without Authentication

```yaml
name: GitMCP
shortDescription: Access any GitHub project's documentation in real-time
description: |
  A cloud-based MCP server for GitHub documentation access.

  ## Features
  - Latest documentation access
  - Zero setup required

  ## What you'll need to connect

  **No Setup Required**: Runs in the cloud with no authentication needed

metadata:
  categories: Developer Tools
icon: https://avatars.githubusercontent.com/u/182288589?v=4
repoURL: https://gitmcp.io

runtime: remote
remoteConfig:
  hostname: gitmcp.io

toolPreview:
  - name: fetch_documentation
    description: Fetch documentation from a GitHub repository
  - name: search_code
    description: Search for code within a repository
    params:
      query: Search query
```

## Optional Configuration Fields

### Allow Multiple Instances

Enable users to add multiple instances of the same server:

```yaml
metadata:
  categories: Communication
  allow-multiple: "true"
```

### Unsupported Tools

Mark tools that shouldn't be exposed in certain configurations:

```yaml
metadata:
  categories: Developer Tools
  unsupportedTools: create_file,delete_file
```

## Validation and CI/CD

When you submit a pull request:

1. **Vulnerability Scanning**: All tools are scanned for potential LLM vulnerabilities
2. **Automated Testing**: Containerized servers are tested against a running Obot instance
3. **Format Validation**: YAML structure is validated

## Resources

- [Obot Documentation](https://docs.obot.ai)
- [MCP Specification](https://modelcontextprotocol.io)
- [Obot Platform](https://obot.ai)
- [GitHub Repository](https://github.com/obot-platform/mcp-catalog)
