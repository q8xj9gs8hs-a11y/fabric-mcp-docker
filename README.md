# Installation

Create a network for container isolation:

```
docker network create fabric-network
```

Install patterns and configure vendor/model of choice:

```sh
mkdir -p "${HOME}/.fabric-config"

docker run -it --rm -v "${HOME}/.fabric-config:/home/appuser/.config/fabric" minatogh/fabric --setup
```

Run the container using a pre-built image:

```sh
docker run --rm -d
        --network fabric-network \
        --name fabric \
        -v "${HOME}/.fabric-config:/home/appuser/.config/fabric" \
        minatogh/fabric --serve --address 0.0.0.0:8080
```

If you want to build the images manually:

```sh
# build minatogh/fabric
docker build -t minatogh/fabric -f scripts/docker/Dockerfile https://github.com/minato-gh/Fabric.git

# build minatogh/fabric-mcp
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

## Run `fabric-mcp` as `http`

Runs via `stdio` by default, change its `docker run` command and alter your `mcp.json` accordingly:

```sh
docker run --rm -d
        --network fabric-network \
        --name fabric-mcp \
        -p 8000:8000 \
        -v "${HOME}/.fabric-config:/home/appuser/.config/fabric" \
        -e FABRIC_BASE_URL=http://fabric:8080
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
