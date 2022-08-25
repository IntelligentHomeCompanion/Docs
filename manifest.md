Each plugin is defined via the `manifest.json` file. 

While these plugins will be running in a node.js environment, at this time it doesn't seem necessary to include NPM functionality, like dependencies, while that may be included in the future, for now lets see what's possible.

The manifest will need to define a friendly name for the plugin, as well of course define the `provides` key. Defining what kind of Provider it is. Really what type of plugin.

Beyond that having some generic sugar, such as the publisher, the repository, issue location, and version.

It may also be a good idea to include an `engines` key, to specify what version of HomeCompanion this is compatible with.

Finally specifically for device providers, there should be some way to define what that plugin supports.

This may mean a key in the manifest that says the model, or device itself, and then there is a modal resolver API, it's hard to say what the best solution here would be, but still important to consider.

Additionally this data may be best fit for a JSON object, to allow multiple methods of identification, such as a matcher that can read the supported MAC's or OUI's at least, or regex device names, something along those lines could live in a specific file named JSON at the root of the plugin.

Otherwise the `manifest.json` file will likely look like so:

```json 
{
  "name": "the-first-plugin",
  "description": "Doesn't do much, just the first plugin manifest draft.",
  "main": "logic.js",
  "version": "1.0.0",
  "repository": "https://github.com/SmartHomeCompanion/Docs",
  "issues": "https://github.com/SmartHomeCompanion/Docs/issues",
  "author": "confused-Techie",
  "license": "MIT",
  "provides": "device_provider",
  "recommends": [
    
  ]
}
```

The `recommends` array key, allows the ability to specify other plugins that are recommended based on the plugins name. Meaning that if a plugin uses a certain `util` plugin, it can recommend its installation here. Or otherwise a device provider can recommend the service that uses said device.
