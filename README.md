# webview


[![Build Status](https://img.shields.io/github/workflow/status/Lightning1337/webview/CI%20Pipeline)](https://github.com/Lightning1337/webview)
[![GoDoc](https://godoc.org/github.com/Lightning1337/webview?status.svg)](https://godoc.org/github.com/Lightning1337/webview)
[![Go Report Card](https://goreportcard.com/badge/github.com/Lightning1337/webview)](https://goreportcard.com/report/github.com/Lightning1337/webview)


A fork of [github.com/webview/webview](https://github.com/webview/webview)

Webview is a cross-platform library for C/C++/Golang to build modern GUIs.

It uses Cocoa/WebKit on macOS, gtk-webkit2 on Linux and Edge on Windows 10.

## Todo

- [x] Add `show()`, `hide()`, `minimize()`, `maximize()`, `set_icon()` and `set_icon_from_file()`
- [x] Fix build errors on Linux and MacOS
- [x] Fix 0xc0000005 error on Windows
- [ ] Add WebView2 helper functions
  - [ ] `is_webview2_runtime_installed()`
  - [ ] `ensure_webview2_runtime()`
- [ ] Implement `set_icon()` and `set_icon_from_file()` for MacOS
- [ ] Add DPI awareness on Windows
- [ ] Add `inject_css()` and `inject_html()` functions
- [ ] Add toast notifications
- [ ] Add dialog windows
- [ ] Implement `hide_to_system_tray()` in Linux (and MacOS?)
- [ ] Clean up and fix Cocoa/Objective-C code

## Webview for Go developers

### Getting started

Install Webview library with `go get`:

```
$ go get github.com/Lightning1337/webview
```

Import the package and start using it:

```go
package main

import "github.com/Lightning1337/webview"

func main() {
	debug := true
	w := webview.New(debug)
	defer w.Destroy()
	w.SetTitle("Minimal webview example")
	w.SetSize(800, 600, webview.HintNone)
	w.SetIconFromFile("path/to/icon.ico")
	w.Navigate("https://en.m.wikipedia.org/wiki/Main_Page")
	w.Run()
}
```

To build the app use the following commands:

```bash
# Linux
$ go build -o webview-example && ./webview-example

# MacOS uses app bundles for GUI apps
$ mkdir -p example.app/Contents/MacOS
$ go build -o example.app/Contents/MacOS/example
$ open example.app # Or click on the app in Finder

# Windows requires special linker flags for GUI apps.
# It's also recommended to use TDM-GCC-64 compiler for CGo.
# http://tdm-gcc.tdragon.net/download
$ go build -ldflags="-H windowsgui" -o webview-example.exe
```

For more details see [godoc](https://godoc.org/github.com/Lightning1337/webview).

### Distributing webview apps

On Linux you get a standalone executable. It will depend on GTK3 and GtkWebkit2, so if you distribute your app in DEB or RPM format include those dependencies. An application icon can be specified by providing a `.desktop` file.

On MacOS you are likely to ship an app bundle. Make the following directory structure and just zip it:

```
example.app
└── Contents
    ├── Info.plist
    ├── MacOS
    |   └── example
    └── Resources
        └── example.icns
```

Here, `Info.plist` is a [property list file](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/AboutInformationPropertyListFiles.html) and `*.icns` is a special icon format. You may convert PNG to icns [online](https://iconverticons.com/online/).

On Windows you probably would like to have a custom icon for your executable. It can be done by providing a resource file, compiling it and linking with it. Typically, `windres` utility is used to compile resources. Also, on Windows, `webview.dll` and `WebView2Loader.dll` must be placed into the same directory with your app executable. 


> **NOTE:**
> To use the `edge_chromium` browser engine ([WebView2](https://docs.microsoft.com/en-us/microsoft-edge/webview2/)), you need to have the [WebView2 Runtime](https://developer.microsoft.com/en-us/microsoft-edge/webview2/#download-section) installed. If you want to prompt the user to install it when they run your app and it isn't installed, call the `EnsureWebView2Runtime()` function before `Run()`. If the WebView2 runtime is not installed, it will fallback to the `edge_html` browser engine.

## Webview for C/C++ developers

Download [webview.h](https://raw.githubusercontent.com/zserge/webview/master/webview.h) and include it in your C/C++ code:

### C++:
```c
// main.cc
#include "webview.h"
#ifdef WIN32
int WINAPI WinMain(HINSTANCE hInt, HINSTANCE hPrevInst, LPSTR lpCmdLine, int nCmdShow) {
#else
int main() {
#endif
  webview::webview w(true, nullptr);
  w.set_title("Minimal example");
  w.set_size(480, 320, WEBVIEW_HINT_NONE);
  w.navigate("https://en.m.wikipedia.org/wiki/Main_Page");
  w.run();
  return 0;
}
```
Build it:

```bash
# Linux
$ c++ main.cc `pkg-config --cflags --libs gtk+-3.0 webkit2gtk-4.0` -o webview-example
# MacOS
$ c++ main.cc -std=c++11 -framework WebKit -o webview-example
# Windows (x64)
$ c++ main.cc -mwindows -L./dll/x64 -lwebview -lWebView2Loader -o webview-example.exe
```

### C:
```c
// main .c
#include "webview.h"
#ifdef WIN32
int WINAPI WinMain(HINSTANCE hInt, HINSTANCE hPrevInst, LPSTR lpCmdLine, int nCmdShow) {
#else
int main() {
#endif
	webview_t w = webview_create(0, NULL);
	webview_set_title(w, "Webview Example");
	webview_set_size(w, 480, 320, WEBVIEW_HINT_NONE);
	webview_navigate(w, "https://en.m.wikipedia.org/wiki/Main_Page");
	webview_run(w);
	webview_destroy(w);
	return 0;
}
```
Build it:

```bash
# Linux
$ g++ main.c `pkg-config --cflags --libs gtk+-3.0 webkit2gtk-4.0` -o webview-example
# MacOS
$ g++ main.c -std=c++11 -framework WebKit -o webview-example
# Windows (x64)
$ g++ main.c -mwindows -L./dll/x64 -lwebview -lWebView2Loader -o webview-example.exe
```

On Windows it is possible to use webview library directly when compiling with cl.exe, but WebView2Loader.dll is still required. To use MinGW you may dynamically link prebuilt webview.dll (this approach is used in Cgo bindings).

Full C/C++ API is described at the top of the `webview.h` file.

## Notes

Execution on OpenBSD requires `wxallowed` [mount(8)](https://man.openbsd.org/mount.8) option.
For Ubuntu Users run `sudo apt install webkit2gtk-4.0`(Try with webkit2gtk-4.0-dev if webkit2gtk-4.0 is not found) to install webkit2gtk-4.0 related items.
FreeBSD is also supported, to install webkit2 run `pkg install webkit2-gtk3`.

## License

Code is distributed under MIT license, feel free to use it in your proprietary
projects as well.

