# Docker
Introduction about Robot Framework in Docker at https://docs.robotframework.org/docs/using_rf_in_ci_systems/docker

Docker is basic tool in software engineering today. However, there are concerns about the attempts of Docker Inc. building some sort of business plan on top of it. Another concern is that docker requires root privileges on the host system. A more and more popular alternative in [podman](https://podman.io/). Podman does not require root privileges and it shares almost the same commands as docker. Thus, you can often use `docker` as an alias for `podman`. For instance, to build an image with podman, the command is `podman build` just like `docker build`.

However, podman is originally developed by Redhat, so you still might be save from creative business plans after all. Podman also provides a desktop ui for all major operating systems. In contrast to Docker Desktop, Podman Desktop is licensed under Apache 2.0 license and is thus free to use.

## Container or Image?

In the beginning the terms `container` and `image` maybe be confusing as they seem to refer to the same thing. And `image` is what you build. `container` is a running instance of the image. You can compare with `classes` and `objects` in Java. A class is the specification, while an `object` is the running instance. You can start several instances of the same class in Java as you can run several containers of the same image in Docker.

- `docker images` command shows the images that are cached on your local environment
- `docker ps` command shows containers that are stored on your local environment (inactive containers with `-all` switch)
- `docker stats` command shows live stats the currently running container instances

## Build Image

Conluding the previous paragraph, we *build* a docker image. Robot Framework describe the basics in our context very well: https://docs.robotframework.org/docs/using_rf_in_ci_systems/docker#creating-a-robot-framework-dockerimage

A few teeaks shall be mentioned:

- **Do not split CLI calls that belong together over several `RUN` statements:** Each `RUN` command will result in a cached layer. If docker decides that a cached layer is good enough, it will not execute the `RUN` line in a later docker build, which leads to weird effects when an image is build that does not look like the Dockerfile. See the `RUN` command [extenstion of the Browser image](./12-custom/Dockerfile-Browser-with-Python311) in this project as example to concatenate commands that belong together.
- **Create a `.dockerignore` file for controlling what files you do *not* want in your image:** The hidden `.git` folder is a classic that you do not want to be in your image. If the `.git` folder is copied unintentionally with `COPY . .` to your image, you accidentally ship your complete git repository with all its history together with your image.