
<img src="Resources/SwiftGodotLogo.svg" width="20"> [![SwiftPM compatible](https://img.shields.io/badge/spm-compatible-brightgreen.svg?style=flat)](https://swift.org/package-manager)
![Platforms](https://img.shields.io/badge/platforms-iOS%20%7C%20Linux%20%7C%20macOS%20%7C%20Windows-333333.svg?style=flat)
[![Swift Package Index](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2Fmigueldeicaza%2FSwiftGodot%2Fbadge%3Ftype%3Dswift-versions)](https://swiftpackageindex.com/migueldeicaza/SwiftGodot)
[![License](https://img.shields.io/badge/license-MIT-lightgrey.svg?maxAge=2592000)](https://raw.githubusercontent.com/migueldeicaza/SwiftGodot/main/LICENSE)

> **Fork note:** This fork contains changes to enable WASM compilation. See [WASM Support](#wasm-support) below.

SwiftGodot provides Swift language bindings for the Godot 4.4 game engine using the [GDExtension](https://docs.godotengine.org/en/stable/tutorials/scripting/gdextension/what_is_gdextension.html) system.

Documentation:
* [Meet Swift Godot](https://migueldeicaza.github.io/SwiftGodotDocs/documentation/swiftgodot)
* [SwiftGodot API Documentation](https://migueldeicaza.github.io/SwiftGodotDocs/documentation/swiftgodot/)
* [Tutorials and walkthroughs](https://migueldeicaza.github.io/SwiftGodotDocs/tutorials/swiftgodot-tutorials/)

# Why SwiftGodot?

* No game stutters caused by GC, unlike C#
* [Swift Godot: Fixing the Multi-million dollar mistake](https://www.youtube.com/watch?v=tzt36EGKEZo)

# Quickly Getting Started

[SwiftGodotKick](https://github.com/EstevanBR/SwiftGodotKick) can create a skeleton GDExtension with Swift.

[SwiftGodotCLI](https://github.com/johnsusek/SwiftGodotCLI) lets you build and run SwiftGodot code without manual project setup.

# Creating an Extension

## Your Swift Code

Create a Swift Library Package:

```swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "MyFirstGame",
    products: [
        .library(name: "MyFirstGame", type: .dynamic, targets: ["MyFirstGame"]),
    ],
    dependencies: [
        .package(url: "https://github.com/migueldeicaza/SwiftGodot", branch: "main")
    ],
    targets: [
        .target(
            name: "MyFirstGame",
            dependencies: ["SwiftGodot"])]
)
```

Create your game logic:

```swift
import SwiftGodot

@Godot(.tool)
class SpinningCube: Node3D {
    public override func _ready () {
        let meshRender = MeshInstance3D()
        meshRender.mesh = BoxMesh()
        addChild(node: meshRender)
    }

    public override func _process(delta: Double) {
        rotateY(angle: delta)
    }
}
```

Register your extension using the macro:

```swift
import SwiftGodot

#initSwiftExtension(cdecl: "swift_entry_point", types: [SpinningCube.self])
```

## Bundling Your Extension

Create a `.gdextension` file:

```ini
[configuration]
entry_symbol = "swift_entry_point"
compatibility_minimum = 4.2

[libraries]
macos.debug = "res://bin/MyFirstGame"
macos.release = "res://bin/MyFirstGame"
linux.debug.x86_64 = "res://bin/MyFirstGame"
linux.release.x86_64 = "res://bin/MyFirstGame"
windows.debug.x86_64 = "res://bin/MyFirstGame"
windows.release.x86_64 = "res://bin/MyFirstGame"
```

## Installing your Extension

Copy the `.gdextension` file and built libraries into your Godot project. Godot will load it automatically.

---

# WASM Support

This fork includes changes to enable SwiftGodot compilation for WebAssembly.

## Changes

1. **WASILibc import** - C library detection for WASI platform
2. **OptionSet Int64** - Fixes overflow on 32-bit WASM
3. **Debug sentinel fix** - `0xdeaddead` → `0` (overflows 32-bit Int)
4. **PropertyUsageFlags fix** - Match Int64 rawValue

## Setup

```bash
# Swift 6.2.3 with WASM SDK
curl -sL https://swiftlang.github.io/swiftly/swiftly-install.sh | bash
swiftly install 6.2.3
swift sdk install swift-6.2.3-RELEASE_wasm

# Emscripten 3.1.74
git clone https://github.com/emscripten-core/emsdk.git ~/emsdk
cd ~/emsdk && ./emsdk install 3.1.74 && ./emsdk activate 3.1.74
```

## Build

```bash
# Compile to WASM
swift build --swift-sdk swift-6.2.3-RELEASE_wasm --target SwiftGodot

# Link as Emscripten side module
emcc -sSIDE_MODULE=2 -sEXPORTED_FUNCTIONS=_swift_entry_point \
    -sALLOW_MEMORY_GROWTH=1 -sERROR_ON_UNDEFINED_SYMBOLS=0 -O2 \
    .build/wasm32-unknown-wasip1/debug/*.build/*.o \
    $SWIFT_WASM_SDK/swift.xctoolchain/usr/lib/swift_static/wasi/libswiftCore.a \
    -o SwiftLogic.wasm
```

## Current Limitation

The binary compiles but **doesn't run** in Godot's web export. Swift WASM SDK targets `wasm32-unknown-wasip1` (WASI), while Godot uses `wasm32-unknown-emscripten`. These have incompatible ABIs. See [issue #447](https://github.com/migueldeicaza/SwiftGodot/issues/447) for details.
