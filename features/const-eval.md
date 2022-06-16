---

title: Constant evaluation
layout: feature
nav_order: 900
feature-status:
  swift:
    status: pending
    details-caption: SE-0359
    details: https://github.com/apple/swift-evolution/blob/main/proposals/0359-build-time-constant-values.md
  rust:
    status: supported
    details: https://doc.rust-lang.org/reference/const_eval.html
excerpt: >- 
  Constant evaluation allows the compiler to calculate the value of constants during compilation.

---

This provides the compiler with information that can be used for additional type safety and optimization.

## Swift
In Swift, constant evaluation support is in its early stages. [SE-0359] aims to formalize the use of `@const` attribute on declarations to signify that they can be determined during compilation. The compiler verifies that this is the case. The `@const` attribute can be applied to:
 * immutable variable declarations
 * function parameters
 * static protocol property requirements

> Note: [SE-0359] does not provide any formalization of how user-defined code is evaluated during compilation. It is assumed that this will be addressed in a subsequent proposal.

## Rust
In Rust, constant evaluation ("const-eval") utilizes the interpreter (`miri`) to run arbitrary user code (including [const functions][const fn]) and evaluate it during compilation. Const-eval is possible in the folowing contexts:
 * constants
 * statics
 * enum discriminants
 * fixed-size arrays
 * generic type arguments



[SE-0359]: https://github.com/apple/swift-evolution/blob/main/proposals/0359-build-time-constant-values.md
[const fn]: https://doc.rust-lang.org/reference/const_eval.html#const-functions