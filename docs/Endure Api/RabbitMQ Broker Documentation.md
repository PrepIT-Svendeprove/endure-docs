# RabbitMQ Broker documentation

## Table of Contents
- [RabbitMQ Broker documentation](#rabbitmq-broker-documentation)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [RabbitMQ Broker Setup](#rabbitmq-broker-setup)
    - [Create a Virtual Host](#create-a-virtual-host)
    - [Create a new user](#create-a-new-user)
    - [Set Permissions](#set-permissions)
  - [AMQP Queues](#amqp-queues)
    - [Naming Convention](#naming-convention)
      - [Format](#format)
    - [Queue declaration](#queue-declaration)
    - [Queues](#queues)
  - [MQTT Topics](#mqtt-queues)
    - [Topic subscription](#topic-subscription)
    - [Consumer topics](#consumer-topics)
    - [Publish topics](#publish-topics)

## Purpose
This document describes how the RabbitMQ broker is configured and managed for the project.

It covers the setup of virtual hosts, user creation and permission assignment.
It also covers information about the AMQP queues and MQTT topics that are actively being used in the project.

## RabbitMQ Broker Setup

### Create a Virtual Host
[RabbitMQ Create Virtual Host](https://www.rabbitmq.com/docs/vhosts#using-cli-tools)

A virtual host in RabbitMQ is used to logically separate messaging environments, such as development, test, or production.

Use the following command to create a new virtual host:
```bash
rabbitmqctl add_vhost <vhost_name>
```
Example:
```bash
rabbitmqctl add_vhost endure
```

### Create a new user
[RabbitMQ User Creation Documentation](https://www.rabbitmq.com/docs/access-control#adding-a-user)

Each application or integration should use its own RabbitMQ user account where appropriate.

A new user can be created with the following command:
```bash
rabbitmqctl add_user <username> <password>
```
Passwords should be generated using:
```bash
openssl rand -base64 32
```
Example:
```bash
rabbitmqctl add_user endure <generated-password>
```
> Passwords should never be stored in plain text, and should never be re-used across users.

### Set Permissions
[RabbitMQ Permissions Documentation](https://www.rabbitmq.com/docs/access-control#grant-permissions)

After creating the user and virtual host, permissions must be assigned so the user can access the virtual host.

Use the following command:
```bash
rabbitmqctl set_permissions -p <vhost_name> <username> ".*" ".*" ".*"
```
Example:
```bash
rabbitmqctl set_permissions -p endure endure ".*" ".*" ".*"
```
The three regex values represent:
- Configure permissions
- Write permissions
- Read permissions

The example above grants full access within the specified virtual host.

## AMQP Queues

### Naming Convention
All registered queues, including future queues, should follow the [format](#format) described below. This helps ensure that queues are named consistently and remain easy to identify and manage without having to inspect the source code.

#### Format
```
{entity}.{action}
```
Example:
```
product.created
```

### Queue declaration
AMQP queues are declared automatically during application startup. The application scans for the message types decorated with the [EventQueueAttribute](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/Attributes/EventQueueAttribute.cs) and uses this information to register the required queues in RabbitMQ.

This approach makes queue registration part of the application setup and ensures that queue definitions are maintained in code alongside the related message models. It also reduces the need for manual queue creation when introducing new message types.

### Queues
Below you can see all of the queues that the application registers.

| Queue | Model |
| ----- | ----- |
| warehouse.updated | [WarehouseUpdatedEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/Warehouse/WarehouseUpdatedEventMessage.cs) |
| storageunit.created | [StorageUnitCreatedEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/StorageUnit/StorageUnitCreatedEventMessage.cs) |
| storageunit.deleted | [StorageUnitDeletedEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/StorageUnit/StorageUnitDeletedEventMessage.cs) |
| storageunit.updated | [StorageUnitUpdatedEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/StorageUnit/StorageUnitUpdatedEventMessage.cs) |
| product.created | [ProductCreatedEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/Product/ProductCreatedEventMessage.cs) |
| product.deleted | [ProductDeleteEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/Product/ProductDeleteEventMessage.cs) | 
| product.updated | [ProductUpdateEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/Product/ProductUpdateEventMessage.cs) |
| productbatch.created | [ProductBatchCreatedEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/ProductBatch/ProductBatchCreatedEventMessage.cs) |
| productbatch.deleted | [ProductBatchDeletedEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/ProductBatch/ProductBatchDeletedEventMessage.cs) |
| auditlog.created | [AuditLogCreatedEventMessage](https://github.com/PrepIT-Svendeprove/endure-api/blob/main/Endure.Dispatcher/EventMessage/AuditLog/AuditLogCreatedEventMessage.cs) |

## MQTT topics

### Topic subscription
MQTT topics are subscribed upon worker start. The workers connect to the default Virtual Host `/`, meaning the user needs to have permission to that virtual host.

### Consumer topics
This is all of the topics that the backend worker consumes.

#### Last Will
The topic is the Last Will topic, that the broker will send out, when it has not heard from a device in **X** time.
In the system it is used as a "heartbeat", to check whether or not a device is active.

The wildcard represents the ClimateDeviceId.
```
lw/{+}
```
*The topic does not include any payload.*

#### Telemetry
The topic is used for when devices collects telemetry data and publishes it to the worker(s).

The wilcard represents the ClimateDeviceId.
```
climate/telemetry/{+}
```
Topic Payload
```json
{
  "temperature": 2.2,
  "humidity": 2.2
}
```

### Publish topics
This is all of the topics that the backend worker publishes.

#### Climate control
The topic is used to control the climate (temperature and humidity) at the specified climatedevice.

The wildcard represents the ClimateDeviceId.
```
climate/control/{+}
```
Topic Payload
```json
{
  "temperature": 3.3,
  "humidity": 3.3
}
```