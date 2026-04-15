# Docker documentation

## Table of Contents
- [Docker documentation](#docker-documentation)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [Required services](#required-services)
    - [Optional services](#optional-services)
  - [Util](#util)
  - [Network](#network)
  - [Volumes](#volumes)
    - [Optional Volumes](#optional-volumes)
  - [Configure required services](#configure-required-services)
    - [PostgreSQL](#postgresql)
      - [Run PostgreSQL](#run-postgresql)
    - [RabbitMQ](#rabbitmq)
      - [Run RabbitMQ](#run-rabbitmq)
    - [Authentik Worker](#authentik-worker)
    - [Authentik Server](#authentik-server)
  - [Configure optional services](#configure-optional-services)
    - [PgAdmin](#pgadmin)
      - [Configure PgAdmin Web Interface](#configure-pgadmin-web-interface)

## Purpose
This document describes how to run and configure the Docker services that the project relies on.

It covers, creation of the Docker network, volumes and configuration.
It also covers things just as password generation that should be used when generating a new password.

## Required services
The project relies on the following services, to be configured and installed.

| Service | Docker Image |
| ------- | ------------ |
| RabbitMq | |
| Authentik Worker | |
| Authentik Server | |
| PostgreSQL | |

Furthermore the project consists of a NextJS, .NET Api and .NET Worker that also needs to be configured.

### Optional services
The project uses services, to improve quality of life, these are not necessary to run.

| Service | Docker Image |
| ------- | ------------ |
| PgAdmin4 | |

## Util
Password generation should be done using the following bash command:
```bash
openssl rand -base64 32
```

## Network
For the Docker container to be able to communicate with each other, we need to create a network that isolates the containers.
```
docker network create endure_net
```
*Creates a new network named `endure_net`, should be used when the containers.*

## Volumes
To create a volume run the command: `docker volume create <volume>`

The services relies on volumes, to store and persist their data.
| Service | Volumes |
| ------- | ------- |
| RabbitMQ | `rabbitmq_lib`, `rabbitmq_logs` |
| Authentik Worker | `authentik_custom_temp`, `authentik_data`, `authentik_certs` |
| Authentik Server | `authentik_data`, `authentik_certs` |
| PostgreSQL | `postgres_data` |

### Optional Volumes
If you have chosen to use any of the [optional](#optional-services), you need to create the volumes for the service that you have chosen.
| Service | Volumes |
| ------- | ------- |
| PgAdmin4 | `pgadmin` |

## Configure required services
*The order of the services, that you run matters as some may depend on others.*

### PostgreSQL
PostgreSQL is used by:
- .NET API / .NET Worker
- Authentik

To keep the setup separated, the application and Authentik should not share the same database user. The PostgreSQL container will create the default application database automatically, while Authentik is created through an initialization script.

Create the folder `postgres-init` and add the file `01-authentik-db.sql`, then add the following SQL into it:
```
CREATE USER authentik WITH PASSWORD '<insert generated password>';
CREATE DATABASE authentik OWNER authentik;
GRANT ALL PRIVILEGES ON DATABASE authentik TO authentik;
```
*[Generate password](#util)*

This file is mounted into the PostgreSQL container and is executed when the database is initialized for the first time.

#### Run PostgreSQL
```bash
docker run -d \
  --name endure-postgres \
  --network endure_net \
  -e POSTGRES_DB=endure \
  -e POSTGRES_USER=<PostgreSQL Username> \
  -e POSTGRES_PASSWORD='<Generated Password>' \
  -p 5432:5432 \
  -v postgres_data:/var/lib/postgresql/18/docker \
  -v ./postgres-init:/docker-entrypoint-initdb.d \
  <PostgreSQL Image>
```
*Replace the `<PostgreSQL Image>` with your local PostgreSQL image, or use `postgres:18`*

*Replace `<PostgreSQL User>` and `<Generated password>` with your own values.*

Notes:
- ``POSTGRES_DB``, ``POSTGRES_USER`` and ``POSTGRES_PASSWORD`` create the default database and user for the application.
- ``./postgres-init`` is used to create the Authentik database and user on first startup.
- The PostgreSQL data is persisted in the `postgres_data` volume.
- Init scripts only run the first time the container initializes the volume.

If PostgreSQL has already been initialized and the SQL file needs to run again, the existing container and volume must be removed first:
```bash
docker rm -f endure-postgres
docker volume rm postgres_data
```

Result:
- `endure` database for the application
- `authentik` database for Authentik

### RabbitMQ
*This documentation builds upon the [RabbitMQ Documentation](./RabbitMQ%20Broker%20Documentation.md)*

RabbitMQ is used by the .NET API and .NET Worker, to communicate between warehouses and between a local warehouse and embedded devices through AMQP and MQTT.

Create the folder `rabbitmq` and a file named `enabled_plugins`:
```
[rabbitmq_management,rabbitmq_mqtt].
```
*The file must be written in Erlang term format.*

The `rabbitmq_mqtt` plugin is required for MQTT communication.

The `rabbitmq_management` plugin is optional, but recommended if you want access to the management UI for monitoring queues, exchanges, users and connections.

To see the available RabbitMQ plugins, run the following command inside the container:
```bash
rabbitmq-plugins list
```
This command shows both available plugins and which plugins are currently enabled.

Plugins can also be enabled or disabled after the container has started, but mounting the `enabled_plugins` file makes the setup reproducible and easier to document.

#### Run RabbitMQ
```bash
docker run -d \
  --name endure-rabbitmq \
  --network endure_net \
  -e RABBITMQ_DEFAULT_USER=<RabbitMQ User> \
  -e RABBITMQ_DEFAULT_PASS=<Generated password> \
  -p 5672:5672 \
  -p 15672:15672 \
  -p 1883:1883 \
  -v rabbitmq_lib:/var/lib/rabbitmq \
  -v rabbitmq_log:/var/log/rabbitmq \
  -v ./rabbitmq/enabled_plugins:/etc/rabbitmq/enabled_plugins:ro \
  <RabbitMQ Image>
```
*Replace the `RabbitMQ Image` with your local RabbitMQ Image, or use `rabbitmq:4.2.5`*

*Replace `<RabbitMQ User>` and `<Generated password>` with your own values.*

Notes:
- Port `5672` is used for AMQP communication.
- Port `15672` is used by the RabbitMQ Management plugin.
- Port `1883` is used for MQTT communication between embedded devices and .NET Services.

### Authentik Worker

### Authentik Server

## Configure optional services

### PgAdmin
PgAdmin is an optional administration tool for PostgreSQL. It can be used to inspect databases, run queries, and verify that the PostgreSQL setup has been created correctly.

Run PgAdmin:
```bash
docker run -d \
  --name endure-pgadmin \
  --network endure_net \
  -e PGADMIN_DEFAULT_EMAIL=<PgAdmin email> \
  -e PGADMIN_DEFAULT_PASSWORD=<Generated Password> \
  -p 5050:80 \
  -v pgadmin:/var/lib/pgadmin \
  <PgAdmin Image>
```
*Replace the `PgAdmin Image` with your local PgAdmin image, or use `dpage/pgadmin4:9`*

*Replace `<PgAdmin email>` and `<Generated Password>` with your own values.*

Notes:
- Port `5050` is used to access the PgAdmin web interface.

#### Configure PgAdmin Web Interface
Open the web interface on port `5050`, with the default email and password.

To connect to PostgreSQL, create a new server in PgAdmin with:
- Host: `endure-postgres`
- Port: `5432`
- Username: `<PostgreSQL User>`
- Password: `<PostgreSQL Password>`

The host should be `endure-postgres` because PgAdmin and PostgreSQL communicate over the shared Docker network.

<!-- 
docker run -d \
  --name endure-pgadmin \
  --network endure_net \
  -e PGADMIN_DEFAULT_EMAIL=admin@admin.com \
  -e PGADMIN_DEFAULT_PASSWORD=kaN0pdW+gGThq2of1/VB9F3r5JY00PgiyAYtFIO5qkE \
  -p 5050:80 \
  -v pgadmin:/var/lib/pgadmin \
  dpage/pgadmin4:9

docker run -d \
  --name authentik_worker \
  --network endure_net \
  --restart unless-stopped \
  --shm-size=512m \
  --user root \
  -e AUTHENTIK_POSTGRESQL__HOST=endure-postgres \
  -e AUTHENTIK_POSTGRESQL__NAME=authentik \
  -e AUTHENTIK_POSTGRESQL__PASSWORD=OXJO5bDjzHpdNmozLHQwsjAZukP3xLmAovWRU8mTLK4= \
  -e AUTHENTIK_POSTGRESQL__USER=authentik_user \
  -e AUTHENTIK_SECRET_KEY=very-secret-key \
  -e AUTHENTIK_BOOTSTRAP_PASSWORD=OXJO5bDjzHpdNmozLHQwsjAZukP3xLmAovWRU8mTLK4= \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v authentik_data:/data \
  -v authentik_certs:/certs \
  -v authentik_custom_temp:/templates \
  authentik/server:2026.2.1 worker

docker run -d \
  --name authentik_server \
  --network endure_net \
  --restart unless-stopped \
  --shm-size=512m \
  --user root \
  -e AUTHENTIK_POSTGRESQL__HOST=endure-postgres \
  -e AUTHENTIK_POSTGRESQL__NAME=authentik \
  -e AUTHENTIK_POSTGRESQL__PASSWORD=OXJO5bDjzHpdNmozLHQwsjAZukP3xLmAovWRU8mTLK4= \
  -e AUTHENTIK_POSTGRESQL__USER=authentik_user \
  -e AUTHENTIK_SECRET_KEY=NW6BfjcUJyo7MIQwVCc5EgkW9E+822/erCWFZY1fQYU= \
  -v authentik_data:/data \
  -v authentik_custom_temp:/templates \
  -p 9000:9000 \
  -p 9443:9443 \
  authentik/server:2026.2.1 server -->

Client: Ph3FgsLXWaC4TAuV2I4QzFSR46FIjDniAeMQpQiv
Secret: NIBH9NZJQqAvhsEaz1D5AuDZKLWXTfkBPQsnLDqFbPTSDKcXsRmbs98TSTwGtuKkDENFe5Fj1lBcmgbK5GoEiw53QJgEFfzfjkiTePgMYdUnD4TmK29whRWdTSlNmkaP