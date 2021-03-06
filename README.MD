# Dockerized Keycloak Server
A (hopefully) simplified approach to standing up a Keycloak server with SSL.

This repo contains a Docker Compose setup for standing up a Keycloak instance and a Postgres DB, with proxied behind Nginx for SSL and the common ENV variables ready to be configured.  The scripts here assume an Ubuntu-like OS, but can be modified for others without much issue.

## Setup
Setup is pretty straightforward, we need to ensure that Docker is installed and then it handles the rest.  SSL is also pretty quick, but does take a few more steps.

### tl;dr:
- `git clone https://github.com/vbhayden/docker-keycloak-server`
- `cd docker-keycloak-server`
- `sudo ./install-reqs.sh`
- `cp example.env .env`
- Populate the `.env` file (see below)
- `sudo ./init-ssl.sh <HOSTNAME>` (see below)
- `sudo docker-compose up -d --build`

### Setting up the Admin Account
This uses a newer version of the Keycloak Docker image, but may still require a restart for the admin user to go through.  This is a bit annoying, but if your admin user doesn't work with the initial admin sign-in, then run

```bash
sudo docker-compose restart keycloak
sudo docker logs keycloak -f 

# wait until you see the Wildfly (X of Y started successfully) message
sudo docker-compose restart nginx
```

### Setting the Config Values
The repo comes with an `example.env` file that you will want to (but really should) configure.  Each service has a few properties exposed by default, but you can refer to the Docker Hub docs for each to add more.

Service|Variable|Why am I setting this?
-|-|-
**Nginx**|`HOSTNAME`|Domain name of the machine used to host this stack, can be `localhost`
**Keycloak**|`KC_ADMIN_USERNAME`|Admin username for Keycloak
-|`KC_ADMIN_PASSWORD`|Admin password for Keycloak
**Postgres**|`KC_DB_USERNAME`|DB username
-|`KC_DB_PASSWORD`|DB password

### Setting up SSL
This stack uses Certbot and Nginx to handle SSL, including temporary self-signed certs to help the process along.  We run the `init-ssl.sh` script during setup to place these into a directory corresponding to Nginx's `HOSTNAME` variable.  

With that, the stack can start without issue but to get an actual SSL cert from Certbot:
```
sudo ./certbot/generate.sh <HOSTNAME>
```
