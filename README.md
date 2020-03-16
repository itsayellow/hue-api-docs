# huedaysync

## Helpful Links

https://github.com/Overboard/discoverhue

https://github.com/quentinsf/qhue

https://developers.meethue.com/develop/hue-api/

## Notes

### Operators

The `"dx"` operator in a condition of a Rule means "whenever this changes,
this condition is triggered as true."  Often the address for this condition
is the `lastupdated` field of a sensor or light.

The `"ddx"` operator in a condition of a Rule means "whenever this changes,
this condition is triggered as true after a specified delay."  The condition
will also include a `"value"` which is a relative timePattern.

### Sensors

#### CLIPGenericStatus

This is a state that contains a `"status"` field that can take on an integer
value.  It also keeps track of `"lastupdated"`

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
