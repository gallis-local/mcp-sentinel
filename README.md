# mcp-sentinel

A Python-based MCP server using FastMCP library that provides integration with Microsoft Sentinel using Azure Identity Authentication.

## Overview

This project implements an MCP server that enables:

- Running KQL queries against Microsoft Sentinel
- Listing All Sentinel Tables
- Fetching a specific Sentinel Table for metadata
- Fetching specific Sentinel Table schema

The server acts as a bridge between development environments and Microsoft Sentinel, allowing for testing and execution of KQL queries. It can be built for SSE or STDIO based on the launch env variable of `MCP_CONNECTION_TYPE` within the FastMCP configuration.

## Features

- **Sentinel Integration**: Execute KQL queries against your Sentinel workspace
- **Authentication Support**: Multiple authentication methods including interactive browser, client secret, and managed identity

## Prerequisites

- Python 3.8+
- Microsoft Sentinel workspace
- Appropriate Azure permissions for Sentinel

## Installation

### Option 1: Install from source (development mode)

1. Clone the repository:
   ```
   git clone https://github.com/gallis-local/SentinelMCPServer.git
   cd SentinelMCPServer
   ```

2. Install the package in development mode:
   ```
   pip install -e .
   ```

   If experencing an install issue, ensure the Windows Registry key for extended python paths is enabled

```
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

### Option 2: Docker STDIO

1. Run the container in STDIO mode:
   ```
   docker run -i --rm \
     -e SENTINEL_SUBSCRIPTION_ID=<your-subscription-id> \
     -e SENTINEL_RESOURCE_GROUP=<your-resource-group> \
     -e SENTINEL_WORKSPACE_NAME=<your-workspace-name> \
     -e SENTINEL_WORKSPACE_ID=<your-workspace-id> \
     -e AUTHENTICATION_TYPE=client_secret \
     -e AZURE_TENANT_ID=<your-tenant-id> \
     -e AZURE_CLIENT_ID=<your-client-id> \
     -e AZURE_CLIENT_SECRET=<your-client-secret> \
     ghcr.io/gallis-local/mcp-sentinel:main
   ```

### Option 3: Docker Compose SSE

1. Create a `.env` file with your environment variables:
   ```
   SENTINEL_SUBSCRIPTION_ID=<your-subscription-id>
   SENTINEL_RESOURCE_GROUP=<your-resource-group>
   SENTINEL_WORKSPACE_NAME=<your-workspace-name>
   SENTINEL_WORKSPACE_ID=<your-workspace-id>
   AUTHENTICATION_TYPE=client_secret
   AZURE_TENANT_ID=<your-tenant-id>
   AZURE_CLIENT_ID=<your-client-id>
   AZURE_CLIENT_SECRET=<your-client-secret>
   ```

2. Start the server using Docker Compose:
   ```
   docker-compose up -d
   ```

3. The MCP server will be available at `http://localhost:8000/sse`

## Usage

### Environment Variables

The following environment variables can be configured:

| Variable | Description | Default Value | Required |
|----------|-------------|---------------|----------|
| `SENTINEL_SUBSCRIPTION_ID` | Azure subscription ID containing the Sentinel workspace | - | Yes |
| `SENTINEL_RESOURCE_GROUP` | Resource group name containing the Sentinel workspace | - | Yes |
| `SENTINEL_WORKSPACE_NAME` | Name of your Sentinel workspace | - | Yes |
| `SENTINEL_WORKSPACE_ID` | Unique identifier of your Sentinel workspace | - | Yes |
| `AUTHENTICATION_TYPE` | Authentication method (`interactive`, `client_secret`, or defaults to managed identity) | `interactive` | No |
| `AZURE_TENANT_ID` | Azure tenant ID (required for `client_secret` auth) | - | For client_secret auth |
| `AZURE_CLIENT_ID` | Application/client ID (required for `client_secret` auth) | - | For client_secret auth |
| `AZURE_CLIENT_SECRET` | Secret value for authentication (required for `client_secret` auth) | - | For client_secret auth |
| `MCP_CONNECTION_TYPE` | Connection type for MCP server (`stdio` or `sse`) | `stdio` | No |

### Starting the Server

After installation, run the MCP server using the command:

```
sentinel_mcp
```

You can also run directly from the repository:

```
python -m sentinel_mcp
```

### Configure VSCode

stdio python method:
```
"mcp": {
   "servers": {          
      "sentinel": {
            "command": "python3",
            "args": ["-m", "sentinel_mcp"],
            "env": {
               "SENTINEL_SUBSCRIPTION_ID": "",
               "SENTINEL_RESOURCE_GROUP": "",
               "SENTINEL_WORKSPACE_ID": "",
               "SENTINEL_WORKSPACE_NAME": ""
            }
      }
   }
}
```

Docker local stdio method:
```json
"mcp": {
   "servers": {          
      "sentinel": {
         "command": "docker",
         "args": [
             "run",
             "-i",
             "--rm",
             "-e", "SENTINEL_SUBSCRIPTION_ID",
             "-e", "SENTINEL_RESOURCE_GROUP",
             "-e", "SENTINEL_WORKSPACE_ID",
             "-e", "SENTINEL_WORKSPACE_NAME",
             "-e", "AUTHENTICATION_TYPE",
             "-e", "AZURE_TENANT_ID",
             "-e", "AZURE_CLIENT_ID",
             "-e", "AZURE_CLIENT_SECRET",
             "ghcr.io/gallis-local/mcp-sentinel:main"
         ],
         "env": {
             "SENTINEL_SUBSCRIPTION_ID": "",
             "SENTINEL_RESOURCE_GROUP": "",
             "SENTINEL_WORKSPACE_ID": "",
             "SENTINEL_WORKSPACE_NAME": "",
             "AUTHENTICATION_TYPE": "client_secret",
             "AZURE_TENANT_ID": "",
             "AZURE_CLIENT_ID": "",
             "AZURE_CLIENT_SECRET": ""
         }
      }
   }
}
```


SSE remote method (docker compose):

```json
"mcp": {
   "servers": {      
      "sentinel": {
            "type": "sse",
            "url": ["http://localhost:8000/sse"]
      }
   }
}
```


### Available Tools

The MCP server provides the following tools:

1. **sentinel_run_query**: Execute KQL queries in Sentinel
2. **sentinel_get_tables**: List all available tables in your Sentinel workspace
3. **sentinel_get_table_schema**: Fetch the schema for a specific Sentinel table
4. **sentinel_get_table_by_name**: Get information for a specific table by name

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request
