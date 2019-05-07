### Simple Hello World Example

In this example I want to show what a Dockerfile looks like and what goes into
creating a simple Hello World web server. We use this Dockerfile to build a Docker
image. This Docker image is then used to launch a Docker container which will run
a simple NGINX HTTP server that only has the purpose to return a very simple
HTML document.

Taking a look at the Dockerfile.1 file, on line 1 we see the [`FROM`](https://docs.docker.com/engine/reference/builder/#from)
keyword. This tells Docker which Docker image to use as a base image for the image
it's about to create. In short, every Docker image that gets created uses another
pre-existing Docker image to build on top of. In this case, I use the
[community Debian Stretch slim image](https://hub.docker.com/_/debian). This image
is a lightweight version of the Debian OS providing just the tooling I need in order
to install other software needed to run the simple NGINX server. If you look on
Dockerhub, you will notice that the "official" community images have lots of base
images to choose from, depending on your needs. There's also the availability of
regular Debian Stretch if you need it but I chose the "slim" variant as it's smaller
than the regular Debian Stretch image (55MB vs 101MB).

So at this point we can run the command `docker build -t hello-world:1 -f Dockerfile_1 .`
and see what happens:

```
$ docker build -t hello-world:1 -f Dockerfile_1 .
Sending build context to Docker daemon  4.096kB
Step 1/1 : FROM debian:stretch-slim
stretch-slim: Pulling from library/debian
27833a3ba0a5: Already exists
Digest: sha256:bade11bf1835c9f09b011b5b1cf9f7428328416410b238d2f937966ea820be74
Status: Downloaded newer image for debian:stretch-slim
 ---> c08899734c03
Successfully built c08899734c03
Successfully tagged hello-world:1
```

So what just happened? What did we tell Docker to do? `docker build` tells Docker
that we want to use a Dockerfile to create a Docker image. We also tell Docker that
we want to tag the resulting image as `hello-world:1`. This means that the image
name is "hello-world" at version 1. We don't necessarily have to provide a version
tag and Docker will instead provide its own "latest" tag. We can build over and over
with the same tag and the resulting image that gets created will just replace the
old one in the repository. We gave the `-f` flag to tell Docker that the Dockerfile
name is actually Dockerfile_1. Without this, Docker will look for a file named Dockerfile
by default. We also provided a dot (`.`). This tells Docker that the context for the
build is the current directory.

Docker sees that the Dockerfile only has one line in it (FROM) so it shows that it
is performing step 1 of 1, and that's pulling in the `debian:stretch-slim` Dockerhub
image. If, in your FROM line you just mention an image name and version, Docker will
automatically try Dockerhub to see if the image exists there and pull it. Otherwise,
you can give it a full URL path to an image (say if you have one in GitLab or Artifactory)
and it will go there to try to pull the image instead. Here it pulls the debian stretch-slim
image from Dockerhub and finishes by saying that it has successfully built image
c08899734c03 and tagged it as "hello-world:1". You might ask what the random set
of characters is. When Docker builds an image, each line in the Dockerfile creates
a "layer" which is then given a random hash to identify. Docker keeps these layers
cached and the resulting Docker image is simply the combination of all of these
layers. The resulting layer is c08899734c03 and we tag that hash as "hello-world:1".

So now we can see that image in our local Docker registry by typing `docker image ls`:

```
$ docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
debian                   stretch-slim        c08899734c03        5 weeks ago         55.3MB
hello-world              1                   c08899734c03        5 weeks ago         55.3MB
```

Notice that the image id for debian:stretch-slim and our image is the same and Docker
tells us both images were created 5 weeks ago and they're also the same size. Why
is that? Well, it's because in our Dockerfile, we don't actually *do* anything
on top of the debian image. Remember when I mentioned that Docker images are just
sets of layers? Because I've not added any layers on top of the base Debian image,
the last layer in our image is still the base debian layer, so the hash remains
the same.

Perhaps this is better exemplified if we do something in our image. Let's try
adding a [`LABEL`](https://docs.docker.com/engine/reference/builder/#label) to our
image. This is just adding some metadata as a key/value pair to our image that we
can use in other processes to more easily identify our image.

Take a look at Dockerfile_2 on line 3 to see that I've added myself as a maintainer
to this image. Let's try to build this via `docker build -t hello-world:2 -f Dockerfile_2 .`

```
$ docker build -t hello-world:2 -f Dockerfile_2 .
Sending build context to Docker daemon  8.704kB
Step 1/2 : FROM debian:stretch-slim
 ---> c08899734c03
Step 2/2 : LABEL maintainer="isuftin@usgs.gov"
 ---> Running in f0ad8e08314a
Removing intermediate container f0ad8e08314a
 ---> 238815bfecbe
Successfully built 238815bfecbe
Successfully tagged hello-world:2
```

We can see that we've added a new step, which includes a new layer added to the
resulting image. First it uses the image we already have (c08899734c03) and then
in step 2, adds a `LABEL`. In doing so, a new layer is added and a new hash is
created, shown by the line `Successfully built 238815bfecbe`. Now let's take a look
at our image listing:

```
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         2                   238815bfecbe        2 minutes ago       55.3MB
debian              stretch-slim        c08899734c03        5 weeks ago         55.3MB
hello-world         1                   c08899734c03        5 weeks ago         55.3MB
```

Now we see that the image `hello-world:2` is actually a separate image from
`debian:stretch-slim` and it was created two minutes ago. As we add more and more
commands in the Dockerfile, newer distinct images get created and added to our
Docker image registry.

So let's start setting up the HTTP server to serve our Hello World html file.

First, I want to make sure that my Docker image is up to date with the latest
security patches and packages at the time I build it. So my first real instruction
will be [`RUN`](https://docs.docker.com/engine/reference/builder/#run), which runs
a shell command during the Docker build. In Dockerfile_3, you can see I've added
`RUN apt-get update && apt-get upgrade -y`. This causes the Debian oS in the Docker
container to update it's package repository and upgrade any outdated packages. I add
the `-y` flag because the Docker builds are non-interactive and the flag tells apt
to assume "yes" to any questions it may have.

```
$ docker build -t hello-world:3 -f Dockerfile_3 .
Sending build context to Docker daemon  11.26kB
Step 1/3 : FROM debian:stretch-slim
 ---> c08899734c03
Step 2/3 : LABEL maintainer="isuftin@usgs.gov"
 ---> Using cache
 ---> 238815bfecbe
Step 3/3 : RUN apt-get update && apt-get upgrade -y && apt-get clean
 ---> Running in b3c076340107
Get:1 http://security-cdn.debian.org/debian-security stretch/updates InRelease [94.3kB]                                                                                                                  
Ign:2 http://cdn-fastly.deb.debian.org/debian stretch
InRelease                                                                                                                     
Get:4 http://security-cdn.debian.org/debian-security stretch/updates/main amd64 Packages [487 kB]
Get:3 http://cdn-fastly.deb.debian.org/debian stretch-updates InRelease [91.0 kB]
Get:5 http://cdn-fastly.deb.debian.org/debian stretch Release [118 kB]
Get:6 http://cdn-fastly.deb.debian.org/debian stretch-updates/main amd64 Packages [31.7 kB]
Get:7 http://cdn-fastly.deb.debian.org/debian stretch Release.gpg [2434 B]
Get:8 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 Packages [7082 kB]
Fetched 7907 kB in 5s (1496 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
Calculating upgrade...
The following packages will be upgraded:
  base-files libsystemd0 libudev1 tzdata
  4 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
  Need to get 748 kB of archives.
  After this operation, 2048 B of additional disk space will be used.
  Get:1 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 base-files amd64 9.9+deb9u9 [67.4 kB]
  Get:2 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libsystemd0 amd64 232-25+deb9u11 [281 kB]
  Get:3 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libudev1 amd64 232-25+deb9u11 [126 kB]
  Get:4 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 tzdata all 2019a-0+deb9u1 [273 kB]
  debconf: delaying package configuration, since apt-utils is not installed
  Fetched 748 kB in 1s (583 kB/s)
  (Reading database ... 6316 files and directories currently installed.)
  Preparing to unpack .../base-files_9.9+deb9u9_amd64.deb ...
  Unpacking base-files (9.9+deb9u9) over (9.9+deb9u8) ...
  Setting up base-files (9.9+deb9u9) ...
  Installing new version of config file /etc/debian_version ...
  (Reading database ... 6316 files and directories currently installed.)
  Preparing to unpack .../libsystemd0_232-25+deb9u11_amd64.deb ...
  Unpacking libsystemd0:amd64 (232-25+deb9u11) over (232-25+deb9u9) ...
  Setting up libsystemd0:amd64 (232-25+deb9u11) ...
  (Reading database ... 6316 files and directories currently installed.)
  Preparing to unpack .../libudev1_232-25+deb9u11_amd64.deb ...
  Unpacking libudev1:amd64 (232-25+deb9u11) over (232-25+deb9u9) ...
  Setting up libudev1:amd64 (232-25+deb9u11) ...
  (Reading database ... 6316 files and directories currently installed.)
  Preparing to unpack .../tzdata_2019a-0+deb9u1_all.deb ...
  Unpacking tzdata (2019a-0+deb9u1) over (2018i-0+deb9u1) ...
  Setting up tzdata (2019a-0+deb9u1) ...
  debconf: unable to initialize frontend: Dialog
  debconf: (TERM is not set, so the dialog frontend is not usable.)
  debconf: falling back to frontend: Readline
  debconf: unable to initialize frontend: Readline
  debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.24.1 /usr/local/share/perl/5.24.1 /usr/lib/x86_64-linux-gnu/perl5/5.24 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.24 /usr/share/perl/5.24 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base .) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
  debconf: falling back to frontend: Teletype

  Current default time zone: 'Etc/UTC'
  Local time is now:      Tue May  7 16:51:54 UTC 2019.
  Universal Time is now:  Tue May  7 16:51:54 UTC 2019.
  Run 'dpkg-reconfigure tzdata' if you wish to change it.

  Processing triggers for libc-bin (2.24-11+deb9u4) ...
  Removing intermediate container 7141fc7f4a7c
   ---> 9683ee152c82
  Successfully built 9683ee152c82
  Successfully tagged hello-world:3
```

We see that Debian has upgraded 4 packages and the final image hash is 194bab9d0479.
Now, if I view my Docker registry's images, I can see that image in there as well:

```
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         3                   9683ee152c82        49 seconds ago      75.9MB
hello-world         2                   238815bfecbe        15 minutes ago      55.3MB
debian              stretch-slim        c08899734c03        5 weeks ago         55.3MB
hello-world         1                   c08899734c03        5 weeks ago         55.3MB
```

Notice that my new image has grown over the base image by 20 megabytes. The reason
for this is because the new packages I've downloaded and installed took up 20 megabytes.
A better way to say this is that the RUN command, which creates a new layer in the
Docker image, creates a 20 megabyte layer. We can look at the history of the
Docker image to see that this is the case:

```
$ docker history hello-world:3
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9683ee152c82        3 minutes ago       /bin/sh -c apt-get update && apt-get upgrade…   20.6MB
238815bfecbe        17 minutes ago      /bin/sh -c #(nop)  LABEL maintainer=isuftin@…   0B
c08899734c03        5 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>           5 weeks ago         /bin/sh -c #(nop) ADD file:4fc310c0cb879c876…   55.3MB
```

It's important to note that layers within a Docker image are static. That is to say
if I have one layer that adds 5 megabytes by using `RUN` to download a 5 megabyte file,
I can have the next `RUN` command delete that file from the Docker image. However,
the resulting Docker image in my registry still has the layer that downloaded the
file as part of the resultant Docker image. The secondary RUN command will have
deleted the file from the Docker image's file system but the Docker image will not
shrink by 5 megabytes in the end. It's best practice that if you are performing
commands that add size to an image and you can remove the files that add to that
size once the command is done, you will want to add that to the `RUN` command that
performs the increase in size. That sounds confusing so maybe an example might work
better here:

```
...

RUN wget http://someplace/20mb_binary_file.bin ./20mb_binary_file.bin
RUN command_that_does something with 20mb file
RUN rm 20mb_binary_file.bin

...
```

With the command above, you will still have your image bloated by 20 megabytes
because that file is removed in another Docker image layer. A better way of doing
this is chaining your commands in a single RUN statement:

```
...

RUN wget http://someplace/20mb_binary_file.bin ./20mb_binary_file.bin && \
      command_that_does something with 20mb file && \
      rm 20mb_binary_file.bin

...
```

This creates a single layer that downloads the 20mb binary file, does something with it
and removes it in one command chain. The size of the resulting layer is dependent
only on whatever increase in file usage is created by the command that does something
with the 20mb file.

You will see this sort of chaining done in Dockerfiles throughout the Docker
community as the desire is to create the smallest image possible since a smaller
image takes up less space and is faster to transfer over the internet to your
Docker registry.

So now I've updated the packages on the Debian system and I wish to now install
the NGINX server. This software is available in the Debian repositories so it's as
easy as using apt to install it. You can see that I've done so in Dockerfile_4. I
add a new `RUN` command `RUN apt-get install nginx -y && apt-get clean`.

```
$ docker build -t hello-world:4 -f Dockerfile_4 .
Sending build context to Docker daemon  20.99kB
Step 1/4 : FROM debian:stretch-slim
 ---> c08899734c03
Step 2/4 : LABEL maintainer="isuftin@usgs.gov"
 ---> Using cache
 ---> 238815bfecbe
Step 3/4 : RUN apt-get update && apt-get upgrade -y && apt-get clean
 ---> Using cache
 ---> 9683ee152c82
Step 4/4 : RUN apt-get install nginx -y && apt-get clean                                                                                                                                                   ---> Running in 4f8378855905                                                                                                                                                                             Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core geoip-database libbsd0 libexpat1
  libfontconfig1 libfreetype6 libgd3 libgeoip1 libicu57 libjbig0
  libjpeg62-turbo libnginx-mod-http-auth-pam libnginx-mod-http-dav-ext
  libnginx-mod-http-echo libnginx-mod-http-geoip
  libnginx-mod-http-image-filter libnginx-mod-http-subs-filter
  libnginx-mod-http-upstream-fair libnginx-mod-http-xslt-filter
  libnginx-mod-mail libnginx-mod-stream libpng16-16 libssl1.1 libtiff5
  libwebp6 libx11-6 libx11-data libxau6 libxcb1 libxdmcp6 libxml2 libxpm4
  libxslt1.1 nginx-common nginx-full sgml-base ucf xml-core
Suggested packages:
  libgd-tools geoip-bin fcgiwrap nginx-doc ssl-cert sgml-base-doc debhelper
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core geoip-database libbsd0 libexpat1
  libfontconfig1 libfreetype6 libgd3 libgeoip1 libicu57 libjbig0
  libjpeg62-turbo libnginx-mod-http-auth-pam libnginx-mod-http-dav-ext
  libnginx-mod-http-echo libnginx-mod-http-geoip
  libnginx-mod-http-image-filter libnginx-mod-http-subs-filter
  libnginx-mod-http-upstream-fair libnginx-mod-http-xslt-filter
  libnginx-mod-mail libnginx-mod-stream libpng16-16 libssl1.1 libtiff5
  libwebp6 libx11-6 libx11-data libxau6 libxcb1 libxdmcp6 libxml2 libxpm4
  libxslt1.1 nginx nginx-common nginx-full sgml-base ucf xml-core
0 upgraded, 40 newly installed, 0 to remove and 0 not upgraded.
Need to get 18.7 MB of archives.
After this operation, 59.3 MB of additional disk space will be used.

[...snip...]

Setting up libnginx-mod-http-image-filter (1.10.3-1+deb9u2) ...
Setting up nginx-full (1.10.3-1+deb9u2) ...
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Setting up nginx (1.10.3-1+deb9u2) ...
Processing triggers for libc-bin (2.24-11+deb9u4) ...
Processing triggers for sgml-base (1.29) ...
Removing intermediate container 4f8378855905
 ---> 28c4b1e174e6
Successfully built 28c4b1e174e6
Successfully tagged hello-world:4
```

I've cut out a lot of the installation steps here to not make this document much
longer than it needs to be.

The first thing I want you to notice near the beginning of the build are the lines
that say `Using cache`. What does this mean? Did you notice that in this build
that Docker did not re-run the apt-get update, apt-get upgrade, apt-get clean
commands? Why? This is because between the current and previous builds, the only
line that's changed in our Docker build is the addition of the nginx installation
via a new `RUN` command. This is one of the nicer things about Docker. It will
reuse layers it's already built and has in the cache to speed up your build. It
only needs to perform anything new you've added. One caveat is that if you inject
new commands in the middle somewhere, it will re-run everything after the new command
regardless of having commands cached already.

For example, if I have a build that looks like:

```
RUN command 1
```

and then a new build that looks like:
```
RUN command 1
RUN command 2
```

Docker will use the cached layer from `RUN command 1` and not re-run it, only running
`RUN command 2`. But if I create a third build that looks like:

```
RUN command 0
RUN command 1
RUN command 2
```

Docker will run all of these over as the insertion of command 0 will invalidate the
cache for the next two steps.

So knowing what we know, we can include all of the installation steps into a single
`RUN` command like in Dockerfile_5 which ends up looking like:

```
RUN apt-get update && \
      apt-get upgrade -y && \
      apt-get install nginx -y && \
      apt-get clean
```

After building the Docker image for Dockerfile_5, we can take a look at the Docker
images we have so far:

```
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         5                   8a9e3f34f53d        3 hours ago         131MB
hello-world         4                   28c4b1e174e6        3 hours ago         133MB
hello-world         3                   9683ee152c82        3 hours ago         75.9MB
hello-world         2                   238815bfecbe        3 hours ago         55.3MB
debian              stretch-slim        c08899734c03        5 weeks ago         55.3MB
hello-world         1                   c08899734c03        5 weeks ago         55.3MB
```

Note that the last image we build it actually 2 megabytes smaller than the image
we built that has us updating the repo and packages and installing nginx on two
separate `RUN` commands. This is because we perform a cleanup as the last step in
the `RUN` command which ensures that the layer that gets created in the Docker
image isn't persisted without being cleaned up.

At this point NGINX should be installed in hello-world:5. However, the image we
built does not know what to do when we start it. If we issue `docker run hello-world:5`
to start a Docker container based on the image of `hello-world:5`, we see that nothing
actually happens. The container exits immediately with an exit code of 0.

That's because when we create a Docker image, we should also include the command
or set of commands that the Docker container should run when it first starts.

We do this by using the [`CMD`](https://docs.docker.com/engine/reference/builder/#cmd)
directive. There is also the option to use [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint)
 but that directive is used moreso to use the Docker container as an executable
rather than a running process. The [Docker documentation](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact) actually documents the difference between the two since
people are often confused which they should use. Note that the two can also be used
together.

In Dockerfile_6, I add the `CMD` directive that kicks off NGINX when the Docker
container runs.

```
$ docker build -t hello-world:6 -f Dockerfile_6 .
Sending build context to Docker daemon   29.7kB
Step 1/4 : FROM debian:stretch-slim
 ---> c08899734c03
Step 2/4 : LABEL maintainer="isuftin@usgs.gov"
 ---> Using cache
 ---> 238815bfecbe
Step 3/4 : RUN apt-get update &&       apt-get upgrade -y &&       apt-get install nginx -y &&       apt-get clean
 ---> Using cache
 ---> 8a9e3f34f53d
Step 4/4 : CMD ["nginx", "-g", "daemon off;"]
 ---> Running in cc2b0e8fdaed
Removing intermediate container cc2b0e8fdaed
 ---> e1b2d0b94186
Successfully built e1b2d0b94186
Successfully tagged hello-world:6

$ docker run --rm -p 8080:80 -d --name hello-world -t hello-world:6
b9c70c9654ec0a29c7f5db2362aa69593a4406adf891cfa06caca75563f79b67

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
b9c70c9654ec        hello-world:6       "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes        0.0.0.0:8080->80/tcp   hello-world
```

As shown above, I've built the `hello-world:6` image and in a second command, I
deploy the container. I've included the `--rm` flag which tells Docker to remove
the stopped container when it exits. Otherwise, the stopped Docker container sticks
around and needs to be manually removed. The `-p` flag tells Docker that I want to
bind the host's port `8080` to the Docker container's port `80` so that when I
launch a web browser and point it to the IP address of the Docker engine (usually
  localhost on Linux instances) on port 8080, I should see the NGINX welcome screen.
The `-d` flag tells Docker that I want to launch the container and detach from
its stdout, which pops me back into bash instead of tailing the stdout of the
running container. The `--name` flag tells Docker that I want the running container's
name to be `hello-world` instead of the random name Docker will assign to it. Finally,
the `-t` flag dictates to Docker which image I want it to run.

When I then issue `docker ps` to find out which containers are running, I see that
the NGINX container is indeed running. If the starting of NGINX hit an error, you
would see no containers running. At that point you could remove the `--rm` flag
from the next run and check the logs of the stopped container once it stops or
you could remove the `-d` flag to not detach once the run kicks off and view
the logs that way. Once the container stops in attached mode, you're put back into
your shell.

At this point, since the Docker container is running, I can try pointing my web
browser to the Docker engine's IP on port 8080. I can also use a text browser which
works much better in a markdown file ;)

```
$ lynx http://localhost:8080/

Welcome to nginx!

If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```

Great. So now I am able to run NGINX in a Docker container using just a `RUN` directive
and a `CMD` directive! But NGINX isn't hosting my own content, is it? I want to
be able to issue a hello world response instead of getting NGINX's welcome page.
I stop the NGINX container via `docker stop hello-world`. At this point NGINX is
no longer running.

Let's tidy up our Docker image slightly in Dockerfile_7.

First, we want NGINX logs, which are saved to `/var/log/nginx/access.log` and
`/var/log/nginx/error.log` to be output to stdout and stderr for Docker to be
able to collect those logs. Being that this runs on Linux, we can simply create
soft links from those two files to `/dev/stdout` and `/dev/stderr` via a `RUN`
command. In Dockerfile_7, I add
`RUN ln -sf /dev/stdout /var/log/nginx/access.log && ln -sf /dev/stderr /var/log/nginx/error.log`
which creates those links.

Next, I want to document in the Docker image that port 80 will be exposed. I will
use the [`EXPOSE`](https://docs.docker.com/engine/reference/builder/#expose) directive.
Note that this directive only serves as a form of documentation and doesn't actually
publish the port. As such, this is not mandatory but nice to have. I add `EXPOSE 80/tcp`.
When using an `EXPOSE` directive, Docker defaults to using TCP but I like to be
explicit. UDP is also an option.

I also add a [`STOPSIGNAL`](https://docs.docker.com/engine/reference/builder/#stopsignal)
directive which tells the Docker container what stop code to send to the running
process in the container when it comes time to exit. I use SIGTERM here as its
considered a polite way to stop a process.

I add a [`VOLUME`](https://docs.docker.com/engine/reference/builder/#volume) directive
to create mountpoints where I plan to add the data I want to serve in the container
at runtime. It looks like `VOLUME ["/var/data/www", "/var/data/images"]`. I expect
that unless I mount anything to the running container, these will be empty directories
in the container.

I've added `apt-get purge -y --auto-remove` into the list of apt commands in order
to ensure a clean system at the end of updates.

Now I want to be able to add my own configuration at run time to NGINX. This allows me
to launch the container with configuration that I've edited on my host. To do so,
I can use [Docker volume mounting](https://docs.docker.com/engine/reference/commandline/run/#mount-volume--v---read-only)
to directly inject files and directories into my container when it starts up. With this
ability, I can edit an NGINX configuration file on my host and launch a container
with this configuration. I do this when I launch the container by adding the `-v` flag
like so:

```
$ docker run --rm -p 8080:80 \
  -v "`pwd`/configuration/nginx.conf.1:/etc/nginx/nginx.conf" \
  --name hello-world \
  -t hello-world:7
```

This tells the Docker engine that when it launches the container, I want it to take
the file that sits on my host in `configuration/nginx.conf.1` (using the root of)
the project as the working directory and mount it to `/etc/nginx/nginx.conf` within
the container. By mounting it to that exact location, I overwrite the configuration
that NGINX ships with. If you want to see that configuration, I've included it in
`configuration/nginx.conf.original`.

Now when I launch the container via the above run command, I try to go to `http://localhost:8080`
and get a 404 page. This is because the configuration now says that the root (`/`)
context requests go to `/data/www` for content and the images context (`/images`)
go to `/data/images`.

A sample of the NGINX log output that shows this 404 looks like:

```
2019/05/07 21:36:50 [error] 7#7: *1 "/data/www/index.html" is not found (2: No such file or directory), client: 192.168.99.1, server: , request: "GET / HTTP/1.1", host: "192.168.99.100:8080"
192.168.99.1 - - [07/May/2019:21:36:50 +0000] "GET / HTTP/1.1" 404 200 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36"
2019/05/07 21:36:53 [error] 7#7: *1 "/data/www/index.html" is not found (2: No such file or directory), client: 192.168.99.1, server: , request: "GET / HTTP/1.1", host: "192.168.99.100:8080"
192.168.99.1 - - [07/May/2019:21:36:53 +0000] "GET / HTTP/1.1" 404 200 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36"
```

As you can see, it does try to go to /data/www to find index.html but cannot and returns a 404.
This means that NGINX has picked up my configuration and is working properly, but
there is nothing being served yet!

So let's try serving some HTML, shall we?

I've included an image and some boilerplate HTML in this project under `configuration/www`
and `configuration/images`. At this point, all I need to do is run the NGINX container
and volume mount those directories into the container at the location where the
NGINX configuration is expecting them.

I do this like so:

```
$ docker run --rm -p 8080:80 \
  -v "`pwd`/configuration/nginx.conf.1:/etc/nginx/nginx.conf" \
  -v "`pwd`/configuration/www:/data/www" \
  -v "`pwd`/configuration/images:/data/images" \
  --name hello-world \
  -t hello-world:7
```

Now when NGINX launches, I am able to get the HTML I expect:

```
$ http http://localhost:8080/
HTTP/1.1 200 OK
Connection: keep-alive
Content-Encoding: gzip
Content-Type: text/html
Date: Tue, 07 May 2019 21:58:46 GMT
ETag: W/"5cd1fedb-19f"
Last-Modified: Tue, 07 May 2019 21:55:39 GMT
Server: nginx/1.10.3
Transfer-Encoding: chunked

<!DOCTYPE html>
<html>
<head>
<title>Hello World!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>You've made it to my NGINX server. Congratulations!</h1>
<p>I want to welcome you by saying HELLO WORLD</p>
<p>
Also, enjoy this sweet USGS logo: <img src="images/USGS_logo.svg" />
</p>
</body>
</html>
```
