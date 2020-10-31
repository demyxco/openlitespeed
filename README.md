# openlitespeed
[![Build Status](https://img.shields.io/travis/demyxco/openlitespeed?style=flat)](https://travis-ci.org/demyxco/openlitespeed)
[![Docker Pulls](https://img.shields.io/docker/pulls/demyx/openlitespeed?style=flat&color=blue)](https://hub.docker.com/r/demyx/openlitespeed)
[![Architecture](https://img.shields.io/badge/linux-amd64-important?style=flat&color=blue)](https://hub.docker.com/r/demyx/openlitespeed)
[![Debian](https://img.shields.io/badge/debian-10.6-informational?style=flat&color=blue)](https://hub.docker.com/r/demyx/openlitespeed)
[![OpenLiteSpeed](https://img.shields.io/badge/openlitespeed-1.7.3-informational?style=flat&color=blue)](https://hub.docker.com/r/demyx/openlitespeed)
[![lsphp](https://img.shields.io/badge/lsphp-7.4.12-informational?style=flat&color=blue)](https://hub.docker.com/r/demyx/openlitespeed)
[![Buy Me A Coffee](https://img.shields.io/badge/buy_me_coffee-$5-informational?style=flat&color=blue)](https://www.buymeacoffee.com/VXqkQK5tb)
[![Become a Patron!](https://img.shields.io/badge/become%20a%20patron-$5-informational?style=flat&color=blue)](https://www.patreon.com/bePatron?u=23406156)

Non-root Docker image running Debian and OpenLiteSpeed.

DEMYX | OPENLITESPEED
--- | ---
TAGS | latest bedrock
PORT | 80 8080
USER | demyx
WORKDIR | /demyx
CONFIG | /etc/demyx
LOG | /var/log/demyx
ENTRYPOINT | ["dumb-init", "demyx"]
WORDPRESS | https://domain.tld/
OPENLITESPEED | https://domain.tld/demyx/ols/

## Usage
- `OPENLITESPEED_ADMIN_IP=myip` to lock down OpenLiteSpeed's admin page
- `OPENLITESPEED_ADMIN_PREFIX=false` if you're exposing ports
- `OPENLITESPEED_BASIC_AUTH_WP=true` to enable basic auth for wp-login.php
- `OPENLITESPEED_XMLRPC=true` to enable xmlrpc.php

```
# Demyx
# https://demyx.sh
#
# This docker-compose.yml is designed for VPS use with SSL/TLS first.
# Be sure to change all the domain.tld domains and credentials before running docker-compose up -d.
#
networks:
  demyx:
    name: demyx
  demyx_socket:
    name: demyx_socket
services:
  demyx_db:
    container_name: demyx_db
    depends_on:
      - demyx_traefik
    environment:
      - MARIADB_DATABASE=demyx
      - MARIADB_USERNAME=demyx
      - MARIADB_PASSWORD=demyx
      - MARIADB_ROOT_PASSWORD=demyx # Mandatory
      - MARIADB_ROOT=/demyx
      - MARIADB_CONFIG=/etc/demyx
      - MARIADB_LOG=/var/log/demyx
      - MARIADB_CHARACTER_SET_SERVER=utf8
      - MARIADB_COLLATION_SERVER=utf8_general_ci
      - MARIADB_DEFAULT_CHARACTER_SET=utf8
      - MARIADB_INNODB_BUFFER_POOL_SIZE=16M
      - MARIADB_INNODB_DATA_FILE_PATH=ibdata1:10M:autoextend
      - MARIADB_INNODB_FLUSH_LOG_AT_TRX_COMMIT=1
      - MARIADB_INNODB_LOCK_WAIT_TIMEOUT=50
      - MARIADB_INNODB_LOG_BUFFER_SIZE=8M
      - MARIADB_INNODB_LOG_FILE_SIZE=5M
      - MARIADB_INNODB_USE_NATIVE_AIO=1
      - MARIADB_INNODB_FILE_PER_TABLE=1
      - MARIADB_KEY_BUFFER_SIZE=20M
      - MARIADB_MAX_ALLOWED_PACKET=16M
      - MARIADB_MAX_CONNECTIONS=1000
      - MARIADB_MYISAM_SORT_BUFFER_SIZE=8M
      - MARIADB_NET_BUFFER_SIZE=8K
      - MARIADB_READ_BUFFER=2M
      - MARIADB_READ_BUFFER_SIZE=256K
      - MARIADB_READ_RND_BUFFER_SIZE=512K
      - MARIADB_SERVER_ID=1
      - MARIADB_SORT_BUFFER_SIZE=20M
      - MARIADB_TABLE_OPEN_CACHE=64
      - MARIADB_WRITE_BUFFER=2M
      - TZ=America/Los_Angeles
    image: demyx/mariadb
    networks:
      - demyx
    restart: unless-stopped
    volumes:
      - demyx_db:/demyx
      - demyx_log:/var/log/demyx
  demyx_socket:
    container_name: demyx_socket
    environment:
      - CONTAINERS=1
    image: demyx/docker-socket-proxy
    networks:
      - demyx_socket
    # Uncomment below if your host OS is CentOS/RHEL/Fedora
    #privileged: true
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
  demyx_traefik:
    container_name: demyx_traefik
    depends_on: 
      - demyx_socket
    environment:
      - DEMYX_ACME_EMAIL=info@domain.tld
    image: demyx/traefik
    networks:
      - demyx
      - demyx_socket
    ports:
      - 80:8081
      - 443:8082
    restart: unless-stopped
    volumes:
      - demyx_log:/var/log/demyx
      - demyx_traefik:/demyx
  demyx_wp:
    container_name: demyx_wp
    depends_on:
      - demyx_db
    environment:
      - WORDPRESS_DB_HOST=demyx_db
      - WORDPRESS_DB_NAME=demyx
      - WORDPRESS_DB_USER=demyx
      - WORDPRESS_DB_PASSWORD=demyx
      - OPENLITESPEED_ADMIN=/demyx/ols
      - OPENLITESPEED_ADMIN_PREFIX=true
      - OPENLITESPEED_ADMIN_IP=ALL # Change this to your IP
      - OPENLITESPEED_ADMIN_USERNAME=demyx
      - OPENLITESPEED_ADMIN_PASSWORD=demyx
      - OPENLITESPEED_BASIC_AUTH_USERNAME=demyx
      - OPENLITESPEED_BASIC_AUTH_PASSWORD=demyx
      - OPENLITESPEED_BASIC_AUTH_WP=false
      - OPENLITESPEED_CACHE=false
      - OPENLITESPEED_CLIENT_THROTTLE_STATIC=250
      - OPENLITESPEED_CLIENT_THROTTLE_DYNAMIC=250
      - OPENLITESPEED_CLIENT_THROTTLE_BANDWIDTH_OUT=0
      - OPENLITESPEED_CLIENT_THROTTLE_BANDWIDTH_IN=0
      - OPENLITESPEED_CLIENT_THROTTLE_SOFT_LIMIT=500
      - OPENLITESPEED_CLIENT_THROTTLE_HARD_LIMIT=750
      - OPENLITESPEED_CLIENT_THROTTLE_BLOCK_BAD_REQUEST=1
      - OPENLITESPEED_CLIENT_THROTTLE_GRACE_PERIOD=30
      - OPENLITESPEED_CLIENT_THROTTLE_BAN_PERIOD=60
      - OPENLITESPEED_CRAWLER_LOAD_LIMIT=5.2
      - OPENLITESPEED_CRAWLER_USLEEP=1000
      - OPENLITESPEED_TUNING_MAX_CONNECTIONS=1000
      - OPENLITESPEED_TUNING_CONNECTION_TIMEOUT=60
      - OPENLITESPEED_TUNING_MAX_KEEP_ALIVE=1000
      - OPENLITESPEED_TUNING_SMART_KEEP_ALIVE=1000
      - OPENLITESPEED_TUNING_KEEP_ALIVE_TIMEOUT=5
      - OPENLITESPEED_PHP_LSAPI_CHILDREN=1000
      - OPENLITESPEED_PHP_OPCACHE=true
      - OPENLITESPEED_PHP_MAX_EXECUTION_TIME=300
      - OPENLITESPEED_PHP_MEMORY=256M
      - OPENLITESPEED_PHP_UPLOAD_LIMIT=128M
      - OPENLITESPEED_RECAPTCHA_ENABLE=1
      - OPENLITESPEED_RECAPTCHA_TYPE=2
      - OPENLITESPEED_RECAPTCHA_CONNECTION_LIMIT=100
      - OPENLITESPEED_XMLRPC=false
      - TZ=America/Los_Angeles
    image: demyx/openlitespeed
    labels:
      # WordPress - https://domain.tld
      - "traefik.enable=true"
      - "traefik.http.routers.domaintld-http.rule=Host(`domain.tld`) || Host(`www.domain.tld`)"
      - "traefik.http.routers.domaintld-http.entrypoints=http"
      - "traefik.http.routers.domaintld-http.service=domaintld-http-port"
      - "traefik.http.services.domaintld-http-port.loadbalancer.server.port=80"
      - "traefik.http.routers.domaintld-http.middlewares=domaintld-redirect"
      - "traefik.http.middlewares.domaintld-redirect.redirectregex.regex=^https?:\/\/(?:www\\.)?(.+)"
      - "traefik.http.middlewares.domaintld-redirect.redirectregex.replacement=https://$${1}"
      - "traefik.http.middlewares.domaintld-redirect.redirectregex.permanent=true"
      - "traefik.http.routers.domaintld-https.rule=Host(`domain.tld`) || Host(`www.domain.tld`)"
      - "traefik.http.routers.domaintld-https.entrypoints=https"
      - "traefik.http.routers.domaintld-https.tls.certresolver=demyx"
      - "traefik.http.routers.domaintld-https.service=domaintld-https-port"
      - "traefik.http.services.domaintld-https-port.loadbalancer.server.port=80"
      # OpenLiteSpeed admin - https://domain.tld/demyx/ols/
      - "traefik.http.routers.domaintld-ols.rule=Host(`domain.tld`) && PathPrefix(`/demyx/ols/`)"
      - "traefik.http.routers.domaintld-ols.middlewares=domaintld-ols-prefix"
      - "traefik.http.middlewares.domaintld-ols-prefix.stripprefix.prefixes=/demyx/ols/"
      - "traefik.http.routers.domaintld-ols.service=domaintld-ols-port"
      - "traefik.http.services.domaintld-ols-port.loadbalancer.server.port=8080"
      - "traefik.http.routers.domaintld-ols.entrypoints=https"
      - "traefik.http.routers.domaintld-ols.tls.certresolver=demyx"
      - "traefik.http.routers.domaintld-ols.priority=99"
      # Proxy admin assets
      - "traefik.http.routers.domaintld-ols-assets.rule=Host(`domain.tld`) && PathPrefix(`/res/`)"
      - "traefik.http.middlewares.domaintld-ols-assets-prefix.stripprefix.prefixes=/demyx/ols/"
      - "traefik.http.routers.domaintld-ols-assets.service=domaintld-ols-assets-port"
      - "traefik.http.services.domaintld-ols-assets-port.loadbalancer.server.port=8080"
      - "traefik.http.routers.domaintld-ols-assets.entrypoints=https"
      - "traefik.http.routers.domaintld-ols-assets.tls.certresolver=demyx"
      - "traefik.http.routers.domaintld-ols-assets.priority=99"
    networks:
      - demyx
    restart: unless-stopped
    volumes:
      - demyx_wp:/demyx
      - demyx_log:/var/log/demyx
version: "3.7"
volumes:
  demyx_db:
    name: demyx_db
  demyx_log:
    name: demyx_log
  demyx_traefik:
    name: demyx_traefik
  demyx_wp:
    name: demyx_wp
```

## Updates & Support
[![Code Size](https://img.shields.io/github/languages/code-size/demyxco/openlitespeed?style=flat&color=blue)](https://github.com/demyxco/openlitespeed)
[![Repository Size](https://img.shields.io/github/repo-size/demyxco/openlitespeed?style=flat&color=blue)](https://github.com/demyxco/openlitespeed)
[![Watches](https://img.shields.io/github/watchers/demyxco/openlitespeed?style=flat&color=blue)](https://github.com/demyxco/openlitespeed)
[![Stars](https://img.shields.io/github/stars/demyxco/openlitespeed?style=flat&color=blue)](https://github.com/demyxco/openlitespeed)
[![Forks](https://img.shields.io/github/forks/demyxco/openlitespeed?style=flat&color=blue)](https://github.com/demyxco/openlitespeed)

* Auto built weekly on Saturdays (America/Los_Angeles)
* Rolling release updates
* For support: [#demyx](https://webchat.freenode.net/?channel=#demyx)
