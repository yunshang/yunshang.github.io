---
title: go mod hello world
date: 2020-07-08 07:44:42
tags:
---
> golang 从1.11开始引入了go mod机制管理依赖包，本文对go mod简单hello world了一下。由于不同版本go mod有所差异，我使用的go环境是1.13版本的。

建立hellogomod文件夹初始化go mod:

```go
go mod init

cat go.mod
module hellogomod

go 1.13
```

将以下内容保存为main.go：

```go

package main

import (
    log "github.com/sirupsen/logrus"
)

func main() {
    log.WithFields(log.Fields{
        "limao": "go mod try",
    }).Info("Hello world!")
}
```

更新go mod：

```go
go mod tidy

cat go.mod
module hellogomod

go 1.13

require github.com/sirupsen/logrus v1.4.2
```

将vender文件cache到本目录：
```

go mod vendor

ll vendor
total 8
drwxr-xr-x  4 limao  staff   128B 12 31 10:14 github.com
drwxr-xr-x  3 limao  staff    96B 12 31 10:14 golang.org
-rw-r--r--  1 limao  staff   250B 12 31 10:14 modules.txt
```

编译时使用vender：

```go
go build -mod=vendor main.go
```