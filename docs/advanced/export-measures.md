# Export Measures

The Export action is a facility to retrieve measures from the storage as a CSV stream.

A set of query parameters allows to filter the objects in order to get only those worthy of interest.

The general syntax of the request is:

    GET /_/thing-manager/measure/export[?key1=val1[&key2=val2[...]]]

## Permission

With a default installation in a development environment, you can choose to stay an anonymous user to experiment the exports, as it has a role giving access to all API actions.

In a stricter environment, the permission must be granted to users through profiles. For that, a `kptm_measure_exporter` role is predefined by the plugin to allow this API action on the measures controller.

## Query Parameters

In case of more than one parameter, the applicable logic is AND.

### Engine Criteria

`n=engineId` targets an e**N**gine. Refer to [Mutiple Engines](multiple-engines.md) for more. Defaults to the default engine defined in the plugin configuration.

### Time Criteria

If the time part is not set, it defaults to 00:00:00. More generally, the supported format est ISO 8601.

`s=yyyy-mm-dd[Thh:mm:ss]` defines the **S**tart date (inclusive).

`e=yyyy-mm-dd[Thh:mm:ss]` defines the **E**nd date (inclusive).

If at least one of the criterias is set, then a range filtering is applied to the `measuredAt` property of the measure.

### Type Criteria

`t=xxx` defines the **T**ype of the measure, as its `type` property.

### Device Criteria

The purpose of these filtering criterias is to restrict the origin of measures to one device or to one model of devices.

`r=reference` defines the **R**eference of a device, with an implicit default model, which is the one of the first registered decoder. This is particularly useful when there is a single decoder.

`m=model` defines the **M**odel of devices. If `r` is also set, it supersedes the default model for the identifying of a single device. Otherwise it will target any device belonging to this model. Obviously the criteria is useless for applications with a single decoder.

## Proposed Filename

When the stream is received, the navigator opens a dialog to save it to a file. The proposed filename depends on the query parameters.

    measures[-type][-model][-reference].csv

The `type` part is present if the `t` query parameter is provided.

The `model` part is present if the `m` or `r` query parameter is provided.

The `reference` part is present if the `r` query parameter is provided.

## Column Headers

The presence and the naming of columns depend on the possible variation of the cell value and on the query parameters.

    id, measuredAt[, measureType][, model][, reference], ...<measure values keys>

The `measureType` column is present if the data may include various measure types, because there are more than a single registered measure model and the `t` query parameter is not provided.

The `model` column is present if the data may include various device models, because there are more than a single registered device model and neither the `m` nor `r` query parameters are provided.

The `reference` column is present if the `r` query parameter is not provided.

The `measure values keys` are the collection of keys from the `valuesMappings` property of one or all registered measure models, with the following rules:

- If the `t` query parameter is provided, only the keys for this type are considered.
- When the column already exists for the same key and the same indexing type ('keyword', 'integer', ...), it is not duplicated but reused.
- When the key has a 'geo_point' indexing type, two columns are generated: `<key>.lat` and `<key>.lon`.

## Cell Values

The `id` cell contains the document unique identifier, automatically generated at the creation time.

The `measuredAt` value, as any other date, is output as a string based on the ISO 8601 format, that is: YYYY-MM-DDTHH:mm:ss.sssZ.

If there are more than one measure type, some columns may not be applicable to some measure types. In that case, the cell is left empty.

## Query Examples

`GET /_/thing-manager/measure/export` exports all measures. Use with caution, may be huge!

`GET /_/thing-manager/measure/export?s=2024-05-01` exports measures with `measuredAt` greater than or equal to May 1st, 2024.

`GET /_/thing-manager/measure/export?s=2024-05-01&e=2024-05-02` exports measures with `measuredAt` equal to May 1st, 2024.

`GET /_/thing-manager/measure/export?r=128.64` exports measures from the device having this reference, with the default model.

`GET /_/thing-manager/measure/export?t=AppPosition&m=App` exports measures giving a position and from devices of this model.

## Results Examples

```{note}
In the CSV contents displayed here:
- Spaces are added just for the reading comfort.
- Some obvious parts may be replaced by a `…` character, to shrink the line length.
```

### Basic Context

For our demonstration, let's say that we have the following definitions as a starting context:

A single measure model with `AppPosition` type and these *valuesMappings*:

    acquiredAt: { type: 'date'},
    type: { type: 'keyword' },
    distance: { type: 'integer' },
    position: { type: 'geo_point' },

A single decoder is implemented, by an `AppDecoder` class, implying that an `App` device model is available. The reference of a device is the combination of two integers joined by a '.' character.

---
Results without query parameters, in `measures.csv`:

    id,                  measuredAt,              reference,acquiredAt,type,distance,position.lat,position.lon
    AdmdpY8BLrzbnV2RW07K,2024-05-23T13:03:26.893Z,128.64,   2024-05-2…,+,   1,       47.12345,    -1.54321
    AtmdpY8BLrzbnV2RW07Z,2024-05-23T13:03:26.893Z,128.65,   2024-05-2…,+,   2,       47.12345,    -1.54321

The `measureType` and `model` columns are absent because there is anyway only one instance in each of these categories.

### Extended Context

Now, suppose we have an additional device model named `Dv2`, and an additional measure model, with `Celsius` type and these mappings:

    acquiredAt: { type: 'date'},
    degrees: { type: 'integer' },

---
Results without query parameters, in `measures.csv`:

    id, measuredAt, measureType, model,reference,acquiredAt, type,distance,position.lat,position.lon,degrees
    Ad…,2024-05-2…, AppPosition, App,  128.64,   2024-05-2…, +,   1,       47.12345,    -1.54321,
    At…,2024-05-2…, AppPosition, App,  128.65,   2024-05-2…, +,   2,       47.12345,    -1.54321,
    Bd…,2024-05-2…, Celsius,     Dv2,  ref1,     2024-05-2…, ,    ,        ,            ,            12
    Bt…,2024-05-2…, Celsius,     Dv2,  ref2,     2024-05-2…, ,    ,        ,            ,            22

Results for `?t=Celsius`, in `measures-Celsius.csv`:

    id, measuredAt, model,reference,acquiredAt, degrees
    Bd…,2024-05-2…, Dv2,  ref1,     2024-05-2…, 12
    Bt…,2024-05-2…, Dv2,  ref2,     2024-05-2…, 22

Results for `?r=128.65`, in `measures-App-128.65.csv`:

    id, measuredAt, measureType, acquiredAt, type,distance,position.lat,position.lon, degrees
    At…,2024-05-2…, AppPosition, 2024-05-2…, +,   2,       47.12345,    -1.54321,

Results for `?t=AppPosition&r=128.65`, in `measures-AppPosition-App-128.65.csv`:

    id, measuredAt, acquiredAt, type,distance,position.lat,position.lon
    At…,2024-05-2…, 2024-05-2…, +,   2,       47.12345,    -1.54321

Results for `?m=Dv2`, in `measures-Dv2.csv`:

    id, measuredAt, measureType, reference,acquiredAt, type,distance,position.lat,position.lon,degrees
    Bd…,2024-05-2…, Celsius,     ref1,     2024-05-2…, ,    ,        ,            ,            12
    Bt…,2024-05-2…, Celsius,     ref2,     2024-05-2…, ,    ,        ,            ,            22

Results for `?m=Dv2&r=ref1`, in `measures-Dv2-ref1.csv`:

    id, measuredAt, measureType, acquiredAt, type,distance,position.lat,position.lon,degrees
    Bd…,2024-05-2…, Celsius,     2024-05-2…, ,    ,        ,            ,            12
