### [What is Docker?](https://opensource.com/resources/what-docker)

Docker is a system for deploying applications or services in a way that tries to be
as agnostic as possible to the host that it's being deployed on.

Docker mainly consists of [xxxx] parts:

- Docker Engine: The running process on the host machine that takes manages the
  running Docker containers, the local Docker registry of images and is what the
  Docker commandline utility communicates with when you run `docker` commands
  
- Dockerfile: The set of commands that are run top-down that define the application
  that is deployed into the running Docker container as well as supporting packages
  needed to run the application

- Docker Image: The frozen binary that represents the result of building the Dockerfile.
  The Docker image, once built, sits in your local Docker repository and can also
  be pushed to remote repositories using the `docker push` command.

- Docker Repository: A cache of Docker images held locally on the host system that
  the Docker engine runs on. Docker can also use remote Docker repositories to push
  images to and pull images from. An example being DockerHub. Other examples include
  Artifactory and GitLab
