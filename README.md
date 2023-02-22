# Golang Webview GUI

A tiny cross-platform webview library for C/C++/Go to build modern cross-platform GUIs.

The goal of the project is to create a common HTML5 UI abstraction layer for the most widely used platforms.

It supports two-way JavaScript bindings (to call JavaScript from C/C++/Go and to call C/C++/Go from JavaScript).

## Platform Support

Platform | Technologies
-------- | ------------
Linux    | [GTK 3][gtk], [WebKitGTK][webkitgtk]
macOS    | Cocoa, [WebKit][webkit]
Windows  | [Windows API][win32-api], [WebView2][ms-webview2]

## Documentation

We have started working on publishing documentation at [webview.dev] but you can always find the most up-to-date documentation right in the source code. Improving the documentation is a continuous effort and you are more than welcome to [offer suggestions][issues-new] or [contribute with content][docs-repo]. Please bear with us if the latest updates are not yet published.

## Prerequisites

Your compiler must support minimum C++11 except for platforms that require a more modern version.

### Linux and BSD

The [GTK][gtk] and [WebKit2GTK][webkitgtk] libraries are required for development and distribution. You need to check your package repositories regarding how to install those those.

Debian-based systems:

* Packages:
  * Development: `apt install libgtk-3-dev libwebkit2gtk-4.0-dev`
  * Production: `apt install libgtk-3-0 libwebkit2gtk-4.0-37`

Fedora-based systems:

* Packages:
  * Development: `dnf install gtk3-devel webkit2gtk4.0-devel`
  * Production: `dnf install gtk3 webkit2gtk4.0`

BSD-based systems:

* FreeBSD packages: `pkg install webkit2-gtk3`
* Execution on BSD-based systems may require adding the `wxallowed` option (see [mount(8)](https://man.openbsd.org/mount.8))  to your fstab to bypass [W^X](https://en.wikipedia.org/wiki/W%5EX "write xor execute") memory protection for your executable. Please see if it works without disabling this security feature first.

### Windows

Your compiler must support C++17 and we recommend to pair it with an up-to-date Windows 10 SDK.

For Visual C++ we recommend Visual Studio 2022 or later. We have a [separate section for MinGW-w64](#mingw-w64-requirements).

Developers and end-users must have the [WebView2 runtime][ms-webview2-rt] installed on their system for any version of Windows before Windows 11.

## Getting Started

If you are a developer of this project then please go to the [development section](#development).

Instructions here are written for GCC when compiling C/C++ code using Unix-style command lines, and assumes that multiple commands are executed in the same shell session. Command lines for Windows use syntax specific to the Command shell but you can use any shell such as PowerShell as long as you adapt the commands accordingly. See the [MinGW-w64 requirements](#mingw-w64-requirements) when building on Windows.

You will have a working app but you are encouraged to explore the [available examples][examples] and try the ones that go beyond the mere basics.

Start with creating a new directory structure for your project:

```sh
mkdir my-project && cd my-project
mkdir build libs "libs/webview"
```

### Windows Preparation

The [WebView2 SDK][ms-webview2-sdk] is required when compiling programs:

```bat
mkdir libs\webview2
curl -sSL "https://www.nuget.org/api/v2/package/Microsoft.Web.WebView2" | tar -xf - -C libs\webview2
```

If you wish to use the official WebView2 loader (`WebView2Loader.dll`) then grab a copy of the DLL (replace `x64` with your target architecture):

```bat
copy /Y libs\webview2\build\native\x64\WebView2Loader.dll build
```

> **Note:** See the [WebView2 loader section](#ms-webview2-loader) for more options.

### C/C++ Preparation

Fetch the webview library:

```sh
curl -sSLo "libs/webview/webview.h" "https://raw.githubusercontent.com/webview/webview/master/webview.h"
curl -sSLo "libs/webview/webview.cc" "https://raw.githubusercontent.com/webview/webview/master/webview.cc"
```


### Getting Started with Go

See [Go package documentation][go-docs] for the Go API documentation, or simply read the source code.

Create a new Go module:

```sh
go mod init example.com/m
```

On Windows you will need to make the WebView2 headers discoverable by cgo (see [Windows Preperation](#windows-preperation)):

```bat
set CGO_CXXFLAGS="-I%cd%\libs\webview2\build\native\include"
```

> **Note:** Argument quoting works for Go 1.18 and later. Quotes can be removed if paths have no spaces.

Save the basic Go example into your project directory:

```sh
curl -sSLo basic.go "https://raw.githubusercontent.com/webview/webview/master/examples/basic.go"
```

Install dependencies:

```sh
go get github.com/webview/webview
```

Build and run the example:

```sh
# Linux, macOS
go build -o build/basic basic.go && ./build/basic
# Windows
go build -ldflags="-H windowsgui" -o build/basic.exe basic.go && "build/basic.exe"
```

### More Examples

The examples shown here are mere pieces of a bigger picture so we encourage you to try [other examples][examples] and explore on your own—you can follow the same procedure. Please [get in touch][issues-new] if you find any issues.

## App Distribution

Distribution of your app is outside the scope of this library but we can give some pointers for you to explore.

### macOS Application Bundle

On macOS you would typically create a bundle for your app with an icon and proper metadata.

A minimalistic bundle typically has the following directory structure:

```
example.app                 bundle
└── Contents
    ├── Info.plist          information property list
    ├── MacOS
    |   └── example         executable
    └── Resources
        └── example.icns    icon
```

Read more about the [structure of bundles][macos-app-bundle] at the Apple Developer site.

> Tip: The `png2icns` tool can create icns files from PNG files. See the `icnsutils` package for Debian-based systems.

### Windows Apps

You would typically create a resource script file (`*.rc`) with information about the app as well as an icon. Since you should have MinGW-w64 readily available then you can compile the file using `windres` and link it into your program. If you instead use Visual C++ then look into the [Windows Resource Compiler][win32-rc].

The directory structure could look like this:

```
my-project/
├── icons/
|   ├── application.ico
|   └── window.ico
├── basic.cc
└── resources.rc
```

`resources.rc`:
```
100 ICON "icons\\application.ico"
32512 ICON "icons\\window.ico"
```

> **Note:** The ID of the icon resource to be used for the window must be `32512` (`IDI_APPLICATION`).

Compile:
```sh
windres -o build/resources.o resources.rc
g++ basic.cc build/resources.o [...]
```

Remember to bundle the DLLs you have not linked statically, e.g. those from MinGW-w64 and optionally `WebView2Loader.dll`.

## MinGW-w64 Requirements

In order to build this library using MinGW-w64 on Windows then it must support C++17 and have an up-to-date Windows SDK. This applies both when explicitly building the C/C++ library as well as when doing so implicitly through Go/cgo.

Distributions that are known to be compatible:

* [LLVM MinGW](https://github.com/mstorsjo/llvm-mingw)
* [MSYS2](https://www.msys2.org/)
* [WinLibs](https://winlibs.com/)

## MS WebView2 Loader

Linking the WebView2 loader part of the Microsoft WebView2 SDK is not a hard requirement when using our webview library, and neither is distributing `WebView2Loader.dll` with your app.

If, however, `WebView2Loader.dll` is loadable at runtime, e.g. from the executable's directory, then it will be used; otherwise our minimalistic implementation will be used instead.

Should you wish to use the official loader then remember to distribute it along with your app unless you link it statically. Linking it statically is possible with Visual C++ but not MinGW-w64.

Here are some of the noteworthy ways our implementation of the loader differs from the official implementation:

* Does not support configuring WebView2 using environment variables such as `WEBVIEW2_BROWSER_EXECUTABLE_FOLDER`.
* Microsoft Edge Insider (preview) channels are not supported.

The following compile-time options can be used to change how the library integrates the WebView2 loader:

* `WEBVIEW_MSWEBVIEW2_BUILTIN_IMPL=<1|0>` - Enables or disables the built-in implementation of the WebView2 loader. Enabling this avoids the need for `WebView2Loader.dll` but if the DLL is present then the DLL takes priority. This option is enabled by default.
* `WEBVIEW_MSWEBVIEW2_EXPLICIT_LINK=<1|0>` - Enables or disables explicit linking of `WebView2Loader.dll`. Enabling this avoids the need for import libraries (`*.lib`). This option is enabled by default if `WEBVIEW_MSWEBVIEW2_BUILTIN_IMPL` is enabled.

## Development

To build the library, examples and run tests, run `script/build.sh` on Unix-based systems and `script/build.bat` on Windows.

> **Note:** These scripts are not in the best condition but a rewrite is being planned. Please bear with us and manually edit the scripts to your liking.

## Limitations

### Browser Features

Since a browser engine is not a full web browser it may not support every feature you may expect from a browser. If you find that a feature does not work as expected then please consult with the browser engine's documentation and [open an issue][issues-new] if you think that the library should support it.

For example, the library does not attempt to support user interaction features like `alert()`, `confirm()` and `prompt()` and other non-essential features like `console.log()`.

### Go Bindings

Calling `Eval()` or `Dispatch()` before `Run()` does not work because the webview instance has only been configured and not yet started.


## Licenses and Copyrights

* Cross-platform webview library: MIT from [webview/webview](https://github.com/webview/webview). Copyright (c) 2017 Serge Zaitsev.
* Simple cross-platform dialog API for go-lang. ISC License from [sqweek/dialog](https://github.com/sqweek/dialog). Copyright (c) 2018, the dialog authors.
* Systray is a cross-platform Go library to place an icon and menu in the notification area, with webview support! Apache License 2.0 from [ghostiam/systray](https://github.com/ghostiam/systray)


## Credits

* https://github.com/xilp/systray
* https://github.com/cratonica/trayhost


## License

Code is distributed under MIT license, feel free to use it in your proprietary projects as well.

[macos-app-bundle]:  https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html
[docs-repo]:         https://github.com/webview/docs
[examples]:          https://github.com/webview/webview/tree/master/examples
[go-docs]:           https://pkg.go.dev/github.com/webview/webview
[gtk]:               https://docs.gtk.org/gtk3/
[issues]:            https://github.com/webview/docs/issues
[issues-new]:        https://github.com/webview/webview/issues/new
[webkit]:            https://webkit.org/
[webkitgtk]:         https://webkitgtk.org/
[webview]:           https://github.com/webview/webview
[webview.dev]:       https://webview.dev
[ms-webview2]:       https://developer.microsoft.com/en-us/microsoft-edge/webview2/
[ms-webview2-sdk]:   https://www.nuget.org/packages/Microsoft.Web.WebView2
[ms-webview2-rt]:    https://developer.microsoft.com/en-us/microsoft-edge/webview2/
[win32-api]:         https://docs.microsoft.com/en-us/windows/win32/apiindex/windows-api-list
[win32-rc]:          https://docs.microsoft.com/en-us/windows/win32/menurc/resource-compiler
