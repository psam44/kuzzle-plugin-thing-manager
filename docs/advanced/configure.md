# Configure The Plugin

Configuring Kuzzle, including plugins, can be done in many ways:
- With a `.kuzzlerc` file.
- With environment variables with a  `kuzzle_` prefix.
- By altering at run time the `config` property of your Application/Backend.

As an example, let's take the third option to change a default setting of the plugin. A change can only occur before the start of the application. A right place is in the constructor, after le `use()`:

```{code-block} typescript
:caption: "`MyApplication.ts` - Configuration at runtime"
:emphasize-lines: 14

import type { ThingManagerConfiguration } from 'kuzzle-plugin-thing-manager'
/* ... */
export class MyApplication extends Backend {
  /* ... */
  constructor(config?: MyApplicationConfig) {
    /* ... */
    /* after this.plugin.use(thingManager) */
    const plugins = this.config.content.plugins
    if (plugins) {
      // if not specified in the options of use(), the name is a purged kebabCase of the class name
      const pluginName = 'thing-manager'
      // there may be no entry yet
      const cfg = plugins[pluginName] ?? (plugins[pluginName] = {}) as ThingManagerConfiguration
      cfg.spareSavePayload = true
    }
  }
  /* ... */
}
```

````{note}
The same effect with a `.kuzzlerc` file would be:
```json
{
  "plugins": {
    "thing-manager": {
      "spareSavePayload": true
    }
  }
}
```
````
