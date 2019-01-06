# Some language features of Jsonnet
- Written on 18/12/31
- Unpublished
- Updated never
## By Leland Later

JSON is the _de facto_ data interchange format. APIs consume and output JSON, humans consume and output JSON. JSON's rigid syntax underlies its ubiquity.

Jsonnet guarantees well-formatted JSON, but it also enables JSON users to programmatically change fields and values (for example, a database conection string field, or the name of an environment). A single Jsonnet file in version control replaces several almost-identical JSON files in the same repository.

This post contains a smattering of Jsonnet language features which demonstrate why Jsonnet is so powerful. For an introduction to Jsonnet, check out the [official Getting Started page](https://jsonnet.org/learning/getting_started.html) or a previous post on this blog, [Getting Started with Jsonnet](https://github.com/llater/Blog/blob/master/jsonnet-series/getting-started/getting-started.md).

### Conditional logic
Use `if`, `else` and `then` expressions to produce different outputs from different inputs:
```
$ cat document.jsonnet
local case = std.extVar("case");
{
    _config+:: {
        formatting: case,
    },
    formattedString:
        if $._config.formatting == "uppercase" 
            then "FORMATTED STRING"
        else if $._config.formatting == "lowercase"
            then "formatted string"
        else if $._config.formatting == "snakecase"
            then "Formatted_String"
        else "Formatted String",
}
$ jsonnet -V case=uppercase document.jsonnet
{
   "formattedString": "FORMATTED STRING"
}
$ jsonnet -V case=snakecase document.jsonnet
{
   "formattedString": "Formatted_String"
}
```

Because this code declares an external variable, `code` must exist as an external variable or the Jsonnet fails to compile:
```
$ jsonnet document.jsonnet
RUNTIME ERROR: undefined external variable: case
	document.jsonnet:1:14-32	thunk <case>
	document.jsonnet:4:21-25	object <anonymous>
	document.jsonnet:7:12-32	thunk <a>
	document.jsonnet:7:12-47	function <anonymous>
	document.jsonnet:7:12-47	object <anonymous>
	During manifestation
$ jsonnet -V case= document.jsonnet
{
   "formattedString": "Formatted String"
}
```

### Functions
To produce many like blocks of JSON with Jsonnet, use functions. Declare functions as any variable, with `local`, and accept parameters with parentheses:
```
$ cat setlist.jsonnet
// Represents a single song performed during a live set.
// Duration is measured in seconds.
local song(name, duration=0, instrument="guitar") = {
    name: name,
    duration: duration,
    instrument: instrument,
};
{
    firstSong: song("Hot Cross Buns", 15, "flute"),
    secondSong: song(name="Crab Canon", duration=940, instrument="harpischord"),
    finale: song("Welcome to the Jungle"),
}
```

`firstSong` demonstrates that parameters need not be named, while `secondSong` demonstrates that this is perfectly allowed. However, don't mix named and unnamed parameters, or compilation will yield this error:
`RUNTIME ERROR: internal error: got positional param after named at index n`

`finale` demonstrates the use of parameter defaults.

Compiling `setlist.jsonnet` yields repeated blocks:
```
$ jsonnet setlist.jsonnet
{
   "finale": {
      "duration": 0,
      "instrument": "guitar",
      "name": "Welcome to the Jungle"
   },
   "firstSong": {
      "duration": 15,
      "instrument": "flute",
      "name": "Hot Cross Buns"
   },
   "secondSong": {
      "duration": 940,
      "instrument": "harpischord",
      "name": "Crab Canon"
   }
}
```

### Looping
Finally, as a practical example, let's generate Kubernetes namespace manifests with a `for` loop. Consider these manifests to be simple JSON objects with one different value per file, a `name`:
```
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "name": <namespace name>
    }
}
```

By declaring all the namespaces in a single list variable, producing a single JSON object output is achieved in a single loop line:
```
$ cat ns.jsonnet
local namespaces = [
    'default',
    'monitoring',
    'util',
    'admin',
];
local namespaceTemplate(name) = {
    apiVersion: "v1",
    kind: "Namespace",
    metadata: {
        "name": name,
    },
};
{
    [name]: namespaceTemplate(name) for name in namespaces
}
$ jsonnet ns.jsonnet
{
   "admin": {
      "apiVersion": "v1",
      "kind": "Namespace",
      "metadata": {
         "name": "admin"
      }
   },
   "default": {
      "apiVersion": "v1",
      "kind": "Namespace",
      "metadata": {
         "name": "default"
      }
   },
   "monitoring": {
      "apiVersion": "v1",
      "kind": "Namespace",
      "metadata": {
         "name": "monitoring"
      }
   },
   "util": {
      "apiVersion": "v1",
      "kind": "Namespace",
      "metadata": {
         "name": "util"
      }
   }
}
```

Notice how `name` appears as `[name]` when used as a field in the Jsonnet code. The Jsonnet documentation terms this a "computed field". There is no outputed field "name", but there is a field for each "name" in the `namespaces` list.

This post highlighted some language features of Jsonnet: conditional logic, functions, and looping. But there are many more! Google [offered Jsonnet as an open source project a few years ago](https://opensource.googleblog.com/2015/04/jsonnet-more-elegant-language-for.html). Since then, Jsonnet has become an important tool in many cool projects, including Houseparty. If you already use Houseparty on your mobile device, why not [download the Mac app](https://houseparty.com/#download)?

#### Tags
- jsonnet
- JSON
- programming languages