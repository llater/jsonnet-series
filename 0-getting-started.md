# Getting started with Jsonnet
- Written on 18/12/26
- Unpublished
- Updated never
## By Leland Later

[Jsonnet](https://jsonnet.org/) is a data templating language. It is specially suited to template JSON, as it is an extension of the [JSON specification](https://tools.ietf.org/html/rfc7159) and provides a robust yet terse standard library for variable interpolation and string manipulation.

Like [YAML](https://yaml.org/spec/), Jsonnet is a superset of the JSON spec (any valid JSON is also valid Jsonnet). After installing Jsonnet with your favorite package manager, try running the tool against some well-formatted JSON:
```
$ cat produce.json
{
    "fruits": [
        "guava",
        "green grapes",
        "raspberries"
    ],
    "treat": "donut"
}
$ jsonnet produce.json
{
   "fruits": [
      "guava",
      "green grapes",
      "raspberries"
   ],
   "treat": "donut"
}
```

Consider an almost identical Jsonnet file which calls for a single external variable:
```
$ cat produce.jsonnet
{
    "fruits": [
        "guava",
        "green grapes",
        "raspberries"
    ],
    "treat": std.extVar("treat"),
}
$ jsonnet -V treat=pineapple produce.jsonnet
{
   "fruits": [
      "guava",
      "green grapes",
      "raspberries"1`   
   ],
   "treat": "pineapple"
}
```

The standard library, referenced as `std`, is a Jsonnet object. `std.extVar(x)` is the first function listed in the [standard library documentation](https://jsonnet.org/ref/stdlib.html).

Often, Jsonnet code contains a top-level `_config` object:
```
$ cat produce.jsonnet
{
    _config+:: {
        treat: std.extVar("treat"),
    },
    "fruits": [
      "guava",
      "green grapes",
      "raspberries"
   ],
   "treat": $._config.treat,
}
```

This pattern has two benefits: it clearly defines the variable name at the top of the code block, and it limits function calls when the same variable is used multiple times. The `::` syntax indicates the `_config` field is hidden (that is, there will be no `_config` field rendered in the JSON output). The `+` syntax indicates that the `_config` object will not replace other `_config` objects in lower-level Jsonnet objects. Rather, a single `_config` object (containing the union of all `_config`s in the Jsonnet code) will be interpolated at runtime.

It is easy to combine pure JSON and Jsonnet code with the `import` function:
```
$ ls
produce.jsonnet	supplies.json
$ cat produce.jsonnet
local s = import 'supplies.json';
{
    _config+:: {
        treat: std.extVar("treat"),
    },
    "fruits": [
      "guava",
      "green grapes",
      "raspberries"
   ],
   "treat": $._config.treat,
   "other": s,
}
$ jsonnet -V treat=donuts produce.jsonnet
{
   "fruits": [
      "guava",
      "green grapes",
      "raspberries"
   ],
   "other": {
      "supplies": [
         "tarpaulin",
         "propane"
      ]
   },
   "treat": "donuts"
}
```

At Houseparty, we live out the benefits of data templating daily. Download our app for iOS, Android and Mac!

#### Tags
- jsonnet
- JSON
- data
- template