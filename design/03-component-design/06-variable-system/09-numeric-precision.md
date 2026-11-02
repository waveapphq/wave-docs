### 3.6.9 Numeric Precision

#### REQ-3.1.3.14 Exact numeric precision for integer and float variables

##### REQ-3.1.3.14 Exact numeric precision for integer and float variables

Numeric values are represented and preserved with exact precision, independent of the rounding
inherent to platform double-precision (IEEE-754) arithmetic. A numeric value a test author types
is the value the system stores, evaluates, compares, and displays - with no silent truncation
introduced by any layer. This applies to scalar integer and float variables, to integer and
float table cells, to integer and float elements of arrays and maps (REQ-3.1.3.19), and to every
other field that takes a number: Block fields, Function inputs, profile values, settings,
configuration values and recipe values.

**Exact representation**: Integer values hold at most 19 digits, the sign aside. Float values
preserve their exact decimal digits up to a fixed precision of 28 significant digits. Because a
double-precision number carries only about 15-17 significant digits, a value such as
`-2123.123213123123699999` would otherwise be corrupted on entry; under exact precision it is
retained verbatim.

**One number rule**: Every field that takes an integer or a float follows the same rule on every
surface and on the server. An integer is an optional leading minus and digits; a float is an
optional leading minus, digits, and optionally a decimal point followed by digits. No exponent,
sign plus, separator, space or other symbol is part of a number, and NaN or an infinity is never
a number. A number written as exact text that follows the rule is that number, so a value too
long for a native number keeps every digit. The editors refuse a keystroke that would break the
rule, and the server refuses on save any value that breaks it, whatever surface sent it. A
field's minimum and maximum are compared on the exact values.

**Single precision everywhere**: The 28-significant-digit bound is uniform across the UI input
boundary and the backend evaluation context. A value that is accepted is a value that
arithmetic preserves; there is no boundary at which a previously accepted value is silently
re-rounded the first time it participates in a computation.

**Exact transport**: Numeric values cross every boundary - UI input to storage, storage to
runtime, runtime to the monitoring surfaces - without losing a digit. A value travels as a native
number only when that native number keeps every typed digit and prints in plain notation;
otherwise it travels as its exact textual representation, and every receiver accepts both forms.
A saved value is stored in that same form, whatever surface or client sent it. No serialization or deserialization step re-introduces rounding or truncation. As a direct
consequence, a precise numeric value that a test never modifies shows no difference between its
defined (Initial) and runtime (Current) views, and comparison surfaces report no phantom delta.

**Values a run produces**: The rule above governs values a person or a recipe enters. A value a
run produces - an Expression result, the checked side of a condition, data read from a file, a
system or a plugin - keeps the conversion of the surface that reads it, and that conversion never
loses a digit.

**Plain notation only**: Float values are represented in plain decimal notation. Scientific or
exponential notation is never produced on any surface or wire.

**Exact arithmetic and operand promotion**: Arithmetic on float values is exact within the
28-significant-digit context. When an expression combines an exact float value with a
lower-precision numeric operand (for example a numeric literal written in an expression), the
operand is promoted into the exact-precision domain so the result remains exact rather than
failing or degrading to double precision. Comparisons and assertions on float values are
evaluated in the exact-precision domain.

**Plugin boundary**: Plugins operate in platform double precision by nature (network clients,
database drivers, shell and scripting runtimes). When an exact float value is passed into a
plugin as a parameter it is reduced to a platform floating-point number at the call boundary;
the variable's own stored value is unaffected. A Function Block is not such a boundary: its
fields reach the Function's inputs as exact numbers. Numeric values that plugins return re-enter the
exact-precision domain when they are stored back into a float-typed variable, keeping the
variable space uniform.
