# Creating a debezium-server docker image

## Overview
To create a docker image for debezium server there are a few steps that need to be taken.
1. installing sdkman
2. Installing docker
3. install java and maven with sdkman
4. building debezium core
5. building debezium-server
6. creating a docker image from debezium-server
7. pushing the image to the artifact registry.

## Installing requirements
### Installing skdman
To install sdkman you should use the official documentation of [sdkman](https://sdkman.io/install/).

### Installing docker
Docker is necessary to build the image but also for some of the tests performed when building debezium core. Follow the [official documentation](https://docs.docker.com/engine/install/) to install docker. (Docker Desktop recommended)

### Installing java and maven with sdkman
Opposing to the official documentation you need specific versions of java and maven to be able to build debezium. For java you need the version `21.0.7` and for maven `3.9.8`.

We use sdkman to install these specific versions:
```
sdk install java 21.0.7-tem
sdk install maven 3.9.8
```
(tem is the temurin version of java 21.0.7)

## Building debezium
### Building debezium core
To build debezium you need to clone (git) [debezium repository](https://github.com/cookiefactory/debezium). 
Navigate to the repository and build debezium with the following commands:
```
cd debezium
mvn clean verify -DskipITs
```
This can take up to 15 minutes to build.

### Building debezium-server
To build debezium you need to clone (git) [debezium-server](https://github.com/cookiefactory/debezium-server). 
Navigate to the repository and build debezium with the following commands:
```
cd debezium-server
mvn clean package -DskipITs -DskipTests -Passembly
```
This can take up to 15 minutes to build.
If you build debezium-core and the artefacts will be stored in the `.m2`folder. Debezium-server will lookup these artifacts to be build (dependencies).

This means that all changes in the code will be already in the artifacts. Because of the flag `-Passembly` a tar.gz will also be created. We need this for the docker image.

### Building the docker image
In the repository [debezium-server](https://github.com/cookiefactory/debezium-server) is a dockerfile.

So from the root folder of the repository you can simply perform:
```
docker build --no-cache -t thomann-debezium/server:3.2.0 .
```
to build a local docker image for debezium.

## Pushing image to artifact registry
To push the local docker image to the google cloud artifact registry you need to authenticate yourself to google with your account. Also you have to set the project. You can do this with the following commands:

```
gcloud auth login       
gcloud config set project data-orchestration-prod-yso
```   
Next you need to configure the artifact registry as remote docker registry:
```
gcloud auth configure-docker europe-west3-docker.pkg.dev       
```
Then you can tag your local image to fit the google cloud artifact registry and push it:
```     
docker tag thomann-debezium/server:3.2.0 europe-west3-docker.pkg.dev/data-orchestration-prod-yso2/common-docker/thomann-debezium/server:3.2.0 

docker push europe-west3-docker.pkg.dev/data-orchestration-prod-yso2/common-docker/thomann-debezium/server:3.2.0          
```

### Changes to code
To be able to build debezium changes needed to be made to the tests performed when building the artefacts. 
Because of different possible cpu architectures a float point precision error might occur and tests could fail.
These changes are documented [here](https://github.com/cookiefactory/debezium/commit/735b353f814ab1d208c543de8ed37466d3975d18)

### Links
https://github.com/debezium/container-images/tree/main/server/3.2

### Debugging
If you encounter problems when building the image you can get more ourput by adding the `progress` flag.
```
docker build --no-cache --progress=plain -t thomann-debezium/server:3.2.0 .
```