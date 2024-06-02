# Packaging Quarkus microservices as containers


First we need a Quarkus application, that will serve as the base for our image. 

The sources for this simple application are located in the `greetings-app` folder. The application contains a single service running on the `/hello` endpoint and returns a single message. The message is configurable and can be changed in the `application.properties` file.

## Running the application

First build try to run the application by executing the `quarkus:dev` command from the project folder:
```
cd podman-lab/greetings-app
mvn quarkus:dev
```

The application and its single service are running on the URL http://localhost:8080/hello. Either open the URL in a web browser or use curl to get the returned response. 

```
curl http://localhost:8080/hello
```
Verify that the message `Hello from Quarkus REST` is returned.

Stop the application with `CTRL + C`.

## Generating the container image
In order for Quarkus to generate an image, it needs an additional dependency `container-image-jib`. There are more extensions that can be used, see: https://quarkus.io/guides/container-image.

From the terminal window add this extension to the application: 

```
mvn quarkus:add-extension -Dextensions='container-image-jib'
```

Following dependency will be automatically added to the `pom.xml` file:
```xml
<dependency>
   <groupId>io.quarkus</groupId>
   <artifactId>quarkus-container-image-jib</artifactId>
</dependency>
```

Before we can build the container image some configuration is required and is added in the `application.properties`. 

```properties title="src/main/resources/application.properties"
# TODO uncomment and set these variables:
quarkus.container-image.build=true
quarkus.container-image.group=quay.io
quarkus.container-image.name=greetings-app
```

These properties will be used to generate our image. `quay.io` in this case is a public container registry that can be used to store the image and share it with others. For more supported parameters, you can check out the documentation: https://quarkus.io/guides/container-image#container-image-options. 

Now we can build a container image from our project:
```
mvn clean install
```

In the output of this command, you should see a following line (the hash may differ):
```
...
[INFO] [io.quarkus.container.image.jib.deployment.JibProcessor] Created container image quay.io/greetings-app:1.0.0-SNAPSHOT (sha256:f55d23a77a88c5c358bcff1171183869327d8fe00874199f99461972dbe72497)
...
```

Now we can verify, if the image is present locally:
```
podman images | grep greetings-app
```

Our image should be listed in the output:
```
REPOSITORY             TAG             IMAGE ID      CREATED            SIZE
quay.io/greetings-app  1.0.0-SNAPSHOT  ffc00d248614  3 minutes ago      437 MB
```

## Running the container
Now we can create and run a container from this image by using `podman run` command. The syntax of this command:
```
podman run [options] image [command [arg …]]
```

We will execute the container with the following parameters: 

```
podman run --name greetings-app -d -p 8080:8080 quay.io/greetings-app:1.0.0-SNAPSHOT
```

In our case the image name is `quay.io/greetings-app:1.0.0-SNAPSHOT`. 
We can also specify additional parameters, such as: 
* name of the newly created container with `--name` parameter
* by using `-d` the container will be run in a detached mode, so `podman` will output its ID and then the containers runs in background 
* port mapping with `-p 8080:8080`, which defines, that the port running on the host `8080` will be mapped to the port `8080` on the container

There are many other parameters you can use, for reference see: https://docs.podman.io/en/latest/markdown/podman-run.1.html 

After executing the `podman run` command, the ID of the container is displayed. 

We can verify if the container is successfully running by using the command: 
```
podman ps
```

The output should look like this (the ID may of course differ):
```
CONTAINER ID  IMAGE                                 COMMAND     CREATED             STATUS             PORTS                   NAMES
f25e84f5356c  quay.io/greetings-app:1.0.0-SNAPSHOT              About a minute ago  Up About a minute  0.0.0.0:8080->8080/tcp  greetings-app
```
^ the status should be `Up`

We can now again verify if the application in our container is also running correctly by accessing the URL http://localhost:8080/hello. 

### Cleaning up
Now we can stop and remove the running container:
```
podman stop greetings-app
podman rm greetings-app
```

If we hadn't run the container with `--name` parameter, we would have to use the container ID here. So in this case, the command would look like this 
```
podman stop f25e84f5356c
podman rm f25e84f5356c
```
You don't have to use the whole ID, only the few first characters that would be used to uniquely identify the container, so `f25e` should be enough. 

We could also remove the image now, but we will need it for the next part of the lab, so do not execute this command yet:
```
podman rmi quay.io/greetings-app:1.0.0-SNAPSHOT
```