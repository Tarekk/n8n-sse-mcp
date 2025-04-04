# n8n-sse-mcp

A minimalistic example for using Python-based MCP (Model Context Protocol) servers within a self-hosted n8n environment via SSE (Server-Sent Events).

## Overview

This repository solves a common issue when using Python-based MCP servers with n8n: the default n8n Docker image doesn't include a Python runtime. We address this by:

1. Setting up n8n with proper configuration to allow the MCP node
2. Creating a separate Python-based container using Supergateway to serve Python-based MCP servers via SSE
3. Providing a complete Docker Compose setup for easy deployment

As a practical example, this repository is configured to use the [MotherDuck MCP server](https://github.com/motherduckdb/mcp-server-motherduck), which allows you to run SQL queries against MotherDuck databases directly from n8n workflows.

## Prerequisites

- Docker and Docker Compose
- Basic knowledge of n8n
- A domain name (for the included Traefik setup)

## Quick Start

1. Clone this repository

   ```
   git clone https://github.com/yourusername/n8n-sse-mcp.git
   cd n8n-sse-mcp
   ```

2. Set up environment variables
   Create a `.env` file with the following variables:

   ```
   DOMAIN_NAME=yourdomain.com
   SUBDOMAIN=n8n
   SSL_EMAIL=your-email@example.com
   GENERIC_TIMEZONE=UTC
   ```

3. Customize the Supergateway command in `docker-compose.yaml`
   Replace `<YOUR_TOKEN>` with your actual MotherDuck token, or modify the command to use a different Python-based MCP server.

4. Create external volumes

   ```
   docker volume create traefik_data
   docker volume create n8n_data
   ```

5. Start the services

   ```
   docker-compose up -d
   ```

6. Access n8n at `https://n8n.yourdomain.com` (or whatever subdomain you configured)

## Components

### 1. n8n Container

The n8n container is configured with:

- Traefik for SSL/TLS termination
- Environment variable `N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true` to enable the MCP node as a tool
- Volume mounts for persistent data

### 2. Supergateway Container

The Supergateway container:

- Is built with Python, Node.js, and npm
- Includes the `uv` package manager for faster Python package installation
- Runs the Supergateway server that translates between stdio and SSE protocols
- Exposes port 8000 for SSE connections

## Example: Using the MotherDuck MCP Server

This repository includes a preconfigured setup for the MotherDuck MCP server. The `docker-compose.yaml` file is already configured with:

```yaml
command:
  - '--stdio'
  - 'uvx mcp-server-motherduck --db-path md: --motherduck-token <YOUR_TOKEN>'
  - '--port'
  - '8000'
```

To use it:

1. Replace `<YOUR_TOKEN>` with your actual MotherDuck token
2. Start the services as described in the Quick Start section
3. In your n8n workflow, you can now execute SQL queries against your MotherDuck databases

Example SQL query using the Execute Tool operation:

```sql
SELECT * FROM your_database.your_table LIMIT 10
```

For more information on available parameters and tools, visit the [MotherDuck MCP server repository](https://github.com/motherduckdb/mcp-server-motherduck).

## Adding the MCP Node to n8n

After your n8n instance is running:

1. Navigate to Settings > Community nodes
2. Click "Install a community node"
3. Enter `n8n-nodes-mcp` and install
4. Restart n8n when prompted

For detailed instructions, visit the [official MCP community node page](https://www.npmjs.com/package/n8n-nodes-mcp).

## Configuring MCP in n8n

1. In your n8n workflow, add a new "MCP Client" node
2. Select "Server-Sent Events (SSE)" as the Connection Type
3. Create new credentials with the following settings:
   - SSE URL: `http://supergateway:8000/sse`
4. Use the MCP node operations like "List Tools", "Execute Tool", etc.

## Example Workflow

1. Create a new workflow in n8n
2. Add an MCP Client node
3. Configure it to use SSE at `http://supergateway:8000/sse`
4. Select "List Tools" operation to see available tools
5. Add another MCP Client node to execute a specific tool

## Customization

### Using Different Python-based MCP Servers

Modify the `command` section in the `supergateway` service in `docker-compose.yaml`:

```yaml
command:
  - '--stdio'
  - 'uvx mcp-server-your-server --your-parameters'
  - '--port'
  - '8000'
```

### Adding Environment Variables

To pass environment variables to the Python MCP server, add them to the `supergateway` service:

```yaml
environment:
  - YOUR_API_KEY=your-key-here
```

## Troubleshooting

- **MCP Node Not Available as Tool**: Ensure `N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true` is set in the n8n environment
- **Supergateway Connection Issues**: Check that port 8000 is exposed and the container is running with `docker-compose ps`
- **Python Package Installation Failures**: Modify the Dockerfile to install additional dependencies

## Resources

- [MCP Community Node](https://www.npmjs.com/package/n8n-nodes-mcp)
- [Supergateway](https://github.com/supercorp-ai/supergateway)
- [n8n Documentation](https://docs.n8n.io)

## License

MIT
