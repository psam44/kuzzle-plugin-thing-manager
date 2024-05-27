<!--
 .. kuzzle-plugin-thing-manager documentation master file, created by
   sphinx-quickstart on Thu Mar 21 19:38:08 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.
 -->

# Kuzzle Plugin - Thing Manager

Kuzzle-Plugin-Thing-Manager (or KPTM in short) is a plugin designed for the [Kuzzle IoT Platform](https://kuzzle.io).

```{important}
This piece of software is a third party initiative, and so is **not** an official distribution from the  publisher.
```

Its purpose is similar to the official [Kuzzle Device Manager](https://docs.kuzzle.io/official-plugins/device-manager/2) plugin (KDM). It provides an API to collect information from sensors (geolocation, temperature, ...) in IoT devices. Any payload formats can be supported because you write your own decoders.

Compared to KDM, KPTM aims to be simpler, with only the basic models which are devices and measures (no assets), and more automation in order to minimize the management effort.

## Support or Contact

Support is provided by [email](mailto:maxcom@laposte.net), in French or English.

## Storage Indexes and Collections

```{note}
The data storage is provided by ElasticSearch. Here, we use the same wordings as mentioned by Kuzzle in [Internal Representation](https://docs.kuzzle.io/core/2/guides/main-concepts/data-storage/#internal-representation).
```
There are at least two indexes:

* An administrative index, acting as an entry point before routage. It is also used for definitions and configurations.
* One or more indexes, to host the mass of data.

### Admin Index

There is a single administrative index, with four collections:

    Index Name:             thing-manager
                                 │
    Collections:   ┌─────────┬───┴────┬─────────┐
                 config   models   payloads   things

The `config` collection stores configuration documents, for the plugin and for the engines.

The `models` collection stores the definitions for the registered models of device and measure.

The `payloads` collection has the purpose to keep a trace of the raw payloads, to help for a deferred investigation in case of error. Too old entries, not useful anymore, should be purged from time to time.

The `things` collection is used as a registry, to know if the device instance is already known of the application. If it is the case, then the entry gives the identifier of the engine this instance is associated with, and the operations proceed in a context of an existing device. If the instance is seen for the first time, it is automatically registered in this collection as associated with an engine (as set by the decoder, with a fallback to a default one), and the operations proceed with the creation of a new device.

### Data Index

There is at least one data index, with three collections:

    Index Name:       default-engine
                             │
    Collections:   ┌─────────┼─────────┐
                 config   devices   measures

Optionally, you may organize data across [multiple engines](advanced/multiple-engines.md).

The `config` collection is created as part of the base action for the creation of an engine, done by the [kuzzle-plugin-commons](https://github.com/kuzzleio/kuzzle-plugin-commons) package. The plugin does not store any documents there.

The `devices` collection stores one document per each managed device, including the possible [Metadata](advanced/metadata.md).

The `measures` collection receices one document per measure.

## Default Roles, Profiles and Users

Based on the [permissions architecture](https://docs.kuzzle.io/core/2/guides/main-concepts/permissions/) of the platform, the plugin makes available these default, ready to use, objects:

        Users         Profiles          Roles                   Actions
    kptm_payload
         └────────> kptm_payload
                         └────────> kptm_payload            All on the Payload Controller

                                    kptm_measure_exporter   Export measures on the Measure Controller

In the names, `kptm` is a prefix, provided by the `ThingManagerConfiguration.acronym` property.

## Contents

The `Advanced Features` section covers features that are optional, not mandatory for a basic usage or that can be deferred in usage after a first proof of operation.

```{toctree}
---
maxdepth: 2
caption: Getting Started
---
plugin
decoder
trial
```

```{toctree}
---
maxdepth: 2
caption: Advanced Features
---
advanced/multiple-measures
advanced/metadata
advanced/content-types
advanced/multiple-engines
advanced/configure
advanced/export-measures
```

```{toctree}
---
hidden:
caption: Project Links
---
Issues <https://github.com/psam44/kuzzle-plugin-thing-manager>
NPM <https://www.npmjs.com/package/kuzzle-plugin-thing-manager>
```
