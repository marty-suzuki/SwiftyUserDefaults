# SwiftyUserDefaults

![Platforms](https://img.shields.io/badge/platforms-ios%20%7C%20osx%20%7C%20watchos%20%7C%20tvos-lightgrey.svg)
[![CI Status](https://api.travis-ci.org/radex/SwiftyUserDefaults.svg?branch=master)](https://travis-ci.org/radex/SwiftyUserDefaults)
[![CocoaPods compatible](https://img.shields.io/badge/CocoaPods-compatible-4BC51D.svg?style=flat)](#cocoapods)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](#carthage)
[![SPM compatible](https://img.shields.io/badge/SPM-compatible-4BC51D.svg?style=flat)](#swift-package-manager)
![Swift version](https://img.shields.io/badge/swift-4.1-orange.svg)
![Swift version](https://img.shields.io/badge/swift-4.2-orange.svg)
![Swift version](https://img.shields.io/badge/swift-5.0-orange.svg)

#### Modern Swift API for `NSUserDefaults`
###### SwiftyUserDefaults makes user defaults enjoyable to use by combining expressive Swifty API with the benefits of static typing. Define your keys in one place, use value types easily, and get extra safety and convenient compile-time checks for free.

Read [documentation for stable version 3.0.1](https://github.com/radex/SwiftyUserDefaults/blob/14b629b035bf6355b46ece22c3851068a488a895/README.md)<br />
Read [migration guide from version 3.x to 4.x](MigrationGuides/migration_3_to_4.md)<br />
Read [migration guide from version 4.0.0-alpha.1 to 4.0.0-alpha.3](MigrationGuides/migration_4_alpha_1_to_4_alpha_2.md)

# Version 4

<p align="center">
    <a href="#features">Features</a> &bull;
    <a href="#usage">Usage</a> &bull;
    <a href="#codable">Codable</a> &bull;
    <a href="#nscoding">NSCoding</a> &bull;
    <a href="#rawrepresentable">RawRepresentable</a> &bull;
    <a href="#extending-existing-types">Extending existing types</a> &bull;
    <a href="#custom-types">Custom types</a> &bull;
    <a href="#kvo">KVO</a> &bull;
    <a href="#launch-arguments">Launch arguments</a> &bull;
    <a href="#utils">Utils</a> &bull;
    <a href="#installation">Installation</a>
</p>

## Features

**There's only one step to start using SwiftyUserDefaults:**

Define your keys!

```swift
extension DefaultsKeys {
    static let username = DefaultsKey<String?>("username")
    static let launchCount = DefaultsKey<Int>("launchCount", defaultValue: 0)
}
```

And just use it ;-)

```swift
// Get and set user defaults easily
let username = Defaults[.username]
Defaults[.hotkeyEnabled] = true

// Modify value types in place
Defaults[.launchCount] += 1
Defaults[.volume] -= 0.1
Defaults[.strings] += "… can easily be extended!"

// Use and modify typed arrays
Defaults[.libraries].append("SwiftyUserDefaults")
Defaults[.libraries][0] += " 2.0"

// Easily work with custom serialized types
Defaults[.color] = NSColor.white
Defaults[.color]?.whiteComponent // => 1.0
```

The convenient dot syntax is only available if you define your keys by extending magic `DefaultsKeys` class. You can also just pass the `DefaultsKey` value in square brackets.

## Usage

### Define your keys

To get the most out of SwiftyUserDefaults, define your user defaults keys ahead of time:

```swift
let colorKey = DefaultsKey<String>("color", defaultValue: "")
```

Just create a `DefaultsKey` object, put the type of the value you want to store in angle brackets, the key name in parentheses, and you're good to go. If you want to have a non-optional value, just provide a `defaultValue` in the key (look at the example above).

You can now use the `Defaults` shortcut to access those values:

```swift
Defaults[colorKey] = "red"
Defaults[colorKey] // => "red", typed as String
```

The compiler won't let you set a wrong value type, and fetching conveniently returns `String`.

### Take shortcuts

For extra convenience, define your keys by extending magic `DefaultsKeys` class and adding static properties:

```swift
extension DefaultsKeys {
    static let username = DefaultsKey<String?>("username")
    static let launchCount = DefaultsKey<Int>("launchCount", defaultValue: 0)
}
```

And use the shortcut dot syntax:

```swift
Defaults[.username] = "joe"
Defaults[.launchCount] += 1
```

### Supported types

SwiftyUserDefaults supports all of the standard `NSUserDefaults` types, like strings, numbers, booleans, arrays and dictionaries.

Here's a full table of built-in single value defaults:

| Single value     | Array                |
| ---------------- | -------------------- |
| `String`         | `[String]`           |
| `Int`            | `[Int]`              |
| `Double`         | `[Double]`           |
| `Bool`           | `[Bool]`             |
| `Data`           | `[Data]`             |
| `Date`           | `[Date]`             |
| `URL`            | `[URL]`              |
| `[String: Any]`  | `[[String: Any]]`    |

But that's not all!

## Codable

Since version 4, `SwiftyUserDefaults` support `Codable`! Just conform to `DefaultsSerializable` in your type:
```swift
final class FrogCodable: Codable, DefaultsSerializable {
    let name: String
 }
```

No implementation needed! By doing this you will get an option to specify an optional `DefaultsKey`:
```swift
let frog = DefaultsKey<FrogCodable?>("frog")
```

Additionally, you've got an array support for free:
```swift
let froggies = DefaultsKey<[FrogCodable]?>("froggies")
```

## NSCoding

`NSCoding` was supported before version 4, but in this version we take the support on another level. No need for custom subscripts anymore!
Support your custom `NSCoding` type the same way as with `Codable` support:
```
final class FrogSerializable: NSObject, NSCoding, DefaultsSerializable { ... }
```

No implementation needed as well! By doing this you will get an option to specify an optional `DefaultsKey`:
```swift
let frog = DefaultsKey<FrogSerializable?>("frog")
```

Additionally, you've got an array support also for free:
```swift
let froggies = DefaultsKey<[FrogSerializable]?>("froggies")
```

## RawRepresentable

And the last but not least, `RawRepresentable` support! Again, the same situation like with `NSCoding` and `Codable`:
```swift
enum BestFroggiesEnum: String, DefaultsSerializable {
    case Andy
    case Dandy
}
```

No implementation needed as well! By doing this you will get an option to specify an optional `DefaultsKey`:
```swift
let frog = DefaultsKey<BestFroggiesEnum?>("frog")
```

Additionally, you've got an array support also for free:
```swift
let froggies = DefaultsKey<[BestFroggiesEnum]?>("froggies")
```

## Extending existing types

Let's say you want to extend a support `UIColor` or any other type that is `NSCoding`, `Codable` or `RawRepresentable`.
Extending it to be `SwiftyUserDefaults`-friendly should be as easy as:
```swift
extension UIColor: DefaultsSerializable {}
```

If it's not, we have two options:<br />
a) It's a custom type that we don't know how to serialize, in this case at [Custom types](#custom-types)<br />
b) It's a bug and it should be supported, in this case please file an issue (+ you can use [custom types](#custom-types) method as a workaround in the meantime)<br />

## Custom types

If you want to add your own custom type that we don't support yet, we've got you covered. We use `DefaultsBridge`s of many kinds to specify how you get/set values and arrays of values. When you look at `DefaultsSerializable` protocol, it expects two properties in each type: `_defaults` and `_defaultsArray`, where both are of type `DefaultsBridge`.

For instance, this is a bridge for single value data storing/retrieving using `NSKeyedArchiver`/`NSKeyedUnarchiver`:
```swift
public final class DefaultsKeyedArchiverBridge<T>: DefaultsBridge<T> {

    public override func get(key: String, userDefaults: UserDefaults) -> T? {
        return userDefaults.data(forKey: key).flatMap(NSKeyedUnarchiver.unarchiveObject) as? T
    }

    public override func save(key: String, value: T?, userDefaults: UserDefaults) {
        userDefaults.set(NSKeyedArchiver.archivedData(withRootObject: value), forKey: key)
    }
}
```

Bridge for default storing/retrieving array values:
```swift
public final class DefaultsArrayBridge<T: Collection>: DefaultsBridge<T> {
    public override func save(key: String, value: T?, userDefaults: UserDefaults) {
        userDefaults.set(value, forKey: key)
    }

    public override func get(key: String, userDefaults: UserDefaults) -> T? {
        return userDefaults.array(forKey: key) as? T
    }
}
```

Now, to use these bridges in our type we simply declare it as follows:
```swift
struct FrogCustomSerializable: DefaultsSerializable {

    static var _defaults: DefaultsBridge<FrogCustomSerializable> { return DefaultsKeyedArchiverBridge() }
    static var _defaultsArray: DefaultsBridge<[FrogCustomSerializable]> { return DefaultsKeyedArchiverBridge() }

    let name: String
}
```

Unfortunately, if you find yourself in a situation where you need a custom bridge, you'll probably need to write your own:
```swift
final class DefaultsFrogBridge: DefaultsBridge<FrogCustomSerializable> {
    override func get(key: String, userDefaults: UserDefaults) -> FrogCustomSerializable? {
        let name = userDefaults.string(forKey: key)
        return name.map(FrogCustomSerializable.init)
    }

    override func save(key: String, value: FrogCustomSerializable?, userDefaults: UserDefaults) {
        userDefaults.set(value?.name, forKey: key)
    }

    public override func isSerialized() -> Bool {
        return true
    }

    public override func deserialize(_ object: Any) -> FrogCustomSerializable? {
        guard let name = object as? String else { return nil }

        return FrogCustomSerializable(name: name)
    }
}

final class DefaultsFrogArrayBridge: DefaultsBridge<[FrogCustomSerializable]> {
    override func get(key: String, userDefaults: UserDefaults) -> [FrogCustomSerializable]? {
        return userDefaults.array(forKey: key)?
            .compactMap { $0 as? String }
            .map(FrogCustomSerializable.init)
    }

    override func save(key: String, value: [FrogCustomSerializable]?, userDefaults: UserDefaults) {
        let values = value?.map { $0.name }
        userDefaults.set(values, forKey: key)
    }

    public override func isSerialized() -> Bool {
        return true
    }

    public override func deserialize(_ object: Any) -> [FrogCustomSerializable]? {
        guard let names = object as? [String] else { return nil }

        return names.map(FrogCustomSerializable.init)
    }
}

struct FrogCustomSerializable: DefaultsSerializable, Equatable {

    static var _defaults: DefaultsBridge<FrogCustomSerializable> { return DefaultsFrogBridge() }
    static var _defaultsArray: DefaultsBridge<[FrogCustomSerializable]> { return DefaultsFrogArrayBridge() }

    let name: String
}
```

To support existing types with different bridges, you can extend it similarly:
```swift
extension Data: DefaultsSerializable {
    public static var _defaults: DefaultsBridge<Data> { return DefaultsDataBridge() }
    public static var _defaultsArray: DefaultsBridge<[Data]> { return DefaultsArrayBridge() }
}
```

Also, take a look at our source code (or tests) to see more examples of bridges. If you find yourself confused with all these bridges, please [create an issue](https://github.com/radex/SwiftyUserDefaults/issues/new) and we will figure something out.

## KVO

KVO is supported for all the types that are `DefaultsSerializable`. However, if you have a custom type, it needs to have correctly defined bridges and serialization in them.

To observe a value:
```swift
let nameKey = DefaultsKey<String>("name", defaultValue: "")
Defaults.observe(key: nameKey) { update in
	// here you can access `oldValue`/`newValue` and few other properties
}
```

By default we are using `[.old, .new]` options for observing, but you can provide your own:
```swift
Defaults.observe(key: nameKey, options: [.initial, .old, .new]) { _ in }
```

## Launch arguments

Do you like to customize your app/script/tests by UserDefaults? Now it's fully supported on our side, statically typed of course.

_Note: for now we support only `Bool`, `Double`, `Int`, `String` values, but if you have any other requests for that feature, please open an issue or PR and we can talk about implementing it in new versions._

### You can pass your arguments in your schema:
<img src="https://i.imgur.com/SDpOBpK.png" alt="Pass launch arguments in Xcode Schema editor." />

### Or you can use launch arguments in XCUIApplication:
```swift
func testExample() {
    let app = XCUIApplication()
    app.launchArguments = ["-skipLogin", "true", "-loginTries", "3", "-lastGameTime", "61.3", "-nickname", "sunshinejr"]
    app.launch()
}
```
### Or pass them as command line arguments!
```bash
./script -skipLogin true -loginTries 3 -lastGameTime 61.3 -nickname sunshinejr
```

## Utils

### Remove all keys

To reset user defaults, use `removeAll` method.

```swift
Defaults.removeAll()
```

### Shared user defaults

If you're sharing your user defaults between different apps or an app and its extensions, you can use SwiftyUserDefaults by overriding the `Defaults` shortcut with your own. Just add in your app:

```swift
var Defaults = UserDefaults(suiteName: "com.my.app")!
```

### Check key

If you want to check if we've got a value for `DefaultsKey`:
```swift
let hasKey = Defaults.hasKey(.skipLogin)
```

## KeyPath dynamicMemberLookup

SwiftyUserDefaults makes KeyPath dynamicMemberLookup usable in Swift 5.1!

```swift
extension DefaultsKeyStore {
    var username: DefaultsKey<String?> { return .init("username") }
    var launchCount: DefaultsKey<Int> { return .init("launchCount", defaultValue: 0) }
}
```

And just use it ;-)

```swift
// Get and set user defaults easily
let username = Defaults.username
Defaults.hotkeyEnabled = true

// Modify value types in place
Defaults.launchCount += 1
Defaults.volume -= 0.1
Defaults.strings += "… can easily be extended!"

// Use and modify typed arrays
Defaults.libraries.append("SwiftyUserDefaults")
Defaults.libraries[0] += " 2.0"

// Easily work with custom serialized types
Defaults.color = NSColor.white
Defaults.color?.whiteComponent // => 1.0
```

## Installation

### Requirements
**Swift** version **>= 4.1**<br />
**iOS** version **>= 8.0**<br />
**macOS** version **>= 10.11**<br />
**tvOS** version **>= 9.0**<br />
**watchOS** version **>= 2.0**

### CocoaPods

If you're using CocoaPods, just add this line to your Podfile:

```ruby
pod 'SwiftyUserDefaults', '~> 4.0'
```

Install by running this command in your terminal:

```sh
pod install
```

Then import the library in all files where you use it:

```swift
import SwiftyUserDefaults
```

### Carthage

Just add to your Cartfile:

```ruby
github "radex/SwiftyUserDefaults" ~> 4.0
```

### Swift Package Manager

Just add to your `Package.swift` under dependencies:
```swift
let package = Package(
    name: "MyPackage",
    products: [...],
    dependencies: [
        .package(url: "https://github.com/radex/SwiftyUserDefaults.git", .upToNextMajor(from: "4.0.0"),
    ],
    targets: [...]
)
```

## More like this

If you like SwiftyUserDefaults, check out [SwiftyTimer](https://github.com/radex/SwiftyTimer), which applies the same swifty approach to `NSTimer`.

You might also be interested in my blog posts which explain the design process behind those libraries:
- [Swifty APIs: NSUserDefaults](http://radex.io/swift/nsuserdefaults/)
- [Statically-typed NSUserDefaults](http://radex.io/swift/nsuserdefaults/static)
- [Swifty APIs: NSTimer](http://radex.io/swift/nstimer/)
- [Swifty methods](http://radex.io/swift/methods/)

## Contributing

If you have comments, complaints or ideas for improvements, feel free to open an issue or a pull request.

## Authors and license

*Maintainer:* Łukasz Mróz
* [github.com/sunshinejr](http://github.com/sunshinejr)
* [twitter.com/thesunshinejr](http://twitter.com/thesunshinejr)
* [sunshinejr.com](https://sunshinejr.com)

*Created by:* Radek Pietruszewski

* [github.com/radex](http://github.com/radex)
* [twitter.com/radexp](http://twitter.com/radexp)
* [radex.io](http://radex.io)
* this.is@radex.io

SwiftyUserDefaults is available under the MIT license. See the LICENSE file for more info.
