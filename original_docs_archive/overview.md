Due to the complexity of this idea, it may be best to focus first on the few general areas of functionality

Additionally a plan needs to be drafted of what the method of functionality will be, instead of hodge-podging it as scenarios necessitate.

Plus to hopefully avoid tech debt, caused by quickly coding my principals of the application, I will try to follow some guidelines.

Here are some good resources to study what I want:
  * https://javascript.plainenglish.io/decoupling-code-in-javascript-with-the-dependency-inversion-principle-6d23342b4aaa (While it says DIP, its not, but still good for encapsulation around dependencies)
  * 
  
Principles:

* SOLID 
* DRY 

These general areas seem to be the following:

IPA: (Intelligent Personal Assistant)
  The functionality surrounding a functional digital assistant. Understanding data and turning it into actions.
  It seems there is a small effort by W3C's Voice Interaction Community Group to attempt at standardizing this.
  And if that is a plan, I would like to follow it as closely as possible, and will likely adopt their language.
  https://w3c.github.io/voiceinteraction/voice%20interaction%20drafts/paArchitecture-1-2.htm

---

# Application Structure 

While this will need to able to handle many events, to help migrate away from direct calls, and managing the state of different plugins and such, most of the plugins will interact via events, other than the generic `load` and `unload`.

Additionally there are even more tasks that should be handled outside of the core, to allow as much extensibility as possible. Although with that said, some choices should be made, to provide the base structure of how these plugins interact with each other to ensure extensibility, while maintaining compatibility.

Although its noteworthy to mention that much of the trust of successfully completing user requests relies on these plugins, since the entire system will be event based, meaning the IPA Service Manager only initiates the requests, instead of moves it along, moving the request along is up to each descending plugin, to emit the correct events.

Some of the major services:

## Core 

### API Server Manager
* Handles incoming requests, and providing them to the right services, such as:
  - Queries consisting of voice data.
  - Queries consisting of text data.
  - Updating the status of remote devices via supported API calls 
  - Responding to supported API calls for internal data.
* Acts on requested posts from internal functions. 
  - Using Supported client API's to update or send data.

### Enabling Service Initialization 
* Calls internal modules to setup, load, and find plugins installed on the system.
* This will use `load` functions on the plugin modules to allow them to setup as needed, and become ready to listen for events.

### IPA Service Manager 
* Managing initial handling of user requests, and emitting the proper events, which then a plugin should respond to.
* Listening to the proper events emitted by a plugin to then handle the request properly, or alternatively we rely on the consuming plugin to listen properly for events, and have simple bounds checking, to allow a failure of the event track, and attempt recovery from a user perspective for it. For example after a given amount of time from the start of the request, if the event track hasn't completed, respond to the user with an error message, and close the HTTP request, to allow a client device to rely the error back to the user.

---

## Event Track / Request Walkthrough 

(All event names are examples and subject to change)
This will need change, as it relies to heavily on events, which while they should be taken advantage of one must remember there is a global API for this, of which all Services should be attached to.

* User makes request to API Endpoint 
* Core Service listening on the endpoint, then takes the request and passes it to the IPA Service Manager 
* IPA Service Manager emits `received`, then Dialog Manager listening for this events, logs this raw request, emitting events based on weather this is textual input, or auditory input. 
  - Textual: Dialog Manager Emits `process_text`
  - Auditory: Dialog Manager Emits `process_audio`
* The corresponding language processor will then enact on this data, returning concise language representations of this text. Emitting `done`
* Again the Dialog Manager can listen for this `done` and can now store the additional data of the request, now ready to slot context details into the text, can emit the `slot` event, at which point the Context Service can now take over, using its History Service, and Knowledge Graph Service to slot context into a request, using any Data Providers necessary via Global APIS.
