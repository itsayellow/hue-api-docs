# Hue API Documentation

by Matthew A. Clapp

[![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg

Notes on using the Hue API.

## Helpful Links

https://github.com/Overboard/discoverhue

https://github.com/quentinsf/qhue

https://developers.meethue.com/develop/hue-api/

## Notes

### Colors

First we get an RGB value.  If this RGB value is from a computer, it probably
has a Gamma function applied to it, which means that every R,G,B value has had
a nonlinear exponential function applied to it.  Thus we need to undo this
function to get linearized R,G,B values before proceeding with future
computations.

From a sensor, presumably we have linear outputs for R,G,B and may omit the
linearization step.

The [Hue api documentation on
colors](https://developers.meethue.com/develop/application-design-guidance/color-conversion-formulas-rgb-to-xy-and-back/)
has a missing group of equations for the "Wide RGB D65 conversion formula".  A
[gist](https://gist.github.com/popcorn245/30afa0f98eea1c2fd34d) was found that
contained the formulas.  **NOTE** this is a different formula than the one in
"How to convert between sRGB and CIEXYZ" linked below!

* `float X = red * 0.649926f + green * 0.103455f + blue * 0.197109f;`
* `float Y = red * 0.234327f + green * 0.743075f + blue * 0.022598f;`
* `float Z = red * 0.0000000f + green * 0.053077f + blue * 1.035763f;`

Then from the  the Hue xy values (little-x, little-y) are as follows.  Little-x
and little-y are X and Y normalized to sum(X,Y,Z).

* `float x = X / (X + Y + Z);`
* `float y = Y / (X + Y + Z);`

Hue also says that `brightness = Y` which presumably means that `bri = 254*Y`.
(??)

### Links

["How to convert between sRGB and
CIEXYZ"](https://www.image-engineering.de/library/technotes/958-how-to-convert-between-srgb-and-ciexyz)

["RGB/XYZ
Matrices"](http://www.brucelindbloom.com/index.html?Eqn_RGB_XYZ_Matrix.html)
* This seems excellent.  What we need to do is to solve for the matrix M that maps between sensor RGB and Hue XY by changing the Hue bulbs color and noting the RGB values.

[Wikipedia "CIE 1931 color
space"](https://en.wikipedia.org/wiki/CIE_1931_color_space)

### Datatypes

#### Time Patterns

The Weekday specification `[bbb]` is a decimal number of the binary
specification where each bit stands for a day of the week:

| Mon | Tue | Wed | Thu | Fri | Sat | Sun |
|-----|-----|-----|-----|-----|-----|-----|
|   64|   32|   16|    8|    4|    2|    1|

### Schedules

Schedules (inexplicably) use the long form of the address in `"command"`, i.e.
/api/\<username\>/...

The `recycle` field should only be true if we include the schedule in a list of
addresses in a resourcelink using hue\_bridge.resourcelinks().  In this case
when the resroucelink is deleted, so will any resources linked to it that
have `recycle` set to true.

`autodelete` is by default true.  This is good, so that any non-recurring
schedule that is past (expired) will be automatically deleted.

### Rules

The response from a hue command returns a response that is a list of dicts,
with each dict containing one key that is (hopefully) "Success" whose value 
is the successful action.  This action string can be used in a rule for
the rule's "action".  This is a handy way of getting the right syntax.

#### Rule Actions

Each action in the list of actions must be simple.  That is, each action's body
must only be key/value pairs, where the values are not another compound object
but either a string or a number or a boolean.

#### Rule Conditions

For some reason the value for `"value"` must be a string, e.g. `"1"` NOT `1`.

While using a condition with `"eq"` and `"0"` seems to work just fine, Philips
Hue rules never use `"eq"` with `"0"`.  They always instead use `"lt"` with
`"1"`.  Why?

#### Rule Operators

`"dx"` only seems to work with sensors.  The `"dx"` operator in a condition of
a Rule means "whenever this changes, this condition is triggered as true."
Often the address for this condition is the `lastupdated` field of a sensor.

`"ddx"` only seems to work with sensors.  The `"ddx"` operator in a condition
of a Rule means "whenever this changes, this condition is triggered as true
after a specified delay."  The condition will also include a `"value"` which is
a relative timePattern.  During the delay, the address must remain stable.
If the address in the `"ddx"` condition changes, the delay will start again.
If you need a delay that is longer than the address will be stable, it is
best to use a rule with a schedule timer instead of `"ddx"`.

### Sensors

#### CLIPGenericStatus

This is a state that contains a `"status"` field that can take on an integer
value.  It seems to be a 31-bit integer, with maximum value allowed of 
2\_147\_483\_647.  It also keeps track of `"lastupdated"`

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

### Resourcelinks

Each resourcelink contains a list of links to other resources on the bridge
(addresses of specific groups, rules, scenes, schedules, sensors).  If the
specified resource has the flag `recycle` set to True, then if all
resourcelinks pointing to it are deleted, it too will be deleted.
