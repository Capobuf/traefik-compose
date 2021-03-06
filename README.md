# traefik-compose #

This is a boilerplate for a traefik proxy with docker as provider.

This repo is composed from 3 files: 

- docker-compose.yaml
- environments
- traefik.toml

## Getting Started ##

### **Requirements** ###

A linux machine with Docker installed (>=18.09.2).

Traefik >=1.7.9

### **How To Clone the repository** ###

`git clone https://github.com/Capobuf/traefik-compose.git`

### How to Generate Hashed Passwd for Basic Auth in Traefik ###

As described in [this Digital Ocean's article](https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-ubuntu-16-04) we can use **htpasswd** utilty for generating an hash from text.
This utility was contained in **apache2-util** package.

So first of all, install the main package:

`sudo apt-get install apache2-utils`

Generating the Hash:

`htpasswd -nb admin your_password_here`

Output it's something like

`admin:$apr1$3jk2MbXm$ZNeJi9pjyYNsKRKLBCg1v1`

This is your hash!

Insert this string in **traefik.toml**, in this section

```toml
[web]
address = ":8080"
  [web.auth.basic]
  users = ["user_name:your_hashed_password_here"]
  ```

As example showed before, something like this:

```toml
[web]
address = ":8080"
  [web.auth.basic]
  users = ["admin:$apr1$3jk2MbXm$ZNeJi9pjyYNsKRKLBCg1v1"]
  ```

### Edit your configuration ###

You should edit in traefik.toml:

```toml
email = "capobuf@gmail.com"
domain = "dotroot.xyz"
```

blalbalbla

### **How To SetUp Environment Variables** ###

We prefer to store sensitive data inside enviroments variables. This is a best practice for security and reusable data across multiple containers.

We setup a **.env** file **in the root of the project.**

For example, we can create multiple **.env** files, each one for an environment: one for testing, one for production and one for staging.

This is an example of **.env** file:

```ENV
ACME_ENABLE=false
ACME_EMAIL=user@domain.tld
ACME_DOMAINS=domain.tld,www.domain.tld

DOCKER_DOMAIN="dev.domain"

TRAEFIK_DEFAULT_ENTRYPOINTS=http
TRAEFIK_ENTRYPOINT_HTTP=--entryPoints="Name:http Address::80"
TRAEFIK_ENTRYPOINT_HTTPS=
TRAEFIK_HOST=domain.local,www.domain.local
```

As we can see, all file it's ENV_Name=VALUE

Ok, we have set the **env** file, now it's turn of DockerFile

```java
version: 3.3
services:
  lb:
    image: traefik:1.5-alpine
      restart: always
      command: --web --acme.storage=/etc/traefik/acme.json --logLevel=info \
         ${TRAEFIK_ENTRYPOINT_HTTP} ${TRAEFIK_ENTRYPOINT_HTTPS}\
                 --defaultentrypoints=${TRAEFIK_DEFAULT_ENTRYPOINTS} \
                         --acme=${ACME_ENABLE} --acme.entrypoint=https --acme.httpchallenge --acme.httpchallenge.entrypoint=http \
                 --acme.domains="${ACME_DOMAINS}" --acme.email="${ACME_EMAIL}" \
                 --docker --docker.domain="${DOCKER_DOMAIN}" --docker.endpoint="unix:///var/run/docker.sock" \
                 --docker.watch=true --docker.exposedbydefault="false"
```

As we can see in this **portion of DockerFile** we use the **env** file for declare the env variable.

## How to Start the Project ##

First, we need to install a GUI for manage our container.
We choose [Portainer](https://www.portainer.io/), very useful tool!

After that, we need to install Traefik for manage all request to our server, this is for analyze, protect and secure our container (i.e. for avoid external attack, manage out-of-law request and more...)

### Understanting Docker Socket ###

First of all, we need to clarify the role of Docker (and Unix) socket.

Sometimes we need to speak directly with Docker Daemon, for manage and run some operation with it.

/var/run/docker.sock file that's the aswer for doing this.

With this path we can communicate from a Daemon in a Container with docker daemon.

An Example? Portainer.

### Understanding Difference btw Dockerfile and Docker-compose.yml ###

In Dockerfile we can write all our settings and parameters, from an **image**, for build another **image**, **not a container**.

In Docker-compose.yaml we can write all settings and parameters for building a **container**, from an image.

Usually, the docker-compose.yaml it's pretty useful for building a Service, that's something like a Stack of Container, wich is linked togheter, without too munch work.

But it's same useful for understanding better all the components we using and debug better in case of failure.

### Understanding Difference btw Docker Start, UP and Build ###

`docker-compose up` it's the best choice. Start/Restart all service and stuff in docker-compose.yaml

`docker build` essentially, telling docker to create an image from our dockerfile.

`docker run` essentially telling docker to create a container with the image we specified, with all mod in dockerfile, even it come from DockerHub.

PS: It's a very Quick&Simple explanation, even some command here appear similar, each one have a specific flag and field of use.

### Install & Configure Portainer ###

Portainer Docu was cleary on the installation process:

``docker volume create portainer_data``

For creating a Volume for Portainer named `portainer_data`.

``docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer``

For Create a Container, in deattached mode, with 9000 as exposed port, listening docker socket, with an volume attached as `portainer_data`

More detailed:

`-d` run Daemon in Background

`-p 9000:9000` expose port 9000 of the container as port 9000 of the server.

`-v /var/run/docker.sock:/var/run/docker.sock` pass Docker Socket to Portainer for manage and run container directly from Portainer. the `-v` flag it's used for attach a volume

`-v portainer_data:/data portainer/portainer` for attach another storage to our container, called `portainer_data`

After that, we can go into server_public_ip:9000 for showing first screen of Portainer, with possibility to set our credentials for login.

We must select "Local" option, for manage our docker installation.

PS: The setup, tells us to pass Docker socket to Portainer, in local option, but we did already :)

### Installing and Configure Traefik ###

In order to install and configure Traefik we need to create a docker-compose.yml file.

Keeping in mind all stuff we learned we know:

  - We must select an image to start from.
  - We must set a Container Name
  - We can add some flag in our container
  - We must declare a Network to Join our container (or going in the default "bridge" Network)
  - We can expose some port
  - We must attach some volume for pass socket, traefik.toml and other config file

  So, if u want to see the docker-compose.yml of our Traefik Container, it's in root of this folder!

  

### Installing Portainer ###

### How To Start a Project ###

Describe here how to getting started:

- ~~requirements, linux machine with linux and docker version xxx~~
- ~~version of traefik etc~~
- ~~how to clone repo~~
- ~~how to set up environment variables~~
- ~~how to generate your hashed password for basic auth of traefik admin panel~~
- hwo to start project
- how to check that everyting is working
- future readings and related projects

## traefik.toml ##

The configuration file of traefik, the _traefik.toml_ file, has 5 main sections:

________________________

### Global configuration ###

Global variables of trafik, like defaultEntrypoints etc.
________________________

### Web configuration backend ###

Enable we service, with traefik control panel on port 8080, with basic auth, statistics and last 20 errors logs.

```toml
[web]
address = ":8080"
  [web.auth.basic]
  users = ["admin:YOUR HASHED PASSWORD HERE"]
  [web.statistics]
  recentErrors = 20
```

________________________

### Docker configuration backend ###

In this project we are using traefik with docker provider, for more information see [traefik docs](https://link)

```toml
[docker]
domain = "your domain here"
watch = true
exposedbydefault = false
```

________________________

### Entrypoints definition (Force HTTPS) ###

To force https over http we need to set entrypoints for http and https, and redirect http over https.

```toml
[entryPoints]
  [entryPoints.http]
  address = ":80"
    [entryPoints.http.redirect]
      entryPoint = "https"
  [entryPoints.https]
  address = ":443"
    [entryPoints.https.tls]
```

________________________

### Let's Encrypt certificate support for Traefik ###

Finally we configure the ACME section to enable traefik to generate certificates for us!

```toml
[acme]
email = "your email"
dnsProvider = "your dns provider"
delayDontCheckDNS = 0
storage = "acme.json"
entryPoint = "https"
onHostRule = true
onDemand = false
```

## Contributing ##

Contributors are welcome, feel fre to fork and open pull requests to collaborate with us!