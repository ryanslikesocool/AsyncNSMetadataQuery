#  Future

Notes and ideas about the future of this package.

---


## Attribute Key Declaration Boilerplate Reduction

### The Present

Declaring attribute keys currently requires a lot of boilerplate.
```swift
extension NSMetadataAttribute {
	enum DisplayNameKey: NSMetadataAttributeKeyProtocol {
		public typealias Value = String

		public static var attributeKey: String {
			NSMetadataItemDisplayNameKey
		}
	}
}

// MARK: - Convenience

// This extension is technically optional,
// but always declared for attribute keys provided by the package.
extension NSMetadataAttributeProtocol where
	Self == NSMetadataAttributeKey<
		NSMetadataAttribute.DisplayNameKey
	>
{ 
	static var displayName: Self { 
		Self()
	}
}
```


### The (Potential) Future

Ideally, this could be simplified with a single `@AttributeKey`<sup>\*</sup> macro,
similar to SwiftUI's
[`@Entry`](https://developer.apple.com/documentation/swiftui/entry())
macro.
<br/>
<sup>\* name not final</sup>
```swift
// ----- Before Macro Expansion -----

extension NSMetadataAttribute {
	@AttributeKey(named: NSMetadataItemDisplayNameKey, ofType: String.self)
	enum DisplayNameKey { }
}
```
```swift
// ----- After Macro Expansion -----

extension NSMetadataAttribute {
	// A `MemberMacro` implementation could validate
	// that the object is an `enum`
	// and doesn't contain any members.
	enum DisplayNameKey { }
}

// The standard boilerplate could be generated
// from an `ExtensionMacro` implementation.
extension NSMetadataAttribute.DisplayNameKey: NSMetadataAttributeKeyProtocol {
	public typealias Value = String

	public static var attributeKey: String {
		NSMetadataItemDisplayNameKey
	}
}

// The shorthand attribute key accessor might be an issue
// if I'm not able to extend `NSMetadataAttributeProtocol`.
// This could cause issues if the attribute key type name
// doesn't follow a standard convention.
extension NSMetadataAttributeProtocol where
	Self == NSMetadataAttributeKey<
		NSMetadataAttribute.DisplayNameKey
	>
{ 
	static var displayName: Self {
		Self()
	}
}
```
However, to use macros,
[Swift Syntax](https://github.com/swiftlang/swift-syntax)
needs to be added as a dependency.
<br/>
Any consumers would have to depend on
and compile Swift Syntax before using this package.
It can substantially increase compile time
and create issues if another package requires
a different version of Swift Syntax.
<br/>
This is a
<ins>[well](https://forums.swift.org/t/compilation-extremely-slow-since-macros-adoption/67921)</ins>
<ins>[documented](https://forums.swift.org/t/macro-adoption-concerns-around-swiftsyntax/66588)</ins>
headache for other developers.

In the meantime, I could consider using
[Sourcery](https://github.com/krzysztofzablocki/Sourcery)
to do essentially the same thing.
Package consumers would still have to write boilerplate.

All that said, there's a finite number of
[pre-defined attribute keys](https://developer.apple.com/documentation/foundation/nsmetadataitem#1681152),
and it's not like I'm extending a quickly-evolving framework.