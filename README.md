Fork of https://github.com/jchv/go-webview2.

## new features

1. use `SetProcessDpiAwarenessContext` and `GetDpiForSystem` to solve HiDPI blurry problem.
```go
//go:build windows
// +build windows

package main

import (
	"fmt"
	"log"
	"syscall"

	"github.com/jchv/go-webview2"
)

func runGUIWebview2(url string) error {
	// 先尝试启动webview2
	w := webview2.NewWithOptions(webview2.WebViewOptions{
		Debug:     false,
		AutoFocus: true,
		WindowOptions: webview2.WindowOptions{
			Title:  "CyberMonk设备网络搜索",
			Width:  scaleDpi(windowWidth),
			Height: scaleDpi(windowHeight),
			IconId: 2, // icon resource id
			Center: true,
		},
	})

	// 如果webview2启动失败，再尝试启动默认浏览器
	if w == nil {
		log.Printf("Failed to load webview2.")
		return errGUIError
	}

	// 如果webview2成功
	defer w.Destroy()
	// w.SetTitle("CyberMonk设备搜索工具")
	w.Navigate(url)
	w.SetSize(int(scaleDpi(windowWidth)), int(scaleDpi(windowHeight)), webview2.HintMin)
	w.Run()

	return nil
}

const windowWidth uint = 900
const windowHeight uint = 750

const origDpi uint = 96

var realDpi uint = origDpi

func scaleDpi(i uint) uint {
	return i * realDpi / origDpi
}

func init() {
	dll := syscall.MustLoadDLL("user32")
	if proc, err := dll.FindProc("SetProcessDpiAwarenessContext"); err == nil {
		aware := -4
		proc.Call(uintptr(aware))
	}

	if proc, err := dll.FindProc("GetDpiForSystem"); err == nil {
		dpi, _, _ := proc.Call()
		fmt.Printf("dpi: %v", dpi)
		realDpi = uint(dpi)
	}
}

```



[![Go](https://github.com/jchv/go-webview2/actions/workflows/go.yml/badge.svg)](https://github.com/jchv/go-webview2/actions/workflows/go.yml) [![Go Report Card](https://goreportcard.com/badge/github.com/jchv/go-webview2)](https://goreportcard.com/report/github.com/jchv/go-webview2) [![Go Reference](https://pkg.go.dev/badge/github.com/jchv/go-webview2.svg)](https://pkg.go.dev/github.com/jchv/go-webview2)

# go-webview2
This package provides an interface for using the Microsoft Edge WebView2 component with Go. It is based on [webview/webview](https://github.com/webview/webview) and provides a compatible API.

Please note that this package only supports Windows, since it provides functionality specific to WebView2. If you wish to use this library for Windows, but use webview/webview for all other operating systems, you could use the [go-webview-selector](https://github.com/jchv/go-webview-selector) package instead. However, you will not be able to use WebView2-specific functionality.

If you wish to build desktop applications in Go using web technologies, please consider [Wails](https://wails.io/). It uses go-webview2 internally on Windows.

## Demo
If you are using Windows 10+, the WebView2 runtime should already be installed. If you don't have it installed, you can download and install a copy from Microsoft's website:

[WebView2 runtime](https://developer.microsoft.com/en-us/microsoft-edge/webview2/)

After that, you should be able to run go-webview2 directly:

```
go run ./cmd/demo
```

This will use go-winloader to load an embedded copy of WebView2Loader.dll. If you want, you can also provide a newer version of WebView2Loader.dll in the DLL search path and it should be picked up instead. It can be acquired from the WebView2 SDK (which is permissively licensed.)
