# Jenkins Practice 9-10 Checklist

Jenkins is available at:

```text
http://localhost:8085
```

## Practice 9

Create a Jenkins Pipeline job for this repository and use `Jenkinsfile` from SCM.

The pipeline contains these CI stages:

```text
Clean -> Compile -> Test -> PMD -> JaCoCo -> Javadoc -> Site -> Package
```

After the build, show these items in Jenkins:

- Pipeline stage result
- Test result from `**/target/surefire-reports/*.xml`
- Binary artifacts from `**/target/**/*.jar` and `**/target/**/*.war`
- Site documentation from `**/target/site/**/*.*`

## Practice 10

Before running the Docker stages, make sure:

- Docker Desktop is running.
- Jenkins can execute `docker version`.
- Docker Desktop proxy is not pointing to a stopped proxy such as `127.0.0.1:7897`.
- Jenkins has a username/password credential for Docker Hub.

Use these build parameters:

```text
DOCKER_IMAGE=<dockerhub-username>/teedy-app
DOCKERFILE=Dockerfile.practice10
DOCKER_CREDENTIALS_ID=dockerhub_credentials
RUN_DOCKER_STAGES=true
PUSH_DOCKER_IMAGE=true
```

The Docker stages do the following:

```text
Build Docker Image -> Push Docker Image -> Run Three Containers
```

The last stage recreates:

```text
teedy-container-8082 -> http://localhost:8082
teedy-container-8083 -> http://localhost:8083
teedy-container-8084 -> http://localhost:8084
```

Useful commands for local verification:

```bash
docker ps --filter "name=teedy-container"
docker images | grep teedy
```
