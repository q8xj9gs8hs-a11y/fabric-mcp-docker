# Installation

Create a network for container isolation:

```
docker network create fabric-network
```

Create configuration files for the bind mount:
```
mkdir -p "${HOME}/.fabric-config"

docker run -it --rm -v "${HOME}/.fabric-config:/home/appuser/.config/fabric" minatogh/fabric --setup
```

Run the container from the pre-built image:

```
docker run --rm -d --network fabric-network --name fabric -v "${HOME}/.fabric-config:/home/appuser/.config/fabric" minatogh/fabric --serve --address 0.0.0.0:8080
```

Add the MCP server to your `mcp.json`:

```json
{
  "mcpServers": {
    "fabric": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--network", "fabric-network",
        "-v", "${HOME}/.fabric-config:/home/appuser/.config/fabric",
        "-e", "FABRIC_BASE_URL",
        "minatogh/fabric-mcp"
      ],
      "env": {
        "FABRIC_BASE_URL": "http://fabric:8080"
    }
  }
}
```

# Manual Installation
