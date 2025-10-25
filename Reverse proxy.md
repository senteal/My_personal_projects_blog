
A reverse proxy could be very help full for my vps lab setup. I run multiple services with webpage on it. I want to access different services by typing subdomain.mydomain.com in the web browser. Also, I need a tls encryption to access them by https for a secured service.

I choose [ Linux/SWAG image](https://hub.docker.com/r/linuxserver/swag) because It provides with a list of preset reverse proxy confs for popular apps and services, including [Mailu](obsidian://open?vault=projects&file=Mailu%2FMailu%20-%20A%20lightweight%20opensource%20SMPT%20server), [vaultwarden](vaultwarden)  and many other services I will use in the future. And it's perfect for container environment. 

## Step 1 - Docker network setup

First we need to create a bridged network for containers to communicate. 

`docker network create dockernet`

this will create a network called "dockernet" for containers to communicate.



## Step 2 - docker compose and join network with aliases

This is the templet I use:

docker-compose.yml
```
version: "2.1"
services:
  swag:
    image: lscr.io/linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=yourdomain.url #configure your domain
      - SUBDOMAINS=www,    #configure the subdomain you wish to use
      - VALIDATION=http
      - CERTPROVIDER= #optional
      - DNSPLUGIN=cloudflare #optional
      - EMAIL=<e-mail> #optional
      - ONLY_SUBDOMAINS=false #optional
      - EXTRA_DOMAINS=<extradomains> #optional
      - STAGING=false #optional
    volumes:
      - </path/to/appdata/config>:/config
    ports:
      - 443:443
      - 80:80 #optional
    restart: unless-stopped
```

start the container using  : `docker compose up -d`

using this command to join dockernet with an alias. So container can access each other by referring the alias, witch will be used in nginx confs 
`docker network connect --alias swag dockernet swag`

## Step 3 proxy-confs


after starting SWAG

```
docker exec -it swag /bin/bash
cd config/nginx/proxy-confs
```
locate proxy-confs directory in swag container. find the preset of the service. In this case, is `mailu.subdomain.conf.sample` :

```
## Version 2025/07/18
# make sure that your mailu container is named front
# make sure that your dns has a cname set for mailu

server {
    listen 443 ssl;
#    listen 443 quic;
    listen [::]:443 ssl;
#    listen [::]:443 quic;

    server_name mailu.*;

    include /config/nginx/ssl.conf;

    client_max_body_size 0;

    # enable for ldap auth (requires ldap-location.conf in the location block)
    #include /config/nginx/ldap-server.conf;

    # enable for Authelia (requires authelia-location.conf in the location block)
    #include /config/nginx/authelia-server.conf;

    # enable for Authentik (requires authentik-location.conf in the location block)
    #include /config/nginx/authentik-server.conf;

    # enable for Tinyauth (requires tinyauth-location.conf in the location block)
    #include /config/nginx/tinyauth-server.conf;

    location / {
        # enable the next two lines for http auth
        #auth_basic "Restricted";
        #auth_basic_user_file /config/nginx/.htpasswd;

        # enable for ldap auth (requires ldap-server.conf in the server block)
        #include /config/nginx/ldap-location.conf;

        # enable for Authelia (requires authelia-server.conf in the server block)
        #include /config/nginx/authelia-location.conf;

        # enable for Authentik (requires authentik-server.conf in the server block)
        #include /config/nginx/authentik-location.conf;

        # enable for Tinyauth (requires tinyauth-server.conf in the server block)
        #include /config/nginx/tinyauth-location.conf;

        include /config/nginx/proxy.conf;
        include /config/nginx/resolver.conf;
        set $upstream_app front;
        set $upstream_port 80;
        set $upstream_proto http;
        proxy_pass $upstream_proto://$upstream_app:$upstream_port;

    }
}
```

on line `server_name mailu.*;` change "mailu" to the chosen subdomain. 
on `set $upstream_app front;` the name should be the same as your specified alias for the service container. 

restart swag: `docker restart swag`

## step 4 - target services settings

take mailu service as example here

### docker-compose.yml configuraiton.

We don't need to expose 80 and 443 port as they will be proxied by swag. 

```
version: '3.3'
services:

  # External dependencies
  redis:
    image: redis:alpine
    restart: always
    volumes:
      - "/mailu/redis:/data"
    depends_on:
      - resolver
    dns:
      - 192.168.203.254

  # Core services
  front:
    image: ${DOCKER_ORG:-ghcr.io/mailu}/${DOCKER_PREFIX:-}nginx:${MAILU_VERSION:-2024.06}
    restart: always
    env_file: mailu.env
    logging:
      driver: journald
      options:
        tag: mailu-front
    ports:
 #     - "127.0.0.1:18080:80"
 #     - "127.0.0.1:18443:443"
      - "185.122.164.210:25:25"
      - "185.122.164.210:465:465"
      - "185.122.164.210:587:587"
        
......
```

for mailu service, we need one addition adjustment;

we need to change secure connection behavior to `TLS_FLAVOR=mail-letsencrypt`. This can be changed in `mailu.env` or mailu docker compose file wizard.

this means mail protocols will be encrypted by mailu container itself. An addition TLS encrypt  required when visiting web interface.  SWAG automatically registers a certificate for the subdomain specified in docker-compose.yml when load up. This will be used by nginx proxy and meet the requirement for visiting mailu web interface.

SSL/TLS encryption is crucial for secured webservice. many apps force to use https to prevent reckless server admins from implementing vulnerable settings .
