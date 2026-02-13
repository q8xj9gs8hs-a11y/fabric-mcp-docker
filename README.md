Docker Compose setup for Fabric AI with MCP server integration

### What is Fabric?

* **Fabric’s Purpose:** To address the integration problem of AI by organizing and managing AI prompts for real-world tasks.
* **Fabric’s Functionality:** Allows users to create, collect, and organize AI solutions (prompts) for use in various tools or directly through Fabric’s command-line interface.
* **Fabric’s Approach:** Breaks down problems into smaller components and applies AI to each component, offering a wide range of pre-built prompts (Patterns) for various life and work activities.

# Setup

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
## Manual Build

Building manually allows you to hardcode the transport method, host, and/or port in the Dockerfile without a `CMD` before building:
```dockerfile
...

ENTRYPOINT ["fabric-mcp", "--transport", "http", "--host", "0.0.0.0", "--port 8000"]
```
Or simply just change the default command:
```dockerfile
...

CMD ["--transport", "http", "--host", "<host>", "--port", "<port>"]
```
1. Clone this repository
```
git clone https://github.com/q8xj9gs8hs-a11y/fabric-mcp-docker.git
cd fabric-mcp-docker
```
2. Build the Dockerfile
```
docker build -t fabric-mcp .
```
3. Pull the official fabric image
```
docker pull kayvan/fabric
```
4. Create the folder to store the configuration files, then go through fabric's setup
```
mkdir -p ~/.fabric-config

docker run --rm -it -v "${HOME}/.fabric-config:/root/.config/fabric" kayvan/fabric --setup
```
5. Create the docker-compose.yml
```yml
services:
  fabric-server:
    image: kayvan/fabric
    container_name: fabric-server
    volumes:
      - "${HOME}/.fabric-config:/root/.config/fabric"
    command: --serve --address 0.0.0.0:8080

  fabric-mcp:
    image: fabric-mcp
    container_name: fabric-mcp
    environment:
      - FABRIC_BASE_URL=http://fabric-server:8080
    ports:
      - "8000:8000"
    volumes:
      - "${HOME}/.fabric-config:/root/.config/fabric"
    depends_on:
      - fabric-server
```
6. Run `docker compose`
```
docker compose --project-name fabric-ai up -d
```
7. Configure your `mcp.json`
```json
{
  "mcpServers": {
    "fabric": {
      "url": "http://localhost:8000/message"
    }
  }
}
```
Notice that one difference from the Quickest method is that we explicitly use the `fabric-mcp` image that we built earlier during step 2:
```yml
fabric-mcp:
  image: fabric-mcp
```
# Notes

The `docker-compose.yml` is the equivalent of running these two containers:
```
docker run --rm -d --name fabric-server -v "${HOME}/.fabric-config:/root/.config/fabric" kayvan/fabric --serve --address 0.0.0.0:8080

docker run --rm -d --name fabric-mcp -e FABRIC_BASE_URL=http://fabric-server:8080 -p 8000:8000 fabric-mcp --transport http --host 0.0.0.0 --port 8000
```
Their corresponding entrypoints are:
```
docker image inspect kayvan/fabric | grep -A 2 '"Entrypoint"'

docker image inspect fabric-mcp | grep -A 2 '"Entrypoint"'
```
```json
"Entrypoint": [
  "fabric"
],

"Entrypoint": [
  "fabric-mcp"
],
```
The advantages of using `docker compose` instead:
* Automatic creation of an isolated container network called `fabric-ai_default`, or more generally, `<project name>_default`.
* Ease of creation and destruction of both containers simultaneously.
* Docker's mature networking automatically manages hostname resolution.

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
