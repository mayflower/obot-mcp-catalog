# How to Add MCP Servers to the Obot Catalog

This guide provides step-by-step instructions for adding new MCP (Model Context Protocol) servers to the Obot catalog. The catalog supports two runtime types: **containerized** servers and **remote** servers.

## Table of Contents

- [Server Types Overview](#server-types-overview)
- [YAML File Structure](#yaml-file-structure)
- [Adding a Containerized Server](#adding-a-containerized-server)
- [Adding a Remote Server](#adding-a-remote-server)
- [Environment Variables](#environment-variables)
- [Tool Preview Documentation](#tool-preview-documentation)
- [Metadata Options](#metadata-options)
- [Validation and CI/CD Pipeline](#validation-and-cicd-pipeline)
- [Complete Examples](#complete-examples)

---

## Server Types Overview

### Containerized Servers

Containerized servers run in isolated Docker containers with user-provided credentials. Each user gets their own server instance. This is the most common type for:

- Database integrations (PostgreSQL, MongoDB, MySQL)
- Services requiring user-specific API keys
- Tools needing isolated execution environments

**Requirements:**
- Container image hosted at `ghcr.io/obot-platform/mcp-images/`
- HTTP endpoint exposed (typically port `8099`)
- MCP protocol implementation

### Remote Servers

Remote servers are hosted MCP services accessible via HTTP. These connect to external services that:

- Handle their own authentication
- Use API keys passed via headers
- Require no local container runtime

**Use cases:**
- GitHub's official MCP server
- Cloud-hosted documentation services
- Third-party MCP endpoints

---

## YAML File Structure

Create a new YAML file in the repository root with the service name (e.g., `myservice.yaml`).

### Required Fields

| Field | Description |
|-------|-------------|
| `name` | Display name of the MCP server |
| `shortDescription` | One-line description (shown in catalog listings) |
| `description` | Detailed markdown description with features and setup instructions |
| `metadata.categories` | Comma-separated list of categories |
| `icon` | URL to the service icon (typically GitHub avatar) |
| `repoURL` | URL to the source repository |
| `runtime` | Either `containerized` or `remote` |
| `toolPreview` | List of tools provided by the server |

### Runtime-Specific Fields

**For containerized servers:**
- `containerizedConfig` - Container configuration
- `env` - Environment variables for credentials

**For remote servers:**
- `remoteConfig` - Remote endpoint configuration

---

## Adding a Containerized Server

### Step 1: Define Basic Metadata

```yaml
name: My Service
shortDescription: Brief one-line description of what this server does
description: |
  A Model Context Protocol server for My Service integration.

  ## Features
  - Feature 1: Description
  - Feature 2: Description
  - Feature 3: Description

  ## What you'll need to connect

  **Required:**
  - **API Key**: Your My Service API key from [myservice.com/api-keys](https://myservice.com/api-keys)

  **Optional:**
  - **Region**: Service region (default: us-east-1)

metadata:
  categories: Developer Tools
icon: https://avatars.githubusercontent.com/u/12345?v=4
repoURL: https://github.com/org/myservice-mcp
```

### Step 2: Configure Container Runtime

```yaml
runtime: containerized
containerizedConfig:
  image: ghcr.io/obot-platform/mcp-images/myservice:1.0.0
  port: 8099
  path: /
  args:
    - myservice-mcp
    - --optional-flag
```

**Container Configuration Fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `image` | Yes | Full container image path with tag |
| `port` | Yes | Port the MCP server listens on (typically `8099`) |
| `path` | Yes | HTTP path for MCP endpoint (typically `/` or `/mcp`) |
| `args` | No | Command-line arguments passed to the container |

### Step 3: Define Environment Variables

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

  - key: DATABASE_URI
    name: Database URI
    required: true
    sensitive: false
    description: Connection string (e.g., postgresql://user:pass@host:5432/db)
```

### Step 4: Add Tool Preview

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
      type: Type of item (optional)
```

---

## Adding a Remote Server

### Remote Server with Authentication

For hosted MCP services requiring API keys via headers:

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
```

### Remote Server Without Authentication

For public MCP services requiring no credentials:

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
```

### Remote Config Options

| Field | Required | Description |
|-------|----------|-------------|
| `hostname` | Yes* | Base hostname for the MCP endpoint |
| `fixedURL` | Yes* | Complete URL if not using hostname |
| `headers` | No | List of required HTTP headers for authentication |

*Either `hostname` or `fixedURL` is required.

**Header Configuration:**

```yaml
headers:
  - name: Display Name        # Shown in UI
    description: Help text    # Guidance for users
    key: X-Api-Key           # Actual HTTP header name
    required: true           # Whether required
    sensitive: true          # Mask value in UI
```

---

## Environment Variables

Environment variables are used with **containerized servers** to pass credentials and configuration.

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `key` | string | Yes | Environment variable name passed to container |
| `name` | string | Yes | Display name shown in the UI |
| `description` | string | Yes | Help text explaining what this variable is for |
| `required` | boolean | Yes | Whether the variable is mandatory |
| `sensitive` | boolean | Yes | If `true`, value is masked in the UI |

### Example Patterns

**API Key (required, sensitive):**
```yaml
- key: API_KEY
  name: API Key
  required: true
  sensitive: true
  description: Your API key from myservice.com
```

**Database Connection (required, not sensitive):**
```yaml
- key: DATABASE_URI
  name: Database URI
  required: true
  sensitive: false
  description: Connection string (e.g., postgresql://user:pass@host:5432/db)
```

**Optional Configuration:**
```yaml
- key: REGION
  name: Region
  required: false
  sensitive: false
  description: Service region (default: us-east-1)
```

**Session Token (optional, sensitive):**
```yaml
- key: AWS_SESSION_TOKEN
  name: AWS Session Token
  required: false
  sensitive: true
  description: Required only for temporary credentials (SSO, STS AssumeRole)
```

---

## Tool Preview Documentation

The `toolPreview` section documents the tools your MCP server provides. This is displayed in the catalog and used for LLM vulnerability scanning.

### Basic Tool (no parameters)

```yaml
- name: list_items
  description: List all items in the account
  params: {}
```

### Tool with Parameters

```yaml
- name: get_item
  description: Get details about a specific item
  params:
    item_id: The unique identifier of the item
    include_metadata: Whether to include metadata (optional)
```

### Tool with Typed Parameters

```yaml
- name: create_item
  description: Create a new item
  params:
    name: Name of the item to create
    type:
      type: string
      description: Type of item (optional)
    count:
      type: integer
      description: Number of items to create
```

---

## Metadata Options

### Categories

Available categories for the `metadata.categories` field:

- `Automation & Browsers`
- `Business`
- `Cloud & Infrastructure` / `Cloud Services`
- `Communication`
- `Databases`
- `Developer Tools`
- `DevOps`
- `Documentation`
- `Finance`
- `Infrastructure`
- `Productivity`
- `Retrieval & Search`

Multiple categories can be specified as comma-separated values:

```yaml
metadata:
  categories: Infrastructure,Cloud Services,DevOps
```

### Allow Multiple Instances

Enable users to add multiple instances of the same server (e.g., multiple Slack workspaces):

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
  unsupportedTools: create_file,delete_file,push_files
```

---

## Validation and CI/CD Pipeline

When you submit a pull request, the following automated checks run:

### 1. LLM Vulnerability Scanning (`vuln-scan.yml`)

All tools in `toolPreview` are analyzed for potential security risks:

- Prompt injection vulnerabilities
- Tool poisoning risks
- Toxic flow patterns
- LLM-based attack vectors

The scanner uses GPT to analyze tool definitions and flag potentially malicious patterns.

**Script:** `scripts/mcp_tool_linter/mcp_tool_linter.py`

### 2. MCP Server Validation (`trigger-automation.yml`)

For containerized servers, automated testing runs against a live Obot instance:

1. Spins up an Obot container
2. Creates a catalog entry from your YAML
3. Runs WebdriverIO tests to validate the MCP server works

**Requirements:**
- Valid `containerizedConfig` with accessible image
- Properly defined `env` variables

### 3. Tool Comparison (for image updates)

When updating container image tags, tools are compared between versions:

```bash
scripts/validation/compare-mcp-tools.sh \
  <package_type> \   # "node" or "python"
  <package_name> \   # Full package name
  <old_version> \
  <new_version> \
  <catalog_name>     # Matches YAML filename
```

This identifies:
- Added tools
- Removed tools (potential breaking changes)
- Modified tool schemas

### 4. Environment Requirements Check

The validation pipeline checks if required environment variables are available:

```bash
python scripts/validation/get-env-requirements.py <catalog_name>
```

If secrets aren't configured in GitHub Actions, validation is skipped with a warning.

---

## Complete Examples

### Example 1: Containerized Database Server (PostgreSQL)

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

### Example 2: Containerized Service with Multiple Credentials (AWS)

```yaml
name: AWS API
shortDescription: Execute AWS CLI commands across all AWS services
description: |
  A Model Context Protocol server for AWS CLI operations.

  ## Features
  - Universal AWS CLI Access
  - Intelligent Command Suggestions
  - Multi-Service Support

  ## What you'll need to connect

  **Required:**
  - **AWS Access Key ID**: Your AWS credential access key
  - **AWS Secret Access Key**: Your AWS credential secret

  **Optional:**
  - **AWS Region**: AWS region (default: us-east-1)
  - **AWS Session Token**: For temporary credentials

metadata:
  categories: Infrastructure,Cloud Services,DevOps
icon: https://avatars.githubusercontent.com/u/3299148?s=48&v=4
repoURL: https://github.com/awslabs/mcp

runtime: containerized
containerizedConfig:
  image: ghcr.io/obot-platform/mcp-images/aws-api:1.1.4
  port: 8099
  path: /
  args:
    - awslabs.aws-api-mcp-server

env:
  - key: AWS_ACCESS_KEY_ID
    name: AWS Access Key ID
    required: true
    sensitive: true
    description: Your AWS Access Key ID

  - key: AWS_SECRET_ACCESS_KEY
    name: AWS Secret Access Key
    required: true
    sensitive: true
    description: AWS Secret Access Key

  - key: AWS_SESSION_TOKEN
    name: AWS Session Token
    required: false
    sensitive: true
    description: Required for temporary credentials (SSO, STS AssumeRole)

  - key: AWS_REGION
    name: AWS Region
    required: false
    sensitive: false
    description: AWS Region (default: us-east-1)

toolPreview:
  - name: call_aws
    description: Execute AWS CLI commands with validation
    params:
      cli_command: The complete AWS CLI command (must start with "aws")
  - name: suggest_aws_commands
    description: Get AI-powered suggestions for AWS CLI commands
    params:
      query: Natural language description of what you want to accomplish
```

### Example 3: Remote Server with Header Authentication (GitHub)

```yaml
name: GitHub
shortDescription: Manage GitHub repositories, issues, PRs, and workflows
description: |
  A hosted MCP server for GitHub integration.

  ## Features
  - Repository Management
  - Issue & PR Automation
  - CI/CD & Workflow Intelligence

  ## What you'll need to connect

  **Required:**
  - **Personal Access Token**: GitHub PAT with repository permissions

metadata:
  categories: Developer Tools
  unsupportedTools: create_or_update_file,push_files
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

### Example 4: Remote Server Without Authentication (GitMCP)

```yaml
name: GitMCP
shortDescription: Access any GitHub project's documentation in real-time
description: |
  A cloud-based MCP server for GitHub documentation access.

  ## Features
  - Latest documentation access
  - Zero setup required
  - Open and free

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

### Example 5: Containerized Server with Optional Config (Context7)

```yaml
name: Context7
shortDescription: Get up-to-date, version-specific code documentation
description: |
  A Model Context Protocol server that provides up-to-date code documentation.

  ## Features
  - Up-to-date Documentation
  - No More Hallucinated APIs
  - Automatic Library Resolution

  ## What you'll need to connect

  **No Setup Required**: Works out-of-the-box.

  **Optional:**
  - **DEFAULT_MINIMUM_TOKENS**: Minimum token count (default: 10000)

metadata:
  categories: Developer Tools
icon: https://avatars.githubusercontent.com/u/74989412?v=4
repoURL: https://github.com/upstash/context7

runtime: containerized
containerizedConfig:
  image: ghcr.io/obot-platform/mcp-images/context7:1.0.31
  port: 8099
  path: /
  args:
    - context7-mcp

env:
  - key: DEFAULT_MINIMUM_TOKENS
    name: Default Minimum Tokens
    required: false
    sensitive: false
    description: Minimum token count for documentation retrieval (default: 10000)

toolPreview:
  - name: resolve-library-id
    description: Resolves a package name to a Context7-compatible library ID
    params:
      libraryName: The name of the library to search for
  - name: get-library-docs
    description: Fetches documentation using a Context7-compatible library ID
    params:
      context7CompatibleLibraryID: Library ID (e.g., /mongodb/docs)
      tokens: Maximum tokens to return (optional)
      topic: Topic to focus on (optional)
```

---

## Checklist for New MCP Servers

Before submitting your PR:

- [ ] YAML file named `servicename.yaml` in repository root
- [ ] `name` and `shortDescription` are concise and descriptive
- [ ] `description` includes Features and "What you'll need to connect" sections
- [ ] `metadata.categories` uses valid category names
- [ ] `icon` URL is valid and accessible
- [ ] `repoURL` points to the source repository
- [ ] `runtime` is either `containerized` or `remote`
- [ ] For containerized: `containerizedConfig` has valid `image`, `port`, `path`
- [ ] For containerized: `env` defines all required/optional variables
- [ ] For remote: `remoteConfig` has valid `hostname` or `fixedURL`
- [ ] For remote with auth: `headers` properly configured
- [ ] `toolPreview` documents all available tools
- [ ] No sensitive defaults in environment variable descriptions
- [ ] Description markdown renders correctly

---

## Resources

- [Obot Documentation](https://docs.obot.ai)
- [MCP Specification](https://modelcontextprotocol.io)
- [Obot Platform](https://obot.ai)
- [MCP Catalog Repository](https://github.com/obot-platform/mcp-catalog)
