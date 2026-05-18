# Practice 10 Docker Commands

Build the production WAR first:

```bash
mvn -B -Pprod package -DskipTests
```

On this machine, Docker is installed on Windows. Start Docker Desktop before running the
Docker commands. If you want to run Docker from WSL Ubuntu 22.04, enable:

Docker Desktop -> Settings -> Resources -> WSL Integration -> Ubuntu-22.04.

If Docker prints `Access is denied` for `C:\Users\孙涵\.docker\config.json`, fix the
file permission or run Docker Desktop/PowerShell with the account that owns that file.

If image build fails with `proxyconnect tcp: dial tcp 127.0.0.1:7897: connect:
connection refused`, Docker Desktop is configured to use a local proxy that is not
running. Either start that proxy, or open Docker Desktop:

Settings -> Resources -> Proxies -> disable manual proxy or clear `127.0.0.1:7897`,
then Apply & Restart.

Build the local Docker image and run the three required containers:

```bash
docker compose -f docker-compose.practice10.yml up -d --build
docker ps --filter "name=teedy-container"
```

Open:

- http://localhost:8082
- http://localhost:8083
- http://localhost:8084

Push the same image to Docker Hub:

```bash
docker tag teedy-practice10:local <dockerhub-username>/teedy-app:latest
docker login
docker push <dockerhub-username>/teedy-app:latest
```

Stop the local demo containers:

```bash
docker compose -f docker-compose.practice10.yml down
```
