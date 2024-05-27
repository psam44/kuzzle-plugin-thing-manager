# Plugin Instantiation

If you followed the instructions from the Kuzzle site about initializing a new application with `Kourou`, or if you did a setup by extracting yourself the [template](https://github.com/kuzzleio/template-kuzzle-project), you should have a code structure as detailed below.

A Plugin must be put in use before starting the application, so a suitable place to instantiate it is in the constructor of the application:

```{code-block} typescript
:caption: "`MyApplication.ts` - Starting structure"
:emphasize-lines: 7

import { Backend } from 'kuzzle'
/* ... */
export class MyApplication extends Backend {
  /* ... */
  constructor(config?: MyApplicationConfig) {
    /* ... */
    /* ... Here the plugin instantiation - to be continued */
  }
  /* ... */
}
```

Instantiate the plugin and proceed to the registration of Device Model(s) and Measure Model(s):

```{code-block} typescript
:caption: "`MyApplication.ts` - Instantiation and registrations"

import { ThingManagerPlugin } from 'kuzzle-plugin-thing-manager'
import type { DeviceModelDefinition, MeasureModelDefinition } from 'kuzzle-plugin-thing-manager'
/* ... */
    /* ... */
    const thingManager = new ThingManagerPlugin()
    const deviceModelDefinition: DeviceModelDefinition = {
      /* ... Here the definition - to be continued */
    }
    thingManager.registerDeviceModel(deviceModelDefinition)
    const measureModelDefinition: MeasureModelDefinition = {
      /* ... Here the definition - to be continued */
    }
    thingManager.registerMeasureModel(measureModelDefinition)
    this.plugin.use(thingManager)
    /* Optionally - Here some plugin configuration settings */
```

```{seealso}
Details related to the above placeholder for the configuration settings can be found at [Configure The Plugin](advanced/configure.md).
```

A Device Model definition must at least provide a payload decoder:

```{code-block} typescript
:caption: "`MyApplication.ts` - Device Model Definition"
:emphasize-lines: 5

import { AppDecoder } from './AppDecoder'
/* ... */
    /* ... */
    const deviceModelDefinition: DeviceModelDefinition = {
      decoder: new AppDecoder(),
    }
```

A Measure Model definition provides:
- An identifier to reference this definition elsewhere. As the identifier will be mentioned in one or more decoder implementations, it is advised to import this string from an aside file dedicated to constants, in order to keep the setting in a single copy.
- The mappings for the measure model properties.

```{code-block} typescript
:caption: "`MyApplication.ts` - Example of a Measure Model Definition"
:emphasize-lines: 5,6

import { APP_POSITION_TYPE } from './consts'
/* ... */
    /* ... */
    const measureModelDefinition: MeasureModelDefinition = {
      type: APP_POSITION_TYPE,
      valuesMappings: {
        acquiredAt: { type: 'date' }, // when the measure was acquired
        type: { type: 'keyword' }, // to qualify the event, such as 'onEnter', 'onExit'
        distance: { type: 'integer' }, // the distance between the real emitter and a collector
        position: { type: 'geo_point' }, // the location of the event
      },
    }
```

```{code-block} typescript
:caption: "`consts.ts` - Example of an identifier for a Measure Model Definition"

// by convention, to make it easier to read traces, use PascalCase for *_TYPE
// and camelCase for *_NAME
export const APP_POSITION_TYPE = 'AppPosition'
```

Now, the next step is to write a [decoder](decoder.md).
