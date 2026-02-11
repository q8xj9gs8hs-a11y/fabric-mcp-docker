Docker Compose setup for Fabric AI with MCP server integration

# Setup

## Quickest
1. Clone this repository
```
git clone https://github.com/q8xj9gs8hs-a11y/fabric-mcp-docker.git
cd fabric-mcp-docker
```
2. Let `docker compose` build everything for you
```
docker compose --project-name fabric-ai up -d
```
3. Configure your `mcp.json`
```json
{
  "mcpServers": {
    "fabric": {
          "url": "http://localhost:8000/message"
    }
  }
}
```
4. To stop the containers
```
docker compose --project-name fabric-ai down
```
## Manual Build
1. Clone the fabric-mcp repository
```
git clone https://github.com/q8xj9gs8hs-a11y/fabric-mcp.git
cd fabric-mcp
```
2. Build the Dockerfile
```
docker build -t fabric-mcp -f docker/Dockerfile .
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
    command: --transport http --host 0.0.0.0 --port 8000
    depends_on:
      - fabric-server
```
Notice that one difference from the Quickest method is that we explicitly use the `fabric-mcp` image that we built earlier during step 2:
```yml
fabric-mcp:
  image: fabric-mcp
```
Building manually also allows you to hardcode the transport method, host, and/or port in the Dockerfile before building:
```dockerfile
...

ENTRYPOINT ["fabric-mcp", "--transport", "http", "--host", "0.0.0.0", "--port 8000"]
```
and
```dockerfile
...

ENTRYPOINT ["fabric", "--serve", "--address", "0.0.0.0:8080"]
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
* Automatic creation of an isolated container network called `fabric-ai_default`, or more generally, `<project name>_default`
* Ease of creation and destruction of both containers simultaneously
