# Exports

In Godot, class members can be exported. This means their value gets saved along
with the resource (such as the scene) they're attached to. They will also be
available for editing in the property editor. Exporting is done by using the
`@Export` annotation.

This document deals with exporting properties, for information about exposing
functions to the Godot world, see the <doc:CustomTypes> document.

## Introduction to Exports

The simplest way of exporting a variable is to annotate it with the `@Export`
attribute, like this:

```swift
import SwiftGodot

@Godot
public class ExportExample: Node3D
{
    @Export
    var number = 5
}
```

In that example the value 5 will be saved, and after building the current
project it will be visible in the property editor.

One of the fundamental benefits of exporting member variables is to have them
visible and editable in the editor. This way, artists and game designers can
modify values that later influence how the program runs. For this, a special export syntax is provided.

`@Export` can be applied to non-static, non-class variables with following types:
1. ``VariantConvertible`` types and their ``Optional``s such as: 
1.1. Swift primitive types (``Bool``, ``String``, ``Int``, ``Double``, ``Float``, any width signed and unsigned version ``BinaryInteger`` (such as ``UInt32``, ``Int64``).
1.2. All builtin Godot types present in ``SwiftGodot``, such as ``VariantArray``, ``Vector3``, ``Callable``, ``Signal``
1.3. All ``Object``-derived types including your own types.
1.4. Your custom ``VariantConvertible`` types.
2. Any Swift closures taking `VariantConvertible` arguments and returning `Void` or `VariantConvertible` value without `async` specifier.
3. Any ``CaseIterable`` enum having integer `RawValue`

The `@Export` macro only works in your `@Godot` class declaration, and will not work on Swift class extensions.

```swift
// Correct
@Godot
class YourClass {
    @Export var int = 42
}

// Incorrect
extension YourClass {
    @Export var anotherInt = 48
}
```

### Basic Usage

```swift
@Export
var number: Int

@Export
var anotherNumber { get { ... } set { ... } }
```

Exported members can specify a default value:

```
@Export
var number: Int = 0

@Export
var text: String? = nil        // Allows for nil

@Export
var greeting = "Hello World"   // Exported field specifies a default value
```

Resources and nodes can be exported.

```
@Export
public var resource: Resource { 
    get { 
        return myInternalResource
    } 
    set { 
        print ("Setting my resource")
    } 
}

@Export
public var node: Node { 
    get { 
        return myInternalNode
    } 
    set { 
        print ("Setting the node")
    } 
}
```

### Grouping Exports

It is possible to group your exported properties inside the Godot Inspector with
the #exportGroup macro. Every exported property after this annotation will be
added to the group.  Start a new group or use #export_group ("") to break out.

```swift
#exportGroup("My Properties")
@Export var number = 3
```

You can also specifiy that only properties with a given prefix be grouped, like
this:

```swift
#exportGroup("My Properties", prefix: "health")
@Export var health_reload_speed = 3
```

Groups cannot be nested, use #exportSubgroup to create subgroups within a group.

```swift
#exportSubgroup("Extra Properties")
#export var string = ""
#export var flag = false
```


### Customizing the Exported Value

You can pass a ``PropertyHint`` parameter to the Export attribute, along with additional 
data to control how the property is surfaced in the editor.

For example, to surface the property as a file and trigger the file selector in the 
UI, use the `.file` value:

```swift
@Export(.file)
var GameFile: String? 
```

String as a path to a directory.

```swift
@Export(.dir)
var gameDirectory: String?
```

String as a path to a file, custom filter provided as hint.

```swift
@Export (.file, "*.txt")
var GameFile: String?
```

Using paths in the global filesystem is also possible, but only in scripts in tool mode.

String as a path to a PNG file in the global filesystem.

```swift
@Export (.globalFile, "*.png")
var toolImage: String?
```

String as a path to a directory in the global filesystem.

```swift
@Export (.globalDir)
var toolDir: String?
```

The multiline annotation tells the editor to show a large input field for editing over multiple lines.

```swift
@Export (.multilineText)
var text: String?
```

### Limiting editor input ranges

Using the range property hint allows you to limit what can be input as a value using the editor.

Allow integer values from 0 to 20.

```swift
@Export(.range, "0,20,")
var number: Int = 0
```

Allow integer values from -10 to 20.

```swift
@Export(.range, "-10,20,")
var number: Int =  0
```


Allow floats from -10 to 20 and snap the value to multiples of 0.2.

```swift
@Export(.range, "-10,20,0.2")
var number = 0
```

If you add the hints `or_greater` and/or `or_less` you can go above or below the limits when 
adjusting the value by typing it instead of using the slider.

```swift
@Export(.range, "0,100,1,or_greater,or_less")
var number: Int = 0
```

### Floats with easing hint

Display a visual representation of the `ease()` function when editing.

```swift
@Export(.expEasing)
public transitionSpeed: Float = 0
```

### Colors

Regular color given as red-green-blue-alpha value.

```swift
@Export
var color: Color { get {} set {} }
```

Color given as red-green-blue value (alpha will always be 1).

```swift
@Export(.colorNoAlpha)
var color: Color { get {} set {} }
```

### Nodes

Since Godot 4.0, nodes can be directly exported without having to use NodePaths.

```swift
@Export
public Node node { get; set; }
```

Custom node classes can also be used, see C# global classes.

Exporting NodePaths like in Godot 3.x is still possible, in case you need it:

```swift
@Export
var nodePath: NodePath

public override func _ready() 
{
    var node = GetNode(nodePath)
}
```

Subclasses of `Node` type will have their `.nodeType` automatically filled, so that Editor will allow
only ``Camera3D`` subtypes to be set to the following exported property:

```swift
@Export // no need to write @Export(.nodeType, "Camera3D")
var camera: Camera3D? = nil
```

### Resources

```
@Export
var resource: Resource { get {} set {} }
```

In the Inspector, you can then drag and drop a resource file from the FileSystem dock into the variable slot.

Opening the inspector dropdown may result in an extremely long list of possible classes to create, however. 
Therefore, if you specify a type derived from Resource such as:

```
@Export
var resource: AnimationNode
```

The drop-down menu will be limited to ``AnimationNode`` and all its inherited classes. 
Custom resource classes can also be used, see Swift global classes.

It must be noted that even if the script is not being run while in the editor, the exported 
properties are still editable. This can be used in conjunction with a script in "tool" mode.

### Arrays

To surface arrays in Godot, use a strong type for it, for example:

```swift
@Export
var myResources: TypedArray<Vector3>
```

Alternatively, if you want to surface an array of Godot objects, or even your own subclasses of those, use `TypedArray<YourCustomType?>`, for example:

```swift
@Export
var myNodes: TypedArray<MySpinnerCube?>
```

### ⚠️ Note ⚠️

`Object` derived types require optional `Element` type.
That's due to Godot absence of guarantee that that `Array[Object]` doesn't contain `nil`s.

### Enumeration Values

To surface enumeration values, you need to use a ``CaseIterable`` enum with `RawValue` conforming to ``BinaryInteger``

```
enum MyEnum: Int, CaseIterable {
    case first
    case second
}

@Godot
class Sample: Node {
    @Export // no need to use @Export(.enum), it will be filled automatically
    var myValue: MyEnum
}
```

