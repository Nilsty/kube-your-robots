# Custom dockerfile

## Choosing a base image
Creating a custom Robot Framework image from scratch, you need a [Python base image](https://hub.docker.com/_/python/) 

The base images do not only differe in python version, but also in the operating systems python installed on to inside the image. Most versions are Debian based. Versions with the token `-slim` are a bit smaller and lack a few default tools. `slim` images work for most cases, while the non-slim version are more like complete Debian Linux with all the tools installed within that you would expact. The advantage of debian based images is that most documentation on the web is probably Ubuntu or Debian based, thus you install new packages with common `apt-get install`.

An exception are `alpine` based images. These images are super small, but often require some to install every little command line tool on top with the command `apk add`. It may require some research to find the right dependencies that need to be installed.

Image size seems of a low priority, but take in mind, your image will be pulled often in your cloud environment or CI, thus single digit seconds or 10s of seconds will add up over time and are infact a cost factor on top of the actual space that your images require.

## Creating Robot Framework Image

A very basic image with Robot Framework:

```
FROM python:3-slim

RUN pip3 install robotframework
```

Running `docker build` on this image will install Robot Framework in an image with the newest verison of Python 3 based on a slim Debian.
