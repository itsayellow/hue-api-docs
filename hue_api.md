# Hue API Documentation

[![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

by Matthew A. Clapp

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg

Notes on using the Hue API.

### Datatypes

#### Time Patterns

The Weekday specification `[bbb]` is a decimal number of the binary
specification where each bit stands for a day of the week:

| Mon | Tue | Wed | Thu | Fri | Sat | Sun |
|-----|-----|-----|-----|-----|-----|-----|
|   64|   32|   16|    8|    4|    2|    1|

#### Colors (x,y)

Hue seems to represent x- and y-values for colors using a fixed-point register.
It is equivalent to an integer divided by 10_000.  It has an absolute
accuracy of 0.0001.

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

There are a maximum number of 8 actions in a given rule.

#### Rule Conditions

For some reason the value for `"value"` must be a string, e.g. `"1"` NOT `1`.

While using a condition with `"eq"` and `"0"` seems to work just fine, Philips
Hue rules never use `"eq"` with `"0"`.  They always instead use `"lt"` with
`"1"`.  Why?

There are a maximum number of conditions in a given rule.  The limit **may** be
8, like actions.

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
