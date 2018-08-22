---
title: "Golang"
authors: ["gio"]
---

#### Installing on Linux

```shell
sudo add-apt-repository ppa:gophers/archive -y
sudo apt-get update
sudo apt-get install golang-1.10-go -y
```

**Installing Glide (Package Manager)**

```shell
sudo add-apt-repository ppa:masterminds/glide -y
sudo apt-get update
sudo apt-get install glide -y
```

#### Install all dependencies recursively

To install all dependencies of a golang project or golang projects recursively with the `go get` command, change directory into the project and simply run:

```shell
go get ./...
```

#### Manipulating Slice

1. https://github.com/golang/go/wiki/SliceTricks
2. https://stackoverflow.com/questions/25025409/delete-element-in-a-slice

#### Manipulating JSON

https://blog.golang.org/json-and-go

#### Useful Standard Library

https://golang.org/pkg/time/#Sleep

#### Rewriting Request Body

We need to modify the content length if we modify http request body.

```golang
req.ContentLength = int64(len(b))
req.Body = ioutil.NopCloser(bytes.NewReader(b))
```

See [this](https://stackoverflow.com/questions/33606330/golang-rewrite-http-request-body).

#### Useful Libraries

1. [CLI App Framework](https://github.com/urfave/cli)
