Here will be the working list of Service Provider Definitions, that control how a plugin is loaded into the global API.

* `device_provider` : This Plugin Modal provides the setup, of a specific device, returning a Device Object.
* `function_provider` : This Plugin Modal provides some functionality, such as a timer, or reading out the current time, getting the weather, getting stock market info, sports game info. Anything else you would expect a Digital Assistant to be able to do.
* `speech` : This Plugin Modal functions as a Speech Manager, managing any passed auditory query data.
* `dialog` : This Plugin Modal functions as the Dialog Manager, maintaining the contextual session.
* `history` : This Plugin Modal functions as the History Manager, maintaining Session History for the `dialog` modal.
* `knowledge` : This Plugin Modal functions as the Knowledge Graph, building knowledge stores for session data.
* `api` : This Plugin Modal extends the API Service to include new endpoints, and web based functionality.
* `util` : This Plugin Modal adds additional utility functions for new utilities that other modals can utilize.
