# huedaysync

## Helpful Links

https://github.com/Overboard/discoverhue

https://github.com/quentinsf/qhue

https://developers.meethue.com/develop/hue-api/

## Notes

### Datatypes

### Time Patterns

The Weekday specification [bbb] is a decimal number of the binary specification
where each bit stands for a day of the week:

Monday = 64, Tuesday = 32, Wednesday = 16, Thursday = 8,
Friday = 4, Saturday = 2, Sunday = 1

| Mon | Tue | Wed | Thu | Fri | Sat | Sun |
|-----|-----|-----|-----|-----|-----|-----|
| 64  | 32  | 16  |  8  |  4  |  2  |  1  |

### Rules

The response from a hue command returns a response that is a list of dicts,
with each dict containing one key that is (hopefully) "Success" whose value 
is the successful action.  This action string can be used in a rule for
the rule's "action".  This is a handy way of getting the right syntax.

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
