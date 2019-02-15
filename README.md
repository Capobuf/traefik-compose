# traefik-compose #

This is a boilerplate for a traefik proxy with docker as provider.

This repo is composed from 3 files: 

- docker-compose.yaml
- environments
- traefik.toml

## Getting Started ##

Describe here how to getting started:

- requirements, linux machine with linux and docker version xxx
- version of traefik etc
- how to clone repo
- how to set up environment variables
- how to generate your hashed password for basic auth of traefik admin panel
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