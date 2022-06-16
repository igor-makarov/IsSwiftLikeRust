---
title: Generics - dynamic types
layout: feature
nav_order: 501
feature-status:
  swift:
    status: pending
    # details: https://docs.swift.org/swift-book/LanguageGuide/Generics.html
  rust:
    status: TODO
    # details: https://doc.rust-lang.org/book/ch10-01-syntax.html
excerpt: >- 
  As described in [Generics - static types](/features/generics-static-types),
  generics allow defining shared behavior and generalizing algorithms.
  This is also allowed in dynamic contexts, with type safety guarantees provide by the runtime.

---

The main concern of using generic types dynamically is to ensure the same degree of type safety at runtime as there is during compilation.

This is usually achieved by a pattern called "boxing/unboxing". In this pattern, a type is "boxed" to allow referring to its generic type, and "unboxed" to safely refer to its concrete type.

## Swift

In Swift, boxed types are also called "existential types" and are referred to by their protocol name.

For a long time, Swift had several limitations for existential uses. These limitations would frequently be surprising to developers and have limited the usefulness of the feature. Oftentimes, workarounds such as type erasing structs would be needed, such as `AnyHashable` and `AnyPublisher`. Other times, it would be impossible to write code that is truly generic at all.

Swift 5.7 has lifted most of the limitations, by implementing several big enhancements:

* [SE-0309] - Unlock existentials for all protocols - allow using a protocol as a type without limitations.
* [SE-0353] - Constrained Existential Types - allow constraining a dynamic type using its type parameters.
* [SE-0352] - Implicitly Opened Existentials - allow "unboxing" a dynamic type to a conrete type.
* [SE-0335] - Introduce existential `any` - a new keyword to explicitly refer to a protocol as a type. Introduced in Swift 5.6, will be mandatory for all protocols in Swift 6.
 
The enhancements to `any` types (existential types) work seamlessly with the enhancements to `some` types (opaque types).

To adapt the example from [static types](/features/generics-static-types) (Swift 5.7):

```swift
// define a generic type
protocol Costume {
    associatedtype BellType
    var bells: [BellType] { get }
}
// define a generic function that checks a concrete type
func hasBells(_ costume: some Costume) -> Bool {
    !costume.bells.isEmpty
}
// define a generic function that works with dynamic types inside an array
func firstWithBells(costumes: some Sequence<any Costume>) -> (any Costume)? {
    costumes.first(where: { hasBells($0) })
}
```

> Note: constraining the associated type `Sequence.Element` with type parameter syntax `Sequence<Element>` is made possible by [SE-0346] (Lightweight same-type requirements for primary associated types, Swift 5.7)

This generic function can accept any `Sequence` that can contain different types of `Costume`s:

```swift 
// define two concrete types conforming to `Costume`
struct GymnastCostume: Costume {
    let name: String
    var bells: [Never] { [] }
}
struct ClownCostume: Costume {
    enum ClownBell { case big, small }
    let name: String
    let bells: [ClownBell]
}
// a generic array of costumes
// the explicit type annotation is there so the array is not inferred to be `[Any]`
let costumesArray: [any Costume] = [
    GymnastCostume(name: "A"),
    ClownCostume(name: "B", bells: [.big, .big, .small])
]
// a concrete result `Optional<Costume>`
let firstCostume = firstWithBells(costumes: costumesArray)
print(firstCostume) // prints "B"
```

[SE-0156]: https://github.com/apple/swift-evolution/blob/main/proposals/0156-subclass-existentials.md
[SE-0309]: https://github.com/apple/swift-evolution/blob/main/proposals/0309-unlock-existential-types-for-all-protocols.md
[SE-0335]: https://github.com/apple/swift-evolution/blob/main/proposals/0335-existential-any.md
[SE-0352]: https://github.com/apple/swift-evolution/blob/main/proposals/0352-implicit-open-existentials.md
[SE-0353]: https://github.com/apple/swift-evolution/blob/main/proposals/0353-constrained-existential-types.md
[SE-0346]: https://github.com/apple/swift-evolution/blob/main/proposals/0346-light-weight-same-type-syntax.md

## Rust

TODO