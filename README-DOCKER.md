# Docker Setup for Redshift MCP Server

This Docker setup runs the `redshift_mcp_server` directly in a containerized environment using an efficient multi-stage build process.

## What This Setup Does

- **Redshift MCP Server**: Provides tools and resources for interacting with Amazon Redshift databases
- **Direct Communication**: Uses stdio transport for direct MCP protocol communication
- **Efficient Build**: Multi-stage Docker build with optimal layer caching using uv

## Prerequisites

- Docker installed
- Access to an Amazon Redshift cluster
- Redshift connection credentials

## Quick Start

1. **Configure Environment Variables**
   
   Copy the example environment file and fill in your Redshift credentials:
   ```bash
   cp env.example .env
   # Edit .env with your actual Redshift connection details
   ```

2. **Build the Docker Image**
   
   ```bash
   docker build -t redshift-mcp-server .
   ```

3. **Run the Container**
   
   ```bash
   docker run --rm -i --env-file .env redshift-mcp-server
   ```

## Configuration

### Environment Variables

Set these in your `.env` file:

| Variable | Description | Default |
|----------|-------------|---------|
| `RS_HOST` | Redshift cluster endpoint | `localhost` |
| `RS_PORT` | Redshift port | `5439` |
| `RS_USER` | Redshift username | `awsuser` |
| `RS_PASSWORD` | Redshift password | (empty) |
| `RS_DATABASE` | Redshift database name | `dev` |
| `RS_SCHEMA` | Redshift schema | `public` |

## Usage Examples

### Using with Claude Desktop

Add this to your Claude Desktop configuration:

```json
{
  "mcpServers": {
    "redshift": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "--env-file", "/absolute/path/to/your/.env",
        "redshift-mcp-server"
      ]
    }
  }
}
```

### Using with Environment Variables Directly

```bash
docker run --rm -i \
  -e RS_HOST=your-cluster.region.redshift.amazonaws.com \
  -e RS_PORT=5439 \
  -e RS_USER=your_username \
  -e RS_PASSWORD=your_password \
  -e RS_DATABASE=your_database \
  -e RS_SCHEMA=public \
  redshift-mcp-server
```

### Using with uv directly (no Docker)

If you prefer to run locally without Docker:

```bash
# Install dependencies
uv sync

# Run the server
uv run python src/redshift_mcp_server/server.py
```

## Docker Commands

### Build the Image

```bash
# Build the image
docker build -t redshift-mcp-server .

# Build with no cache (if you need a clean rebuild)
docker build --no-cache -t redshift-mcp-server .
```

### Run the Container

```bash
# Run with .env file
docker run --rm -i --env-file .env redshift-mcp-server

# Run with environment variables
docker run --rm -i \
  -e RS_HOST=your-host \
  -e RS_USER=your-user \
  -e RS_PASSWORD=your-password \
  -e RS_DATABASE=your-database \
  redshift-mcp-server

# Run in background (for testing)
docker run -d --name redshift-mcp --env-file .env redshift-mcp-server

# Stop background container
docker stop redshift-mcp
docker rm redshift-mcp
```

### Debugging

```bash
# View container logs
docker logs redshift-mcp

# Connect to running container
docker exec -it redshift-mcp sh

# Run container with shell for debugging
docker run --rm -it --env-file .env redshift-mcp-server sh
```

## Features Available

The containerized setup provides access to all Redshift MCP server features:

### Resources
- `rs:///schemas` - List all schemas
- `rs:///{schema}/tables` - List tables in a schema
- `rs:///{schema}/{table}/ddl` - Get table DDL
- `rs:///{schema}/{table}/statistic` - Get table statistics

### Tools
- `execute_sql` - Execute SQL queries
- `analyze_table` - Analyze tables for statistics
- `get_execution_plan` - Get query execution plans

## Docker Build Features

This setup uses an advanced multi-stage Docker build with several optimizations:

### Build Efficiency
- **Multi-stage build**: Separates dependency installation from final runtime image
- **Layer caching**: Dependencies are cached separately from application code
- **Bytecode compilation**: Python bytecode is pre-compiled for faster startup
- **Alpine Linux**: Smaller final image size

### Security
- **Minimal image**: Only includes necessary runtime dependencies
- **No development tools**: Production image excludes development dependencies

### Performance
- **uv integration**: Fast Python package management and virtual environment
- **Optimized layers**: Dependencies installed separately for better caching
- **Frozen lockfile**: Uses exact dependency versions from uv.lock

## Troubleshooting

### Common Issues

1. **Connection Refused**: Make sure your Redshift cluster is accessible and credentials are correct
2. **Permission Denied**: Ensure your Redshift user has necessary permissions
3. **Container Won't Start**: Check Docker logs with `docker logs <container-name>`
4. **Build Failures**: Ensure uv.lock and pyproject.toml are present and up to date

### Debugging Commands

```bash
# Check if image was built correctly
docker images | grep redshift-mcp-server

# Run container with shell for debugging
docker run --rm -it --env-file .env redshift-mcp-server sh

# Test the server locally (no Docker)
uv run python src/redshift_mcp_server/server.py

# Rebuild image from scratch
docker build --no-cache -t redshift-mcp-server .

# Check container environment
docker run --rm --env-file .env redshift-mcp-server env
```

## Security Notes

- Never commit your `.env` file with real credentials
- Use IAM roles or temporary credentials when possible
- Restrict Redshift access to necessary schemas/tables only
- Consider using VPC and security groups to limit network access

## Development

For development, you can mount your source code to avoid rebuilding:

```bash
# Mount source code for development
docker run --rm -i --env-file .env \
  -v $(pwd)/src:/app/src \
  redshift-mcp-server
```

Or run directly with uv for faster iteration:

```bash
# Install in development mode
uv sync --dev

# Run with development dependencies
uv run python src/redshift_mcp_server/server.py
```

## About the Build Process

This Dockerfile follows best practices for Python containerization:

1. **Build Stage**: Uses `ghcr.io/astral-sh/uv:python3.13-alpine` for fast dependency resolution
2. **Runtime Stage**: Uses `python:3.13-alpine` for minimal production image
3. **Efficiency**: Leverages Docker build cache and uv's speed
4. **Simplicity**: Direct execution without complex startup scripts

The result is a fast, secure, and efficient containerized MCP server.

## Common Workflows

```bash
# Complete workflow: build and run
docker build -t redshift-mcp-server .
docker run --rm -i --env-file .env redshift-mcp-server

# Development workflow: build, run with volume mount
docker build -t redshift-mcp-server .
docker run --rm -i --env-file .env -v $(pwd)/src:/app/src redshift-mcp-server

# Testing workflow: run in background, check logs, stop
docker run -d --name test-mcp --env-file .env redshift-mcp-server
docker logs test-mcp
docker stop test-mcp && docker rm test-mcp
``` 