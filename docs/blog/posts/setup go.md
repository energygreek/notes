---
draft: true
comments: true
date: 2025-12-11
tags: []
---

# setup go

## install go
pacman -Syu
pacman -S go

## install plugin vsix
https://github.com/golang/vscode-go/releases/tag/v0.50.0


## setup go proxy

go env -w  GOPROXY=https://goproxy.cn,direct   

## install lsp
go install -v golang.org/x/tools/gopls@latest
go install -v honnef.co/go/tools/cmd/staticcheck@latest
go install -v github.com/cweill/gotests/gotests@v1.6.0
go install -v github.com/josharian/impl@v1.4.0
go install -v github.com/haya14busa/goplay/cmd/goplay@v1.0.0
go install -v github.com/go-delve/delve/cmd/dlv@latest

## create new project
mkdir myproject
cd myproject
go mod init myproject

### create main.go

```go
package main

import (
	"fmt"

	"github.com/vishvananda/netlink"
)

func main() {
	links, err := netlink.LinkList()
	if err != nil {
		panic(err)
	}
	for _, l := range links {
		fmt.Println(l.Attrs().Name)
	}
}
```

## build
run command `go mod tidy` first or directly run `go build` will automaticlly download dependency