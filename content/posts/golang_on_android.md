---
title: "Golang on android"
date: 2023-03-20T14:33:10+01:00
draft: true
---

You need Golang 1.16 at least 

I am using go version go1.20.2 linux/amd64

You need go path in PATH

```
export PATH="$PATH:$(go env GOPATH)/bin"
```

Install

```
go install golang.org/x/mobile/cmd/gomobile@latest
gomobile init
go get golang.org/x/mobile/bind
```

Install NDK

```
git clone https://aur.archlinux.org/android-ndk.git
cd android-ndk
makepkg -si

export ANDROID_NDK_HOME="/opt/android-ndk"
```

make a go program 

```go
package notmain

func main() {
    fmt.Println("hello world")
}
```

build

```
gomobile bind --target android -androidapi 19 .
```

