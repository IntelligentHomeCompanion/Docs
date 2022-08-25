This is likely one of the most complex elements to define. As it needs to be able to define all language applicable to commands and questions, while ideally broad enough to support future changes. As so many elements will rely on it, such as the Dialog Manager, potentially History Manager, and every single Provides Service Plugin, it will not be easy to change except in the early stages.

So it is very important to have a solid definition of this before continuing development.

---

Examples of `intents` used today.

When referring to `intents` within Oracle, they describe them as a definition of `utterances` defined within a skill. Such as defining your `intents` to be 'I want to order a Pizza', 'I feel like eating a pizza'. This heavily relies on the Speech Service to be able to decode this meaning and apply it generally to translate language to these predefined intents. Which really seems it has to infer meaning from two sets of sentences, rather than decide from one, and use that to control functions directly.

A closer comparison in `intents` in the wild may be with the use of IBM's Watson. Using an `intent` classification model to find meaning in users words, allowing the use of `Annotated Mentions`, variables prefixed by '@' to describe a variable in what the user is saying. 

IBM's use of `intents` seems to match much more closely with what the goals of this application are, although still does user an example set to help train their classification model. Which may not exactly suit our needs. 

Although as the scope of finding meaning in auditory or even text data is not my forte this may be the only course of action.

Beyond that there are a few important things to consider:

* An `intent` needs to be able to represent the action intended.
* Needs to be flexible enough that a contextual system can be used to slot data into variables or `Annotated Mentions`
* Finally must be strict enough that it can easily be consumed by down the chain providers, in order to properly act on that data. 

The goal of an intent system is to put the burden of responsibility of understanding text on the `Dialog Manager`. Then from there, a `Provider Service` just needs to understand the queries and act on them.


---

The important data that must be communicated via the `intent` syntax:
* The data expected back.
* The type of request (query, command)
* The item to act on
* The location to consider 
* The value to provide 

Looking at several different Query Languages (which makes sense for this type of application)
`Gellish` stands out as a contender, or something to largely borrow from. Intended as an ontology language for data storage and communication its an extendable conceptual data modeling language.

Meant to return data to queries based on an internal Knowledge Table.

Considering the syntax here its possible this could be expanded for our uses.

```
QUERY - RETURN:_temperature_:f @where/'Los Angeles':city @when/1661404626:epoch
```

This is an example `intent` that could be derived from 'Whats the temperature right now?', this would expect that a `Dialog Manager` contextual slotting system is able to determine that the lack of usage of a location means the user is asking about their current location. Additionally that they want the results in Fahrenheit. And lastly convert the 'now' portion to the current epoch time.

Representing the data is this way follows a simple extensible syntax.

`COMMAND - RETURN:_DATA_TYPE_:modifier @variable/value:modifier`

Using the syntax in such a way allows additional modifiers to be added at any time and that new Provides Services can utilize them, while still allowing older Provides Services to use the lack of data.

Structuring the `intent` syntax in such a way, means its easy to log, and easy for a user to understand while troubleshooting. This system would likely be accompanied by an `intent` object thats actually being returned, that allows easy object creation based off the syntax so there is simple interaction in JavaScript such as using `intent.where` to receive the city name from the example above.

The general idea is that the `COMMAND` is able to specify the type of request, generally `QUERY` or `REQUEST`, meaning if the user is asking a question or asking that an action takes place.

From here each item is aligning with `@variable/value:modifier` except in the case of knowing the return data, which is the data thats expected to be given back to the user. Which is denoted by `RETURN:` followed by a preset possible `_DATA_TYPE_` these data types will attempt to be all encompassing of what the user is asking for. Which themselves do accept modifiers. 

Depending on the modifier used, this could even natively use a `util` function. Such as the case with temperature, the modifier could show the value being expected, and when the value is returned, it should also declare what its format is, so that if the two don't match a utility function can handle the conversion automatically.

Finally the `@variable/` will be based on a preset list of variables, following the variable is the value, which itself accepts modifiers to declare the format the variable is listed in.

Adding modifiers to the variables means that not only can the return data type be converted to whatever the user asked for, it means the end `Provider Service` is able to ask for the data in whatever format it expects to.

Some examples of the `intents` expected from certain utterances:

* 'What time is it right now?' 
  - Context: User lives in 'Los Angeles' city, they prefer a 12 hour time representation, the current time in epoch.
  - Intent: `QUERY - RETURN:_time_:12 @where/'Los Angeles':city @when/1661404626:epoch`

* 'Where am I?'
  - Context: User's GPS Location (In this case represented as Latitude and Longitude), prefers location data as latitude and longitude 
  - Intent: `QUERY - RETURN:_location_:latlong @where/'34.01696468290318, -118.28878131315437':latlong`

* 'Turn off the Lights'
  - Context: Using `Devices` to determine the device based on its friendly name 'Lights', or using context to know they are the only lights in a system, or at the users current location.
  - Intent: `REQUEST - RETURN:_action_:on_off @where/'DEVICE_ID' @what/false`
  - Notes: See here that the `RETURN`'s Data Type has a modifier of a trait. The modifiers of an action should relate to a predefined trait, to allow the `Provides Service` to properly call the Devices Object Controller function.
  
* 'Set the lights brightness to 50%'
  - Context: Using `Devices` to determine the devices DEVICE_ID
  - Intent: `REQUEST - RETURN:_action_:bightness @where/'DEVICE_ID' @what/50:percent`
  
### Specials 

The `specials` are then additional modifiers to how an action is acted apon. For example if a user asked to 'Turn off the Lights in five minutes.' we could use a `special` to add the timing functionality of this request. Or if a user asked 'Turn off the lights and lock the door.' a `special` would be used for the double commands.

`Specials` are **always** handled first, being passed to a `Provides Service` that can handle said special, which then will re-execute the commands in accordance to these `specials`. For example in the case of 'Turn off the lights and lock the door.' A `Provides Service` that can handle a special would take this request, and execute them in order. First passing the `intent` for 'Turn off the Lights' back to the `Provider Selection Service` to handle appropriately, likely with an optional flag to indicate that the request is not complete, as to prevent the system from respond to the API call early by ending the command train. Then it would invoke the `Provider Selection Service` on the secondary `intent` for 'lock the door' afterwards, this time with any flags removed, to allow the request to complete.

The available `Specials`:
* `#AND` : Execute the first `intent` then the one following it.
* `#TIMER/1:min` : Execute the `intent` after the time specified, in this case which looks similar to the variable, value, and modifier declaration as before. In this case meaning the value is '1' and the modifier declaration being 'min' to mean in minutes.

An example `intent` with specials:

* 'Turn off the lights and tell me the temperature'
  - Context: User's current location, preferred temperature format, and the `Devices` DEVICE_ID for 'the lights'
  - Intent: `REQUEST - RETURN:_action_:on_off @where/'DEVICE_ID' @what/false #AND QUERY - RETURN:_temperature_:f @where/'Los Angeles':city @when/1661404626:epoch`

---

You can see now that this format that is loosely borrowed from Query Languages with a special interest on Gellish can allow an extremely extensible language to represent `intent`s, plus allow the opportunity for new declarations later on, while maintaining backwards compatibility, and with built in `utils` on the `intent` object, can allow conversion of the supported data types from any format possible to whatever the user wants, or the consumer `Provides Service` needs.

---

# Intent Value Declaration:

## Variables 

Variables prefixed with `@` are built in values, that automatically use an appropriate data type to allow conversions and methods.

* `@where` : Used to specify a location. DATA_TYPE: '\_location\_', expect if the `RETURN:` is an `_action_` datatype. In those instances, it is automatically a reference to the proper Device, based on the DEVICE_ID passed.
* `@when` : Used to specify a the time the data needs to be relevant to. **Not used for timed actions** DATA_TYPE: '\_time\_'
* `@what` : Used to pass data along to the resulting service. In most cases this will be relevant to a device action, in which case will inherit its DATA_TYPE based on the DATA_TYPE associated to the trait being used.

## Specials 

Specials are used to change the control flow of an `intent`

* `#AND` : Used to chain multiple `intent`s together. Has no DATA_TYPE 
* `#TIMER` : Used to execute and `intent` after the specified time. DATA_TYPE '\_time\_'

## Data Types

* `_time_` :
* `_location_` :
* `_action_` :
* `_temperature_` :
* `_controlBinary_` : Used when the Device Trait is `on_off` expects the `@what/` value to be a boolean, `true` or `false`. Where `true` is on, and `false` is off.
* `_controlRange_` : Used when the Device Trait is `brightness` or a normalized range. Expects the `@what/` value to be a numbered range between 0-100. Or something compatible with such via the modifier of the value. 
