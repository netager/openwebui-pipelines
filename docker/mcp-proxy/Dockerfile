FROM python:3.11-slim

WORKDIR /app

# Install required packages
RUN pip install mcpo uv

# Create config directory
RUN mkdir -p /app/config

# Create MCP config file
COPY mcp-config.json /app/config/mcp-config.json

# Run using the config file
CMD ["mcpo", "--host", "0.0.0.0", "--port", "8000", "--config", "/app/config/mcp-config.json"]
