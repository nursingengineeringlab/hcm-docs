# System Overview

The HMS uses a cloud of microservices and a cloud of sensors to collect, store,
and process patient data. It is then presented to patients and their health
providers using the presentation layer.

In this system overview, we introduce each module and subsystem following the
data flow.

## Cloud of Sensors

Within the HMS systems, the sensors forms a cloud on the edge of the system.
This cloud on the edge collects anonymized patient data using sensors, and
transport those data to the cloud of microservices for storage, processing and
presentation.

Each patient has sensors assigned and attached to them. Those sensors collects
vitals data of the patients, and send them out via Bluetooth Low Energy to the
edge nodes, consisting of a fleet of Raspberry Pi based low-power compute nodes.

The Raspberry Pi compute nodes performs a protocol translation by parsing the
BLE packets coming from the sensors and repackagng the information into Protobuf
format for transmission over MQTT protocol, which is the ingressed into the
Cloud of Microservices.

## Cloud of Microservices

The server side of the 
