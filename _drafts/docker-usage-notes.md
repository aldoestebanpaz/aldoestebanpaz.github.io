# Docker usage notes

## Images

### List images

Run this command to list images available locally:

```sh
docker images | grep <IMAGE_NAME>
```

An image will be listed more than once if it has multiple repository names or tags. This single image (identifiable by its matching IMAGE ID) uses up the SIZE listed only once.

### Get detailed information on an image

```sh
docker image inspect <IMAGE_NAME> | less
```

Example, the following command get all the tags of an image that already has a container create:

```sh
docker image inspect --format={{.RepoTags}} $(docker inspect --format='{{.Config.Image}}' myproj)
```

### Build a docker image

```sh
cd /my/repo
docker build -t <IMAGE_NAME> .
```

### Save an image as a file

```sh
docker save -o myimage.image <IMAGE_NAME>
```

### Load an image file

Read from STDIN:

```sh
docker load < myimage.image
```

Read from file:

```sh
docker load -i myimage.image
```

## Containers

### Create a container

#### Create and run it automatically

Containers without '--network' flag are attached automatically to the default bridge network. It means that `--network bridge` is implicit.

```sh
docker run ... -name <CONTAINER_NAME> <IMAGE_NAME>[:TAG]
```

##### Detached mode

To run in detached mode, that means in background without blocking the command line, use `-d`:

```sh
docker run ... -d -name <CONTAINER_NAME> <IMAGE_NAME>[:TAG]
```

##### Pull mage from another repository

**1 - Login**

NOTE: You have to make this step for the first time only.

```sh
sudo docker login myartifactory.local
```

**2 - Pull or directly pull-and-run the images**

```sh
sudo docker pull myartifactory.local/docker-repo/sampleproject/origin/development/SampleProject:latest
# or
docker run ... -d --name myproj myartifactory.local/docker-repo/sampleproject/origin/development/SampleProject:latest
```

##### A most common example of running a container

The following command:
- Runs in detached mode (-d),
- maps the port of the host 8083 to the port 8080 of the container (-p),
- limits the ram usage to 1024m (-m),
- mounts the directory /opt/myproduct/config and /opt/myproduct/logs of the host, to /config and /logs in the container (-v),
- sets an environment variable (-e),
- configures it to restart automatically (--restart=always),
- uses the latest version of the container.

```sh
docker run -d -p 8083:8080 -m 1024m -v /opt/myproduct/config:/config -v /opt/myproduct/logs:/logs -e ENVIRONMENT=Staging
--restart=always --name <CONTAINER_NAME> <IMAGE_NAME>:latest
```

#### Create and run it automatically attached to a specific network

This command attach the container to another network different than bridge.

```sh
docker run ... -d --network <NETWORK_NAME> -name <CONTAINER_NAME> <IMAGE_NAME>[:TAG]
```

#### Create a container and attach it to an existing network

```sh
docker create ...
# repeat
docker network connect <NETWORK_NAME> <CONTAINER_ID>
docker start <CONTAINER_ID>
```

#### Create a network and attach containers

Create the internal network:

```sh
docker network create --internal sample-network
```

Create the containers. Note it is not possible to do `docker run` because this command tries to create the network whenever it is executed:

```sh
# 1
docker create \
  ... \
  --name myproj \
  myartifactory.local/docker-repo/sampleproject/origin/development/SampleProject:latest

# 2
docker create \
  ... \
  -p 80:80 \
  -p 443:443 \
  -v /opt/nginx:/etc/nginx:ro \
  --name proxy \
  nginx
```

Connect the containers to the existing network:

```sh
sudo docker network connect sample-network myproj
sudo docker network connect sample-network proxy
```

Start the containers:

```sh
sudo docker start myproj
sudo docker start proxy
```

### Restart a container

```sh
docker restart <CONTAINER_ID>
```

### Stop a container

(Recommended) Using stop (send SIGTERM, and then SIGKILL after grace period):

```sh
docker stop <CONTAINER_ID>
```

Using kill (sends the default SIGKILL signal to the container):

```sh
docker kill <CONTAINER_ID>
```

### Stop all containers

```sh
docker kill $(docker ps -q)
```

### List containers

#### List running containers

```sh
docker ps
```

#### List all containers

```sh
docker ps -a
```

### Get detailed information on a container

#### Get all detailed information

```sh
docker inspect <CONTAINER_ID> | less
```

#### Get specific values

```sh
docker inspect -f "{{json .Args}}" <CONTAINER_ID> | python -m json.tool
# sample output
# [
#     "nginx",
#     "-g",
#     "daemon off;"
# ]
docker inspect -f "{{json .Config.Cmd}}" <CONTAINER_ID> | python -m json.tool
# sample output
# [
#     "nginx",
#     "-g",
#     "daemon off;"
# ]
docker inspect -f "{{json .Config.Env}}" <CONTAINER_ID> | python -m json.tool
# sample output
# [
#     "ASPNETCORE_ENVIRONMENT=Staging",
#     "ASPNETCORE_HTTPS_PORT=443",
#     "ASPNETCORE_Kestrel__Certificates__Default__Path=/app/certs/cert.crt",
#     "ASPNETCORE_Kestrel__Certificates__Default__KeyPath=/app/certs/cert.key",
#     "SERVER_ENABLE_HTTP_LOGS=1",
#     "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
#     "ASPNETCORE_URLS=http://+:80",
#     "DOTNET_RUNNING_IN_CONTAINER=true",
#     "DOTNET_VERSION=6.0.2",
#     "ASPNET_VERSION=6.0.2",
#     "Logging__Console__FormatterName=Json"
# ]
docker inspect -f "{{json .Config.Entrypoint}}" <CONTAINER_ID> | python -m json.tool
# sample output
# [
#     "dotnet",
#     "myapp.dll"
# ]
docker inspect -f "{{json .Config.ExposedPorts}}" <CONTAINER_ID> | python -m json.tool
# sample output
# {
#     "5000/tcp": {},
#     "5001/tcp": {}
# }
docker inspect -f "{{json .Config.Image}}" <CONTAINER_ID>
# sample output
# "myapp"
docker inspect -f "{{json .Config.WorkingDir}}" <CONTAINER_ID>
# sample output
# "/app"
docker inspect -f "{{json .HostConfig.Binds}}" <CONTAINER_ID> | python -m json.tool
# sample output:
# [
#     "/opt/myapp/config:/app/config:ro",
#     "/opt/myapp/logs:/app/logs",
#     "/opt/myapp/etc/stg/ssl/certs:/app/certs:ro"
# ]
docker inspect -f "{{json .HostConfig.PortBindings}}" <CONTAINER_ID> | python -m json.tool
# sample output:
# {
#     "443/tcp": [
#         {
#             "HostIp": "",
#             "HostPort": "443"
#         }
#     ],
#     "80/tcp": [
#         {
#             "HostIp": "",
#             "HostPort": "80"
#         }
#     ]
# }
docker inspect -f "{{json .Mounts}}" <CONTAINER_ID> | python -m json.tool
# sample output:
# [
#     {
#         "Destination": "/app/config",
#         "Mode": "ro",
#         "Propagation": "rprivate",
#         "RW": false,
#         "Source": "/opt/myapp/config",
#         "Type": "bind"
#     },
#     {
#         "Destination": "/app/logs",
#         "Mode": "",
#         "Propagation": "rprivate",
#         "RW": true,
#         "Source": "/opt/myapp/logs",
#         "Type": "bind"
#     },
#     {
#         "Destination": "/app/certs",
#         "Mode": "ro",
#         "Propagation": "rprivate",
#         "RW": false,
#         "Source": "/opt/myapp/etc/stg/ssl/certs",
#         "Type": "bind"
#     }
# ]
docker inspect -f "{{json .NetworkSettings.Networks}}" <CONTAINER_ID> | python -m json.tool
# sample output
# {
#     "myapp-network": {
#         "Aliases": [
#             "45a43bc3c76b"
#         ],
#         "DriverOpts": null,
#         "EndpointID": "500806d720908e1933d4c6c7c4cd95976455aa49950b68e839d492dc0b62f811",
#         "Gateway": "172.20.0.1",
#         "GlobalIPv6Address": "",
#         "GlobalIPv6PrefixLen": 0,
#         "IPAMConfig": null,
#         "IPAddress": "172.20.0.2",
#         "IPPrefixLen": 16,
#         "IPv6Gateway": "",
#         "Links": null,
#         "MacAddress": "02:42:ac:14:00:02",
#         "NetworkID": "69435a289dacc638c8f08b94f3bbc497599d7926e2483015e46fdd9220727594"
#     }
# }
docker inspect -f "{{json .NetworkSettings.Ports}}" <CONTAINER_ID> | python -m json.tool
# sample output
# {
#     "443/tcp": [
#         {
#             "HostIp": "0.0.0.0",
#             "HostPort": "443"
#         },
#         {
#             "HostIp": "::",
#             "HostPort": "443"
#         }
#     ],
#     "80/tcp": [
#         {
#             "HostIp": "0.0.0.0",
#             "HostPort": "80"
#         },
#         {
#             "HostIp": "::",
#             "HostPort": "80"
#         }
#     ]
# }
docker inspect -f "{{json .Path}}" <CONTAINER_ID>
# sample output
# "/docker-entrypoint.sh"
docker inspect -f "{{json .State}}" <CONTAINER_ID> | python -m json.tool
# sample output:
# {
#     "Dead": false,
#     "Error": "",
#     "ExitCode": 0,
#     "FinishedAt": "0001-01-01T00:00:00Z",
#     "OOMKilled": false,
#     "Paused": false,
#     "Pid": 4278,
#     "Restarting": false,
#     "Running": true,
#     "StartedAt": "2022-03-05T08:05:00.215522312Z",
#     "Status": "running"
# }
```

### Delete a container

```sh
docker rm -f <CONTAINER_ID>
```

### Delete all containers

```sh
docker rm -f $(docker ps -a -q)
```

### Run a container and delete it automatically after it exits

Normally, a Docker container persists after it has exited, which allows you to run the container again, inspect its filesystem, and so on. However, sometimes you want to run a container and delete it immediately after it exits. Docker provides the --rm command line option for this purpose:

```sh
docker run --rm <IMAGE_NAME>
```

### Run a container with bash shell and delete it automatically after it exits

This command is usefull for exploring the content of a docker image without starting the default entrypoint.

```sh
docker run --rm -ti --entrypoint=bash <IMAGE_NAME>
```

### Execute command in running container

```sh
docker exec <CONTAINER_ID> <COMMAND>
# e.g. docker exec <CONTAINER_ID> ifconfig
```

### Get into a running container with an interactive shell

```sh
docker exec -it <CONTAINER_ID> bash
# or
docker exec -it <CONTAINER_ID> /bin/bash
```

### Run a container just to get an interactive shell

The following example shows how to run a temporary container just to get an interactive shell into it. Once the user exits, the container will be deleted automatically:

```sh
sudo docker run --rm \
  -it \
  -v ~/bkp:/etc/nginx \
  nginx:latest \
  /bin/bash
```

### Copy content from a container to the host

```sh
docker cp <CONTAINER_ID>:<FILE_LOCATION_IN_CONTAINER> ./<FILE_NAME>
```

### Logs

To print logs:

```sh
docker logs <CONTAINER_ID>
```

Keep listening log entries:

```sh
docker logs -f <CONTAINER_ID>
```

## Networks

### Create a network

#### Create a custom bridge network

```sh
docker network create <NETWORK_NAME>
```

#### Create a custom bridge network for internal usage only

The '—internal' flag prevents any external communication from the bridge. This means that, even when you try to publish ports with '-p', these are not accessible on the host and the ports are not available to any client that can reach the host.

```sh
docker network create --internal <NETWORK_NAME>
```

### List networks

```sh
docker network ls
```

### Delete a network

```sh
docker network rm <NETWORK_NAME>
```

#### Delete all custom networks not used by at least one container

```sh
docker network prune
```

### Get detailed information on an network

With this command you can see the containers attached to an specific network.

```sh
docker network inspect <NETWORK_NAME> | less
```

#### Get detailed information on the default network

```
docker network inspect bridge | less
```

### The bridge network

If you do not specify a network using the '--network' flag, and you do specify a network driver, your container is connected to the default bridge network by default. Containers connected to the default bridge network can communicate, but only by IP address.

Whenever you start Docker, a bridge network gets created and all newly started containers will connect automatically to the default bridge network.

You can use this whenever you want your containers running in isolation to connect and communicate with each other. Since containers run in isolation, the bridge network solves the port conflict problem. Containers running in the same bridge network can communicate with each other, and Docker uses iptables on the host machine to prevent access outside of the bridge.

The downside with the bridge driver is that it's not recommended for production; the containers communicate via IP address instead of automatic service discovery to resolve an IP address to the container name. Every time you run a container, a different IP address gets assigned to it. It may work well for local development or CI/CD, but it's definitely not a sustainable approach for applications running in production.

Another reason not to use it in production is that it will allow unrelated containers to communicate with each other, which could be a security risk. The solution here is creating custom bridge networks. Custom bridge networks are networks on the host machine separated from the rest of the host network stack.

## Mapped ports

### Show what is the binded port (the port accesible by clients outside)

```sh
docker port <CONTAINER_ID> <PRIVATE_PORT>
```

### List all mapped ports

```sh
docker port <CONTAINER_ID>
```

## Dockerfile

### The EXPOSE instruction

The documentation explicetly states:

The EXPOSE instruction informs Docker that the container listens on the specified network ports at runtime. You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified.

The EXPOSE instruction does not actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published.

### Difference between expose and publish

Basically, there are four options:

- Neither specify the `EXPOSE` instruction nor the `-p` parameter.
- Only specify `EXPOSE`.
- Only specify `-p`.
- Specify `EXPOSE` and `-p`.

1) If you specify neither EXPOSE nor -p, the service in the container still is accessible in an inter-container communication.

2) If you EXPOSE a port, it works the same as (1). EXPOSE functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published. With or without the EXPOSE instruction it is still posible an inter-container communication.

3) If you publish a port with -p, the service in the container is accessible from anywhere. Remember that EXPOSE is just for documentation, so regardless of the EXPOSE settings, you can use the -p flag in any port. If you open a port to the public with -p, but do not EXPOSE, Docker doesn't complain.

4) If you EXPOSE and publish a port with -p, the service in the container is accessible from anywhere same as (3), and exposed ports appears in `docker inspect -f "{{json .Config.ExposedPorts}}" <CONTAINER_ID>`.

### User-defined networks and exposed ports

You can create networks with `docker network` for communication among containers (inter-container communication) without the need to expose or publish specific ports, because the containers connected to the network can communicate with each other over any port.

## Registries

A Docker registry is a storage and distribution system for named Docker images. The same image might have multiple different versions, identified by their tags. A Docker registry is organized into Docker repositories , where a repository holds all the versions of a specific image.

### Use an insecure registry

While it's highly recommended to secure your registry using a TLS certificate issued by a known CA, you can choose to use self-signed certificates, or use your registry over an unencrypted HTTP connection.

Either of these choices involves security trade-offs and additional configuration steps.

With insecure registries enabled (the `"insecure-registries"` property), Docker goes through the following steps:

- First, try using HTTPS. If HTTPS is available but the certificate is invalid, ignore the error about the certificate.
- If HTTPS is not available, fall back to HTTP.

More info:
- [Test an insecure registry](https://docs.docker.com/registry/insecure/)
- [Daemon CLI (dockerd)](https://docs.docker.com/engine/reference/commandline/dockerd/)

#### HTTP only registry

This procedure configures Docker to entirely disregard security for your registry.

This is very insecure and is not recommended. It exposes your registry to trivial man-in-the-middle (MITM) attacks. Only use this solution for isolated testing or in a tightly controlled, air-gapped environment.

1. Edit `/etc/docker/daemon.json` (If the `daemon.json` file does not exist, create it) and add the `"insecure-registries"` array:

```json
{
  "insecure-registries" : ["myregistrydomain.com:5000"]
}
```

2. Restart Docker for the changes to take effect: `systemctl reload docker`.

## Docker and git for CI build

Example using an aspnet core project:

```sh
REPOSITORY="myartifactory.local"
PROJECT_NAME=sampleproject

# repository example: myartifactory.local/docker-repo/sampleproject/origin/development/SampleProject
LOCATION="${REPOSITORY}/docker-repo/${PROJECT_NAME}/${GIT_BRANCH}"
IMAGE_NAME=$(basename -s .git $(git config --get remote.origin.url))


VERSION=$(grep -oPm1 "(?<=<Version>)[^<]+" < SampleProject.csproj)

# time as yyyy-MM-dd.HHmmss
# option 1, works with git^2.6
# FORMATTED_TIME=$(git show -s --format=%cd --date=format:%Y-%m-%d.%H%M%S $GIT_COMMIT)
# option 2, works with old versions of git
GIT_COMMIT_TIME=$(git show -s --format=%cd --date=iso8601 HEAD)
FORMATTED_TIME="\
$(cut -d " " -f 1 <<< ${GIT_COMMIT_TIME}).\
$(cut -d " " -f 2 <<< ${GIT_COMMIT_TIME} | tr -d :)"

# short hash
GIT_COMMIT_SHORT_HASH=$(git rev-parse --short=7 $GIT_COMMIT)

# tag example: v1.0.0.2021-12-01.143955.56156b5
TAG="v${VERSION}.${FORMATTED_TIME}.${GIT_COMMIT_SHORT_HASH}"

# Login with docker into the repository
echo ${docker_build_pwd} | docker login --username docker_build --password-stdin ${REPOSITORY}

# Build, add tags, and publish
docker build -t ${LOCATION}/${IMAGE_NAME}:${TAG} -t ${LOCATION}/${IMAGE_NAME}:latest .
docker push ${LOCATION}/${IMAGE_NAME}

# Save a local copy
docker save -o "${IMAGE_NAME}.image" "${LOCATION}/${IMAGE_NAME}:${TAG}"
```
