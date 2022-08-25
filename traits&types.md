Much of this is based off the fantastic definitions of these by Google.
* [Google's Traits](https://developers.google.com/assistant/smarthome/traits)
* [Google's Types](https://developers.google.com/assistant/smarthome/guides)

Although for now, this list will be simplified, with some functionality that's determined as possible with the current scope of the project.

# Traits 

* `brightness` : Absolute brightness setting in a normalized range from 0-100. 
* `color_setting` : Applies to devices such as Lights that can change colour.
* `on_off` : Applies to any device that has a binary On or Off state. Boolean True is On, and False is Off.

# Types 

* `LIGHT` : Light devices can be turned on and off. They may have additional features, such as dimming and the ability to change colour.
* `OUTLET` : An outlet device to delivery power to connected devices. Usually just a binary, on/off.
