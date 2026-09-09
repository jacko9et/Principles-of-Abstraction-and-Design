# Origin Language Semantics Design

> Positioning: the semantics layer = the **atomic operations** provided by the language environment and their interpretations—what value each atomic operation acts on, what effect occurs, what it produces, and how it responds on failure. This document answers "what each atomic operation does".

## Positioning and scope

- The semantics layer is a design of the language environment: it defines atomic operations and their interpretations;
- Fourfold grounding: every atomic operation is an **operation** (operations are also values); its interpretation = the environment's interpretation of the operation;
- Connections with existing documents:
  - The structural operations of *Language Structure Design* "The minimal operation set" (`=` binding / lambda / function declaration / field access / if / sequential structure / type declaration / instance construction and method invocation / failure capture / return / loops) are not repeated here;
  - The "interpretation table for individual atomic operations" left over by *Type System Design* (the "Skeleton boundary" of the weak typing chapter) lands here;
  - Operation interpretation rules are mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent")—the interpretation of every atomic operation does not change between strong/weak typing modes;
  - Text form of operations: operation names are names (see *Syntax Design* "The text of operation names"); there are no keywords.

## The minimal set of atomic operations

### 1. Arithmetic operations

- Operation names: `+` / `-` / `*` / `/`—infix connecting operations (see *Syntax Design* "How the text environment interprets these values", item 6), in the form `x + y` (interpretation order is defined in the syntax layer);
- Interpretation: acts on a **pair of values of the same numeric kind**—two int → int, two long → long, two double → double, two float → float (operations interpret themselves; see *Type System Design* "The nature of 'weak'"); byte is a byte-data unit and does not participate in arithmetic;
- Interpretation of `-` = the difference of two values of the same numeric kind (binary); a unary minus is not in the minimal interpretation rules—negative-number expression can be refined in later versions;
- Mixed types (e.g. int and long) are outside the interpretation rules → the operation cannot complete its effect → a diagnostic value (see "The unified response to operation failure" below)—consistent with "no implicit relations between numeric types" (see *Type System Design* "The mechanism of type relation judgment");
- Failure cases: `/` division by zero; overflow (the result exceeds the precision of the corresponding type—int / long precision is determined by a configuration policy; see *Type System Design* "Atomic type values");
- The other interpretation of `+`: acting on two Strings → concatenation producing a String (a Java convention); mixing String with numeric values is outside the interpretation rules → a diagnostic value (no implicit conversion; see *Type System Design* "The nature of 'weak'", "no conversion");
- Failure response: see "The unified response to operation failure" below.

### 2. Comparison operations

- Operation names: `==` (equality judgment), `<` (less than), `>` (greater than)—infix connecting operations (see *Syntax Design* "How the text environment interprets these values", item 6), in the forms `a == b` / `a < b` / `a > b`;
- Interpretation of `==`—acts on **any two values**:
  1. First judge the **distinguishability** of the two values: if they are **indistinguishable** (they are one value), their payloads are necessarily identical—produce true and do not compare payloads;
  2. If the two values are **distinguishable** (they are two values), compare their **payloads**: identical payloads produce true, different payloads produce false;
  - "Payload" = the carried content of a value: for int it is the number, for String the character sequence, for a structure the structural form—the payload comparison of each kind of value is provided by the environment's interpretation;
  - Unknown is merely the mark of "not interpreted" and has no essential difference from an ordinary value—every value has its carried content;
- Interpretation of `<` / `>`: acts on two values of the same numeric kind (int / long / double / float), judges by numeric order, produces a boolean; mixed types are outside the interpretation rules → a diagnostic value (see "The unified response to operation failure" below); byte equality uses `==` (already universal);
- Connection with the type system: `==` acts on any values, and its operation declaration is parameterized (see *Type System Design* "Type construction operations");
- Failure cases: none (any two values can complete a `==` judgment; any two values of the same numeric kind can complete a `<` / `>` judgment);
- Necessity: necessary—comparison is the boolean source for if conditions.

### 3. Conversion: type values are applicable

- Forms: `int(x)` / `long(x)` / `float(x)` / `double(x)` / `byte(x)` / `String(x)`—`int` and the like are **type values** (see *Type System Design* "Atomic type values"); when a type name is immediately followed by a parameter parenthesis, the environment interprets its application as **conversion** (the same form as instance construction `Person("Jack")`; see *Language Structure Design* "Instance construction and method invocation");
- Mechanism: conversion is not an independent operation; it is **one environment interpretation rule**—numeric and String atomic type values are applicable. Type values are still values (types are values; see *Type System Design* "The skeleton of the type universe"); the name `int` binds the same type value—in an application position it is interpreted by the environment as conversion; no second `int` is introduced;
- Applicability boundary:
  - **Numeric atomic type values** (int / long / float / double / byte) are applicable: acting on any numeric type → numeric conversion; acting on a String → **parsing** (`int("42")` produces 42; a parse failure → a diagnostic value; see "The unified response to operation failure" below);
  - **The String type value** is applicable: acting on **any value** → the string representation of that value (`String(42)`, `String(x)` where x is a char value obtained via `s.at(n)`, `String(true)`); idempotent: `String(s)` on an already-String s produces s;
  - Composite type values (e.g. `(A -> B)`), char / boolean type values: no application interpretation is defined;
- Interpretation notes:
  - All cross-type channels are covered, and all conversions are explicit (consistent with "no implicit relations, conversions are explicit" in *Type System Design* "The mechanism of type relation judgment");
  - Same-type conversion is idempotent: `int(x)` when x is already int produces x;
  - Narrowing out of range (e.g. `byte(256)`) → the operation cannot complete its effect → a diagnostic value (see "The unified response to operation failure" below);
  - Precision loss in `float(x)` is not treated as failure (the decision-maker explicitly chooses narrowing);
- Necessity: necessary—long / float / byte have no direct text forms (see *Syntax Design* "How the text environment interprets these values"); without a conversion path there is no way to obtain values of those types; interconversion between numbers and strings is a required dependency for composition-covering conversion methods (see *Language Structure Design* "Type declaration").

### 4. Arrays

- Form: `[1, 2, 3]`—square-bracket text (a syntax usage convention of the vast majority of programming languages; see *Syntax Design* "Parentheses and braces are operations: delimiting"); **the form and the name borrow from syntax usage conventions and inherit no language's semantic promises**—Java arrays promise fixed length, contiguous, homogeneous storage; we make none of those promises;
- Interpretation: acts on any number of values (including zero), producing an array value—an **ordered aggregation** of those values;
- **What "ordered" means**: elements are aggregated in the **written order** in the source code (the written order is preserved)—the aggregation must have this promise: if order were not explicitly promised (e.g. an unordered set), that would implicitly promise "automatic sorting", which would be absurd; ordered = written order preserved, **not** automatic sorting;
- The semantic promise of arrays = ordered aggregation; **the physical representation (contiguous storage or linked) is decided by the environment—the semantics layer makes no promise**; the mutability of array values is defined by the environment; this layer presumes nothing (isomorphic with the value mutability in *Language Structure Design* "Operation construction: lambda");
- **Element access and length**: `a[n]`—take the n-th element in written order (n starts at 0), producing that element's value; n out of range → a diagnostic value (see "The unified response to operation failure" below); `a.length`—the length field = the element count (int);
- Connection with the type system: an array type is the homogeneous-repetition special case `int[]` (see *Type System Design* "Type construction operations"); strong typing mode checks element homogeneity, weak typing mode does not; the aggregation itself is mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent")—the check is the type system's involvement;
- **Capacity hint**: the `T[N]` form (e.g. `int[10] a = [];`)—type `T[]` + a capacity hint N (an int value); N is an **expected element-count hint** to the implementing environment—the environment may reserve memory accordingly or ignore it (the semantics promise no physical allocation: the hint is structure, the reservation is the environment's response); isomorphic with the int precision policy (see *Type System Design* "Atomic type values": the decision-maker declares explicitly, the environment responds);
- The capacity hint **can be omitted**: when the initial text already gives the element count (`int[] b = [1, 2, 3];`), the hint is redundant—omission is the convention; the hint matters when the element count differs from the expectation: empty-initialization reservation (`int[10] a = [];`), expected growth (`int[5] c = [1, 2, 3];`);
- The capacity hint **does not promise the element count**: the semantic element count of an array = the number of aggregated values, unrelated to N; N is not "N default elements" (there is no default-value concept—decided);
- The empty array `[]` is legal;
- Failure cases: none (any values can be aggregated);
- Necessity: necessary—ordered aggregation is a required capability; without a construction form, array values cannot come into being.

### 5. String operations

- **Carried by a type**: `at` / `length` are **members of the String type value** (the type declaration the environment provides for the String atomic type; see *Language Structure Design* "Type declaration")—String is a type value (everything is a value), and a type value's payload carries its members (at is a method, length is a field; method and field values are all values); members are **not in the global environment binding table**—writing `at` alone finds no binding (operation name unbound; the existing semantics in *Type System Design* "Failure semantics")—they can only be obtained through a String instance;
- Forms: `s.at(n)`—at is a **method**: the dot connecting operation, with the immediately adjacent parenthesized combination `(n)` as the parameter (see *Syntax Design* "How the text environment interprets these values", item 4); `s.length`—length is a **field** (a member name with no adjacent parenthesized combination), the field access form (see *Language Structure Design* "Field access");
- Interpretation of `s.at(n)`: receiver s (String) + parameter n (int)—take the n-th character of s by **grapheme cluster**, producing a char; n out of range or the receiver is not a String → the operation cannot complete its effect → a diagnostic value (see "The unified response to operation failure" below);
- Interpretation of `s.length`: receiver s (String)—produce the value of s's length field (an int obtained by counting grapheme clusters); the receiver is not a String → a diagnostic value (see "The unified response to operation failure" below);
- Grapheme-cluster **boundary division** is a required dependency of these two members; grapheme-cluster **ordering** is an extension capability and can be refined in later versions (char is unordered—`<` / `>` do not interpret char; char equality uses `==`);
- The channel for obtaining char: char has no direct text; `s.at(n)` is the way to obtain a char;
- Necessity: necessary—char is on the minimal list (see *Type System Design* "Atomic type values"); without s.at there is no way to obtain a char.

### 6. Output

- Operation name: `print`—application form `print("Hello World!")` (a Java convention);
- Interpretation: acts on one value, produces an effect (presents the value to the decision-maker—the manner of presentation is decided by the environment), and produces the output value itself;
- Fourfold grounding: an operation (a boundary operation between the environment and the decision-maker)—values are abstractions inside the environment; print lets a value reach the decision-maker;
- Failure cases: none (any value can be presented; the physical manner of presentation is decided by the environment);
- Necessity: necessary—without print, the result of interpretation cannot be observed by the decision-maker.

## The unified response to operation failure

- When an operation's name **is bound** but its effect **cannot be completed** (division by zero, overflow, etc.), the operation produces a **diagnostic value**: the operation judges that its parameters cannot complete its effect, refuses, and reports back to the environment;
- The diagnostic value propagates upward along the composition hierarchy and can be captured by higher-level operations or reach the decision-maker (the propagation mechanism in *Type System Design* "Rejection and diagnostics");
- Semantic division of labor (consistent with *Language Structure Design* "Overview of structural forms" / "Field access"):
  - **Operation name unbound** (name lookup failure) → **weak typing mode falls back to Unknown** (the core anchor carries "not interpreted"); **strong typing mode produces a diagnostic** (the check steps in; statically decidable);
  - **The operation recognizes the parameters but judges that it cannot complete its effect** → a **diagnostic value**—this response is **mode-independent** (see *Type System Design* "Operation interpretation rules are mode-independent"): identical in weak/strong typing modes;
- Relation to weak typing mode: "weak typing mode produces no type diagnostics" refers to type diagnostics—this response is an operation-level diagnostic value; the two do not conflict.

## Design principles (carried over from existing documents)

- Every atomic operation passes the fourfold self-check;
- Interpretation rules are mode-independent;
- Minimal necessity: operations grow on demand (the environment can register new operations; the universe is not closed).
