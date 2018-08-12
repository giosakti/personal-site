---
title: "Golang"
authors: ["gio"]
---

#### Installing on Linux

```shell
$ sudo add-apt-repository ppa:gophers/archive -y
$ sudo apt-get update
$ sudo apt-get install golang-1.10-go -y
```

**Installing Glide (Package Manager)**

```shell
$ sudo add-apt-repository ppa:masterminds/glide -y
$ sudo apt-get update
$ sudo apt-get install glide -y
```

#### Rewriting Request Body

We need to modify the content length if we modify http request body.

```golang
req.ContentLength = int64(len(b))
req.Body = ioutil.NopCloser(bytes.NewReader(b))
```

See [this](https://stackoverflow.com/questions/33606330/golang-rewrite-http-request-body).
