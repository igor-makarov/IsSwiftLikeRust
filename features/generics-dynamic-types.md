---
title: Generics - dynamic types
layout: feature
nav_order: 501
feature-status:
  swift:
    status: pending
  rust:
    status: supported
excerpt: >- 
  Generics can refer to dynamic types with constraints, with type safety guarantees provided by the runtime.

---

As described in ["Generics - static types"](/features/generics-static-types), we can define generic algorithms that can operate on concrete types. This page explores the considerations of using generalized types without knowing the underlying concrete types.

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

> Note: constraining the associated type `Sequence.Element` with type parameter syntax `Sequence<Element>` is made possible by [SE-0346] (Lightweight same-type requirements for primary associated types, Swift 5.7) and [SE-0358] (Primary Associated Types in the Standard Library, _in review_).

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
[SE-0358]: https://github.com/apple/swift-evolution/blob/main/proposals/0358-primary-associated-types-in-stdlib.md

## Rust

Rust also allows using generalized types by their interfaces. The term for this is "trait object" and it's denoted using the `dyn` keyword.

Trait objects **must** be references, an as such are usually declared as:
* `&dyn Trait`
* `Box<dyn Trait>`

To implement a generalized function that finds the first costume with bells:

```rust
// define a generic type
pub trait Costume {
    fn description(&self) -> String;
    fn has_bells(&self) -> bool;
}
// define a generic dynamic function
fn first_with_bells<'a, 'b, I>(iter: &'a mut I) -> Option<Box<dyn Costume + 'b>>
where I: Iterator<Item = Box<dyn Costume + 'b>> {
    iter.find(|c| c.has_bells())
}
```

Use it with two distinct types: 


```rust
// first concrete type
struct GymnastCostume {
    name: &'static str,
    has_bells: bool,
}
impl Costume for GymnastCostume {
    fn description(&self) -> String {
        let name = self.name;
        format!("Gymnast {name}").to_string()
    }
    fn has_bells(&self) -> bool {
        self.has_bells
    }
}
// second concrete type
struct ClownCostume {
    name: &'static str,
}
impl Costume for ClownCostume {
    fn description(&self) -> String {
        let name = self.name;
        format!("Clown {name}").to_string()
    }
    fn has_bells(&self) -> bool {
        true
    }
}
// use it
fn main() {
    // heterogenous vector
    let costumes_vec: Vec<Box<dyn Costume>> = vec![
        Box::new(GymnastCostume { name: "A", has_bells: false }),
        Box::new(ClownCostume { name: "B" }),
    ];
    let mut iter = costumes_vec.into_iter();
    let first_costume = first_with_bells(&mut iter);
    println!("{:?}", first_costume.unwrap().description()); // prints "Clown B"
}
```

[Rust example on godbolt.org]: https://rust.godbolt.org/#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAMzwBtMA7AQwFtMQByARg9KtQYEAysib0QXACx8BBAKoBnTAAUAHpwAMvAFYTStJg1DEArgoKkl9ZATwDKjdAGFUtEywYgATKUcAZPAZMADl3ACNMYhAAVg1SAAdUBUI7Bhc3D28EpJSBAKDQlgio2MtMa1sBIQImYgJ0908fK0wbVOragnyQ8MiYuPNOhszmmrruwuL%2BgEpLVBNiZHYOAHoVgGosGiD1pnXgRki8ZHWCAE94zABSDQBBeJMw0%2BImQnWXc3dMdauAdgAhG63dYg9ZUBibTAKZDEPDxSoMCBXLwANisVGm6wAtFcAMzYdbVWFGPGAu6gsEQhBMBQAfQitFoCiRqPRmJx%2BPWYVQrlJQL%2BABEgWtIdtvnsDkFYSd0GdmCxjmCTAw2gIgeCwXhiOZaQB3QgIenlJl4pxgDhMUjrc1hK0ASTx2AghD6P1R5r2LBMBHWdvZjvWAHl4alTf9UKpTbKIR8CF83f9rRwwo7U3ddQhIt87SBfQRIkwiMRTXb8ywfriBetw5HcU5o%2B8knG2Amkyn8QG/mTgaCXcQAHTbdBI35OZB/JzrZD96l0hlMiDTab835Cu78u4imjan1oFXETD504Xa53cymGzrADiZw8NPqTfjXaBFPl7DdKPNg1sJ3PpBfoKzkajIKLm3KuP%2B66rkCeAsPEtCNp8Lb8MQ163sw5ixk%2BAIASCGpYNCsIhgILJouUGLYgGRKBMAPw4eSFKgvQPpvhWVbov2b58gxjFgiQLCFmAkDIl4N53uYdH/Fx0FeF40z9kQtLnjRi64XRa49nhVI0sBC7ImRtAURyBLgQhz48YxHFAfOChqYKK4aSKSh7ugU4CDCh7fOclxAueJiXk4tCoLqaSPi25maesb65vpX41D%2B6x/g5MFwQhWHISQ7xBSF6XfBFFL4VCMJwgipFspRnLUUYklqRSzFRaweWVol5GcY13GRQV/GCcJsmBcFEJdtJQqyfJinKUYqkWfZFkatZxrMvp5XGVyPJmfRnWggQpinpFM23PtIpmN8hDqhCAmBIuNUWSKmb5sQqCSvMCjrAAbq0RZqfVaBIVCtLvcguYAGqtGGEZRnKiHNtcHacniVYA0JVzRN2vEgjWIAgEEuoQGJGEPr9kkNWwMWyQdslWvNIG5lQYhKOpsy1aCGNY5gOP9TlYV5QCxPviJgKyQzkF7dEQq4qjTGHusXo%2Bn2bFub9dIA/2gSKX2qni19UvbjqP3Q/LOsEHqBq6YtqIy%2BsfbLprFnxMSBC0AwQksl4XYgHiABi9kU5qO60nrXz9squovPEi79gRxXEYiS6kusIp26rL0iRzEIC67UEChwsy0Jw0S8J4HBaKQqCcAASmYPoKPMix5V4uI8KQBCaNnswANb9PonCSAXLcl5wvCgXEzdF9npBwLASCYKorTeiQ5CULUwAKMohjlEICADZwjdoHBdCFqkq9BLQG9b6PpC7/EdB9MQXAolwcSX9fxCBt6p8hX30%2BtLcxDLwPvgz2QNUfAhdeD8EECIMQ7ApAyEEIoFQ6hz66C4PoQwxhK76DwGEUCkBZioGjqBDgg8a5LD0OYEBR916bw/twXg21MDLEbiHJg8Rt5j1zhwfOpBC7F1LhwbAgC56oVMOYE4XB%2Bz337BodYEAK7mCtLgQgmVkQN2mHQluS5SAd1KBwnu3C%2B58MHiAYeGic6cC8L3c%2Bhim6mNIO9bUqQQCSCAA%3D%3D
