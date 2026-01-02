# Model Context Protocol (MCP) in Claude Code: A Comprehensive Guide

## Table of Contents
1. [What is MCP and Why It Matters](#what-is-mcp-and-why-it-matters)
2. [Connecting to External Services](#connecting-to-external-services)
3. [Configuring MCP in Claude Code](#configuring-mcp-in-claude-code)
4. [Available MCP Servers and Tools](#available-mcp-servers-and-tools)
5. [Creating Custom MCP Servers](#creating-custom-mcp-servers)
6. [Security Considerations](#security-considerations)
7. [Debugging MCP Connections](#debugging-mcp-connections)
8. [Best Practices](#best-practices)

---

## What is MCP and Why It Matters

The **Model Context Protocol (MCP)** is an open-source standard introduced by Anthropic in November 2024 that enables AI applications to connect to external systems in a standardized way. Often described as "the USB-C port for AI applications," MCP allows AI models like Claude to seamlessly interact with data sources, tools, and workflows.

### Architecture Overview

MCP uses a **client-server architecture** where AI applications (like Claude Code) act as MCP clients that connect to MCP servers. These servers provide access to:
- **Data sources**: Local files, databases, cloud storage
- **Tools**: Search engines, calculators, APIs, system commands
- **Workflows**: Specialized prompts and task automation

### Core Primitives

The MCP specification defines building blocks called **primitives** through JSON-RPC messages:

**Server-Side Primitives:**
1. **Tools**: Model-controlled functions that the AI can invoke directly
2. **Resources**: App-controlled data that the AI can read (files, database records)
3. **Prompts**: User-controlled templates for structured interactions

**Client-Side Primitives:**
1. **Roots**: Define working directories and scope
2. **Sampling**: Enable servers to request AI completions

### Why MCP Matters

Since its launch, MCP has seen rapid adoption with:
- **Thousands of MCP servers** built by the community
- **SDKs available** for all major programming languages (TypeScript, Python, Java, Go, etc.)
- **Industry adoption** by major AI providers including OpenAI and Google DeepMind
- **Linux Foundation stewardship**: In December 2025, Anthropic donated MCP to the Agentic AI Foundation (AAIF), a directed fund under the Linux Foundation

MCP solves the problem of fragmented AI integrations by providing a universal protocol, eliminating the need for custom connectors for each tool or data source.

---

## Connecting to External Services

MCP enables Claude Code to connect to hundreds of external services through three transport mechanisms:

### Transport Types

#### 1. HTTP Transport (Recommended)
Best for remote, cloud-based MCP servers. This is the most widely supported transport for production deployments.

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  }
}
```

#### 2. Stdio Transport
For local processes running on your machine with direct system access. Ideal for filesystem operations, local databases, and development.

```json
{
  "mcpServers": {
    "local-server": {
      "type": "stdio",
      "command": "node",
      "args": ["/path/to/server/index.js"],
      "env": {
        "DATABASE_URL": "${DB_CONNECTION_STRING}"
      }
    }
  }
}
```

#### 3. SSE Transport (Deprecated)
Server-Sent Events transport is deprecated; use HTTP instead for new implementations.

### Common Integration Scenarios

**Database Connections:**
- PostgreSQL, MySQL, MongoDB, BigQuery
- Vector databases (Pinecone, Weaviate)
- NoSQL databases (DynamoDB, CosmosDB)

**Cloud Services:**
- AWS (S3, Lambda, CloudWatch)
- Azure (Storage, Cosmos DB, DevOps)
- Google Cloud Platform

**APIs and Platforms:**
- GitHub (repositories, PRs, issues)
- Slack (channels, messages, workflows)
- Salesforce (CRM data, contacts)
- Zapier (5,000+ app integrations)

**File Systems:**
- Local filesystem access
- Remote storage (SFTP, S3, SMB, NFS)
- Version control (Git repositories)

---

## Configuring MCP in Claude Code

### Configuration File Locations

Claude Code recognizes MCP servers from different configuration scopes:

1. **Project Scope**: `.mcp.json` in project root (shared with team, version controlled)
2. **User Scope**: `~/.claude.json` in home directory (personal configurations)
3. **Local Scope**: Local overrides that take precedence
4. **Enterprise Scope**: `managed-mcp.json` for centralized organizational control

### Configuration File Structure

The standard `.mcp.json` format uses an `mcpServers` object:

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/directory"
      ]
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    }
  }
}
```

### Environment Variable Expansion

Use environment variables to keep sensitive data out of version control:

```json
{
  "mcpServers": {
    "api-service": {
      "type": "http",
      "url": "${API_BASE_URL}",
      "env": {
        "API_KEY": "${SERVICE_API_KEY}",
        "TIMEOUT": "${API_TIMEOUT:-30000}"
      }
    }
  }
}
```

Syntax:
- `${VAR}`: Required variable (fails if not set)
- `${VAR:-default}`: Optional with default value

### Platform-Specific Configuration

**Windows Users (Non-WSL):**
Local stdio servers using `npx` require the `cmd /c` wrapper:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "cmd",
      "args": [
        "/c",
        "npx",
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "C:\\Users\\username\\projects"
      ]
    }
  }
}
```

Without this wrapper, you'll encounter "Connection closed" errors.

### Managing Configurations via CLI

Claude Code provides CLI commands for managing MCP servers:

```bash
# Add a new MCP server
claude mcp add server-name --scope user

# List all configured servers
claude mcp list

# Remove a server
claude mcp remove server-name

# Test a server connection
claude mcp get server-name
```

### Activating MCP Servers

After configuration:
1. **Restart Claude Code** for changes to take effect
2. Use the **`/mcp` slash command** to view and manage servers
3. **@mention servers** to toggle them on/off during sessions
4. Look for the **MCP tools icon** indicating available tools

---

## Available MCP Servers and Tools

The MCP ecosystem includes hundreds of servers. Here are the most popular categories:

### Official Reference Servers

Maintained by the MCP steering group:

| Server | Description | Use Case |
|--------|-------------|----------|
| **filesystem** | Secure file operations with access controls | Reading code, editing files, directory navigation |
| **git** | Tools to read, search, and manipulate Git repositories | Version control operations, diff viewing, commit history |
| **fetch** | Web content fetching and conversion | Scraping web content, API calls, data retrieval |
| **memory** | Knowledge graph-based persistent memory | Maintaining context across conversations |
| **sequential-thinking** | Dynamic problem-solving through thought sequences | Complex reasoning, step-by-step analysis |
| **time** | Time and timezone conversion capabilities | Scheduling, time calculations |

### Database Servers

**Relational Databases:**
- **PostgreSQL**: Full-featured database integration with schema inspection
- **MySQL**: Configurable access controls and query execution
- **BigQuery**: Google Cloud data warehouse integration

**NoSQL & Vector Databases:**
- **MongoDB**: Query and analyze MongoDB collections
- **MongoDB Lens**: Advanced MongoDB analytics
- **Pinecone**: Vector database for semantic search
- **ClickHouse**: Analytics and data retrieval
- **Supabase**: Database, auth, and edge functions

**Cloud Database Platforms:**
- **Neon**: Natural language database management
- **AWS DynamoDB**: NoSQL database operations
- **Azure Cosmos DB**: Multi-model database access

### Filesystem & Storage Servers

- **filesystem** (official): Local filesystem with directory restrictions
- **fast-filesystem-mcp**: Advanced operations with large file handling
- **FileStash**: Remote storage (SFTP, S3, FTP, SMB, NFS, WebDAV, Git)
- **AWS S3**: Cloud object storage
- **Azure Blob Storage**: Cloud file storage

### API Integration Servers

**Development Platforms:**
- **GitHub**: Repository management, PRs, issues, workflows
- **GitLab**: CI/CD, repositories, merge requests
- **Bitbucket**: Repository and pull request management

**Communication & Collaboration:**
- **Slack**: Real-time conversations, channels, workflows
- **Microsoft Teams**: Chat and collaboration
- **Discord**: Community messaging

**Business Platforms:**
- **Salesforce**: CRM data, leads, accounts, opportunities
- **HubSpot**: Marketing and sales automation
- **Zendesk**: Customer support tickets

**Integration Platforms:**
- **Zapier**: Connect to 5,000+ apps without custom code
- **Pipedream**: Serverless code execution and event-driven workflows

### Enterprise & Cloud Servers

- **AWS Bedrock**: Access AWS AI services
- **Azure DevOps**: Build pipelines, work items, releases
- **Databricks**: Delta Lake and ML pipeline integration
- **K2view**: Multi-source enterprise data virtualization

### Specialty Servers

- **Brave Search**: Web search integration
- **Exa**: AI-powered search
- **Puppeteer**: Browser automation and web scraping
- **Playwright**: End-to-end testing and automation
- **Screenshot**: Capture web pages
- **Weather**: Real-time weather data

### Discovering More Servers

Several curated lists help discover MCP servers:

- **GitHub MCP Registry**: Official registry for discovering servers
- **GitMCP.io**: MCP servers for any GitHub project
- **Awesome MCP Servers**: Community-curated lists
  - [punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)
  - [wong2/awesome-mcp-servers](https://github.com/wong2/awesome-mcp-servers)
- **MCP.so**: Directory of MCP servers
- **MCPServers.org**: Searchable server directory

---

## Creating Custom MCP Servers

Building custom MCP servers allows you to expose your own services, APIs, and data sources to Claude Code.

### Prerequisites

**For TypeScript:**
- Node.js version 16 or higher
- npm or yarn package manager

**For Python:**
- Python 3.9 or higher
- MCP SDK 1.2.0+
- `uv` or `pip` for package management

### Creating a TypeScript MCP Server

#### 1. Initialize Your Project

```bash
# Create project directory
mkdir my-mcp-server
cd my-mcp-server

# Initialize npm project
npm init -y

# Install MCP SDK
npm install @modelcontextprotocol/sdk zod
```

#### 2. Create Server Implementation

Create `index.ts`:

```typescript
#!/usr/bin/env node

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

// Create server instance
const server = new Server(
  {
    name: "my-custom-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// Define tools
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "get_weather",
        description: "Get weather information for a city",
        inputSchema: {
          type: "object",
          properties: {
            city: {
              type: "string",
              description: "City name",
            },
          },
          required: ["city"],
        },
      },
    ],
  };
});

// Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "get_weather") {
    const city = request.params.arguments?.city as string;

    // Your implementation here
    const weatherData = await fetchWeatherData(city);

    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(weatherData, null, 2),
        },
      ],
    };
  }

  throw new Error(`Unknown tool: ${request.params.name}`);
});

// Start server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);

  // IMPORTANT: Never write to stdout in stdio servers
  console.error("MCP Server running on stdio");
}

main().catch((error) => {
  console.error("Server error:", error);
  process.exit(1);
});
```

#### 3. Build and Configure

```bash
# Add build script to package.json
# "build": "tsc"

# Build the server
npm run build

# Test locally
node dist/index.js
```

Add to `.mcp.json`:

```json
{
  "mcpServers": {
    "my-custom-server": {
      "type": "stdio",
      "command": "node",
      "args": ["/absolute/path/to/my-mcp-server/dist/index.js"]
    }
  }
}
```

### Creating a Python MCP Server

#### 1. Initialize Project with uv

```bash
# Create project using uv
uv init my-mcp-server
cd my-mcp-server

# Create virtual environment
uv venv

# Install MCP SDK
uv add mcp[cli] httpx
```

#### 2. Create Server Implementation

Create `server.py`:

```python
#!/usr/bin/env python3

import asyncio
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

# Create server instance
app = Server("my-python-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    """List available tools."""
    return [
        Tool(
            name="calculate",
            description="Perform basic calculations",
            inputSchema={
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "Math expression to evaluate"
                    }
                },
                "required": ["expression"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    """Handle tool calls."""
    if name == "calculate":
        expression = arguments["expression"]
        try:
            result = eval(expression)  # Note: Use safe evaluation in production
            return [TextContent(type="text", text=str(result))]
        except Exception as e:
            return [TextContent(type="text", text=f"Error: {str(e)}")]

    raise ValueError(f"Unknown tool: {name}")

async def main():
    """Run the server."""
    # IMPORTANT: Use logging.info(), never print() in stdio servers
    import logging
    logging.basicConfig(level=logging.INFO)
    logging.info("Starting MCP server")

    async with stdio_server() as streams:
        await app.run(
            streams[0],
            streams[1],
            app.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

#### 3. Configure in Claude Code

Add to `.mcp.json`:

```json
{
  "mcpServers": {
    "my-python-server": {
      "type": "stdio",
      "command": "uv",
      "args": ["run", "server.py"],
      "env": {
        "PYTHONPATH": "/path/to/my-mcp-server"
      }
    }
  }
}
```

### Using FastMCP (Python)

FastMCP provides a simpler Python API using decorators:

```python
from fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
def add_numbers(a: int, b: int) -> int:
    """Add two numbers together."""
    return a + b

@mcp.resource("config://app")
def get_config() -> dict:
    """Get application configuration."""
    return {"version": "1.0.0", "env": "production"}
```

### Key Implementation Tips

1. **Stdio Servers - Output Restrictions**:
   - **Never write to stdout** (it's reserved for MCP protocol)
   - Python: Use `logging.info()` instead of `print()`
   - JavaScript: Use `console.error()` instead of `console.log()`

2. **Type Safety**:
   - Use Zod (TypeScript) or Pydantic (Python) for schema validation
   - Define clear input schemas for all tools

3. **Error Handling**:
   - Always catch and return meaningful error messages
   - Don't let exceptions crash the server

4. **Documentation**:
   - Provide clear descriptions for tools and parameters
   - Use docstrings (Python) or JSDoc (TypeScript)

5. **Testing**:
   - Test servers independently before integrating
   - Use the MCP Inspector for debugging

---

## Security Considerations

MCP servers can access sensitive data and perform privileged operations. Security must be a primary concern.

### Critical Security Risks

Recent security analyses have identified:
- **Multiple CVEs** (CVSS 7.3-9.6) affecting 437,000+ installations
- **43% of analyzed servers** vulnerable to command injection
- **Confused deputy vulnerabilities** in proxy servers
- **Prompt injection risks** through external data sources

### Authentication & Authorization

#### OAuth 2.1 with PKCE (Required for Remote Servers)

Remote MCP servers must implement OAuth 2.1 with PKCE:

```json
{
  "mcpServers": {
    "secure-api": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "auth": {
        "type": "oauth2",
        "authorizationUrl": "https://auth.example.com/authorize",
        "tokenUrl": "https://auth.example.com/token",
        "scope": "read write"
      }
    }
  }
}
```

Use `/mcp` command in Claude Code to authenticate.

#### Fine-Grained Access Control

Implement least-privilege access:

```typescript
// Tool-level authorization
server.setRequestHandler(CallToolRequestSchema, async (request, context) => {
  const tool = request.params.name;
  const user = context.user;

  if (!hasPermission(user, tool)) {
    throw new Error(`User ${user} not authorized for tool ${tool}`);
  }

  // Execute tool
});
```

### Supply Chain Security

#### 1. Trust Only Verified Sources
- Use official MCP servers from trusted repositories
- Check GitHub stars, contributors, and maintenance activity
- Verify developer signatures when available

#### 2. Code Signing & Verification
```bash
# Verify npm package signatures
npm audit

# Check for known vulnerabilities
npm audit fix
```

#### 3. Security Scanning in CI/CD
Implement in your build pipeline:

**Static Application Security Testing (SAST):**
```yaml
# .github/workflows/security.yml
- name: Run SAST scan
  run: npm run security:scan
```

**Software Composition Analysis (SCA):**
```yaml
- name: Check dependencies
  run: npm audit --audit-level=moderate
```

### Data Protection

#### Minimal Data Collection

```python
@app.call_tool()
async def process_request(name: str, arguments: dict):
    # Only log necessary information
    logger.info(f"Tool called: {name}")  # Good
    # logger.info(f"Full args: {arguments}")  # Bad - may contain PII
```

#### Sensitive Data Handling

```typescript
// Filter sensitive fields before logging
function sanitize(data: any): any {
  const sensitive = ['password', 'token', 'apiKey', 'secret'];
  const sanitized = { ...data };

  for (const key of sensitive) {
    if (key in sanitized) {
      sanitized[key] = '***REDACTED***';
    }
  }

  return sanitized;
}
```

### Logging & Monitoring

#### Comprehensive Audit Logs

Log every MCP interaction:

```typescript
interface AuditLog {
  timestamp: string;
  user: string;
  tool: string;
  parameters: any;
  result: any;
  status: 'success' | 'error';
}

function logToolCall(log: AuditLog) {
  // Store securely
  auditLogger.info(sanitize(log));

  // Send to SIEM if available
  if (siemIntegration) {
    siemIntegration.send(log);
  }
}
```

#### MCP Protocol Logging

Enable structured logging:

```json
{
  "mcpServers": {
    "monitored-server": {
      "type": "stdio",
      "command": "node",
      "args": ["server.js"],
      "env": {
        "LOG_LEVEL": "debug",
        "LOG_FILE": "/var/log/mcp/server.log"
      }
    }
  }
}
```

### Sandboxing & Isolation

#### Container-Based Isolation

Run MCP servers in containers:

```dockerfile
# Dockerfile
FROM node:20-alpine

# Create non-root user
RUN addgroup -g 1001 -S mcpserver && \
    adduser -S mcpserver -u 1001

# Copy server files
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .

# Drop privileges
USER mcpserver

CMD ["node", "server.js"]
```

```json
{
  "mcpServers": {
    "isolated-server": {
      "type": "http",
      "url": "http://localhost:3000/mcp",
      "command": "docker",
      "args": ["run", "-p", "3000:3000", "my-mcp-server"]
    }
  }
}
```

#### File System Restrictions

Limit filesystem access:

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects"  // Restricted to this directory only
      ]
    }
  }
}
```

### Enterprise Security Controls

#### Allowlists & Denylists

```json
{
  "mcpSecurity": {
    "allowlist": {
      "servers": ["filesystem", "git", "github"],
      "commands": ["npx", "node", "python3"],
      "urls": ["https://api.internal.company.com/*"]
    },
    "denylist": {
      "servers": ["untrusted-server"],
      "commands": ["curl", "wget", "bash"]
    }
  }
}
```

#### Managed MCP Configuration

Deploy organization-wide configurations:

```json
// /etc/claude/managed-mcp.json
{
  "mcpServers": {
    "company-database": {
      "type": "http",
      "url": "https://mcp.company.com/database",
      "required": true,
      "userModifiable": false
    }
  },
  "security": {
    "allowUserServers": false,
    "requireSignedServers": true,
    "maxConcurrentConnections": 5
  }
}
```

### OWASP Recommendations

Follow OWASP GenAI Security Project guidelines:

1. **Tool Poisoning Prevention**: Validate all tool definitions
2. **Prompt Injection Mitigation**: Sanitize external inputs
3. **Memory Poisoning Protection**: Validate knowledge graph entries
4. **Human-in-the-Loop**: Require approval for sensitive operations

```typescript
// Require confirmation for destructive operations
if (isDestructiveOperation(tool)) {
  const approved = await requestUserConfirmation(
    `Allow ${tool} to ${description}?`
  );

  if (!approved) {
    throw new Error('Operation cancelled by user');
  }
}
```

---

## Debugging MCP Connections

### Common Issues and Solutions

#### 1. "Connection closed" Errors

**Symptoms**: Server fails to start or immediately disconnects

**Causes & Solutions**:

**Windows without cmd /c wrapper:**
```json
// Wrong
{
  "command": "npx",
  "args": ["-y", "server"]
}

// Correct
{
  "command": "cmd",
  "args": ["/c", "npx", "-y", "server"]
}
```

**Stdout contamination:**
```python
# Wrong - writes to stdout
print("Starting server")

# Correct - writes to stderr
import logging
logging.info("Starting server")
```

#### 2. Server Not Appearing in Claude Code

**Checklist**:
1. Restart Claude Code after configuration changes
2. Verify `.mcp.json` syntax with JSON validator
3. Check file location (project root or `~/.claude.json`)
4. Confirm scope priority (local > project > user)

**Debugging command**:
```bash
# Test server configuration
claude mcp get server-name

# List all servers
claude mcp list
```

#### 3. Authentication Failures

**HTTP Servers**:
```bash
# Check if OAuth token is valid
curl -H "Authorization: Bearer $TOKEN" https://api.example.com/mcp
```

**Environment variables**:
```bash
# Verify variables are set
echo $GITHUB_TOKEN
echo $DATABASE_URL

# Check variable expansion
claude mcp get server-name --verbose
```

#### 4. Timeout Issues

Increase timeout limits:

```json
{
  "mcpServers": {
    "slow-server": {
      "type": "stdio",
      "command": "python",
      "args": ["server.py"],
      "env": {
        "MCP_TIMEOUT": "60000"  // 60 seconds
      }
    }
  }
}
```

Or globally:
```bash
export MCP_TIMEOUT=60000
```

#### 5. Token Limit Warnings

Increase output token limit:

```bash
# Increase to 50,000 tokens
export MAX_MCP_OUTPUT_TOKENS=50000
```

Or in server configuration:
```json
{
  "mcpServers": {
    "large-output-server": {
      "env": {
        "MAX_MCP_OUTPUT_TOKENS": "50000"
      }
    }
  }
}
```

### Debugging Tools

#### MCP Inspector

Use the official MCP Inspector for interactive debugging:

```bash
# Install globally
npm install -g @modelcontextprotocol/inspector

# Launch inspector
mcp-inspector
```

The inspector provides:
- Interactive tool testing
- Request/response visualization
- Schema validation
- Performance monitoring

#### Claude Code CLI

```bash
# View MCP server status
claude mcp status

# Test connection to specific server
claude mcp test server-name

# View detailed logs
claude mcp logs server-name

# Enable debug mode
claude mcp debug server-name
```

#### Server-Side Logging

**TypeScript**:
```typescript
// Always log to stderr in stdio servers
console.error('Server started');
console.error('Tool called:', toolName);
console.error('Result:', JSON.stringify(result, null, 2));
```

**Python**:
```python
import logging

# Configure logging to file
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('/tmp/mcp-server.log'),
        logging.StreamHandler()  # stderr
    ]
)

logger = logging.getLogger(__name__)
logger.debug('Server started')
logger.info('Tool called: %s', tool_name)
```

#### Network Debugging

For HTTP servers:

```bash
# Test server endpoint
curl -X POST https://api.example.com/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":1}'

# Check network connectivity
nc -zv api.example.com 443

# Monitor traffic
tcpdump -i any -s 0 -A 'host api.example.com'
```

### Verbose Mode

Enable detailed logging:

```json
{
  "mcpServers": {
    "debug-server": {
      "type": "stdio",
      "command": "node",
      "args": ["server.js"],
      "env": {
        "DEBUG": "*",
        "MCP_LOG_LEVEL": "debug",
        "NODE_ENV": "development"
      }
    }
  }
}
```

### Using the /mcp Command

Within Claude Code:
```
/mcp
```

This opens an interactive interface showing:
- All configured MCP servers
- Current status (enabled/disabled/error)
- Authentication status
- Available tools
- Toggle switches to enable/disable servers

---

## Best Practices

### Configuration Management

#### 1. Use Environment Variables for Secrets

**Never commit secrets to version control:**

```json
// .mcp.json (committed to git)
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL}",
      "env": {
        "API_KEY": "${SERVICE_API_KEY}"
      }
    }
  }
}
```

```bash
# .env (NOT committed, in .gitignore)
API_BASE_URL=https://api.example.com
SERVICE_API_KEY=sk_live_abc123xyz789
```

#### 2. Scope Configuration Appropriately

- **Project scope** (`.mcp.json`): Team-shared tools (filesystem, git)
- **User scope** (`~/.claude.json`): Personal tools (private APIs, experimental servers)
- **Local scope**: Temporary overrides and testing

#### 3. Minimize Active Servers

Enable only needed servers to reduce context window usage:

```
# Within Claude Code, toggle servers:
@filesystem    # Enable filesystem server
@unused-server # Disable unused server
```

Each enabled server adds tool definitions to Claude's context, even when not actively used.

### Development Workflow

#### 1. Test Locally First

```bash
# Test server independently
node server.js

# Use MCP Inspector for interactive testing
mcp-inspector

# Verify with CLI
claude mcp test my-server
```

#### 2. Implement Graceful Error Handling

```typescript
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  try {
    const result = await executeTool(request.params.name, request.params.arguments);
    return { content: [{ type: "text", text: result }] };
  } catch (error) {
    // Return user-friendly error message
    return {
      content: [{
        type: "text",
        text: `Error: ${error.message}. Please check your parameters.`
      }],
      isError: true
    };
  }
});
```

#### 3. Version Your Servers

```json
{
  "mcpServers": {
    "my-server-v2": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "my-server@2.0.0"]
    }
  }
}
```

### Performance Optimization

#### 1. Cache Expensive Operations

```typescript
const cache = new Map<string, any>();

async function fetchData(key: string): Promise<any> {
  if (cache.has(key)) {
    return cache.get(key);
  }

  const data = await expensiveOperation(key);
  cache.set(key, data);

  // Set expiration
  setTimeout(() => cache.delete(key), 5 * 60 * 1000); // 5 minutes

  return data;
}
```

#### 2. Implement Streaming for Large Data

```typescript
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === 'read_large_file') {
    const stream = createReadStream(filepath);

    return {
      content: [{
        type: "text",
        text: await streamToString(stream)
      }]
    };
  }
});
```

#### 3. Set Appropriate Timeouts

```json
{
  "mcpServers": {
    "quick-server": {
      "env": {
        "REQUEST_TIMEOUT": "5000"  // 5 seconds for fast operations
      }
    },
    "batch-processor": {
      "env": {
        "REQUEST_TIMEOUT": "300000"  // 5 minutes for batch jobs
      }
    }
  }
}
```

### Documentation & Maintenance

#### 1. Document Your Servers

Create a `README.md` for custom servers:

```markdown
# My MCP Server

## Description
Provides access to internal API services.

## Configuration
\`\`\`json
{
  "mcpServers": {
    "my-server": {
      "type": "stdio",
      "command": "node",
      "args": ["dist/index.js"],
      "env": {
        "API_KEY": "${MY_API_KEY}"
      }
    }
  }
}
\`\`\`

## Required Environment Variables
- `MY_API_KEY`: API key from https://api.example.com

## Available Tools
- `search`: Search the knowledge base
- `create`: Create new entries
- `update`: Update existing entries
```

#### 2. Keep Dependencies Updated

```bash
# Check for updates
npm outdated

# Update MCP SDK
npm update @modelcontextprotocol/sdk

# Security updates
npm audit fix
```

#### 3. Monitor Server Health

Implement health checks:

```typescript
server.setRequestHandler(ListToolsRequestSchema, async () => {
  // Check dependencies
  await checkDatabaseConnection();
  await checkExternalAPIs();

  return { tools: [...] };
});
```

### Team Collaboration

#### 1. Share Configurations via Git

```bash
# Commit project-wide MCP config
git add .mcp.json
git commit -m "Add filesystem and git MCP servers"
git push
```

Team members automatically get the configuration.

#### 2. Document Environment Setup

Create `.env.example`:

```bash
# .env.example
API_BASE_URL=https://api.example.com
SERVICE_API_KEY=your_key_here
DATABASE_URL=postgresql://localhost/mydb
GITHUB_TOKEN=ghp_yourtokenhere
```

#### 3. Use Consistent Naming

Adopt naming conventions:
- `{service}-{environment}`: `postgres-prod`, `postgres-dev`
- `{provider}-{resource}`: `aws-s3`, `github-repos`
- `{purpose}-{version}`: `analytics-v2`, `legacy-api-v1`

### Security Best Practices Summary

1. **Principle of Least Privilege**: Grant minimum necessary permissions
2. **Defense in Depth**: Multiple security layers
3. **Zero Trust**: Verify every request
4. **Audit Everything**: Comprehensive logging
5. **Regular Updates**: Keep dependencies current
6. **Security Scanning**: Automated vulnerability detection
7. **Incident Response**: Plan for security events
8. **User Education**: Train team on secure usage

### When to Use MCP vs. Direct Integration

**Use MCP when:**
- Building reusable integrations across multiple AI applications
- Need standardized protocol for tool access
- Want community-supported servers
- Require enterprise governance controls

**Use direct integration when:**
- Simple, one-off API calls
- No need for persistence or state
- Performance is critical (direct API faster than MCP layer)
- Tool is too simple to justify MCP overhead

---

## Conclusion

The Model Context Protocol represents a paradigm shift in how AI applications integrate with external systems. By providing a universal standard, MCP enables:

- **Seamless integration** with databases, APIs, and tools
- **Reusable components** across different AI applications
- **Community-driven ecosystem** with hundreds of pre-built servers
- **Enterprise-grade security** with standardized controls
- **Future-proof architecture** backed by industry leaders

Whether you're connecting Claude Code to your company's internal systems, building custom integrations, or leveraging community servers, MCP provides the foundation for powerful AI-driven workflows.

### Getting Started Checklist

- [ ] Install Claude Code and verify MCP support (`/mcp` command)
- [ ] Create `.mcp.json` in your project root
- [ ] Configure essential servers (filesystem, git)
- [ ] Set up environment variables for secrets
- [ ] Test connections with `claude mcp list` and `/mcp`
- [ ] Explore community servers for your use cases
- [ ] Review security best practices for your organization
- [ ] Build custom servers for proprietary systems
- [ ] Implement monitoring and logging
- [ ] Document configurations for your team

### Additional Resources

**Official Documentation:**
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Claude Code MCP Docs](https://code.claude.com/docs/en/mcp)
- [MCP GitHub Organization](https://github.com/modelcontextprotocol)

**SDKs:**
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk)

**Learning:**
- [Anthropic MCP Course](https://anthropic.skilljar.com/introduction-to-model-context-protocol)
- [FreeCodeCamp TypeScript Tutorial](https://www.freecodecamp.org/news/how-to-build-a-custom-mcp-server-with-typescript-a-handbook-for-developers/)

**Discovery:**
- [GitHub MCP Registry](https://github.blog/ai-and-ml/github-copilot/meet-the-github-mcp-registry-the-fastest-way-to-discover-mcp-servers/)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- [GitMCP.io](https://gitmcp.io/)

**Community:**
- [MCP Discord Community](https://discord.gg/modelcontextprotocol)
- [GitHub Discussions](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions)

---

## Sources

- [What is the Model Context Protocol (MCP)?](https://modelcontextprotocol.io/)
- [Introducing the Model Context Protocol - Anthropic](https://www.anthropic.com/news/model-context-protocol)
- [Model Context Protocol GitHub](https://github.com/modelcontextprotocol/modelcontextprotocol)
- [Connect Claude Code to tools via MCP](https://code.claude.com/docs/en/mcp)
- [Configuring MCP Tools in Claude Code - Scott Spence](https://scottspence.com/posts/configuring-mcp-tools-in-claude-code)
- [GitHub's Official MCP Server](https://github.com/github/github-mcp-server)
- [Model Context Protocol Servers Repository](https://github.com/modelcontextprotocol/servers)
- [Awesome MCP Servers - punkpeye](https://github.com/punkpeye/awesome-mcp-servers)
- [Security Best Practices - Model Context Protocol](https://modelcontextprotocol.io/specification/draft/basic/security_best_practices)
- [Model Context Protocol: Understanding security risks](https://www.redhat.com/en/blog/model-context-protocol-mcp-understanding-security-risks-and-controls)
- [OWASP Guide for Securely Using Third-Party MCP Servers](https://genai.owasp.org/resource/cheatsheet-a-practical-guide-for-securely-using-third-party-mcp-servers-1-0/)
- [How to Build a Custom MCP Server with TypeScript - FreeCodeCamp](https://www.freecodecamp.org/news/how-to-build-a-custom-mcp-server-with-typescript-a-handbook-for-developers/)
- [Build an MCP server - Official Docs](https://modelcontextprotocol.io/docs/develop/build-server)
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Top 10 MCP Servers & Clients - DataCamp](https://www.datacamp.com/blog/top-mcp-servers-and-clients)
- [GitHub MCP Registry](https://github.blog/ai-and-ml/github-copilot/meet-the-github-mcp-registry-the-fastest-way-to-discover-mcp-servers/)
