# Multiple Measures

In a very basic scheme, a payload consists of a single measure. More realistically, an emitter wants to deliver more than one measure in a same transfer.

However, the Kuzzle platform supports the payload only as a json object, not a json array.

So, to be conformant, we just need to wrap the array in an object, with a key. This key can be any value, except the empty string which is not supported in a document created in ElasticSearch, and doesn't need to have any applicative meaning.

For our demonstration, let's say now that the incoming payload is something like this sample:

```{code-block} json
:caption: Json payload sample for multiple measures

{
  "_": [
    {
      "timestamp": 1672247313635, "type": "onEnter", "major": 512, "minor": 16, "distance": 12,
      "latitude": 47.12345, "longitude": -1.54321, "geotime": 1672247313635
    },
    {
      "timestamp": 1672247376123, "type": "onExit", "major": 512, "minor": 16, "distance": 17,
      "latitude": 47.54321, "longitude": -1.12345, "geotime": 1672247376123
    }
  ]
}
```

Our choice for a short, neutral, not significant, key is the underscore character.

The  [basic implementation](../decoder.md) of the decoder is extended with:

```{code-block} typescript
:caption: "`AppDecoder.ts` - Multiple measures in a payload"

const WRAPPER_JSON_KEY = '_'
/* ... */

    /* inside validate() ... */
    this.assertProperties(payload, [WRAPPER_JSON_KEY])
    const items: PayloadItem[] = payload[WRAPPER_JSON_KEY]
    if (!Array.isArray(items)) {
      throw new PreconditionError(
        `Attribute "${WRAPPER_JSON_KEY}" in payload must be of type "array"`
      )
    }
    for (const item of items) {
      /* check an individual item, as previously ... */
    }
    return true

    /* inside decode() ... */
    const items: PayloadItem[] = payload[WRAPPER_JSON_KEY]
    for (const item of items) {
      /* process an individual item, as previously ... */
    }
```
