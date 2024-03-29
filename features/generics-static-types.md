---
title: Generics - static types
layout: feature
nav_order: 500
feature-status:
  swift:
    status: supported
    details: https://docs.swift.org/swift-book/LanguageGuide/Generics.html
  rust:
    status: supported
    details: https://doc.rust-lang.org/book/ch10-01-syntax.html
excerpt: >- 
  Generics are types (classes, values, functions) that have type parameters.

---

In the context of this article, we will examine generics as they apply to statically-typed code.  
In other words, while the algorithms are generic (e.g. comparing values), the actual code that uses them has concrete types (e.g. comparing integers).

To see the dynamically typed equivalent, see ["Generics - dynamic types"](/features/generics-dynamic-types).

## Swift

Generics are supported in all recent versions of Swift. To define a generic algorithm:

```swift
// define a generic type 
protocol Costume {
  var hasBells: Bool { get }
}
// define a generic function
func firstWithBells<C, S>(costumes: S) -> C? where C: Costume, S: Sequence, S.Element == C {
    costumes.first(where: { $0.hasBells })
}
```

The use of such an algorithm is bound during compilation to a concrete type at the point of use:

```swift 
// define a concrete type conforming to `Costume`
struct GymnastCostume: Costume {
    let name: String
    let hasBells: Bool
}
// a concrete array of structs `Array<GymnastCostume>`
let gymnastCostumesArray = [
    GymnastCostume(name: "A", hasBells: false),
    GymnastCostume(name: "B", hasBells: true)
]
// a concrete result `Optional<GymnastCostume>`
let firstGymnastCostume = firstWithBells(costumes: gymnastCostumesArray)
print(firstGymnastCostume) // prints "B"
```

### Opaque types

To avoid typing out long generic type signatures, Swift allows using the `some` keyword as a shorthand.

This concept is called an "opaque type": the type is still concrete in this case, and is inferred during compilation from the context.

Using opaque types is possible in several contexts:
 * function result types ([SE-0244] - Opaque Result Types, Swift 5.1)
 * function parameters ([SE-0341] - Opaque Parameter Declarations, Swift 5.7)
 * type parameters within types ([SE-0328] - Structural opaque result types, Swift 5.7)

The same function as above, with opaque types (Swift 5.7):

```swift
func firstWithBells(costumes: some Sequence<some Costume>) -> (some Costume)? {
    costumes.first(where: { $0.hasBells })
}
```

> Note: constraining the associated type `Sequence.Element` with type parameter syntax `Sequence<Element>` is made possible by [SE-0346] (Lightweight same-type requirements for primary associated types, Swift 5.7) and [SE-0358] (Primary Associated Types in the Standard Library, _in review_).

[SE-0244]: https://github.com/apple/swift-evolution/blob/main/proposals/0244-opaque-result-types.md
[SE-0328]: https://github.com/apple/swift-evolution/blob/main/proposals/0328-structural-opaque-result-types.md
[SE-0341]: https://github.com/apple/swift-evolution/blob/main/proposals/0341-opaque-parameters.md
[SE-0346]: https://github.com/apple/swift-evolution/blob/main/proposals/0346-light-weight-same-type-syntax.md
[SE-0358]: https://github.com/apple/swift-evolution/blob/main/proposals/0358-primary-associated-types-in-stdlib.md

## Rust

Generic algorithms are also supported in Rust. 

```rust
// define a generic type 
pub trait Costume {
    fn has_bells(&self) -> bool;
}
// define a generic function
fn first_with_bells<'a, I, C>(iter: &'a mut I) -> Option<&'a C>
where I: Iterator<Item = &'a C>, C: Costume {
    iter.find(|&c| c.has_bells())
}
```

And then a concrete implementation:

```rust
// define a concrete type
#[derive(Debug)]
struct GymnastCostume {
    name: &'static str,
    has_bells: bool,
}
// create a trait impl
impl Costume for GymnastCostume {
    fn has_bells(&self) -> bool {
        self.has_bells
    }
}
// use it
fn main() {
    let gymnast_costumes_vec = vec![
        GymnastCostume { name: "A", has_bells: false },
        GymnastCostume { name: "B", has_bells: true },
    ];
    let mut iter = gymnast_costumes_vec.iter();
    let first_gymnast_costume = first_with_bells(&mut iter);
    println!("{:?}", first_gymnast_costume); // prints "B"
}
```

[Rust example on godbolt.org]: https://rust.godbolt.org/#g:!((g:!((g:!((h:codeEditor,i:(filename:'1',fontScale:14,fontUsePx:'0',j:1,lang:rust,selection:(endColumn:2,endLineNumber:32,positionColumn:2,positionLineNumber:32,selectionStartColumn:1,selectionStartLineNumber:11,startColumn:1,startLineNumber:11),source:'//+define+a+generic+type+%0Apub+trait+Costume+%7B%0A++++fn+has_bells(%26self)+-%3E+bool%3B%0A%7D%0A//+define+a+generic+function%0Afn+first_with_bells%3C!'a,+I,+C%3E(iter:+%26!'a+mut+I)+-%3E+Option%3C%26!'a+C%3E%0Awhere+I:+Iterator%3CItem+%3D+%26!'a+C%3E,+C:+Costume+%7B%0A++++iter.find(%7C%26c%7C+c.has_bells())%0A%7D%0A%0A//+define+a+concrete+type%0A%23%5Bderive(Debug)%5D%0Astruct+GymnastCostume+%7B%0A++++name:+%26!'static+str,%0A++++has_bells:+bool,%0A%7D%0A//+create+a+trait+impl%0Aimpl+Costume+for+GymnastCostume+%7B%0A++++fn+has_bells(%26self)+-%3E+bool+%7B%0A++++++++self.has_bells%0A++++%7D%0A%7D%0A//+use+it%0Afn+main()+%7B%0A++++let+gymnast_costumes_vec+%3D+vec!!%5B%0A++++++++GymnastCostume+%7B+name:+%22A%22,+has_bells:+false+%7D,%0A++++++++GymnastCostume+%7B+name:+%22B%22,+has_bells:+true+%7D,%0A++++%5D%3B%0A++++let+mut+iter+%3D+gymnast_costumes_vec.iter()%3B%0A++++let+first_gymnast_costume+%3D+first_with_bells(%26mut+iter)%3B%0A++++println!!(%22%7B:%3F%7D%22,+first_gymnast_costume)%3B+//+prints+%22B%22%0A%7D%0A'),l:'5',n:'0',o:'Rust+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((h:executor,i:(argsPanelShown:'1',compilationPanelShown:'0',compiler:r1610,compilerOutShown:'0',execArgs:'',execStdin:'',fontScale:14,fontUsePx:'0',j:1,lang:rust,libs:!(),options:'',source:1,stdinPanelShown:'1',tree:'1',wrap:'1'),l:'5',n:'0',o:'Executor+rustc+1.61.0+(Rust,+Editor+%231)',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0')),l:'2',n:'0',o:'',t:'0')),version:4

### 'impl Trait'

Like Swift, Rust has a notion of *unnamed* trait-bound types.

These can be used as anonymous type parameters:
```rust
// accept all concrete types that implement `Trait`
fn foo(arg: impl Trait) {
}
```

And as abstract return types:
```rust
fn foo() -> impl Trait {
  // return some concrete type that implements `Trait`
}
```

