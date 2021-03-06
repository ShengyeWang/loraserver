# Changelog

## Development

**Changes:**

* RU 864 - 869 band configuration has been updated (see [#113](https://github.com/brocaar/loraserver/issues/113))

## 0.13.3

**Features:**

* The following band configurations have been added:
	* AS 923
	* CN 779 - 787
	* EU 433
	* KR 920 - 923
	* RU 864 - 869
* Flags for repeater compatibility configuration and dwell-time limitation
  (400ms) have been added (see [configuration](configuration.md))

## 0.13.2

**Features:**

* De-duplication delay can be configured with `--deduplication-delay` or
  `DEDUPLICATION_DELAY` environment variable (default 200ms)
* Get downlink data delay (delay between uplink delivery and getting the
  downlink data from the application server) can be configured with
  `--get-downlink-data-delay`  or `GET_DOWNLINK_DATA_DELAY` environment variable

**Bugfixes:**

* Fix duplicated gateway MAC in application-server and network-controller API
  call

## 0.13.1

**Bugfixes:**

* Fix crash when node has ADR enabled, but it is disabled in LoRa Server

## 0.13.0

**Features:**

* Adaptive data-rate support. See [features](features.md) for information about
  ADR. Note:
  
	* [LoRa App Server](https://docs.loraserver.io/lora-app-server/) 0.2.0 or
      higher is required
	* ADR is currently only implemented for the EU 863-870 ISM band
	* This is an experimental feature

**Fixes:**

* Validate RX2 data-rate (this was causing a panic)

## 0.12.5

**Security:**

* This release fixes a `FCnt` related security issue. Instead of keeping the
  uplink `FCnt` value in sync with the `FCnt` of the uplink transmission, it
  is incremented (uplink `FCnt + 1`) after it has been processed by
  LoRa Server.

## 0.12.4

* Fix regression that caused a FCnt roll-over to result in an invalid MIC
  error. This was caused by validating the MIC before expanding the 16 bit
  FCnt to the full 32 bit value. (thanks @andrepferreira)

## 0.12.3

* Relax frame-counter option.

## 0.12.2

* Implement China 470-510 ISM band.
* Improve logic to decide which gateway to use for downlink transmission.

## 0.12.1

* Fix multiple LoRa Server instances processing the same gateway payloads
  (resulting in the gateway count multiplied by the number of LoRa Server
  instances).

## 0.12.0

This release decouples the node "inventory" part from LoRa Server. This
introduces some breaking (API) changes, but in the end this will make it easier
to integrate LoRa Server into your own platform as you're not limited anymore
by it's datastructure.

### API

Between all LoRa Server project components [gRPC](http://gprc.io) is used
for communication. Optionally, this can be secured by (client) certificates.
The RESTful JSON api and api methods to manage channels, applications and nodes
has been removed from LoRa Server. The node-session api methodds are still
part of LoRa Server, but are only exposed by gRPC.

### Application-server

An application-server component and [API](https://github.com/brocaar/loraserver/blob/master/api/as/as.proto)
was introduced to be responsible for the "inventory" part. This component is
called by LoRa Server when a node tries to join the network, when data is
received and to retrieve data for downlink transmissions.

The inventory part has been migrated to a new project called
[LoRa App Server](http://docs.loraserver.io/lora-app-server/). See it's
changelog for instructions how to migrate.

### Configuration

As components have been dropped and introduced, you'll probably need to update
your LoRa Server configuration. 

### Important

Before upgrading, make sure you have a backup of all data in the PostgreSQL
and Redis database!

## 0.11.0

* Implement receive window (RX1 or RX2) and RX2 data-rate option in node and
  node-session API (and web-interface).

## 0.10.1

* Fix overwriting existing node-session (owned by different DevEUI)
  (thanks @iBrick)

## 0.10.0

* Implement (optional) JWT token authentication and authorization for the gRPC
  and RESTful JSON API. See [api documentation](https://docs.loraserver.io/loraserver/api/).
* Implement support for TLS
* Serve the web-interface, RESTful interface and gRPC interface on the same port
  (defined by `--http-bind`). When TLS is disabled, the gRPC interface is
  served from a different port (defined by `--grpc-insecure-bind`).
* Fix: delete node-session (if it exists) on node delete

## 0.9.2

* Fix Swagger base path.

## 0.9.1

* Fix `cli.ActionFunc` deprecation warning.

## 0.9.0

**WARNING:** if you're using the JSON-RPC interface, this will be a breaking
upgrade, as the JSON-RPC API has been replaced by a gRPC API.

In order to keep the possiblity to access the API from web-based applications
(e.g. the web-interface), a RESTful JSON API has been implemented on top
of the gRPC API (using [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)).

Please refer to the LoRa Server documentation for more information:
[https://docs.loraserver.io/loraserver/api/](https://docs.loraserver.io/loraserver/api/).

## 0.8.2

* Validate the join-request DevEUI belongs to the given AppEUI
* Implement `Node.FlushTXPayloadQueue` API method
* Update `GatewayStatsPacket` struct (`CustomData` and `TXPacketsEmitted`, to
  be implemented by the lora-gateway-bridge).


## 0.8.1

* Bugfix: 'fix unknown channel for frequency' error when using custom-channels (`CFList`)
  (thanks @arjansplit)

## 0.8.0

* Implement network-controller backend
* Implement support for sending and receiving MAC commands (no support for proprietary commands yet)
* Refactor test scenarios
* Web-interface: nodes can now be accessed from the applications tab (nodes button)

**Note:** You need to update to LoRa Semtech Bridge 2.0.1+ or 1.1.4+ since
it fixes a mac command related marshaling issue.

## 0.7.0

* Complete join-accept payload with:
	* RXDelay
	* DLSettings (RX2 data-rate and RX1 data-rate offset)
	* CFList (optional channel-list, see LoRaWAN specs to see if this
	  option is available for your region)

  All values can be set / created throught the API or web-interface

## 0.6.1

* Band configuration must now be specified with the ``--band`` argument
  (no more separate binaries per ism band)
* RX info notifications (``application/[AppEUI]/node/[DevEUI]/rxinfo``)

## 0.6.0

* Implement various notifications to the application:
	* Node join accept (``application/[AppEUI]/node/[DevEUI]/join``)
	* Errors (e.g. max payload size exceeded) (``application/[AppEUI]/node/[DevEUI]/error``)
	* ACK of confirmed data down (``application/[AppEUI]/node/[DevEUI]/ack``)
* Handle duplicated downlink payloads (when running multiple LoRa Server instances each server
  is receiving the TXPayload from MQTT, just one needs to handle it)
* New ISM bands:
	* US 902-928 band (thanks @gzwsc2007 for testing)
	* AU 915-928 band (thanks @Mehradzie for implementing and testing)
* Fix: use only one receive-window (thanks @gzwsc2007)

## 0.5.1

* Expose RX RSSI (signal strength) to application
* Provide binaries for multiple platforms

## 0.5.0

Note: this release is incompatible with lora-semtech-bridge <= 1.0.1

* Replaced hardcoded tx related settings by lorawan/band defined variables
* Minor changes to TX / RX structs
* Change gateway encoding to json (from gob encoding)
* Source-code re-structure (internal code is now under `internal/...`,
  exported packet related structs are now under `models/...`)

## 0.4.1

* Update mqtt vendor to fix various connection issues
* Fix shutting down server when mqtt server is unresponsive

## 0.4.0

* Implement confirmed data up
* Implement (confirmed) data down
* Implement graceful shutdown
* Re-subscribe on mqtt connection error (thanks @Magicking)
* Fix FCnt input bug in web-interface (number was casted to a string, which was rejected by the API)

## 0.3.1

* Bugfix related to ``FCnt`` increment (thanks @ivajloip)

## 0.3.0

* MQTT topics updated (`node/[DevEUI]/rx` is now `application/[AppEUI]/node/[DevEUI]/rx`)
* Restructured RPC API (per domain)
* Auto generated API docs (in web-interface)

## 0.2.1

* `lorawan` packet was updated (with MType fix)

## 0.2.0

* Web-interface for application and node management
* *LoRa Server* is now a single binary with embedded migrations and static files

## 0.1.0

* Initial release
