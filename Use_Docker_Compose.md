If your development environment requires a database for test data, or another
service running in another container, docker-compose can be used to orchestrate
multiple containers when the devcontainer is launched. Official
[documentation](https://code.visualstudio.com/docs/remote/devcontainerjson-reference#_docker-compose-specific-properties).

The docker-compose files can 

```json
  "dockerComposeFile": [
    "./docker-compose.yaml",
    "../docker-compose.yaml"
  ],
  "runServices": [
    "DevContainer",
    "Database"
  ],
  // The service to connect to for development
  "service": "DevContainer",
```

`./docker-compose.yaml`
```yaml
version: "3.3"
services:
  DevContainer:
    container_name: DevContainer
    # If you need to make changes to the container image,
    # uncomment the lines below and comment the image field
    # build:
      # context: ./
      # dockerfile: .devcontainer/Dockerfile
    # image: devcontainer:dev
    image: <registry/image:tag>
    volumes:
      - type: bind
        source: .
        target: /workspace
        consistency: cached
    ports:
      - "8080:8080"
      - "8443:8443"
```
`../docker-compose.yaml`
```yaml
version: "3.4"
services:
  ServiceA:
    container_name: ServiceA
    build:
      context: ./
      dockerfile: ./service-a/service-a.dockerfile
      target: production
    image: server:production
    ports:
      - "443:443"
    environment:
      - NODE_ENV=production

  Database:
    container_name: Database
    build:
      context: ./
      dockerfile: ./database/Database.dockerfile
      target: production
    image: database:production
    ports:
      - "5432:5432"
    volumes:
      - type: volume
        source: dbdata
        target: /var/lib/postgresql/data

networks:
  default:
    name: project-network

volumes:
  dbdata:
```