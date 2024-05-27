# Metadata

Metadata are not to be confused with measures. It is information about the device, the environment or anything else that can vary but it is not the main purpose of the tracking.

The best candidate of metadata is the battery level of the devices that report measures. You want to be informed to plan the replacement of the battery, but it is not a tracked property of a sensor in the IoT device.

Similar to measures, metadata supported by a decoder must be declared by overriding the `metas` property:

```{code-block} typescript
:caption: "`AppDecoder.ts` - Metadata declaration example"
:emphasize-lines: 7

import type { NamedMetas } from 'kuzzle-plugin-thing-manager'
/* ... */
const BATTERY_NAME = 'batteryCapacity'
/* ... */

  /* inside the class ... */
  override metas: NamedMetas = [ // default is an empty array
    {
      name: BATTERY_NAME,
      valuesMappings: {
        capacity: { type: 'integer' },
      },
    },
  ] as const
```

For our demonstration, let's say that the incoming payload is something like this extended sample. Compared to the basic sample, note the presence of a `bp` field (as **b**attery **p**ercentage).

```{code-block} json
:caption: Json payload sample
:emphasize-lines: 7

{
  "timestamp": 1672247313635,
  "type": "onEnter",
  "major": 512, "minor": 16,
  "distance": 12,
  "latitude": 47.12345, "longitude": -1.54321, "geotime": 1672247313635,
  "bp": 100
}
```

For the typing, we need to enrich the declaration of the payload structure with fields related to metadata. We can either be strict by mentioning each field, or either be loose with an index signature (the choice here, even if there is only one field for our example).

Here is the additional code in the decoder:

```{code-block} typescript
:caption: "`AppDecoder.ts` - decode() implementation"
:emphasize-lines: 4,9

// structure of an item received in payload
interface PayloadItem {
  /* ... as previously */
  [extra: string]: string | number
}

  /* inside decode() ... */
  if (item['bp']) {
    decodedPayload.addMeta(deviceReference, {
      name: BATTERY_NAME,
      capturedAt: item.timestamp,
      values: {
        capacity: +item['bp'],
      },
    })
  }
```
