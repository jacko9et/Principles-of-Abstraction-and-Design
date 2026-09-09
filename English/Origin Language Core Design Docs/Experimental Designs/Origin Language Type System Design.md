# Origin Language Type System Design

> Positioning: the type system is Origin's **environment-layer rule set**, not part of the core. This document establishes its design principles and structural skeleton, and refines concrete rules step by step in later chapters.

## Positioning (one sentence)

> **Type annotation belongs to structure; type checking belongs to the environment.** Strong and weak typing are not two languages, but the same structure under different interpretation policies of the environment; the mode is declared explicitly by the decision-maker **on the source code side** (fixed once written, because of structural invariance), and the compile-time and run-time environments read that declaration and choose the corresponding interpretation policy.

## Design principles

### P1 Separation of structure and interpretation

Type annotations (type syntax, type symbols, type relation descriptions) belong to **structure**; type checking (whether to check, when to reject, how to infer) belongs to the **environment's interpretation rules**. The two are orthogonal.

- Rationale: a direct corollary of the Origin core's "structure preserved, environment responds". Source code only describes relations and carries no execution rules; checking is the environment's response to structure.

### P2 Core neutrality

The Origin core builds in no type rules and holds no position on strong/weak typing. The choice of strong/weak typing and the implementation of type rules are entirely borne by environments (compile-time or run-time).

- Rationale: types are "desired capabilities", not "existing facts". The core keeps only the minimal abstraction that cannot be removed (the Unknown anchor + the compiler application interface) and does not overstep to make type decisions for environments.

### P3 Policy declaration on the source code side

The choice of strong/weak typing is declared explicitly by the decision-maker **in the source code** (at the head of the source code). Because of structural invariance, a mode declaration, once written, is fixed and is part of the structure; the environment (compile-time / run-time) reads that declaration and chooses the corresponding interpretation policy. The environment is forbidden to guess when the source code has no declaration. Declarations come in two levels (project-level and region-level); see "The two-level forms of mode declaration" below.

- Rationale: structure preserved, environment responds—a mode declaration is structure, and choosing a policy is the environment's response to that structure. Explicit over implicit.

### P4 Weak typing by default

When the source code declares no mode, the environment interprets under **weak typing mode**.

- Rationale: the compatibility direction determines the default. Weak typing can be compatible with strong typing (structures that pass under strong typing behave identically under weak typing; see P6), while strong typing cannot be compatible with weak typing (structures in weak-typing form would be rejected under strong typing). The default takes the more compatible side, guaranteeing that any source code can run; a decision-maker who wants strict checking simply declares strong typing mode explicitly.

### P5 Isomorphic forms, different interpretations

Strong and weak typing are **not two type systems**, but the interpretation of the same set of structures (including type annotations) under two checking policies. The policy difference shows only in "whether checking steps in and when to reject", not in structural forms.

- Rationale: avoid two parallel languages; only by keeping the structure single can "structure unchanged, environment responds" hold.

### P6 Monotonic compatibility

Type checking policies form an ordered relation: **the strong typing policy is a tightening of the weak typing policy** (weak ⊃ strong). A structure that passes under the strong typing policy must also pass under the weak typing policy, with identical behavior; the only added constraints of the strong typing policy are—types are enforced, and a missing type that cannot be inferred is rejected (see "Type enforcement policy" below).

- Rationale: guarantee that policy switching produces no surprising behavior; strong typing mode is the subset semantics of weak typing mode—switching only adds constraints and does not change existing meanings. P4's default choice follows from this.

### P7 Compile time and run time each do their part, both obeying the source code declaration

Both the compile-time and the run-time environments read the source code's mode declaration; the division of checking duties (static vs dynamic checking) belongs to each environment, but both sides' interpretation policies obey the same source code declaration and must not contradict each other.

- Rationale: compilation and execution are two environment layers in the first place; the mode is set uniformly by the source code declaration, and duties are divided locally by the environment currently interpreting the structure.

### P8 Determinism

Under the same structure and the same policy configuration, repeated interpretations produce identical results.

- Rationale: carried over from the determinism requirement of *Implementer's Guide*. Policies are part of the interpretation rules; interpretation must be predictable and traceable.

### P9 Traceable diagnostics

When a policy rejection occurs, the diagnostic must locate the concrete structure and the concrete judgment; silent failure is not allowed.

- Rationale: diagnostics are the environment's responsibility. Every rejection of a checking policy should be predictable and locatable by the decision-maker.

### P10 Types introduce no new category

Type symbols are themselves values (distinguishable and able to participate in compositions); the type system introduces no new category independent of "value". Before any new concept is added, it must answer: which of the fourfold does it fall on? Is it "necessary" or "a desired capability"?

- Rationale: consistent with the core's fourfold ontology (value · operation · environment · composition); desired capabilities move out of the core and grow in the environment.

### P11 Mode penetration is explicitly controlled

The strong typing mode environment **may** decide whether to allow weak-typing expressions to **penetrate** in (i.e. using weak-typing expressions in strong-typed code). If allowed, two conditions must be met: ① the source code declares the penetration region with an explicit marker (auditable); ② when the penetration region's result returns to strong typing mode, it must pass an explicit **boundary conversion** and run-time check—otherwise strong typing's guarantees leak silently. The reverse direction needs no penetration: strong-typing-form expressions are interpreted directly under the weak typing policy in weak typing mode.

- Rationale: penetration is decided by the environment currently interpreting the structure (see *Origin Language Core Abstract Design*, the "Penetration" section). Strong ⊂ weak only says "strong-typed-form code holds in weak typing mode"; it does not say "weak-typing-form expressions can be exempt from checking in strong typing mode". Allowing penetration is the environment's decision, but the boundary must be explicit; otherwise P6 and P9 are violated.

### P12 The dual view of checking policies

A checking policy **is a value in the structural view (source code side)** and can be operated on; it **is an environment in the environment view (run-time side)** and cannot be operated on.

- Source code side: the type system environment provides two policies (strong typing policy / weak typing policy) as selectable values; the decision-maker picks one through the mode declaration at the head of the source code (P3) and tells the environment—a mode declaration is a structure that "references a policy value". As a value, a policy can be referenced, passed, and participate in compositions.
- Run-time side: the selected policy becomes the environment interpreting the current source code (a given condition) and cannot be operated on by the program being interpreted—there is no "switching the checking policy at run time".
- The two views occur mutually exclusively, with no recursive dependency (see *Origin Language Core Abstract Design*, the "Environment" chapter): at run time it is impossible to operate on "the environment that is interpreting itself"; on the source code side, what is operated on is only a policy value that has not yet become an environment.
- The forms of concrete meta-operations (querying, passing, constructing new policies, etc.) are left to later chapters; among them, "constructing a new policy" belongs to the capability of "source code constructs environments" and will be refined further in later chapters.

- Rationale: consistent with "value, operation, environment are different abstractions of the same reality at different observation levels"; it also guarantees P8 determinism—the run-time policy is immutable, and interpretation is consistent under the same configuration.

## The semantic skeleton of weak typing mode

> Note: weak typing mode is the default mode (P4), and its semantics depend on penetration boundary conversion and P6 monotonic compatibility; this section gives the semantic skeleton, with details extended in later chapters.

### 1. The nature of "weak": no checking, no conversion, operations interpret themselves

The "weakness" of weak typing mode is **non-involvement at the type system level**:

1. **No checking**: the environment rejects no structure on account of type relations. Values can enter any operand position; "type rejection" is not a decidable state in weak typing mode. No special type concept like Any is introduced for this (P10).
2. **No conversion**: the type system provides no global implicit conversion mechanism. Implicit conversion would have the type system "not involving" and "involving" at the same time—conceptually self-contradictory—and would silently change values, conflicting with P9 traceable diagnostics.
3. **Operations interpret themselves**: the interpretation rules of an atomic operation for its parameters are defined by the operation itself (operations are provided and interpreted by the environment). If an operation needs lenient behavior, that is that operation's interpretation rule, not a type system mechanism.

- The "global implicit conversion" (auto-adapting weak typing) is abandoned. Rationale: it mixes the type system's duty into the operation's duty; operation-level leniency is kept but attributed to the operation itself.

### 2. Operation interpretation rules are mode-independent

The interpretation rules of the same atomic operation are **identical** in strong/weak typing modes; the difference between strong/weak typing modes is only whether the type system's checking steps in (strong typing mode checks; weak typing mode does not), and it changes no operation's own interpretation.

- Rationale: this is P6 monotonic compatibility guaranteed at the operation layer—otherwise "a structure passing in strong typing mode behaves identically in weak typing mode" would be broken at the operation layer.

### 3. The status of type positions in weak typing mode: no response

- Types belong to structure (P1). The weak typing mode environment does **not respond** to types: types do not participate in checking, produce no diagnostics, and do not affect interpretation results.
- Weak-typed source code may or may not write types; written types exist only as structure, and the environment's choice does not look at them.
- "Partial use of types" (e.g. as hints for diagnostics or optimization) belongs to the environment's optional enhancements; it is outside the current skeleton and can be extended later as needed.

### 4. Failure semantics: falling back to the Unknown anchor

- Weak typing mode rejects no structures, so there is no "type rejection". The only failure at the type system level is: **operation name unbound** (name lookup failure; see *Semantics Design* "The unified response to operation failure").
- In that case the result falls back to the core public anchor: carried by Unknown (payload = the name and parameters, etc.; the form is decided by the environment), and it can continue to be passed, staged, or used as a boundary return value.
- Unknown is not a diagnostic: weak typing mode produces no type diagnostics. The name and parameters carried by Unknown provide traceability and do not conflict with P9 (P9 constrains "rejection and conversion"; weak typing mode has neither).
- "Run-time type checking", if it exists, appears only at strong typing's penetration boundary; there is none inside weak typing mode.

### 5. Shared structural layer: type position syntax is unique

- Types have **one way of writing** (the type position in declaration structures; see *Language Structure Design* "Binding" / "Operation construction: lambda" / "Type declaration"), shared by strong/weak typing modes; there are no two ways of writing types depending on the mode.
- The same typed source code: in strong typing mode types are enforced (response); in weak typing mode types are ignored (no response).
- Corollary: the mode declaration (P3) is a special structure at the head of the source code, part of the same "structural layer is unique" as type positions.

### 6. Skeleton boundary (outside the current skeleton; extended later)

- The concrete interpretation table of individual atomic operations—belongs to the operations themselves and is not defined here.
- The concrete checking of strong typing mode—see "The semantic skeleton of strong typing mode" below.
- Diagnostic formats, run-time checking mechanisms, and the optional "partial use of types"—grow later.

## The semantic skeleton of strong typing mode

> Note: weak typing mode is the default mode (P4); strong typing mode is the explicitly declared tightening mode (P6). This section gives the semantic skeleton of strong typing mode, symmetric with "The semantic skeleton of weak typing mode" above; the type universe and type writing are extended in later chapters.

### 1. The nature of "strong": judgment, no conversion

The "strength" of strong typing mode is **the type system stepping in to check**:

1. **Judgment**: the environment responds to types in structures and judges type relations; a relation that does not hold → rejection.
2. **No conversion**: the type system only judges and does no implicit conversion—all conversions must be written explicitly. Consistent with weak typing mode (see "The nature of 'weak'" above): the type system changes no values in either mode; the difference is only "whether judgment steps in".
3. **Judgment and conversion are separated**: type relation judgment comes with no implicit conversion; operation interpretation rules are mode-independent (see "Operation interpretation rules are mode-independent" above) is reflected here—checking changes neither operations nor values.

### 2. What is checked

Strong typing mode judges three kinds of type relations:

1. **Bindable positions** (operation parameters / returns / fields / name bindings) have explicitly written or inferable types;
2. **Operation invocation**: in every invocation, the type relation between the operands' types and the operation declaration's parameter types holds;
3. **Penetration boundaries**: when a weak-typing region's result returns to strong typing mode, the boundary conversion holds (see "Region-level declaration" below).

Judgment results have three states: **holds / does not hold / unknown**. "Unknown" (missing type and not inferable) is uniformly treated as rejection; silent passage is not allowed.

### 3. Type enforcement policy: explicit-first + restricted local inference

- **Explicit boundaries**: parameters, returns, fields, and names bound into the environment—must have explicitly written types (they are interface contracts and cannot be inferred); **the lambda special case**: no extra type is written when binding a lambda—the lambda carries its own parameter types, and the return type is determined by the body's return (see *Language Structure Design* "Operation construction: lambda"); the type comes from the lambda itself, not from inference;
- **Local inference**: only at local positions of the initialized value structure is inferring a type from a value allowed; failed inference or going beyond the local scope → a diagnostic.
- Missing type and not inferable = rejection (a diagnostic)—this is the source of "strong"'s strictness.

### 4. Rejection and diagnostics (strictly distinguished from Unknown)

- A rejection in strong typing mode triggers a **diagnostic** (a diagnostic is an operation; its result is a diagnostic value). Diagnostics likewise fall within the fourfold ontology:
  - **Source code side**: a diagnostic is a **composition**—the relational structure in the source code describing "where and under what conditions a diagnostic is triggered";
  - **Run-time side**: a diagnostic is an **operation**—it really takes effect (produces a diagnostic value and reports back to the decision-maker);
  - **The diagnostic value**: is a **value**—distinguishable and passable, and can be captured or handled by higher-level operations in the structure (consistent with *Implementer's Guide* "On errors and boundary cases": a diagnostic value is a special value returned by the environment, brought into the composition relations rather than excluded).
- Contrast with weak typing mode: weak = operation name unbound → the Unknown value keeps passing; strong = the type relation does not hold → a diagnostic is produced (the operation takes effect; the result is a diagnostic value). The diagnostic's influence is **narrowed to the nearest environment**: the (nearest) environment that is judging the structure **terminates this interpretation of the structure**; the diagnostic value propagates upward along the composition hierarchy and can be captured by higher-level operations or reach the decision-maker; **the structure itself is still a value**—it is not invalidated and can be handed to other environments for interpretation.
- Distinction from Unknown: both are values and both are passable, but the semantics differ—Unknown is the core anchor carrying "not interpreted" structures; **what is reported to the decision-maker is the diagnostic value**, which carries the environment's conclusion of "judged and rejected". The diagnostic (operation) itself is not the feedback; the feedback is its effect's product (the diagnostic value).
- Timing division (P7): statically decidable → compile-time diagnostics; dynamic judgments (penetration boundaries) → run-time diagnostics.
- Diagnostics are traceable (P9): located to the concrete structure and the concrete rule.

### 5. Skeleton boundary (outside the current skeleton; extended later)

- **The type universe**: base types, composite types, and the concrete kinds and judgments of type relations—grown item by item.
- **Inference algorithms** and **diagnostic formats**—left for later chapters.
- Numeric widening is decided (see "The mechanism of type relation judgment" below): no implicit relations; widening like int → long must be an explicit conversion.

## The two-level forms of mode declaration

> Here declaration forms and penetration markers are unified into two levels of the same mechanism: project-level (the B form) and region-level (the A form).

### 1. Project-level declaration (entry file; one declaration takes effect globally)

- Declaration position: the **entry file**—the entry of the whole project's source code collection. Declared once, it serves as the whole project's interpretation policy; other files do not repeat the declaration (the declaration takes effect for the whole project).
- Mechanism: the entry declaration is an operation of "source code constructs environments" (see *Origin Language Core Abstract Design*, the "Source code" chapter)—the decision-maker chooses a policy for the project environment. Once in effect, the policy becomes the environment's given rule and, per P12, cannot change on the run-time side (no switching during a project run).
- Default: with no project-level declaration, the whole project is interpreted under weak typing mode (P4).
- Fourfold grounding: **environment** (an environment construction parameter), not a structural property—every file in the project remains mode-neutral.
- Form: the mode declaration at the head of the entry file, e.g. `mode(strong);` as a standalone item (structural form, semicolon-terminated).

### 2. Region-level declaration (`mode(P) { ... }`, locally explicit)

- Form: a mode declaration is an ordinary **operation** provided by the **type system environment**, taking a policy value as the parameter and braces as the wrapped structure: `mode(strong) { ... }` / `mode(weak) { ... }`—the "operation name (parameters) { body }" form of the same family as `if (cond) { ... }` (see *Syntax Design* "Parentheses and braces are operations").
- Two typical uses:
  1. **Local override inside a project**: the project as a whole is strong-typed, with a certain region explicitly weak-typed;
  2. **Cross-boundary loading**: when the main program loads a standalone lightweight script, the script internally uses `mode(weak) { ... }` to explicitly declare interpretation under weak typing mode; the other landing of passing the mode in at the call entry at load time is "Passed-in invocation" below—both are P11's explicit penetration markers (auditable).
- Boundary: when the result of `mode(weak) { ... }` returns to the outer (strong typing) mode, it passes boundary conversion and a run-time check (`expect(T) { ... }`; see "The expect assertion operation" below).
- Fourfold grounding: **composition** (operation acting on values), sharing the same policy value with the project-level declaration (P12; see "Shared structural layer" above, structural layer is unique).

### 3. The relation between the two levels

- Project-level declaration = the environment's given policy; region-level declaration = a locally requested interpretation explicitly declared inside a structure—in essence penetration (P11).
- The two differ in form, granularity, and fourfold grounding, but reference the same set of policy values (strong / weak).
- Concrete syntax symbols, the way the entry file is designated, and the concrete form of boundary conversion are left to the syntax layer and later chapters.

### 4. The concrete form of boundary conversion: the expect assertion operation

- Form: `expect(T) { ... }`—an **assertion operation** provided by the **type system environment**, acting on (the type value T, the brace body: the penetrated structure)—the "operation name (parameters) { body }" form of the same family as `if (cond) { ... }`.
- Semantics: interpret the penetrated structure to obtain a value v, and check "v's run-time type holds at positions of T" (equal or substitutable) per the relation judgment of "The mechanism of type relation judgment" below—the basis is run-time type information (see "Type values are retained at runtime" below: type values are retained at runtime, and values from weak-typing regions carry type tags);
- **Boundary conversion does not change values**: expect only confirms / grants type identity and does no value transformation (numeric widening etc. are separate explicit operations)—judgment and conversion are separated (see "The nature of 'strong'" above).
- Pass → v enters strong typing mode with type T; fail → a diagnostic value is produced (see "Rejection and diagnostics" above: the nearest environment terminates this interpretation, and the diagnostic value propagates upward along the composition hierarchy).
- **Mode-independent** (see "Operation interpretation rules are mode-independent" above): expect's check is the operation's own interpretation rule and is equally usable in weak typing mode (the decision-maker asserts voluntarily).
- This is the landing form of P11 condition ②.

### 5. Passed-in invocation: passing the mode at load time

- Form: `load(link, weak)`—the loading operation `load` (provided by the type system environment), acting on (the script link link: a String value; the script's interpretation mode: a policy value); the application form is the same family as `print(...)` / `Person("Jack")`;
- Semantics: the environment loads the script structure pointed to by link and interprets it under the passed-in policy—a script's interpretation mode has three sources: **passed-in at load, script-internal declaration, project-level declaration**; the priority chain (isomorphic with the int precision policy; see "Atomic type values" below): **passed-in at load > script-internal declaration > project-level declaration > default weak typing (P4)**—with no passed-in policy the script-internal declaration takes effect; with neither, the project-level declaration applies (the project-level declaration takes effect for the whole project; see "Project-level declaration" above); with no project-level declaration, default weak typing;
- Relation to P11: **interpretation under the project-level declaration does not constitute penetration**—the loaded script shares the project's mode; there is no boundary difference; **an explicit pass-in at load (e.g. `load(link, weak)`) = an explicit penetration marker** (the decision-maker explicitly declares the loaded script's policy at the call entry, auditable); when the loaded script's result returns to the outer (strong typing) mode, it passes the `expect(T) { ... }` boundary check (see "The expect assertion operation" above);
- Fourfold grounding: load is an **operation** (provided by the environment; loading is the environment's interpretation); link is a String value; the policy is a value;
- **Implementation extension**: this section settles the form and semantic skeleton; the minimal implementation is not required to support loading immediately—extend the implementation along this form later;
- Necessity: a desired capability (the settled way to compose external scripts)—not required by the minimal implementation, but the design must have the form first.

## The structural-layer skeleton of type positions

> Note: this section specifies the **structural landing of types**: types are written directly in declaration structures; annotation composition is no longer independent syntax.

### 1. Type positions: types written directly in declaration structures

- Types are written in the **declaration structures of bindable positions**, type-first (Java-style): name binding `int x = 1;` (see *Language Structure Design* "Binding"), fields `String name;` and method declarations (parameter / return types) (see *Language Structure Design* "Type declaration"), lambda parameters `(int a) -> ...` (see *Language Structure Design* "Operation construction: lambda"); a lambda binding writes no extra type (the lambda carries its own parameter types + return determines the return type; see *Language Structure Design* "Operation construction: lambda");
- Fourfold grounding: types are **values** (P10); appearing in declaration structures they are part of the structure—the annotation operation is retired; zero new categories;
- The lambda return type is not written: determined by the body's `return` value (see *Language Structure Design* "Operation construction: lambda").

### 2. Type position separated from binding

- The binding (`=` operation) and the type position (the type value in the declaration structure) are structurally separated: `int x = 1;`—`int x` is the declaration structure (type + name), `=` is the binding operation;
- Benefits: the binding structure remains mode-neutral; types can exist independently (transparent in weak typing mode, checked in strong typing mode).

### 3. Type positions = bindable positions

- Types are written in the declaration structures of "bindable positions": name bindings / parameters / returns / fields (see "What is checked" above)—the field types and method declarations (parameter / return types) in a type declaration (see *Language Structure Design* "Type declaration") are the types of those positions;
- These positions are defined by the language environment's declaration structures; this skeleton declares the dependency but does not define the declaration structures themselves in the type system document (they belong to the language structure layer).

### 4. Type positions in weak typing mode: transparent

- The weak typing mode environment **recognizes** the type values in declaration structures but does **not respond** (see "The status of type positions in weak typing mode" above): interpreting `int x = 1;` = binding x → 1; the type value is transparent to the result—no checking, no diagnostics.

### 5. The form of type values

- Type symbols are string values; the `int` / `String` in a declaration structure is a **type value**.
- Type values come from the **type system environment's** atomic type values or are generated by type construction operations (composite types)—type construction is left to the type universe chapter.
- A type value appearing in a structure introduces no new syntax category (P10).

### 6. Skeleton boundary (outside the current skeleton; extended later)

- The type universe (the domain of type values: base types, composite types)—the next chapter.
- Declaration structures and binding structures—the language structure layer.

## The skeleton of the type universe

> Note: this section specifies the **constitution mechanism** of the type universe: where type values come from, how they are constructed, and how relations are judged; the concrete type list and the kinds of relations can be extended step by step as needed.

### 1. The constitution of the type universe

- The type universe = **atomic type values provided by the type system environment** + **type construction operations** (producing composite type values).
- Everything falls within the fourfold ontology: types are **values**, type construction is an **operation**—no new categories (P10).
- The "universe" is not a closed list: the environment can register new atomic type values; new types are born from composition by construction operations (capability growth).

### 2. Atomic type values

- Atomic types are type values provided by the **type system environment** (indivisible relative to that environment; consistent with the atomic relativity in *Origin Language Core Abstract Design*, the "Composition" chapter).
- Design principles:
  - One type, one semantics: no primitive/wrapper split (e.g. only one of int and Integer is kept);
  - Numeric types use **semantic naming** (not bound to bit widths):
    - int (signed integer), long (wider signed integer)—**precision is an environment configuration policy**, not hardcoded in code:
      - Policy source one: the decision-maker gives it explicitly in the project-level declaration (the mechanism in "The two-level forms of mode declaration" above);
      - Policy source two: **environment sensing**—at environment construction, the environment reads the underlying environment's parameters (e.g. the operating system's word size) and selects precision per a **predefined policy table** (a multi-policy mode): the environment has a built-in set of precision policies, e.g. 8-bit systems int=8/long=16, 32-bit systems int=32/long=64, 64-bit systems int=32/long=64 (on 64-bit, 32-bit int is still the optimal choice); **no fixed formulas like "multiples of the word size"**;
      - **Default = sensing** (consult the policy table, follow the hardware); an explicit declaration overrides the default; the policy table is extensible—when new hardware appears, add a policy row without changing code;
      - **Fallback**: in extreme cases (no configuration and no sensing), start the default policy—int = 32 (baseline the 32-bit policy table behavior; long = 64);
      - The priority chain of precision policies: explicit declaration > environment sensing (table lookup) > default policy;
      - Sensing is the environment's penetration of other environments (in the broad sense): other environments' content (here, the word-size parameter) enters the current environment and becomes part of its interpretation rules. Note: the penetration defined in *Origin Language Core Abstract Design* "Penetration" is "mapping other environments' interpretations into the current environment"; here it is taken in the broad sense—other environments' **content** enters the current environment;
      - Future hardware evolution needs no code changes, not even configuration changes; the source code's semantics is unchanged (structure preserved, environment responds);
    - float (single precision), double (double precision)—the name is the precision (IEEE 754);
    - byte (8-bit unsigned)—the byte is an industry-standard unit; its semantics is 8 bits, not policy-ized.
  - The precision policy is immutable at run time (isomorphic with P12); int / long overflow behavior belongs to operation-layer interpretation rules and is not defined here.
  - When interoperation with the underlying environment needs fixed widths, use the explicit conversion operations provided by the **type system environment** (operation names carry the format, e.g. 64-bit fixed-width encoding)—the language's type universe introduces no fixed-width type names (isomorphic with the character-encoding boundary).
  - boolean, String, and other base types necessary for computation / storage are on the minimal list.
  - **Members rest on type values**: the members of the String type (the at method / the length field) are payload components of the String type value—the type declaration the environment provides for the atomic type (see *Language Structure Design* "Type declaration"); they do not enter the global binding table, and members cannot exist detached from the type (the forms are in *Semantics Design* "String operations").
- **char is on the minimal list**: its semantics adopts the **extended grapheme cluster** (close to human intuition—display is for humans):
  - Encoding (UTF-8 / UTF-16, etc.) is a representation-layer matter: the type layer makes no promise; the environment is free internally;
  - char ↔ byte-sequence conversion is done by explicit encode/decode operations provided by the **type system environment**; the decision-maker chooses at the boundary (operations as adapters, conversions explicit); concrete encoding operation names are defined in later extensions;
  - Byte-level precise control uses the `byte` type + explicit encoding operations (bytes are bytes, characters are characters);
  - A string's character sequence is defined by grapheme clusters—depending on the grapheme-cluster **boundary division** rule (necessary); the grapheme-cluster **ordering** rule is an extension capability (lexicographic comparison) and can be supplemented in later versions;
- The minimal list: int / long / float / double / byte / boolean / String / char. Other widths and unsigned variants grow by environment registration on demand (the universe is not closed), with naming likewise following the semantic rules.

### 3. Type construction operations (composite types)

- A type constructor = **an operation that takes types as parameters and produces a new type value**—generics are type constructors; no new concept is introduced (P10).
- Composite types are also values: generated by construction operations acting on type values, they can continue to participate in construction as parameters (e.g. `(A -> B[])`).
- **The composite mechanism is unified into one rule: "a composition is interpreted as a type"**—a composition whose parameter positions hold type values is interpreted by the environment as a type description.
- The minimal list:
  - Function types: `(A -> B)` (parameter type A → return type B; multi-parameter forms are left to the syntax layer; the skeleton settles "parameter type sequence + return type"); **no return = the return type position is omitted**: `(A ->)` means taking an A and returning nothing—no placeholder type is introduced (no "one value" to mean "no value");
  - **Structural-form types**: composition itself is aggregation (a core-layer capability); a structural type = a composition-form description: field names = operation names, field types = type values at parameter positions (e.g. `(point (int x) (int y))`, type-first in the same direction as field declarations)—no separate type constructor category is established; **the homogeneous-repetition form (arrays) is a special case of structural-form types**: `int[]` describes "all array elements are int"—array type checking = matching judgment on the repetition form (all element positions uniformly match T); the array type form `T[]`—`[]` is a type construction operation name (postfix form; see *Syntax Design* "How the text environment interprets these values", item 7), and the type universe fixes no other names; the capacity hint `T[N]` (see *Semantics Design* "Arrays") attaches to declarations and is a reservation hint for the implementation layer, **not participating in type judgment** (`int[10]` and `int[]` are the same type value); the nominal is obtained by binding the form description to a name via the binding operation (`point-type = ...`);
  - **Type declarations** (see *Language Structure Design* "Type declaration"): `Name { ... }`—name + brace type body; the environment's interpretation = construct a nominal structural type value carrying fields and methods and bind the name—the "composition interpreted as a type" form description settles fields, the type declaration adds methods; instance construction = type values are applicable (see *Language Structure Design* "Instance construction and method invocation", the same mechanism as numeric conversion);
  - **Generics (type parameters)**: `Box<T> { ... }` declares type parameters; instantiation = type constructor application `Box(int)` (type values are applicable—the parameter is a type value → produces a type value)—generics are the form landing of type constructors (this mechanism) (see *Language Structure Design* "Type declaration", type parameters);
- **Design basis (why the two forms are no longer merged)**: function types describe **operation values** (the operation: the input→output effect), while structural-form types describe **composition values** (the composition: the relational structure of operation name + parameter positions)—the two fall on different elements of the fourfold; the mechanism is already unified into one rule (composition interpreted as a type), and the two forms are the convergence limit required by the ontology. Supporting evidence: no-return (`(A ->)`) belongs only to function types—an effect can have no product, but an aggregation cannot have no product.
- Later extensions (not on the minimal list): union types (sum types, the carrier of Option / Result, companion to "no null"), sets / maps, etc.—registered by environments on demand.

### 4. The mechanism of type relation judgment

- Relation judgment = an **operation** provided by the **type system environment**, acting on two type values and producing a judgment result (a value).
- Judgment and conversion are separated (see "The nature of 'strong'" above): relation judgment only produces conclusions and comes with no conversion.
- Judgment results serve the three kinds of checks in "What is checked" above; "does not hold" triggers a diagnostic (see "Rejection and diagnostics" above).
- The relation set—**equality** and **substitutability** (a composition corollary, not an independently declared relation):
  - **Equality**: type values are equal (same structure)—types are values; equality judgment is a core capability;
  - **Substitutability**: v's type composes T by declaration—**directly or transitively along the composition chain** (`C + B`, `B + A` ⇒ C holds for A)—and all of T's bodyless methods have implementations in v's type (implementations may come from member promotion; the nominal corollary of *Language Structure Design* "Type declaration" composition declarations, not structural inference)—judgment: v used at positions of T ⇔ v's type = T or v's type is substitutable for T;
  - **The parameter positions of type constructor applications and array types = invariant**: `Box(Dog)` at positions of `Box(Animal)`, and `Dog[]` at positions of `Animal[]`, both do not hold (judged only by equality);
  - **No implicit relations between numeric types**: widening like int → long must be an explicit conversion (judgment and conversion separated; see "The nature of 'strong'" above).
- Not in the relation set (left to the implementation layer): type–structural-form matching (e.g. matching `int[]` with a concrete array), generic instantiation (classified under equality judgment).
- Concrete symbols and declaration forms are left to the syntax layer.

### 5. Type values are retained at runtime

- Types are values; values exist at runtime. Erasing types = erasing values, which conflicts with the ontology.
- Therefore: type values receive the same treatment as other values and are not erased at runtime (avoiding the semantic cost of type erasure; run-time checks, penetration boundary checks, and run-time operations on type values all take type values as their basis).
- In weak typing mode type values likewise exist (not checked, but still passable).

### 6. The type universe in weak typing mode

- Type construction operations produce type values in both modes ("recognizing"); the difference is only "whether type values participate in checking"—this is the refinement of "The status of type positions in weak typing mode" above ("recognize but not respond") at the type universe layer.
- In weak typing mode: type values do not participate in checking and produce no diagnostics, but they can exist and be passed as values without affecting run-time interpretation results.

## Document evolution conventions

- This document only describes "how the type system is interpreted by the environment" and does not modify *Origin Language Core Abstract Design* or *Implementer's Guide*.
- If later designs find conflicts between the principles and the core abstraction, go back to the core document to check; the principles yield to the core.
