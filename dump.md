The initial plans for this were overly reliant on events. I think it may be best to look at Atom's pluggable system.

Which while the codebase is a mess, does provide some good ideas. Mainly that services are defined per package, and the global API is vast.

In this case, since the goal of HomeCompanion is to be as pluggable as possible, many of the global API fields and methods will actually be plugins, that are exposed via the global API,
which the choice of attaching them there is based on the service they provide.

Many services will be under the namespace of the general service they belong to.

While events should still be utilized its hard to say.

Really what may be a good solution is to have the global API, and each namespace service attaches its services beneath it, which themselves are plugins, but the namespace core service can then emit events based on the data it receives.

That way, all a plugin that is a child of a major service needs to do, is listen to its parent for the events it cares about, and once done hands its transformed data to its parent, at which point can hand it off to the next service in the train.

This way we ensure the a pluggable environment, where every important action is it's self a plugin, with full support for whatever structure they want, while just understanding the events emitted by their parent, and nothing else.

Plus this allows the cohesion layer to be able to respond accordingly if they receive no data back, or have a known error with their children.

This general functional structure could look like so:

---

Core : The core is the container of all sub services, in charge of starting up services, and beginning device/service discovery.
  
  .IPA : The IPA Service is the handoff point between the Core Service and many others. 
          The IPA (Intelligent Personal Assistant) receives appropriate requests and their data from Core, and hands it off to the next Parent Service.
          The IPA is also what does the responding to Core's requests, waiting for the appropriate actions to occur, then handing the resulting data off.
    .Speech Manager Service : The first child or plugin service, is in charge of handling the auditory query data. It is expected to take this data, 
                              And return usable text data.
    .Dialog Manager : A child of IPA or plugin modal that is expected to maintain the session. This item is also able to depend on several Services itself
                      It is able to rely on Services such as the .History Manager, which maintains session history, and historical Dialog Data.
                      Or even use a .Knowledge Graph Service, which can help build knowledge trees or state data about the user their interacting with.
                      But mainly the Dialog Manager is intended to be able to slot context into text data, additionally the Dialog Manager may rely on a separate parser such as an NLU, 
                      to turn this speech into usable commands, or `intents`. A syntax needs to be created for these intents, but they are the expected output of this service.
    .Provider Selection Service: Another plugin modal, or child of the IPA this is the service that takes in `intents` and routes them to the proper services/devices.
                                  Ultimately this is the director of what to do with a command, whether that's to invoke a plugin to find the weather, or turn on the lights, 
                                  The .Provider Selection Service must determine how to route the call, and invoke its desired service.
    .Provides Service : A collection of plugin modals, all used by the .Provider Selection Service. While the .Provider Selection Service determines how to route a call, it will send it to a .Provides Service.
                        Each .Provides Service should be specific in its task, intending to handle a single type of `intent` and find or complete the proper query, and return the status of that.
                        These are intended to be the most common plugins created within an ecosystem like this, since each may look something like 'Provides Temperature', 'Turns on/off Phillips Lights', 'Chromecast Controller'
                        and so on, for anything that can be thought of.
  .Device Service : The Device Service is a Core Service that is in charge of storing devices found, and initializing the setup of said devices. 
    .Devices : Another Core Service, that is the namespace for interacting with existing and setup devices.
    .Device Provides Service : A collection of plugin modals, that each are specific in function, in returning a Devices object that allows interaction. Likely filling in functions of a predetermined pattern, to allow other services 
                                to interact with said device, with zero knowledge on how it functions under the hood. We will utilize `traits` and `types` to allow this interaction to be as agnostic as possible. 
                                For example: A Device with `type: light` and `traits` such as `turn_on`, and `turn_off` an .IPA.Provides Service would be able to, once it has the correct device, would be able to call a function such as 
                                `global.companion.device_service.devices.DEVICE_ID.turnOn()` and expect that the device will turn on, no matter what the URL of this device is, or how the packet is structured to tell it to do so.
  .Plugin Service : A Core Service that is the controller of loading, and starting any/all plugins installed on the system. As well as disabling/enabling of plugins already on the system.
  .API Service : A Core Service, that should be decoupled from Core that manages the API server and its endpoints.
    .API Extension Service : It may be a smart idea to allow and extension service, that extends the endpoints, and responses to these new endpoints, allowing something like a WebService that hosts and controls a webpage, or 
                              Just additional JSON endpoints for interacting with the service.
                              Really if this route where to be chosen, it may be smart to include the expected endpoints as an API Extension Service, to expand customizability.
  .Config Service : A Core Service tasked with finding, reading and initializing the configuration. Likely supporting a way to get a single configuration out to each plugin. Likely via name.
  .Repository Service : A Core Service tasked with the finding, and installation of all plugins that are available. 

---

With this core structure specified, heres a quick example of how a common interaction would go:

API_Service receives a request via an endpoint. Seeing its text data, made to a command endpoint it knows somewhere the user said something. It is also passed some information about where this occurred, 
  likely with parameters to identify the user, or include a DEVICE_ID of which device made the request. This DEVICE_ID should be the same as an ID accessible with .Device_Service.Devices.DEVICE_ID...
  The API_Service then can use `global.companion.ipa.consume()` passing all of this data. 
Now when API Receives the data, with it, it can make some choices about how to handle it, but generally will then pass it along to its child via its own interface like `global.companion.ipa.dialog_manager` or even `this.dialog_manager`
  Passing the data again thats relevant. 
When the Dialog Manager receives the data it can choose what to do, either parsing it with more plugins or built in functions, then using a plugin/built in modal to add context from a History Manager or Knowledge Graph Service 
  it will inject this with context, then return it to the IPA with something like `global.companion.ipa.emit()` to indicate that it is done parsing all of the text data, and has now handed out an `intent`
When the IPA receives its intent from the Dialog Manager, it can now give it to the Provider Selection Service like `global.companion.provider_selection.new()` 
When the Provider Selection Service receives an intent from the IPA it will need to choose how to process it, and determine the best matched Provides Service to handle this. It can do this by checking the device its asked to control 
  and querying `global.compaion.device_service` for the device ID, then ask `global.companion.device_service.devices.DEVICE_ID` about its traits. If they match the intended function it can additionally find out its other parameters such as the devices it is, such as a device or modal name, then find a Provides Service that supports that device and trait type. At which points asks that Provides Service to do what it does best.
  (While on this subject it seems very repetitive to have a device have a function that is able to complete a trait, and a provides service that completes a trait. It may make more sense to either allow this to be bundled, or have a single 
    provides service named device_controller that is then able to interact with known traits and call the proper device for it.)
Once the Provides Service is done with its action, it will tell the Provider Selection Service its done with `global.companion.provider_selection.done()` and from here this can give the results to `global.companion.ipa.respond()`
  At this point the IPA will again give the response data to the dialog_manager to indicate that its received a response, and will expect to be given as a return a proper response to this request. Either auditory data, or a simple chime to indicate a task has completed.
Finally then the IPA responds to the await of the API_Service with this intended response, and it is returned to the client.

---

With all of this said, its easy to see how there are many parts that need to work together, but other than a defined structure of some common functions, and a specific manifest file, and finally a hyper specific `intent` spec, the rest is up to a developer on how they want to integrate, and what they want to do. While using something like a manifest won't allow for the user of imports, that may be a good idea to avoid NPM installations and offload the issue of vulnerabilities to the Core service, rather than others. Although it may be possible to add a .Utils Core Service that can have many requires, that exposes functions that end up being common. Such as an expose for HTTP requests via a low level language, or a Utils for logging, and so on.

This method does allow developers to create a lot, and for each system to vary from another, letting the user decide the system they want, using this as a basis.

This does mean that a few components MUST be made before much development can begin.

* The manifest.json syntax, to indicate how resources of the service are defined.
* The `intent` syntax, this will likely require the most research as it needs to be able to represent any command or question a user could pose. But does benefit in that it can be strict 
* Traits, types, this can likely be reviewed from Google's expansive coverage on the topic, and may be used largely to start off in.
* Otherwise simply defining the final structure of these global API's is whats needed to get this going

Then with the creation of all Core services, and a basis of plugins which will likely be the default plugins can allow this to be functional.
