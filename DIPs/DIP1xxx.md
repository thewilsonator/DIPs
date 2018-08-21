# extern(C++) array

| Field           | Value                                                           |
|-----------------|-----------------------------------------------------------------|
| DIP:            | (number/id -- assigned by DIP Manager)                          |
| Review Count:   | 0                                                               |
| Author:         | Nicholas Wilson, Jacob Carlborg                                 |
| Implementation: | https://github.com/dlang/dmd/pull/8120                          |
| Status:         | Will be set by the DIP manager (e.g. "Approved" or "Rejected")  |

## Abstract

`T[]` is extended to `extern(C++)` with mangling and ABI defined as if it were
```D
extern(C++) 
struct __dslice(T)
{
    size_t length;
    T*     ptr;
}
```

but not POD. This is useful for making D code eaiser to expose to C++.

### Reference

[Example C++ header for __dslice](https://github.com/thewilsonator/interop/blob/master/c%2B%2B/dslice.h)

[gsl::span](https://github.com/isocpp/CppCoreGuidelines/blob/master/docs/gsl-intro.md#gslspan-what-is-gslspan-and-what-is-it-for) ([implementation](https://github.com/Microsoft/GSL/blob/master/include/gsl/span)

(`gsl::span` is the library pre-release of `std::span` and is identical exect for name).

## Contents
* [Rationale](#rationale)
* [Description](#description)
* [Breaking Changes and Deprecations](#breaking-changes-and-deprecations)
* [Copyright & License](#copyright--license)
* [Reviews](#reviews)

## Rationale

There is currently no type equvalent for D's `T[]` for extern(C++).
This has lead to a number of different non-ideal workarounds such as defining the above maually 
(e.g. [LDC](https://github.com/ldc-developers/ldc/blob/438ff5873e1aa768333a41dcd6d682c15ad088e7/gen/ldctraits.d),
replete with their own [bugs](https://github.com/ldc-developers/ldc/commit/8db147a03600a7c976d2cc021f4f5f88c4780f65)),
passing the pointer and length sepately (which is unsafe and prone to error) and
using an `extern(C)` style interface (which is just as unsafe and for strings usually involves redunant calls to `strlen`).

"But I want a `std::vector` interface!"

Due to C++'s implicit conversions the `__dslice` in the example header accepts all the C++ random access containters.

```C++
// Defined in D as extern(C++) void foo(const(char)[] a) { ... }
void foo(__dslice<const char> a);

void bar()
{
    std::vector<char> vc = ...;
    std::string s = "blah"s;
    std::string_view sv = "blah"sv;
    foo(vc);
    foo(s);
    foo(sv);
}
```

"But that means I'm going to need to pollute my C++ code with `__dslice`!"

Again, due to C++ implicit conversions, using the `__dslice` in the example header any `T[]` returned from a function will,
if necessary, be cast to `gsl::span`/`std::span` such that the C++ code can still be conformant to the C++ core guidelines.

```C++
// Defined in D as extern(C++) const(char)[] baz() { ... }
__dslice<const char> baz();
void takesASpan(std::span<const char> s);
void quux()
{
    std::span<const char> s = baz();
    takesASpan(s);
    takesASpan(baz());
}
```

Why not mangle as `gsl::span`/`std::span`?

We need to have a distict type so that it is available (`std::span` is only available in C++20 and `gsl::span` is an external library) 
the layout is known and guarunteed to match `T[]`. 

## Description

`T[]` where it appears in an `extern(C++)` type or function signature is to be treated as if it were 
```D
extern(C++) 
struct __dslice(T)
{
size_t length;
T*     ptr;
}
```

but not POD, for both mangling and ABI.

## Breaking Changes and Deprecations

N/A


## Copyright & License

Copyright (c) 2018 by the D Language Foundation

Licensed under [Creative Commons Zero 1.0](https://creativecommons.org/publicdomain/zero/1.0/legalcode.txt)

## Reviews

The DIP Manager will supplement this section with a summary of each review stage
of the DIP process beyond the Draft Review.
