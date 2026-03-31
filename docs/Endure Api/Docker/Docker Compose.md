## Table of contents
- [Table of contents](#table-of-contents)
  - [Purpose](#purpose)
  - [Included services](#included-services)
    - [postgres:](#postgres)
    - [pgadmin:](#pgadmin)
    - [rabbitmq:](#rabbitmq)
  - [Docker Compose configuration](#docker-compose-configuration)

### Purpose
The Docker Compose file is used to start the services required for running the Endure system in a local development environment.

For production, go to [Deploy to production](/docs/Endure%20Api/Docker/Deploy%20to%20production.md)

### Included services
The docker compose file contains the services used by the Endure system in a local development environment.

*This does not contain a deeper explanation of the ports used, go to [Services](/docs/Endure%20Api/Docker/Services.md) to get that.*

#### postgres:
This service runs a PostgreSQL instance used to store data collected by the Endure system.

**postgres default credentials**:
- Database: endure 
- Username: endure 
- Password: endure

#### pgadmin:
This service runs pgAdmin 4, which provides a web based management interface for PostgreSQL. It is optional and is mainly used for inspecting and managing the local database.

**pgadmin default credentials**:
- Username: endure@endure.com
- Password: endure

#### rabbitmq:
This service runs RabbitMQ, which is used as the message broker for AMQP and MQTT communication between IoT devices and the Endure system across warehouses.

**rabbitmq default credentials**:
- Username: endure 
- Password: endure

*The credentials provided for the services are only used in local development.*

### Docker Compose configuration
The full Docker Compose configuration used for local development is shown below:
```yaml
name: Endure

services:
  postgres:
    image: postgres:18
    environment:
      POSTGRES_DB: endure
      POSTGRES_USER: endure
      POSTGRES_PASSWORD: endure
    volumes:
      - postgres:/var/lib/postgresql/18/docker
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U endure -d endure -h localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    ports:
      - "5432:5432"

  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: endure@endure.com
      PGADMIN_DEFAULT_PASSWORD: endure
    volumes:
      - pgadmin:/var/lib/pgadmin
    ports:
      - "5050:80"
  
  rabbitmq:
    image: rabbitmq:4.2.5
    environment:
      RABBITMQ_DEFAULT_USER: endure
      RABBITMQ_DEFAULT_PASS: endure
    configs:
      - source: rabbitmq-plugins
        target: /etc/rabbitmq/enabled_plugins
    volumes:
      - rabbitmq-lib:/var/lib/rabbitmq
      - rabbitmq-log:/var/log/rabbitmq
    ports:
      - "5672:5672"     
      - "15672:15672"   
      - "1883:1883"     

configs:
  rabbitmq-plugins:
    content: "[rabbitmq_management,rabbitmq_mqtt]."
    
volumes:
  postgres:
  pgadmin:
  rabbitmq-lib:
  rabbitmq-log:
```