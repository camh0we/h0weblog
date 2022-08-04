---
layout: post
title: gootstrap
author: camh0we
tags: golang git bash cli
---

[gootstrap](https://github.com/camh0we/gootstrap) is a tool that bootstraps a new `go` project the way i like it

running [`./gootstrap.sh`](https://github.com/camh0we/gootstrap/blob/main/gootstrap.sh) from an empty directory will result in a directory structure like this:

```bash
.
├── Makefile
├── bin
│   └── newproj
├── cmd
│   └── newproj
│       └── main.go
├── go.mod
├── internal
└── pkg
```

the main package just prints "hello world"

```go
package main

import "fmt"

func main() {
	fmt.Println("### ~~~ hello world")
}
```

empty `/internal` and `/pkg` directories are created to organize future packages

`Makefile` provides basic commands to build, test, and run
