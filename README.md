# Easy Installation 

```sh
# Clone this repository
git clone https://github.com/minato-gh/fabric-mcp-docker.git
cd fabric-mcp-docker

# Start both services
docker compose up -d
```

Configure your `mcp.json`:

```json
{
  "mcpServers": {
    "fabric": {
      "url": http://localhost:8000/message
  }
}
```

# Manual Installation 

Create a network for container isolation:

```
docker network create fabric-network
```

Pull the image:

```sh
# Pre-built image
docker pull minatogh/fabric:latest

# Or build manually
docker build -t minatogh/fabric -f scripts/docker/Dockerfile https://github.com/minato-gh/Fabric.git
```

Prepare the container:

```sh
# This is the directory that will be bind mounted onto the fabric container
mkdir -p "${HOME}/.fabric-config"

# Proceed with installing the patterns and strategies, and choosing a vendor and default model
docker run -it --rm -v "${HOME}/.fabric-config:/home/appuser/.config/fabric" minatogh/fabric --setup
```

Run the container:

```sh
# Listens at the container's port 8080 on all network interfaces in the background
docker run --rm -d \
        --network fabric-network \
        --name fabric \
        -v "${HOME}/.fabric-config:/home/appuser/.config/fabric" \
        minatogh/fabric --serve --address 0.0.0.0:8080
```

Pull the MCP server image:

```sh
# Pre-built image
docker pull minatogh/fabric-mcp:latest

# Or build manually
docker build -t minatogh/fabric-mcp -f docker/Dockerfile https://github.com/minato-gh/fabric-mcp.git
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

The MCP server will access the host `fabric`, via connection on the same network, at port `8080`.

## Run `fabric-mcp` as `http`

Runs via `stdio` by default, run the container yourself and alter your `mcp.json` accordingly:

```sh
docker run --rm -d \
        --network fabric-network \
        --name fabric-mcp \
        -p 8000:8000 \
        -v "${HOME}/.fabric-config:/home/appuser/.config/fabric" \
        -e FABRIC_BASE_URL=http://fabric:8080 \
        minatogh/fabric-mcp --transport http --host 0.0.0.0 --port 8000
```

```json
{
  "mcpServers": {
    "fabric": {
      "url": http://localhost:8000/message
  }
}
```

This exposes `fabric-mcp`'s port `8000`, which runs as a server in the background listening to all network interfaces, to the host machine's port `8000`. This is where your MCP client will reach it at, rather than running the container itself.

# LICENSE

MIT
