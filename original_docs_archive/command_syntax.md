This might actually be complete trash, and maybe should take heed of the traits and types syntax from Google's smart home.

This may make life much simpler.
https://developers.google.com/assistant/smarthome/traits?hl=en

Some inspiration has been taken from Google's NLU (Natural Language Understanding) syntax thats passed to a custom NLU parser.

The general object expected from the `langaugeParser` will be:

```json 
{
  "action_type": "ACTION_TYPE",
  "expected_response": "EXPECTED_RESPONSE",
  "data": {
    "who": "Who the action is directed at",
    "what": "What the question is about, or what device/service is the command directed at",
    "where": "Where the question pertains to, or where the device/service is located that the command is directed at",
    "when": "When the question pertains to, or when the command on the device should take place"
  }
}
```

The above values within `data` can be variables, prefixed by `@_`:

* `@_HERE` : Will indicate either the question specifically said here, or left out the required data to determine a  location or time and can confidently assume `here`. In the consideration of time `@_HERE` is the same as now, just represented by a single variable. Additionally in the case of an action `@_HERE` means the command was directed at the device that accepted the initial request, likely by usage of a command like `Turn down your volume`.

* `@_NAMED` : Will indicate that the text following after a space is a named variable such as `@_NAMED 'Home'` or `@_NAMED 'Los Angeles'` or `@_NAMED 'The Big TV'`. Anytime a non-formal name is used to represent data.

Additionally for a question it is safe to leave out the `who` key entirely.

In the case of `what` when dealing with a command, this is intended to contain the action thats intended to occur.
This action will not always be a simple one word command, and in some cases need to be more complex, for example `Play The Office on the TV`, the action or `what` can't simply be `PLAY`, so in the case of an action, additional variables are supported, formatted like so `"what": "@_ACTION PLAY : @_ITEM 'The Office'"`.

The supported variables here:

* `@_ACTION` : The type of action that is to occur. 
* `@_ITEM` : The item the action should occur with.
* `@_DIRECTION` : The direction the type of action should occur.

The `action_type` is currently expected to only equate to two separate values:

* `QUESTION` : Used when the user intends to ask a question.
* `COMMAND` : Used when the user intends for something to happen as a result of their action.

Additionally the `expected_response` is expected to be of the following:

* `ACTION` : When the user expects something to happen.
* `ACTION_DELAY` : When the user expects something to happen after some time.
* `RESPOND` : When the user expects an immediate response to their question.
* `RESPOND_DELAY` : When the user expects a response after some time.

---

Here are some examples of what certain text should return from the `languageParser` keep in mind this text has not yet been passed to the `textParser`

### Example Command 1

"Turn off the Lights"

```json 
{
  "action_type": "COMMAND",
  "expected_response": "ACTION",
  "data": {
    "who": "@_NAMED 'Lights'",
    "what": "OFF",
    "where": "@_HERE",
    "when": "@_HERE"
  }
}
```

### Example Command 2

"Play The Office on the Main TV"

```json 
{
  "action_type": "COMMAND",
  "expected_response": "ACTION",
  "data": {
    "who": "@_NAMED 'Main TV'",
    "what": "@_ACTION PLAY : @_ITEM 'The Office'",
    "where": "@_HERE",
    "when": "@_HERE"
  }
}
```

## Action List:

* `PLAY` : When media is intended to be played.
* `PAUSE` : When media is intended to be paused.
* `STOP` : When media playback is intended to be ended or canceled.
* `OFF` : When a device is intended to be turned off.
* `ON` : When a device is intended to be turned on.
* `DIM` : When a device is intended to have is brightness or speed affected granularly. `@_DIRECTION` can be used here to indicate the direction of `UP` or `DOWN`, or something even more specific than that, such as providing media playback with a time stamp, or providing lights with an exact brightness or color value.
