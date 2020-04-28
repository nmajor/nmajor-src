---
layout: post
title: FusionAuth Authentication Service with Docker
hero: ''
tags:
- authentication
- docker
date: 

---
Lack of full featured authentication and authorization solutions in the node.js world is disappointing. I've been interested in offloading all my user/identity and auth management into a service. I considered aws cognito and auth0, but from what I've read about cognito, its not easy to implement, and auth0 can be expensive.

Luckly I found a couple opensource solutions that can easily be deployed with docker. 

* [Keycloak](https://www.keycloak.org/ "https://www.keycloak.org/")
* [FusionAuth](https://fusionauth.io/ "https://fusionauth.io/")

After researching both of these, FusionAuth seemed to have easier documentation to follow and I really liked the web interface so I decided to give it a try.

# FusionAuth Demo

Trying it out is really easy, you can follow the docker setup example [here](https://fusionauth.io/docs/v1/tech/installation-guide/docker "https://fusionauth.io/docs/v1/tech/installation-guide/docker"), but basically all you have to do is run these commands:

    curl -o docker-compose.yml https://raw.githubusercontent.com/FusionAuth/fusionauth-containers/master/docker/fusionauth/docker-compose.yml
    curl -o .env https://raw.githubusercontent.com/FusionAuth/fusionauth-containers/master/docker/fusionauth/.env
    docker-compose up

If you investigate the docker-compose.yml file you'll see that it runs 3 services, a postgres service, an elasticsearch service, and the actual fusionauth service. 

After you run `docker-compose up` you can visit [http://localhost:9011/](http://localhost:9011/) and it will walk you through creating your first user.

# FusionAuth SignUp Page