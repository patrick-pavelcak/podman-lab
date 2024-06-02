# Podman Compose

Podman Compose is a tool to orchestrate multiple containers that make up an application. It is especially useful for complex applications that require multiple interconnected services, such as a web server, database, and cache. Applications are defined using a YAML file. This file specifies the services, networks, and volumes required by the application, along with their configurations.

## Writing a `compose.yaml` file
We are going to use the image we built in the previous part as a base for our new composition. 

Edit the `compose.yaml` file and fill it in with the values we used to run the container using the `podman run` command in the previous part. The file can use different name, but then you will have to pass the name of the file as a parameter, when using `podman compose` command. 

When editing yaml files you have to be careful and not mess the indentation, otherwise it will be not possible to parse the file. 


```yaml
version: '3.7'

services:
  greeting-app:
    container_name: #define the container name
    image: #define the image to be run
    ports:
      - # define the port mapping, in the following format "HOST:CONTAINER"
    environment:
      - # set the GREETING_MESSAGE environment variable
```

After the values are set correctly, you can create the container using the `podman compose up` command:
```
podman compose up
```
If our file used a different name (other than `compose.yaml` or `docker-compose.yaml`, that is supported because of legacy reasons), you would also have `-f` parameter and specify the name. 

Verify in browser, that the URL http://localhost:8080/hello is reachable and returns the expected response from the `compose.yaml` file.

## Cleaning up

In order to only stop the composition, you can use this command:
```
podman compose stop
```

This will only stop the containers specified in the yaml file. If you want to remove them alltogether, than you should use
```
podman compose down
```