# Additional Content Types

By default, the Kuzzle platform supports the receive of payloads in usual content types such as json, x-www-form-urlencoded and form-data.

It may be easier or necessary to accept another type. For our demonstration, let's take a CSV format.

```{note}
The hereunder instructions are an alternative to the implementation explained in the [Additional Content Types](https://docs.kuzzle.io/core/2/guides/advanced/content-types/) page of the official documentation, aimed to be simpler and more integrated with your code.
```

Action 1 of 2: Declare the additionnal content types that you want to be accepted by the backend by overriding the `additionalContentTypes` property of the decoder:

```{code-block} typescript
:caption: "`AppDecoder.ts` - Additional content types declaration"
:emphasize-lines: 2

  /* inside the class ... */
  override additionalContentTypes = [ // default is an empty array
    'text/csv',
  ]
```

These are automatically added for you to the `server.protocols.http.additionalContentTypes` configuration setting (without duplicates).

Action 2 of 2: For the conversion of the entering payload, override the `jsonify` property:

```{code-block} typescript
:caption: "`AppDecoder.ts` - Jsonify implementation"
:emphasize-lines: 6

import type { Jsonify } from 'kuzzle-plugin-thing-manager'
/* ... */

  /* inside the class ... */
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  override jsonify: Jsonify = (payload, contentType) => {
    /* ... to be continued */
  }
```

For our demonstration, we consider that there is no header line, no quotes, and the separator character is the comma.

For the explanation, let's give a name to the possible cells:
- timestamp, type, major, minor, distance: These are always present.
- latitude, longitude, geotime: These three are together present or absent.
- bp_N: A special notation following this pattern: \<metadata key>\_\<metadata_value>. It allows to support as many metadata as wanted and a means to parse them in a generic way.

With that naming scheme, the supported variants of a line content are:
- timestamp, type, major, minor, distance
- timestamp, type, major, minor, distance, latitude, longitude, geotime
- timestamp, type, major, minor, distance, bp_N
- timestamp, type, major, minor, distance, latitude, longitude, geotime, bp_N

The incoming payload may be something like the following samples:

```{code-block}
:caption: CSV payload samples

1672247313635,onEnter,512,16,12
1672247313635,onEnter,512,17,12,47.12345,-1.54321,1672247313635
1672247313635,onEnter,512,18,12,bp_97
1672247313635,onEnter,512,19,12,47.12345,-1.54321,1672247313635,bp_97
```

```{seealso}
The payload has likely many lines, so the [Multiple Measures](multiple-measures.md) technic is adopted here as well.
```

According to the previous expectations, here is an implementation:

```{code-block} typescript
:caption: "`AppDecoder.ts` - Jsonify implementation"

    /* continuation ... */
    const items: PayloadItem[] = []
    for (const line of payload.toString().split('\n')) {
      // first, extract the metadata if present, and from whichever columns they are located
      const extras = new Map<string, string>()
      const fields = line.split(',').filter((field) => {
        if (field.includes('_')) {
          const kv = field.split('_', 2)
          extras.set(kv[0]!, kv[1]!)
          return false
        }
        return true
      })

      const len = fields.length
      if (len >= 5) { // required fields
        const item: PayloadItem = {
          timestamp: +fields[0]!,
          type: fields[1]!,
          major: +fields[2]!,
          minor: +fields[3]!,
          distance: +fields[4]!,
        }
        if (len >= 8) { // optional fields
          item.latitude = +fields[5]!
          item.longitude = +fields[6]!
          item.geotime = +fields[7]!
        }
        extras.forEach((value, key) => { // re-inject the metadata
          item[key] = value
        })
        items.push(item)
      }
    }
    const value: JSONObject = { [WRAPPER_JSON_KEY]: items }
    const jsonText = JSON.stringify(value)
    return Buffer.from(jsonText)
```
