# Compiling Jsonnet with Go
- Written on 19/1/8
- Unpublished
- Updated never
## By Leland Later

What do Kubernetes, Prometheus and Docker have in common? They are all software: software written for networked systems. Incidentally, they are all written in Go. Go is a systems programming language especially suited to networked systems (ie. microservices, network-reliant services).

Jsonnet was initially written in C++ but at this time there is also a functional [Go implementation](https://github.com/google/go-jsonnet).

To cover:
- hypothesize # of cores
- profiling CPU of Go app
- monitoring it with Prometheus
- build a Go CLI
  - `jsonnet` to run jsonnet command with built-in cluster args
  - `apply` to apply a compiled jsonnet output to the Kubernetes API
- test / profile Go implementation vs. C++ implementation