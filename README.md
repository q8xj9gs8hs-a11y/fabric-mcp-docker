Docker Compose setup for Fabric AI with MCP server integration

## Quickest
1. Clone this repository
```
git clone https://github.com/q8xj9gs8hs-a11y/fabric-mcp-docker.git
cd fabric-mcp-docker
```
2. Pull the official fabric image
```
docker pull kayvan/fabric
```
3. Create the folder to store the configuration files, then go through fabric's setup
```
mkdir -p ~/.fabric-config

docker run --rm -it -v "${HOME}/.fabric-config:/root/.config/fabric" kayvan/fabric --setup
```
4. Start the project with `docker compose` (automatically builds the MCP server for you)
```
docker compose --project-name fabric-ai up -d
```
5. Configure your `mcp.json`
```json
{
  "mcpServers": {
    "fabric": {
      "url": "http://localhost:8000/message"
    }
  }
}
```
To stop the containers:
```
docker compose --project-name fabric-ai down
```
For consecutive startups, you can skip building `fabric-mcp`:
```
docker compose --project-name fabric-ai up -d --no-build
```

# Using fabric-mcp via `stdio`

If you'd rather use fabric-mcp with `--transport stdio`, you can change the `docker-compose.yml` and `mcp.json` like so:

First, create a container network:
```
docker network create fabric-network
```
Then, `docker-compose.yml` will be adjusted to just maintain `fabric-server` connected to `fabric-network`:
```yml
services:
  fabric-server:
    image: kayvan/fabric
    container_name: fabric-server
    volumes:
      - "${HOME}/.fabric-config:/root/.config/fabric"
    command: --serve --address 0.0.0.0:8080
    networks:
      - fabric-network
networks:
  fabric-network:
    driver: bridge
```
Your `mcp.json` will be set up as:
```json
{
  "mcpServers": {
    "fabric": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--network",
        "fabric-network",
        "-v",
        "${HOME}/.fabric-config:/root/.config/fabric",
        "-e",
        "FABRIC_BASE_URL",
        "fabric-mcp",
        "--transport",
        "stdio"
      ],
      "env": {
        "FABRIC_BASE_URL": "http://fabric-server:8080"
    }
  }
}
```

## LICENSE

MIT
