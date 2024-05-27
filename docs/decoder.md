# Write A Decoder

For each payload kind sent by your IoT devices, you have to write a custom decoder.

You can organize the classes and the source files as you want. For this documentation, we will simply name the decoder as `AppDecoder` and place its source file in the same folder as that of the application source file.

A decoder is a class inheriting from the `Decoder` class provided by the plugin, and must provide an implementation for the abstract `decode()` method.

```{code-block} typescript
:caption: "`AppDecoder.ts` - Basic inheritance"
:emphasize-lines: 12

import type { JSONObject, KuzzleRequest } from 'kuzzle'
import { Decoder } from 'kuzzle-plugin-thing-manager'
import type { DecodedPayload } from 'kuzzle-plugin-thing-manager'

export class AppDecoder extends Decoder {
  // deviceModel: inferred from the class name => 'App'
  // action: inferred from the device model => 'app'
  // routes: default applies => POST /_/thing-manager/payload/app
  /* ... to be continued */

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  override decode(
    decodedPayload: DecodedPayload<AppDecoder>,
    payload: JSONObject,
    request: KuzzleRequest, // probably unused context
  ): void {
    /* ... to be continued */
  }
}
```

For our demonstration, let's say that the incoming payload is something like this sample:

```{code-block} json
:caption: Json payload sample

{
  "timestamp": 1672247313635,
  "type": "onEnter",
  "major": 512, "minor": 16,
  "distance": 12,
  "latitude": 47.12345, "longitude": -1.54321, "geotime": 1672247313635
}
```

Measures expected by this decoder must be declared by overriding the `measures` property:

```{code-block} typescript
:caption: "`AppDecoder.ts` - Measures declaration"
:emphasize-lines: 10

import type { NamedMeasures } from 'kuzzle-plugin-thing-manager'
import { APP_POSITION_TYPE } from './consts'

// by convention, to make it easier to read traces, use camelCase for *_NAME
// and PascalCase for *_TYPE
const APP_POSITION_NAME = 'appPosition'
/* ... */

  /* inside the class ... */
  override measures: NamedMeasures = [ // default is an empty array
    {
      name: APP_POSITION_NAME,
      type: APP_POSITION_TYPE,
    },
  ] as const
```

As seen in [Plugin Instantiation](plugin.md), `APP_POSITION_TYPE` points to a registered measure model, potentially used many times and by other decoders. `APP_POSITION_NAME` names here a local usage of that model.

To better understand these two meanings, let's see another example with temperatures:

```{code-block} typescript
:caption: Example

import { TEMPERATURE_TYPE } from './consts'
const INTERIOR_TEMPERATURE_NAME = 'interiorTemperature'
const EXTERIOR_TEMPERATURE_NAME = 'exteriorTemperature'
  /* ... */
  override measures: NamedMeasures = [
    { name: INTERIOR_TEMPERATURE_NAME, type: TEMPERATURE_TYPE },
    { name: EXTERIOR_TEMPERATURE_NAME, type: TEMPERATURE_TYPE },
  ] as const
```

Here is how we can implement the decoding:

```{code-block} typescript
:caption: "`AppDecoder.ts` - decode() implementation"
:emphasize-lines: 28

// structure of the measure values given to decodedPayload
interface AppEventMeasure {
  acquiredAt: number
  type: string
  distance: number
  position: {
    lat: number
    lon: number
  }
}

// structure of an item received in payload
interface PayloadItem {
  timestamp: number
  type: string
  major: number
  minor: number
  distance: number
  latitude?: number // location may not be available
  longitude?: number
  geotime?: number // timestamp of the geolocation, may be outdated in a sparing last-known strategy
}
/* ... */

  /* inside decode() ... */
  const item: PayloadItem = payload
  const deviceReference = `${item.major}.${item.minor}`
  decodedPayload.addMeasure<AppEventMeasure>(deviceReference, {
    name: APP_POSITION_NAME,
    type: APP_POSITION_TYPE,
    measuredAt: item.timestamp,
    values: {
      acquiredAt: item.geotime || 0,
      type: item.type,
      distance: item.distance,
      position: { lat: item.latitude || 0, lon: item.longitude || 0 }, // a geo_point mapping type
    },
  })
```

The preceding code processes a payload whose content is supposed to be as expected. How can we have such confidence? By implementing a custom `validate()` method. By default, this method doesn't check anything.

```{code-block} typescript
:caption: "`AppDecoder.ts` - validate() implementation"
:emphasize-lines: 6

import { PreconditionError } from 'kuzzle'
/* ... */

  /* inside the class ... */
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  override async validate(
    payload: JSONObject,
    request: KuzzleRequest, // probably unused context
  ): Promise<boolean> | never {
    const item: PayloadItem = payload
    this.assertProperties(
      item,
      ['timestamp', 'type', 'major', 'minor', 'distance'], // the other properties may be missing
    )
    if (typeof item.distance !== 'number')
      throw new PreconditionError('Attribute "distance" in payload must be a number')
    /* add any relevant checks ... */
    return true
  }
```

```{note}
The `assertProperties()` method is a helper provided by the base class to verify that some properties are present in an object.
```

Additional advanced features of decoders are documented in dedicated pages:
- [Multiple Measures](advanced/multiple-measures.md) in a payload
- [Metadata](advanced/metadata.md) about the device
- [Content Types](advanced/content-types.md) other than json
- [Multiple Engines](advanced/multiple-engines.md)
