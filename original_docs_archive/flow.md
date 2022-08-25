The hopeful flow of the application:

## Application:

This serves as the core. The backend server essentially that runs all the major features.

That can take in many plugins, which expand the devices supported, the functionality, and so on.

The core here just links them all together, and even the text parser is a plugin,

and even the website UI will be a plugin, since hopefully one day, this can be an app that runs on something like 
a raspberry pi hooked into a small device, with an available STL 

## Device Flow:

Service discovery constantly runs in the background, and you can view all discovered devices.

When one is picked to be added the server is asked to add it.

The server will then look at the service being provided and check all of the plugins, defined 
services they support, and look through them to see if one can be found, if one is found 
then the device information is passed to the plugin to set it up.

Once the device has been setup to be usable its added to the global device list.

Now that same integration knows how to handle the device, and parsed commands can be given, if the friendly name matches to the proper plugin in order to act on the device appropriately.


### Setup?

When a processed device is asked to be setup, Plugins uses the service name, and `canSetup` function of each deviceIntegration to check if that device can be setup. If it can, then it returns `true` and Plugins returns 
`setup: true, idx: NUM`, idx being the number in the `deviceIntegrations` array.

From here, the device is called to be setup, to Plugins, which in turn calls the `setup` function from the `deviceIntegration` which is expected to return a controller.

This controller can then be returned to the `Devices` to append that controller to it. Otherwise it should return a `string` error message, of why it failed.
