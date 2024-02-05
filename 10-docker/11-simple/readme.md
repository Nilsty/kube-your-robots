# Docker - the easy way for Robot Framework
The Robot Framework Webservice is a good example for a use case using Docker. The image of RF webservice allows to start a container with test cases within that can be executed on demand through a REST api. 

The image can be pulled with 

```
docker pull ghcr.io/marketsquare/robotframework-webservice:latest
```

To run your test cases as a service, you could integrate your test cases in your image like this:

```
FROM python:3.11

# exposing the port for accessing the webservice
EXPOSE 8080

# copy tests to image and requirements.txt
COPY my/local/tests .
COPY requirements.txt .

# install dependencies
RUN pip3 install --user -r requirements.txt

ENTRYPOINT [ "python","-m","RobotFrameworkService.main","-p","8080","-t","." ]
```

After building this image with `docker build` you can run it by:

```
docker run --rm --publish 8080:8080 \
           <your image name>
``

When running this image as container, the *entrypoint* will start the webservice listening on port 8080. (because `-p 8080:8080` mapped your machine's port 8080 to the container port 8080). On that port you can trigger the test cases wrappend in the container on demand. You can view all endpoints of the webservice on localhost:8080/docs

You can also start the container and hook your test cases from an external volume in to the container by running:

```
docker run --rm --publish 5003:5003 \
           --volume <host directory of test cases>:/robot/tests \
           <your image>
```

The `volume` switchs maps a host folder on to a container folder.
