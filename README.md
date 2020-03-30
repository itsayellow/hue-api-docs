# huedaysync

## Helpful Links

https://github.com/Overboard/discoverhue

https://github.com/quentinsf/qhue

https://developers.meethue.com/develop/hue-api/

## Notes

### Colors

The Hue api documentation (https://developers.meethue.com/develop/application-design-guidance/color-conversion-formulas-rgb-to-xy-and-back/) has a missing group of equations for the "Wide RGB D65 conversion formula".  A gist (https://gist.github.com/popcorn245/30afa0f98eea1c2fd34d) was found that contained the formulas:

* `float X = red * 0.649926f + green * 0.103455f + blue * 0.197109f;`
* `float Y = red * 0.234327f + green * 0.743075f + blue * 0.022598f;`
* `float Z = red * 0.0000000f + green * 0.053077f + blue * 1.035763f;`


### Datatypes

#### Time Patterns

The Weekday specification `[bbb]` is a decimal number of the binary
specification where each bit stands for a day of the week:

| Mon | Tue | Wed | Thu | Fri | Sat | Sun |
|-----|-----|-----|-----|-----|-----|-----|
|   64|   32|   16|    8|    4|    2|    1|

### Schedules

Schedules (inexplicably) use the long form of the address in `"command"`, i.e. /api/\<username\>/...

### Rules

The response from a hue command returns a response that is a list of dicts,
with each dict containing one key that is (hopefully) "Success" whose value 
is the successful action.  This action string can be used in a rule for
the rule's "action".  This is a handy way of getting the right syntax.

#### Rule Actions

Each action in the list of actions must be simple.  That is, each action's body must only be key/value pairs, where the values are not another compound object but either a string or a number or a boolean.

#### Rule Conditions

For some reason the value for `"value"` must be a string, e.g. `"1"` NOT `1`.

While using a condition with `"eq"` and `"0"` seems to work just fine, Philips Hue rules never use `"eq"` with `"0"`.  They always instead use `"lt"` with `"1"`.  Why?

#### Rule Operators

`"dx"` only seems to work with sensors.  The `"dx"` operator in a condition of
a Rule means "whenever this changes, this condition is triggered as true."
Often the address for this condition is the `lastupdated` field of a sensor.

`"ddx"` only seems to work with sensors.  The `"ddx"` operator in a condition
of a Rule means "whenever this changes, this condition is triggered as true
after a specified delay."  The condition will also include a `"value"` which is
a relative timePattern.

Other conditions accompanying a `"ddx"` condition sample their states after the
`"ddx"` delay, NOT during the original event.

### Sensors

#### CLIPGenericStatus

This is a state that contains a `"status"` field that can take on an integer
value.  It seems to be a 31-bit integer, with maximum value allowed of 
2_147_483_647.  It also keeps track of `"lastupdated"`

To Set (example is one item in list of `"actions"` in a Rule):
```
    {
        "address": "/sensors/15/state",
        "body": {
            "status": 1
        },
        "method": "PUT"
    }
```

#### CLIPGenericFlag

This is a state that contains a one-bit `"flag"` field that can take on a
boolean value.  It also keeps track of `"lastupdated"`

To Set (example is one item in list of `"actions"` in a Rule):
```
    {
        "address": "/sensors/15/state",
        "body": {
            "flag": false
        },
        "method": "PUT"
    }
```
