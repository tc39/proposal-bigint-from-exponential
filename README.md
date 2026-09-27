# Bigint from exponential

## Status

[The TC39 Process](https://tc39.es/process-document/)

**Stage**: 1

**Champions**:
- Richard Gibson ([@gibson042](https://github.com/gibson042))

**Specification**: http://tc39.es/proposal-bigint-from-exponential/

## Motivation

Exponential notation is useful for dealing with big integers, but unavailable for direct use in defining a bigint.

Use of `_` separators helps, but isn't quite enough for sufficiently large values.
`BigInt(…)` is lossy above 2<sup>53</sup>.
Manual conversion is possible (e.g., `/^(?:(0|[1-9][0-9]*)(?:[.]([0-9]*))?|[.]([0-9]+))(?:[Ee]([+-]?[0-9]+))?$/` validates and matches the relevant whole-number/fraction/decimal-exponent parts), but cumbersome and tedious.

## Use cases

This was discovered in [Amount](https://github.com/tc39/proposal-amount/issues/107), but also comes up in other contexts (GitHub reports [20.3e3 files matching pattern `BigInt[(][^)a-z]*[0-9]e[0-9]`](https://github.com/search?q=%2FBigInt%5B%28%5D%5B%5E%29a-z%5D*%5B0-9%5De%5B0-9%5D%2F&type=code)):
* parsing JSON like `{ "scale": 1e6 }` with source text access
* working with currencies and/or financial values, particularly fine-grained cryptocurrencies (e.g., in [algorand-js](https://github.com/Folks-Finance/algorand-js-sdk/blob/c5c7f730c9ae69692d634cc7d85c9c0c928b1df0/src/math-lib.ts#L4))
* working with high-resolution dates or times (e.g., in the [ECMAScript Temporal polyfill](https://github.com/js-temporal/temporal-polyfill/blob/c55b211de913320d8aba32c456bb45914739f414/lib/bigintmath.ts#L9-11), and [WASI libraries](https://github.com/cloudflare/workers-wasi/blob/55d7dc2374f6ccf7a863127b4635de87f920b2e0/src/index.ts#L321))

Many use cases are static, where the exponential notation is part of the source text, while others are dynamic and therefore require a built-in function.

## Description

For static use cases, we propose syntactic support like `1e6n`[^1].

[^1]: This might be unprecedented in widespread programming languages (cf. https://github.com/tc39/ecma262/pull/3857#issuecomment-4960986324).

For dynamic use cases, we propose expanding the behavior of StringToBigInt as suggested by https://github.com/tc39/ecma262/pull/3857, supporting `BigInt("1e6")`/`BigUint64Array.from(["1e6"])`/etc.
Note that this also affects operator behavior (e.g., `1_000_000n == "1e6"` and `1_000_000n <= "1e6"` and `1_000_000n >= "1e6"`, just like the Number analogs `1_000_000 == "1e6"` and `1_000_000 <= "1e6"` and `1_000_000 >= "1e6"`) absent explicit changes to preserve backwards compatibility.

Alternatives like `BigInt.parse("1e6")` or `BigInt.fromString("1e6")` could also be considered.

Dynamic use cases technically cover the static ones, albeit with erosion of developer experience and implementer satisfaction.

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
* converted to a Stage 1 proposal: July 2026 TC39 plenary ([slides](https://docs.google.com/presentation/d/1kPYmR8LkoV4AZpSM_ezHCyR2givdk9OiQlwiXHdah4M/edit?usp=drive_link))

## Implementations

### Polyfill/transpiler implementations

None yet.

### Native implementations

Check here after Stage 2.7.

## Frequently asked questions

**Q**: Should dynamic parsing also support `_` separators?

**A**: Not through `BigInt(string)`, but maybe through a different API.

**Q**: Should dynamic parsing be configurable?

**A**: Not without doing the same for numbers, which seems like a separate proposal.

**Q**: Are decimals allowed as in e.g. `BigInt("86.4e12") === 86_400_000_000_000n`?

**A**: Yes, and this is important for precision-preserving canonical forms (e.g., differentiating `1.000e3n` from `1e3n`).

**Q**: Are negative exponents allowed as in e.g. `1230e-1n === 123n`?

**A**: To be determined. Restricting support to non-negative exponents would not be difficult.

**Q**: Is it web-compatible to change the behavior of `1_000_000n == "1e6"` and `1_000_000n <= "1e6"` and `1_000_000n >= "1e6"`?

**A**: To be determined, but if not then it is still possible to carve out exceptions while still supporting the primary use cases.
