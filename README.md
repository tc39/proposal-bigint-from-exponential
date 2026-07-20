# Bigint from exponential

## Status

Champion(s): Richard Gibson ([@gibson042](https://github.com/gibson042))

Stage: 1

## Motivation

Exponential notation is useful for dealing with big integers, but unavailable for direct use in defining a bigint.

Use of `_` separators helps, but isn't quite enough for sufficiently large values.
Manual conversion is possible (e.g., `/^(?:(0|[1-9][0-9]*)(?:[.]([0-9]*))?|[.]([0-9]+))(?:[Ee]([+-]?[0-9]+))?$/` validates and matches the relevant whole-number/fraction/decimal-exponent parts), but cumbersome and tedious.

## Use cases

This was discovered in [Amount](https://github.com/tc39/proposal-amount/issues/107), but also comes elsewhere (e.g., parsing JSON like `{ "scale": 1e6 }` with source text access, or when working with currencies, dates, or times).

Many use cases are static, where the exponential notation is part of the source text, while others are dynamic and therefore require a built-in function.

## Description

Support for static use cases might be provided syntactically, e.g. `1e6n`.
If so, this might be unprecedented in widespread programming languages (cf. https://github.com/tc39/ecma262/pull/3857#issuecomment-4960986324).

Support for dynamic use cases could be provided by expanding the behavior of `BigInt("1e6")` as suggested by https://github.com/tc39/ecma262/pull/3857, or alternatively by something like `BigInt.parse("1e6")` or `BigInt.fromString("1e6")`.
Note that the former likely implies corresponding changes to operator behavior, e.g. `1_000_000n == "1e6"` and `1_000_000n <= "1e6"` and `1_000_000n >= "1e6"`, just like the Number analogs (`1_000_000 == "1e6"` and `1_000_000 <= "1e6"` and `1_000_000 >= "1e6"`).

Dynamic use cases cover the static ones, albeit with erosion of developer experience and implementer satisfaction.

### Prior art

Some npm packages already support parsing exponential-notation strings into arbitrary-precision integers (or analogous strings):
* [big-integer](https://www.npmjs.com/package/big-integer): `bigInt("9.007199254740993e15")`
* [json-bigint](https://www.npmjs.com/package/json-bigint): `parse("9.007199254740993e15")`
* [from-exponential](https://www.npmjs.com/package/from-exponential): `fromExponential("9.007199254740993e15")`

And there is an even larger population of arbitrary-precision decimal libraries that generalize such behavior beyond integers.

Across other programming languages, .NET `BigInteger.Parse(str, options)` supports `AllowDecimalPoint` and `AllowExponent` options, but that seems to be as far as it goes:

Language/API | Leading `+`/`-` | Wrapping whitespace | Radix prefixes<br>(e.g. `0x`) | Separators<br>(e.g. `_`) | Decimal/exponent
-- | -- | -- | -- | -- | --
Go [`(new(big.Int)).SetString(str, 0)`](https://pkg.go.dev/math/big#Int.SetString) | yes | no | yes | yes | no
Java [`new BigInteger(str, radix)`](https://docs.oracle.com/javase//7/docs/api/java/math/BigInteger.html#BigInteger(java.lang.String,%20int)) | yes | no | _n/a_ | no | no
JavaScript [`BigInt(str)`](https://tc39.es/ecma262/multipage/numbers-and-dates.html#sec-bigint-constructor-number-value) | yes | yes | yes | no | no
.NET [`BigInteger.Parse(str, options)`](https://learn.microsoft.com/en-us/dotnet/api/system.numerics.biginteger.parse?view=net-10.0#system-numerics-biginteger-parse(system-string-system-globalization-numberstyles-system-iformatprovider)) | optional | optional | no | optional | optional
Python [`int(str, base=0)`](https://docs.python.org/3/library/functions.html#int) | yes | yes | yes | yes | no
Rust [`num_bigint` `from_str_radix(&str, radix)`](https://docs.rs/num-bigint/latest/num_bigint/struct.BigInt.html#method.from_str_radix) | yes | no | _n/a_ | yes | no

And for context, ECMAScript is unusual in its strictness:

Language/API | Floating-point → arbitrary-precision integer
-- | --
Go [`(*big.Float).Int(new(big.Int))`](https://pkg.go.dev/math/big#Float.Int) | truncates toward zero
Java [`bigDecimal.toBigInteger()`](https://docs.oracle.com/javase//7/docs/api/java/math/BigDecimal.html#toBigInteger()) | truncates toward zero
JavaScript [`BigInt(flt)`](https://tc39.es/ecma262/multipage/numbers-and-dates.html#sec-bigint-constructor-number-value) | rejects non-integer
.NET [`new BigInteger(flt)`](https://learn.microsoft.com/en-us/dotnet/api/system.numerics.biginteger?view=net-10.0#instantiate-a-biginteger-object) | truncates toward zero
Python [`int(flt)`](https://docs.python.org/3/library/functions.html#int) | truncates toward zero
Rust [`num_bigint` `from_f64(flt)`](https://docs.rs/num-bigint/latest/num_bigint/struct.BigInt.html#method.from_f64) | truncates toward zero

## Presentation history

* as [normative PR #3857](https://github.com/tc39/ecma262/pull/3857): May 2026 TC39 plenary ([notes](https://github.com/tc39/notes/blob/main/meetings/2026-05/may-19.md#needs-consensus-pr-support-bigint-coercion-of-integers-expressed-as-exponential-notation-strings-3857))
* converted to a Stage 1 proposal: July 2026 TC39 plenary

## Implementations

### Polyfill/transpiler implementations

None yet.

### Native implementations

Check here after Stage 2.7.

## Frequently asked questions

**Q**: Should dynamic parsing also support `_` separators?

**A**: Not through `BigInt(string)`, but maybe through a different API.

**Q**: Should dynamic parsing be configurable?

**A**: Also to be determined.
