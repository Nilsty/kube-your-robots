# Docker
Introduction about Robot Framework in Docker at https://docs.robotframework.org/docs/using_rf_in_ci_systems/docker

Docker is basic tool in software engineering today. However, there are concerns about the attempts of Docker Inc. building some sort of business plan on top of it. Another concern is that docker requires root privileges on the host system. A more and more popular alternative in [podman](https://podman.io/). Podman does not require root privileges and it shares almost the same commands as docker. Thus, you can often use `docker` as an alias for `podman`. For instance, to build an image with podman, the command is `podman build` just like `docker build`.

However, podman is originally developed by Redhat, so you still might be save from creative business plans after all. Podman also provides a desktop ui for all major operating systems. In contrast to Docker Desktop, Podman Desktop is licensed under Apache 2.0 license and is thus free to use.
