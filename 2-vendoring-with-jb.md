# Vendoring Jsonnet code with jb, the jsonnet-bundler
- Written on 19/1/6
- Unpublished
- Updated never
## By Leland Later

Jsonnet is a powerful tool in its own right. Adding Jsonnet to a codebase turns the stickiest parts of configuration and data description into cake. A few lines of Jsonnet can replace many lines of human-unreadable JSON.

But what about adding someone else's Jsonnet code to your project? Vendoring is a common pattern to enumerate and include external dependencies. [npm](https://github.com/npm/cli), the JavaScript package manager, is perhaps the best known example of a vendoring solution.

In a [previous post on this blog](https://github.com/llater/Blog/blob/master/jsonnet-series/getting-started/getting-started.md), the `_config` pattern was introduced as a means to clearly define variables in Jsonnet code. Leverage the `_config` pattern and vendored code to supercharge your Jsonnet with others'!

First, this post will explain how to install a Go tool from scratch ([the jsonnet-bundler](https://github.com/jsonnet-bundler/jsonnet-bundler), `jb`, is written in Go). Then it will explain how to vendor external Jsonnet code.

To install `jb`, the jsonnet-bundler, the [Go programming language](https://golang.org/) must be installed. Install it with your favorite package manager or check out the [installation documentation](https://golang.org/doc/install) for other methods.

Run these commands (or similar ones) to install `jb` and add it to your executable path:
```
$ go get -u github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb
$ cd "$(go env GOPATH)/src/github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb"
$ go build
$ mv jb /usr/local/bin
```

Run the command, `jb`, to ensure the tool was installed successfully.

Before adding external Jsonnet code to a `vendor/` subdirectory, prepare the project's directory with `jb init`. This command creates a `jsonnetfile.json`, which will list dependencies as they are added to the project. Right now it's empty:
```
$ cat jsonnetfile.json
{}
```

To add external Jsonnet dependencies run `jb install`. To update those dependencies, run `jb update`. These commands alter the `jsonnetfile.json` and a `jsonnetfile.lock.json` file, repsectively. Check these files into version control so others can compile this program with the vendored code.

If you are familiar with `npm`, this pattern likely looks familiar.

[ksonnet](https://github.com/ksonnet/ksonnet-lib) is a popular Jsonnet library. Use this library to generate airtight Kubernetes manifests in JSON.

First, vendor the library with `jb install https://github.com/ksonnet/ksonnet-lib`. Import the ksonnet library with the `import` built-in, as if it was in your local directory:
```
$ cat app.libsonnet
local k = import 'ksonnet-lib/ksonnet.beta.2/k.libsonnet';
...
```

Now, to compile the code with an additional import path (the vendor directory), add the `-J` flag:
```
$ jsonnet -J vendor app.libsonnet
```

Without this flag, you'll receive an error like this:
```
$ jsonnet app.libsonnet
RUNTIME ERROR: couldn't open import "ksonnet-lib/ksonnet.beta.2/k.libsonnet": no match locally or in the Jsonnet library paths.
	app.libsonnet:1:11-58	thunk <k>
	app.libsonnet:17:1
```

`vendor/` will not live in version control; rather, each time local compilation takes place, if `vendor/` does not exist, it will be generated with `jb install`.

Vendoring is imperfect as a dependency management solution in many software uses cases, but for Jsonnet, vendoring is a capable and simple to understand method.

Have you been on Houseparty recently? Send your family a [Facemail](https://medium.com/@houseparty/youve-got-facemail-fdc71ef8c561)!

#### Tags
- Jsonnet
- jsonnet-bundler
- Golang
- Go
- Kubernetes
- ksonnet
- k8s