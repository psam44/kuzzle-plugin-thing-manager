# Multiple Engines

An engine is a concept brought to plugins by the [kuzzle-plugin-commons](https://github.com/kuzzleio/kuzzle-plugin-commons) package.

Basically, it allows to have a set of indexes and related collections, with potentially different features, and to route the incoming payloads to a specific index, for whatever reason (volume, tenant, origin, etc.).

Usually, there is no such need and all the traffic is allocated to the single default engine/index, whose name is defined in the plugin configuration.

Otherwise, use your own algorithm to decide which engine is in charge to host the device data and tell it to the `decodedPayload`:

```{code-block} typescript
:caption: "`AppDecoder.ts` - decode() implementation"

  /* inside decode() ... */
  decodedPayload.setEngineId(deviceReference, 'my-engine')
```

There is nothing else to do. If the index doesn't exist yet, it will be created on the fly.
