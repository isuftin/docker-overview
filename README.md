### [What is Docker?](https://opensource.com/resources/what-docker)

Docker is a system for deploying applications or services in a way that tries to be
as agnostic as possible to the host that it's being deployed on. [The main differences
between a virtual machine and Docker](https://www.backblaze.com/blog/vm-vs-containers/) is that
a Virtual Machine, when created, have a fully running operating system along with
its own memory management, disk space, device drivers and runs on top of the host OS's
hypervisor (or a virtual machine monitor). Each VM is set to take up x memory, y
CPUs and z disk space from the  host. Each VM may be considered its own emulated
machine running on the host. In contrast, Docker containers run on the host's Docker
engine. Each container running on a Docker engine share the host's kernel and typically
the binaries and libraries on the host as well. Containers launch almost immediately
whereas virtual machines need to boot the OS of the guest. Containers should be
considered single-purpose. By that I mean they should only run a single service
(web server, messaging queue, database, etc). A VM tends to be used as a general-purpose
guest OS, running an application stack all in one VM or perhaps the VM is used as
a development station for a developer.

Docker mainly consists of the following parts:

- [Docker Engine](https://docs.docker.com/engine/): The running process on the host machine that takes manages the
  running Docker containers, the local Docker registry of images and is what the
  Docker commandline utility communicates with when you run `docker` commands.
  The Docker engine and the commandline client typically get installed in a single
  package on the operating system.

- [Docker Commandline](https://docs.docker.com/engine/reference/commandline/cli/):
  When installing Docker, the user has access to use the Docker cli via the
  `docker` command. This is what issues commands to the Docker engine running on
  the host. In some instances, you can also use the Docker cli to communicate to
  remote Docker engines running on remote hosts.

- [Dockerfile](https://docs.docker.com/engine/reference/builder/): The set of
  commands that are run top-down that define the application that is deployed
  into the running Docker container as well as supporting packages needed to run
  the application

- [Docker Image](https://docs.docker.com/engine/reference/commandline/images/):
  The frozen binary that represents the result of building the Dockerfile.
  The Docker image, once built, sits in your local Docker repository and can also
  be pushed to remote repositories using the `docker push` command. The image is known
  to the Docker engine by a unique hash tat gets created at the tail end of the build
  process. You are able to attribute a human readable name to a Docker image. A
  custom Docker image name consists of an image name (hello-world) and a version
  (0.0.1, 0.0.2-development, latest).

- [Docker Container](https://www.docker.com/resources/what-container): If a Docker
  image can be considered an executable file located on your file system, a Docker
  container may be considered the process that runs when you execute the executable.
  If the Docker image is a recipe, the Docker container is the cake. You may have
  as many running containers (instances of a Docker image) as your system can handle.
  And of course you may also launch as many containers of as many images that are in
  your registry as your system can handle.  

- [Docker Registry](https://docs.docker.com/registry/introduction/) (local):
  A cache of Docker images held locally on the host system that
  the Docker engine runs on. The Docker engine manages the local Docker registry
  and images built locally automatically end up in the local Docker regitry.

- Remote Docker Registry: Docker can also use remote Docker repositories to push
  images to and pull images from. An example being DockerHub. Other examples include
  Artifactory and GitLab

Some Docker tooling includes:

- [Docker Compose](https://docs.docker.com/compose/overview/): Using a YAML file,
  Docker Compose allows you to configure how one or a set of containers should be launched
  and largely replaces having to type out absurdly long `docker run` commands. Check
  out the [list of features](https://docs.docker.com/compose/overview/#features). After
  experimenting with docker via the commandline, you will probably quickly switch over
  to docker-compose to save your sanity. Docker compose acts as a wrapper for the Docker
  commandline.

- [Docker Machine](https://docs.docker.com/machine/overview/): Earlier in this document
  I stated that you're able to run the Docker commandline against a remote Docker engine.
  Docker for Mac and Windows are a work in progress. The Docker engine runs natively in
  Linux against the Linux kernel. As a result, if you are on MacOS or in Windows, you
  may consider using Docker Machine. Docker Machine will create 1..n virtual machines
  using VirtualBox, VMWare and other virtualization solutions. The virtual machines are
  a very slim build of [CoreOS](https://coreos.com/). The VMs have Docker pre-installed
  and when Docker Machine finishes booting a new VM, your Docker commandline client actually
  then issues commands to the VM running on your host. This can be very convenient but takes
  some getting used to if you're running multiple Docker Machine VMs at once (a use cause
    would be clustering Docker engines to form a Docker swarm)

- [Docker Swarm](https://docs.docker.com/engine/swarm/): Not necessarily an external aspect of
Docker, typically Docker runs as a single engine running multiple containers on a single host.
There is, however, Docker Swarm. When Docker is running in swarm mode, multiple Docker engines
from multiple Docker hosts network together to form a cluster of manager and worker nodes. Instead
of running a Docker container on the local host's Docker engine, you issue commands to the Docker
manager cluster to launch a Docker service (basically a definition of a Docker container and
  associated configuration). The managers take care of figuring out on which Docker engine that
service will be launched, monitors the health of the service, balances the service among the hosts,
scales the service as needed and removes it when told to do so.  This is an advanced topic for
Docker. Similar strategies are provided by Kubernetes, AWS ECS, EKS and AWS Fargate
