All services are registered under `global.companion` and can be invoked with `companion`

`companion`
  - `.ipa` : Intelligent Personal Assistant - Core modal 
    - `.speech` : Speech Manager - Manages auditory query data, plugin modal 
    - `.dialog` : Dialog Manager - Maintains the contextual session, and more importantly understands the text based data, plugin modal 
    - `.history` : History Manager - Maintains Session History, for use of `.dialog`, plugin modal 
    - `.knowledge` : Knowledge Graph - Builds knowledge stores for session data, for use of `.dialog`, plugin modal 
    - `.provider_selection` : Provider Selection Service - Routes an `intent` to the proper services.
    - `.provides` : Provides Service - Collection of modals, that fullfill a specific `intent`, plugin modal collection.
  - `.device` : Device Service - Core Modal 
    - `.devices` : Devices Namespace - Core Service namespace to interact with loaded devices, core modal 
      - `.Devices Provides Service` : A euphemism, for exposing device objects via the device name, with a chosen Device Provides Service. Core Modal 
  - `.plugin` : Plugin Service - Core Modal controller of loading and starting plugins.
  - `.api` : API Service - Core Modal, sets up and tears down API functionality, exposing endpoints for modification.
    - `.ext` : API Extension Service : plugin modal to extend API endpoints and their functionality. Worthy to note, a core modal will be included as an `.api.ext` modal to include basis API endpoints.
  - `.config` : Configuration Service - Core Modal loading and parsing configuration data.
    - `.Config Providers` : A euphemism for exposing plugin configurations via their name 
  - `.repo` : Repository Service - Core Modal tasked with finding, and installation of all plugins.
  - `.util` : Utilities Service - Exposes common functions, or dependencies


Additionally many of these services will have required or included functions/methods. Which will be shown here as they are in the beta state.

## `ipa`

* `.consume()` : Used as the initial handoff for data to the IPA, to begin the handling of this data.
* `.emit()` : Used once the Dialog Manager has finished acting on the data. Returning an `intent` to the IPA Service.
* `.respond()` : Used to inform the IPA that an action has been completed, whatever that is. Passing results back. At this point these results will be handed again to the Dialog Manager.

## `provider_selection`

* `.new()` : Used for passing an `intent` to the Provider Selection Service. 
* `.done()` : Used for informing the Provider Selection Service that a specific Provides modal is complete with it's action.
