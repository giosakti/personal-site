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

**Installing golang/dep (Package Manager)**

```shell
# OSX
brew install dep

# Linux
curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
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

#### Rewriting Request Body

We need to modify the content length if we modify http request body.

```golang
req.ContentLength = int64(len(b))
req.Body = ioutil.NopCloser(bytes.NewReader(b))
```

See [this](https://stackoverflow.com/questions/33606330/golang-rewrite-http-request-body).

#### Mocking

Do this inside one of your project

```shell
dep ensure -add github.com/golang/mock/gomock
go get github.com/golang/mock/mockgen
# Ensure your $GOPATH/bin is in your $PATH
mkdir mock
mockgen -destination=mock/mock_<file_name>.go -package=mock <package> <interface>
```

#### Limit of HTTP Handler Concurrent Connection

https://stackoverflow.com/questions/47385692/limited-concurrent-connections-in-go

#### Useful Standard Library

https://golang.org/pkg/time/#Sleep

#### Useful Libraries

1. [CLI App Framework](https://github.com/urfave/cli)

#### Best Practices

- https://github.com/istio/istio/wiki/Dev-Writing-Fast-and-Lean-Code
