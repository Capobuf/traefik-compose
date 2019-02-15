# traefik-compose #

This is a boilerplate for a traefik proxy with docker as provider.

This repo is composed from 3 files: 

- docker-compose.yaml
- environments
- traefik.toml

## Getting Started ##

### Requirements ###

We are using an Ubuntu Machine (18.04.02) Hosted on DigitalOcean.

Really, it's a Droplet, with an Image of Docker pre-installed. 

### How To Clone a Repo ###

In order to using pre-esisting repository submitted by Original Creators of Traefik and more we need to clone the repository (something like download the entire folder with all file)

For do this, we need to check if `git` command it's available on our machine.
So type in terminal:

`git --version`

In case we have already installed `git` we was able to see

`git version X.XX.X`

If not, we can install `git` via apt (on debian-based distro) with:

`sudo apt-get install git`

So, for doing a "clone" of a repo we can use:

`git clone https://github.com/Capobuf/traefik-compose.git`

In this case, we clone this repo, but replacing the url we can clone another repo!

### Version of Traefik ###
We using latest image available on DockerHub Registry of Trafeik, using :latest tag on repo/imagetag command.

### DockerFile ###
A DockerFile, it's a text file, without extension, that allow us to create an instruction for build an image or run a container.

In this file, we must declare an base image from starting to work!

i.e.

```toml
FROM debian:stretch-slim
```
To this point, we can add all commands we want.
Every, single command, create a layer on image.
That's really helpful for modify and understand error!






### How To SetUp Environment Variables ###

First of all, the Env Variables are simply a dynamic variable/path of our system resources. We're using this variable to pass information from a process to other.
Usually, all Env have Capital letters, for distinguish the Env from other contex.

i.e $USER in my linux shell response something like **Capobuf**. 

But in @daton89 shell, response **daton**

As we can see, depends of environment!

Docker support import from a file for set all Env Variable direct in docker compose.
This method it's more safer & faster!


This file it's called **.env** and **must be placed in the same folder of docker compose file.**

For example, we can create two different **.env** files, one for testing, one for production environment.

Note that Dockerfile wants syntax like VAL=VAL for taking the variable, support # for commenting and blank line are simply ignored.

An example of **.env** file it's something like this:

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
As we can see in this **portion of DockerFile** we use the **env** file for declare the env variable, without importing it.


### How to Generate Hashed Passwd for Basic Auth in Traefik ###

As described in [Digital Ocean Documentation](https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-ubuntu-16-04) we can use **htpasswd** utilty for generating an hash from text.
This utility was contained in **apache2-util** package.

So first of all, install the main package:

`sudo apt-get install apache2-utils`

For Generating the Hash:

`htpasswd -nb admin your_password_here`

Output it's something like

`admin:$apr1$3jk2MbXm$ZNeJi9pjyYNsKRKLBCg1v1`

This is your hash!

Insert this string in **traefik.toml**, in this section

```toml
[web]
address = ":8080"
  [web.auth.basic]
  users = ["your_hashed_password_here"]
  ```

As example showed before, something like this:

```toml
[web]
address = ":8080"
  [web.auth.basic]
  users = ["admin:$apr1$3jk2MbXm$ZNeJi9pjyYNsKRKLBCg1v1"]
  ```

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