# Serial Configuration Guide

This is a simple guide to how to configure the cooling system using the serial connection available.

To configure the device using serial, you simple have to set the following parameters, and it will try and connect at
start up or if a command has been send to call for it.

| break     | length   | message string   |
|-----------|----------|------------------|
| 0xAA 0xAA | 0x3-0x0F | `PROPERTY=VALUE` |

using the above structure of a message there are the following properties to be set.

| Property        | Type    | Value Range                                      | Description                                                                |
|-----------------|---------|--------------------------------------------------|----------------------------------------------------------------------------|
| TARGET_TEMP     | Integer | 2-(Ambient Temp)                                 | The target temperature of the cooling system.                              |
| TARGET_HUMIDITY | Integer | 40-(Ambient Humidity)                            | The target humidity of the cooling system. (NOT IMPLEMENTED)               |
| MQTT_BROKER     | String  | Max 113 Characters (if only property in message) | The MQTT brokers ip or address                                             |
| MQTT_USER       | String  | Max 48 Characters                                | The User for the MQTT broker.                                              |
| MQTT_PASS       | String  | Max 48 Characters                                | The Password for the MQTT User.                                            |
| CMD             | String  | Max (Largest CMD name)                           | Set this to run a command on target according the the command table below. |
