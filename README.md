# traefik-compose #

This is a boilerplate for a traefik proxy with docker as provider.

This repo is composed from 3 files: 

- docker-compose.yaml
- environments
- traefik.toml

## Getting Started ##
________________________


### **Requirements** ###

A linux machine with Docker installed (>=18.09.2).

Traefik >=1.7.9

### How To Clone the repository ###

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
________________________

### Edit your configuration ###

You need to insert your personal data in .env file, for docker-compose file and traefik.toml settings:

```toml
email = "your@email.com"
domain = "yourdomain.xyz"
```


________________________

### **How To SetUp Environment Variables** ###

We prefer to store sensitive data inside enviroments variables. This is a best practice for security and reusable data across multiple containers.

We setup a **.env** file **in the root of the project.**

This is an example of **.env** file:

```ENV
WP_CONTAINER_NAME=projectname_wp
DB_CONTAINER_NAME=projectname_db
DB_PASSWD=your_password
```

And this is an example of application of .env file in a docker-compose.yml

```YML
version: '3.1'

services:

  main:
    image: image
    container_name: $WP_CONTAINER_NAME #Here
    networks:
      - net
    environment:
      WORDPRESS_DB_PASSWORD: $DB_PASSWD #Here
      WORDPRESS_DB_HOST: db:3306

    labels:
      - "traefik.backend = $WP_CONTAINER_NAME" #Here
      - "traefik.docker.network = net"
      - "traefik.frontend.rule = Host:nameofservice.yourdomain.xyz"
      - "traefik.enable = true"
      - "traefik.port = 80"
```

________________________

## How to Start the Project ##

### Installing Traefik ###

We need to install Traefik for manage all request to our server, this is for analyze, protect and secure our container (i.e. for avoid external attack, manage out-of-law request and more...)

```YML
version: '3'

### Here we Have Traefik ###

services:
 reverse-proxy:             #Name of Service
  image: traefik            #The official Traefik docker image
  
  container_name: traefik   #Name of Container
    
  command: --api --docker   #Enables the web UI and tells Træfik to listen to docker
    
  networks:                 #Connected Networks
   - prx                    #Name of the Network

  ports:                    #Exposed port
   - "80:80"                #Expose HTTP port
   - "443:443"              #Expose HTTPS port

  labels:                   #Labels needed for pass some value to traefik
   - traefik.frontend.rule=Host:proxy.yourdomain.xyz #Rule for hostname redirection
   - traefik.port=8080      #Port that Traefik must use
    
  volumes:                  #Declare volumes to attach to container
   - /var/run/docker.sock:/var/run/docker.sock #Pass docker socket to Traefik
   - ./traefik.toml:/traefik.toml #Pass toml configuration as volume
   - ./acme.json:/acme.json #ACME Settings for let's encrypt feature

```

________________________


### Installing Portainer ###
First, we need to install a GUI for manage our container.
We choose [Portainer](https://www.portainer.io/), very useful tool!

```YML
version: '3'

### Here we Have Portainer ###

 container-manager:           #Name of Service

  image: portainer/portainer  #Official Portainer Image from Docker Registry

  container_name: portainer   #Container Name

  restart: always             #Policy of restart, try to restart every time go down

  depends_on:                 #Wait follow service before starting
   - reverse-proxy            #Wait Traefik for starting

  networks:                   #Declare wich network join container
   - prx                      #Name of Network
      
  ports:                      #Ports to be exposed
   - "9000:9000"              #Declare port for WebUI

  volumes:                    #Declare volumes to attach to container
   - /var/run/docker.sock:/var/run/docker.sock #Here we pass docker socket to Portainer
   - portainer_data:/data     #Define portainer_data for make persistent session of portainer

  labels:                     #Labels needed for pass some value to traefik
   - "traefik.backend=portainer" #Give the name for backend
   - "traefik.frontend.rule=Host:portainer.yourdomain.xyz" #Rule for hostname redirection
   - "traefik.port=9000"      #Port that traefik must use
   - "traefik.enable=true"    #Enable traefik on this container
   - "traefik.docker.network=prx" #Specify wich network Traefik must use for connect to container

volumes:                      #Declare volume
  portainer_data:             #Declared for Portainer data
```


The complete docker-compose.yml it's available on the root directory.
________________________

### Installing Wordpress ###

```YML
version: '3.1'

services:

  wordpress:
    image: wordpress  #Image of Wordpress
    container_name: $WP_CONTAINER_NAME #Name of Container
    depends_on: 
      - database  #Waiting Database for Starting
    networks: #Network to connecting
      - wordp #Name of Network
    environment:
      WORDPRESS_DB_PASSWORD: $MYSQL_PASSWORD  #DB Password For Connecting to Database
      WORDPRESS_DB_HOST: database:3306  #Connect parameters for database
    volumes:
      - ./config/php.conf.uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
      - ./wp-app:/var/www/html  #Entire 
    labels:
      - "traefik.backend=$WP_CONTAINER_NAME" #Name of Backend for Traefik
      - "traefik.docker.network=wordp"  #Select default network for Traefik Communication with container
      - "traefik.frontend.rule=Host:nameofservice.yourdomain.xyz" #Url for reaching container
      - "traefik.enable=true" #Enable Traefik expose on this container
      - "traefik.port=80" #Select port to use in Traefik

### Here we have Database ###

  database:
    image: mariadb  #Image of MariaDB
    container_name: $DB_CONTAINER_NAME #Name of Container
    networks: #Network to Connecting
      - wordp #Name of network to connect
    
    environment:
      MYSQL_ROOT_PASSWORD: $MYSQL_PASSWORD #Database Root Password
      MYSQL_USER: $MYSQL_USER #User of Database
    
    volumes:
      - ./wp-data:/docker-entrypoint-initdb.d #Database data

networks: #Declare Networks
  wordp:  #Name of network
    external: false #Create Network
```

As we can see, there's some $ENV_VARIABLE, that's variable are explained in .env file in "Wordpress" directory.

More detail about Wordpress installation behind Traefik are explained in it's Readme.md file.

### How To Start a Project ###

Describe here how to getting started:

- hwo to start project
- how to check that everyting is working
- future readings and related projects

## traefik.toml ##

The configuration file of traefik, the _traefik.toml_ file, has 5 main sections:

________________________

### Global configuration ###

Global variables of trafik. 
In this first area we can setting debug settings, select default entrypoints and more.

Here an example:
```toml
debug=true  #Enable Debug

logLevel = "ERROR"  #Select Debug Level

defaultEntryPoints = ["http", "https"] #Default Entrypoint definition
```
________________________

### Web configuration backend ###

Enable web service, with traefik control panel on port 8080, with basic auth, statistics and last 20 errors logs.

```toml
[entryPoints] #Define EntryPoint
  [entryPoints.traefik] #Define Traefik EntryPoint
    address = ":8080" #Define Port
    [entryPoints.traefik.auth]  
      [entryPoints.traefik.auth.basic]  #Define Basic Auth
        users = ["user_name:your_hashed_password_here"]
```
________________________



### Redirect all Request in HTTPS (Force HTTPS) ###

We can redirect all HTTP request to HTTPS.

```toml
  [entryPoints.http]
    address = ":80"
      [entryPoints.http.redirect]
        entryPoint = "https"
  [entryPoints.https]
    address = ":443"
      [entryPoints.https.tls]
```
________________________


### Traefik Settings (API) ###
Here we can define some global settings for Traefik.

```toml
[api]

  entryPoint = "traefik" #Name of default entrypoint
  dashboard = true #Enable Dashboard
```

________________________

### Docker Configuration Backend ###

In this project we are using traefik with docker provider, for more information see [Traefik Docs](https://docs.traefik.io/configuration/backends/docker/)

```toml
[docker] #Define Docker Part

domain = "your domain here" #Default Based Domain

watch = true #Watch changes of docker

exposedbydefault = false #Expose only container with label "traefik.enable=true"
```

________________________

### Let's Encrypt certificate support for Traefik ###

Finally we configure the ACME section to enable traefik to generate certificates for us, using Let's Encrypt service.

```toml

[acme]
email = "your email"
storage = "acme.json"
entryPoint = "https" #Entrypoint to apply Certificate
onHostRule = true #Generate Certificate for all exposed container
```

## Contributing ##

Contributors are welcome, feel fre to fork and open pull requests to collaborate with us!