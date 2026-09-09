# Origin Language Language Structure Design

> Positioning: the language structure layer = the minimal structural conventions the decision-maker uses to write source code—composed of the atomic operations and structural forms provided by the **language environment**. This document answers "what shapes source code can take".

## Positioning and scope

- The language structure layer is a design of the **language environment**: it defines the minimal operation set and structural forms the decision-maker uses when writing structures.
- Fourfold grounding: every concept defined in this layer (including every operation) must pass the fourfold self-check—**which** of value / operation / environment / composition the concept falls on, and whether **the concept itself** is a value (operations are also values).
- Relation to existing documents:
  - The core (the Unknown anchor + the compiler application interface): the language structure layer does not modify the core; it grows on top of it;
  - *Type System Design* "Type positions = bindable positions": the type system references this layer's "bindable positions", defined by this layer;
  - The syntax layer (future): this layer defines "structural forms"; the syntax layer defines "how text is written as structure"—the two layers are separated.

## Overview of structural forms (carried over from the core; not a new design)

- Minimal source code = one composition: `(op v1 v2 ...)`—the relational structure formed by an operation acting on values.
- Atomic values are provided by the environment (integers, strings, booleans... corresponding to the atomic types in *Type System Design* "Atomic type values").
- **Name reference**: a bare name appearing in a structure is interpreted by the environment as "look up in the environment bindings"—the name is structure, the lookup is the environment's response.
- **Composition interpretation**: `(op args...)` is interpreted by the environment as "op acts on args"—the core interface `operation(input)` as it appears in the language layer.
- **Java-style application form**: application = a value immediately followed by parameter parentheses—`f(a, b)` (operation value application), `Person("Jack")` (type value application = construction), `s.at(n)` (member method); `f(args)` and `(f args)` are two textual forms of the same application structure (syntax is not unique); infix operations (`+ - * / == < >`, binding `=`, member access `.`) act on the value structures on both sides (see *Syntax Design* "How the text environment interprets these values", item 6).
  - **The operation position**: the structure at the first position of a composition is interpreted first; if it produces an **operation value**, that operation value acts on the remaining parameters—a name (whose lookup yields an operation value) is only the simplest case of the operation position;
  - **Application is not an independent operation**: it is the composition's inherent interpretation rule—re-inventing the core interface as an independent operation violates the minimal abstraction;
  - The operation position fails to produce an operation value after interpretation: weak typing mode falls back to Unknown (name lookup failure, etc.; see *Semantics Design* "The unified response to operation failure"), strong typing mode produces a diagnostic—connected with the existing failure semantics; no new rules.

## The minimal operation set

### 1. Binding: =, an environment-mutating operation with no return

- Form: `int x = 1;`—binding structure = `[Type] name = valueStructure;` (`=` is the binding operation name, a symbolic name; see *Syntax Design* "How the text environment interprets these values", item 5; `;` is the connecting operation separating structures); the type may be omitted (weak typing mode has untyped binding: `x = 1;`), strong typing mode requires an explicit type (see *Type System Design* "Type enforcement policy", "explicit boundaries");
- Fourfold grounding: `=` is an **operation** (an atomic operation provided by the language environment), acting on (name, type value (if given), the interpreted value); the binding itself is a value (operations are also values);
- What it acts on is the **environment**: the effect of binding = constructing / updating a name binding in the current environment—*Origin Language Core Abstract Design*, the "Source code" chapter, "source code constructs environments", on the smallest scale;
- **Product: none**—binding is a purely environment-mutating operation and produces no passing value (the function type form = the return type position is omitted, connecting with *Type System Design* "Type construction operations").
  - This layer does not keep the design of "returning the bound value" or "returning the name": the former would require specifying which value is returned, and its semantics can be replaced by name reference; the latter confuses symbols with values. No return is the safer choice.
- **Name reference**: after binding, a bare name in a structure is interpreted by the environment as "binding lookup"—the name is structure, the lookup is the environment's response (see "Overview of structural forms" above).
- Interface with the type system: the name left of `=` is one of the "bindable positions" (see *Type System Design* "What is checked", name binding); the type sits in the declaration structure (`int x`, the same family as type-body field declarations; see "Type declaration" below), and the type position is separated from the binding structure (see *Type System Design* "The structural-layer skeleton of type positions");
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent"): the binding operation's interpretation rules are the same in both modes.

**Binding is mutable: rebinding = reassignment**:

- **Same-name rebinding = overwrite (reassignment)**: one unified form, no separate set—the environment finds no name = definition; it finds the name = reassignment (consistent with mainstream habits); reassignment form `x = 2;` (the type is already declared; no need to repeat it);
- **Identical-payload rebinding is not checked**: overwriting with an identical payload is harmless (merely redundant); checking would require comparing two values' payloads at runtime (first judge distinguishability, then compare payloads when distinguishable; see *Semantics Design* "Comparison operations"); do not introduce complexity for a harmless redundancy;
- Connection with the type system: in strong typing mode the reassigned new value must pass the declared type check of the bound name (see *Type System Design* "What is checked", operation invocation judgment); weak typing mode does not check—type safety is not damaged by mutable binding;
- Mutable binding stays consistent with the unified rebinding form and common usage habits; interpretation remains deterministic—the same structure under the same interpretation environment has a stable result.

### 2. Operation construction: lambda, producing an operation value

- Form: `(int a, int b) -> body`—a parameter list (`Type name`, comma-separated) + `->` (the arrow connecting operation; see *Syntax Design* "How the text environment interprets these values", item 6) + a body (a single structure, or a brace block `{ s1; s2; }`);
- Fourfold grounding: lambda is an **operation** (provided by the language environment) whose effect is to **construct an operation value**—lambda itself is a value, and its product is also a value.
- **The body is not evaluated at construction**: the parameter structures and the body structure are kept inside the operation value; the body is interpreted only at "application"—the body is structure; the environment interprets it at application time (structure preserved, environment responds).
- **Product = an operation value**: unlike binding (no return), lambda is a constructor; the operation value is subsequently bound to a name by the binding operation (see "Binding" above)—operation construction and name binding are separated.
- **Interpretation environment: carries the construction-time environment**—a lambda value carries the environment at construction; at application, the body is interpreted in the carried environment.
  - Ontological basis: environments are also values (the external-observer view); a lambda value = parameter structures + body structure + the construction-time environment;
  - **Carrying = sharing**: what is carried is the **environment value itself** at construction time—at application, bindings are looked up in that environment value; rebindings in the same environment after construction are visible at application;
  - The interpretation that looks up bindings along the call chain is excluded here, because it depends on the call chain, makes behavior unpredictable, and conflicts with the determinism requirement.
- Parameters are "bindable positions" (see *Type System Design* "What is checked"): parameter types go into declaration positions (`int a`, the same direction as field declarations; see *Type System Design* "The structural-layer skeleton of type positions"); strong typing mode requires explicit types ("Type enforcement policy", explicit boundaries); weak typing mode may omit types (`(a, b) -> body`); the return type is not written and is determined by the body's `return` value (see "Return" below);
- **The type of a lambda binding**: determined by the lambda itself—the parameter types are already written in the parameter list, and the return type is determined by the body's return; `twice = (int n) -> n * 2;` **needs no additional type position** (also true in strong typing mode: the type comes from the lambda itself, not from inference—the lambda special case of *Type System Design* "Type enforcement policy", explicit boundaries);
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent").
- Necessity: necessary—operations are values; without a way to construct new operation values, the language cannot grow.

**Function declaration (named functions)**:

- Form: `int add(int a, int b) { return a + b; }`; no return = the return type is omitted: `doSomething(int a) { a + 1; }` (no return in the body; each structure is interpreted in order);
- Same form family as method declarations (see "Type declaration" below): `[return type] Name(parameter declarations...) body`—distinguished by position: inside a type body = method declaration; at the structure layer (top level / inside a sequential structure) = function declaration;
- Declaration is binding: a function declaration carries its own name binding (same family as the type declaration `Person { ... }`); it does not go through `=` separately;
- Division of labor with lambda: function declaration = named function; lambda = anonymous function value (bindable, passable)—`int add(int a, int b) { return a + b; }` and `add = (int a, int b) -> a + b;` are semantically equivalent (syntax is not unique; see *Syntax Design* "Positioning and scope");
- Distinction rule: parenthesized content starting with a type value immediately followed by a name (`int a`) → function declaration; otherwise → body-carrying application (if / mode / expect; see *Syntax Design* "Parentheses and braces are operations");
- Fourfold grounding: structure (the environment's interpretation = construct an operation value + bind a name); zero new categories;
- Connection with the type system: strong typing mode requires explicit parameter types ("Type enforcement policy", explicit boundaries); omitted return type = no return (the body has no return);
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent").

**Parameters can be reassigned**: parameter names are bound in a **local environment** constructed at application; reassigning a parameter name changes only the binding relation in that local environment and does not affect the caller's environment.

- A binding relation = a "name → value" relation in an environment. When `f(a)` is applied: the argument structure a is interpreted by the environment to obtain a value, and the local environment establishes the relation "parameter name x → that value"; if a is a name already bound to that value, then what appears in the two relations of the caller's environment and the local environment is **that value**—the value appears in multiple relations and remains itself;
- Whether the values in the two relations are distinguishable is judged by the environment's distinction strategy (see *Origin Language Core Abstract Design*, the "Value" chapter: values must be distinguishable; the distinction strategy is provided by the environment);
- **Reassigning a parameter name** (e.g. `x = 2;`): the relation "x → value" in the local environment now points to the new value—that relation changes; the relation of a in the caller's environment is unaffected; a value's own state is not changed by changes to binding relations;
- **Mutability of values**: the core abstract design does not specify whether values are mutable or whether a value before and after a change is distinguishable—it is judged by the environment's distinction strategy; this layer's binding semantics presume no answer and only state the "name → value" relation and its changes.

### 3. Field access: a field name is an operation

- Form: `point.x`—the dot **connecting operation** (see *Syntax Design* "How the text environment interprets these values", item 4): the field name itself is an access operation that acts on an aggregate structure value and produces the field's value; **no unified get operation name is introduced**;
- Fourfold grounding: every field name is an **operation** (an atomic operation provided by the language environment), acting on an aggregate structure value and producing the field value; a field name is not an operation name in the global binding table—it is a **member name** and enters interpretation only through the dot connecting operation (members exist through values and cannot appear alone);
- Connection with the type system: *Type System Design* "Type construction operations", "field name = operation name"—the field names in a structural-form type are this layer's access operations; in strong typing mode the check of a field access = operation invocation judgment ("What is checked", category 2).
- Failure semantics: accessing a nonexistent field → weak typing mode falls back to Unknown (member name unbound; see *Semantics Design* "The unified response to operation failure"); strong typing mode produces a diagnostic—consistent with the existing failure semantics.
- **Member methods rest on type values**: types are also values (see *Type System Design* "The skeleton of the type universe"), and member methods (operations are also values) sit in the payload of type values (see *Type System Design* "Atomic type values"); when a member-name operation acts on an **instance value**, it takes the method out via the type value carried by the instance (type values retained at runtime; see *Type System Design* "Type values are retained at runtime")—member resolution is isomorphic with this mechanism: a member name is an operation that acts on an instance value and produces a receiver-bound method operation value (see "Instance construction and method invocation" below).
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent").
- Necessity: necessary—aggregate structures must be able to yield part of their values; otherwise compositions can only be passed whole and their internals cannot be used.

### 4. Conditional: if, lazy selective interpretation

- Form: `if (cond) { ... } else { ... }`—`(cond)` is the condition parenthesized structure, `{ ... }` is a branch block (sequential structure; see "Sequential structure" below), `else` connects the false branch; else is optional: no else and a false condition → produce "nothing" (same as binding's no return);
- Fourfold grounding: if is an **operation** (an atomic operation provided by the language environment), acting on (condition structure, true-branch structure, false-branch structure (if any)).
- **Lazy selection**: if interprets only the selected branch; the other branch is not interpreted—this is if's reason for existence: if both branches were interpreted, the condition would lose its meaning.
- **Qualification as a special operation**: if differs from ordinary application interpretation (all parameters interpreted first)—it **selectively interprets branches**.
- Truth/falsity judgment of the condition: prescribed by if's interpretation rules (interpret the condition structure, then judge truth per the environment's convention)—lenient behavior belongs to the operation itself, not to the type system's mechanisms (see *Type System Design* "The nature of 'weak'", operations interpret themselves).
- Connection with the type system: in strong typing mode, if's operation declaration (condition boolean, both branches of the same type) is checked per *Type System Design* "What is checked", category 2, operation invocation judgment.
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent").
- Necessity: necessary—without branching, structures cannot express "choose by condition" relations.

### 5. Sequential structure: the brace structure, interpreted in order

- Form: `{ s1; s2; ... }`—brace-delimited (see *Syntax Design* "Parentheses and braces are operations"); the structures inside are interpreted in order, producing the interpretation result of **the last** structure (the effects of earlier structures have occurred; their results are discarded); the block operation name is retired (the brace structure is the only form of sequential structure);
- Fourfold grounding: the sequential structure is **structure**; the environment's interpretation = in-order interpretation (the environment's response);
- **Why it is necessary**: binding's no-return design requires the sequential structure to carry subsequent computation (`{ int x = 1; x * 2 }`); without a sequential structure, computation after a binding cannot be expressed.
- **Special interpretation**: the sequential structure interprets in order and keeps only the last result—different from ordinary application where "all parameters are interpreted first".
- Failure semantics: a structure in the middle fails to interpret → the diagnostic value propagates upward along the composition hierarchy, and the remaining structures are not interpreted—the diagnostic's influence is narrowed to the nearest environment, which terminates this interpretation of the failing structure (isomorphic with *Type System Design* "Rejection and diagnostics").
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent").
- Necessity: necessary—the companion structure of binding's no-return; otherwise "compute after defining" cannot be expressed.

### 6. Summary of the minimal operation set

| Operation | Form | Fourfold grounding | Product | Notes |
|---|---|---|---|---|
| Binding | `int x = 1;` | Operation (environment-mutating) | none | reading the value is done by name reference |
| Operation construction | `(int a) -> body` | Operation (constructs an operation value) | operation value | carries the construction-time environment; parameters carry their own types |
| Function declaration | `int add(int a, int b) { return a + b; }` | Structure (constructs an operation value + binds) | none | same form as method declarations; declaration is binding |
| Field access | `point.x` | Operation (field name is an operation) | field value | dot connecting operation; no unified get |
| Conditional | `if (cond) { ... } else { ... }` | Operation (lazy selection) | selected branch result | special operation; else optional |
| Sequential | `{ s1; s2; }` | Structure (in-order interpretation) | last structure's result | block retired |
| Type declaration | `Person { String name; ... }` | Structure (environment interpretation = construct a type value + bind) | type value | fields + methods; composition `B + A`, generics `Box<T>` |
| Instance construction | `Person("Jack")` | Type values are applicable (an environment promise) | instance value | same mechanism as numeric conversion |
| Method invocation | `s.at(n)` | Dot connection + application | method result | receiver left of the dot |
| Failure capture | `try { ... } catch (e) { ... }` | Operation (selective interpretation) | none (structure) | special operation; e binds the diagnostic value |
| Return | `return v;` | Operation (terminates the body's interpretation) | value of v | special operation; product of method / lambda bodies |
| Loops | `while (cond) { ... }` / `for (int x : a) { ... }` | Operation (repeated interpretation) | none | special operations; counting for can be refined in later versions |

- Application: not a new operation—the inherent rule of composition interpretation (see "Overview of structural forms" above); application form `f(args)` (a value immediately followed by parameter parentheses); type values are applicable = numeric type conversion (see *Semantics Design* "Conversion") or structural-type instance construction (see "Instance construction and method invocation" below)—the environment's interpretation promise for type values in application position.
- Qualification for special operations: only operations that do not follow the "interpret all parameters first" rule (if lazy selection, sequential in-order interpretation, try selective interpretation) need special treatment.
- Binding mutability: same-name rebinding = overwrite (reassignment), one unified form without set; identical-payload rebinding is not checked; parameters can be reassigned (a local environment relation change that does not affect the caller); value mutability is defined by the environment, and this layer presumes nothing (see "Operation construction: lambda" above).

### 7. Type declaration: `Name { ... }`, constructing and binding a type value

- Form:
  ```
  Person {
    String name;
    int add(int a, int b) { return a + b; }
    String greet() { return this.name; }
  }
  ```
- Overall form: `Person { ... }` = type name + brace **type body**—the environment's interpretation of the adjacent "name + braces" structure = **type declaration**: construct a **type value** (types are values; see *Type System Design* "The skeleton of the type universe") and bind the name Person to it—declaration carries its own binding and does not go through the binding operation separately; the type name is therefore an ordinary name, and instance construction (see "Instance construction and method invocation" below) and member access (see "Field access" above) obtain the type value through name lookup;
- Fourfold grounding: a type declaration is **structure**; the environment's interpretation = construct a type value + bind (environment mutation); braces are delimiting operations (see *Syntax Design* "Parentheses and braces are operations", including the type body / sequential structure distinction rules); zero new categories;
- **The type body** = a declaration list: the declarations inside are interpreted in order as components of the type value's payload (different from the sequential structure—the product is the type value's payload, not the result of interpreting the last structure);
  - **Field declaration** = `Type fieldName` (separated by `;`—`;` is a connecting operation; see *Syntax Design* "Whitespace is an operation")—corresponding to `String name;`; isomorphic with *Type System Design* "Type construction operations" structural-form types (field name = operation name);
  - **Method declaration** = `[return type] methodName(parameter declarations...) methodBody`—no return = return type omitted (isomorphic with the function type's "no return = return type position omitted"; see *Type System Design* "Type construction operations"); parameter declaration = `Type name` (same direction as field declarations);
  - **Constructor declaration** = `TypeName(parameter declarations...) constructorBody`—no return type, name = the type's name (distinguished from methods); **constructor overloading** is allowed (matched by parameter declarations); body = a structure sequence; field initialization uses `this.fieldName = value;`;
  - **Method body** = a brace sequential structure (see *Syntax Design* "Parentheses and braces are operations"), with structures ending in semicolons: at application, interpreted in order per sequential structure semantics; **a method with a return type: the body must produce via `return v;`** (the body finishes without return → a diagnostic value; see *Semantics Design* "The unified response to operation failure"); a method without a return type: the body needs no return (each structure is interpreted in order; the product is nothing);
- **The interpretation environment of method bodies**: the instance environment = a local environment (isomorphic with the environment construction of parameter reassignment in "Operation construction: lambda" above), with bindings = `this → the receiver instance` + `parameterName → parameterValue`; field access goes through `this.name` (see "Field access" above)—the receiver is explicit; no implicit field visibility is established;
- Connection with the type system: in strong typing mode, field types and method declarations (parameter / return types) are type values; checks happen at instance construction (see "Instance construction and method invocation" below) and method invocation (see "Instance construction and method invocation" below) per operation invocation judgment (see *Type System Design* "What is checked", category 2); fields / method parameters / returns are "bindable positions" (see *Type System Design* "Type positions = bindable positions");
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent");
- Necessity: necessary—types are values; without a form that declares type values carrying fields and methods, nominal types and member operations cannot grow.

**Composition declaration: `Name + ParentType... { ... }`**:

- Form:
  ```
  A {
    String name;
    int id;
  }

  B + A {
    int age;
  }
  ```
- Form: `B + A { ... }`—type name + `+` composition connection + parent type name + type body (multiple composition: `C + A + B { ... }`)—B composes A: **all of A's members (fields and methods) are promoted into B** (`b.name`, `b.show()` are directly usable); composition is a core concept of the fourfold ontology—reuse is composition: inheritance, standalone interface declaration, and standalone composite declaration **are all merged into composition** (no inheritance is introduced; an "interface" is just a type declaration containing bodyless methods; "substitutability" is just a nominal corollary of the composition declaration);
- **Declaration order runs through three places**: member ownership, constructor parameter order, and conflict resolution (one order used in three places);
- **Constructor parameter order** = left to right in the type declaration: own fields in declaration order → uncovered fields of composition components (in composition declaration order)—`B(18, "Jack", 42)` corresponds to B's age → A's name, id;
- **Overriding**: an explicitly declared same-name member in the type body overrides the parent type's same-name member:
  - **Constructor parameters take only sub-structure fields**—an overridden parent field does not occupy a parameter position;
  - An overridden parent component field is maintained by the sub-structure field—**same-type override: the value is shared directly** (no conversion needed); **different-type (conflicting) override: via two-directional conversion methods** (view adapters, provided as needed):
    - **Child → parent**: `Tparent __fieldName(Tchild fieldName) { ... }`—fills the parent component field at construction and syncs when the child field is later reassigned; the parent view (positions of type A) uses it for reads;
    - **Parent → child**: `Tchild __fieldName(Tparent fieldName) { ... }`—the child view uses it for reads: when a **name of a sub-structure type receives a parent structure instance** (`B b = a;`), b's field is converted from the parent instance through it;
    - Method name = `__` + field name; the parameter name equals the field name; parameter / return types distinguish the direction; a missing direction → a diagnostic;
  - **A name of a sub-structure type receiving a parent structure instance** (`B b = a;`): B's promoted members (from parent types) are usable; B's own declared fields have no values (access → a diagnostic in strong typing mode / fall back to Unknown in weak typing mode; see *Semantics Design* "The unified response to operation failure");
  - Same-name, same-type fields from multiple parents: no forced declaration; per composition declaration order, the first declarer takes effect;
  - **Same-name, different-type fields from multiple parents with no sub-type override**: member promotion conflict—the environment cannot construct the type value → a diagnostic value (mode-independent; see *Semantics Design* "The unified response to operation failure", "recognizes but cannot complete the effect"); disambiguation = the sub-type explicitly declares the field (override);
- **Substitutability qualification**: when a parent type contains bodyless methods (interface methods, see below), the composition declaration `Dog + Animal` is simultaneously **a nominal declaration that Dog implements Animal**—after Dog explicitly declares implementations of all bodyless methods, **Dog can be used at positions of type Animal** (parameters / name bindings / returns); parameter position judgment: v's type = T, or v's type composes T by declaration—**directly or transitively along the composition chain** (`C + B`, `B + A` ⇒ C holds for A)—and all of T's bodyless methods have implementations in v's type (implementations may come from member promotion; nominal and statically decidable, not structural inference); runtime method invocation finds the method implementation via the type value carried by the instance (naturally holds); `expect(T)` judgment stays in sync (see *Type System Design* "The expect assertion operation");
- **Bodyless methods**: a method declaration without a body (`String name();`) = an interface method (an interface is a type declaration containing bodyless methods; there is no independent interface concept)—the type cannot be instantiated: strong typing mode rejects construction statically (a diagnostic); weak typing mode does not prevent construction, but invoking a bodyless method → the operation recognizes the method but has no implementation body and cannot complete its effect → a diagnostic value (see *Semantics Design* "The unified response to operation failure");
- Fourfold grounding: a composition declaration is **structure** (the environment's interpretation = construct a composition type value + bind); zero new categories;
- Necessity: a desired capability (member reuse)—composition is an existing core concept; member reuse must have a form.

**Type parameters (generics)**:

- Form:
  ```
  Box<T> {
    T value;
    T get() { return this.value; }
  }
  ```
- Declaration: `Name<T> { ... }`—`<T>` declares a type parameter (T is a type value inside the type body and can occupy field / parameter / return type positions);
- Instantiation = **type values are applicable** (the same mechanism as `Person("Jack")`): `Box(int)`—a name immediately followed by parameter parentheses whose content is a **type value** → produces a new type value (type constructor application); content is an ordinary value → instance construction—distinguished by whether the parameter is a type value or a value;
- Instance construction: `Box(int)(42)`—first instantiate the type, then construct the instance; parameter check conforms to T (`Box(int)("a")` → a diagnostic);
- Nesting holds naturally: `Box(Box(int))`, `Box(int)[]`;
- Fourfold grounding: type parameters are names (values); instantiation = type values are applicable (an environment promise); zero new categories;
- Necessity: a desired capability (type parameters)—the type constructor mechanism provides the landing for this form.

### 8. Instance construction and method invocation: type values are applicable

- **Instance construction**: `Person("Jack")`—the name Person (looked up to obtain the type value) immediately followed by parameter parentheses, **type values are applicable**: a type value applied to values = construct an instance of the type, producing an instance value; **with an explicit constructor → match the constructor's parameter declarations** (no match → a diagnostic value); **without an explicit constructor → parameters correspond to fields in declaration order** (composition types' field order: see the declaration order in "Type declaration" above);
- **The same mechanism as numeric conversion**: type values are applicable—numeric type application = conversion (see *Semantics Design* "Conversion"); structural-type application = instance construction; applying a type value = producing a value of that type;
- Parameter count / types not conforming to the field declarations → the operation cannot complete its effect → a diagnostic value (the unified response in *Semantics Design* "The unified response to operation failure");
- **Method invocation**: `s.at(n)`—the dot connecting operation (see *Syntax Design* "How the text environment interprets these values", item 4): the member-name operation at acts on the instance value s left of the dot, takes the method out via the type value carried by s, and produces a **method operation value bound to the receiver s**; the parameters in the immediately adjacent parenthesized combination `(n)` are applied to that operation value—the receiver is left of the dot (s), and the parameters are in the parentheses (corresponding to Java's s.at(n));
- **Application rules of method operation values**: a method value = (method declaration, implementation body, receiver)—obtained only through `s.at` resolution; the receiver is always s; applying a method value to parameters = interpret the implementation body in the instance environment (instance environment = this + parameter bindings; see "Type declaration" above); parameter type mismatch → a diagnostic value;
- Fourfold grounding: a type value (a value) in application position, and the environment's application interpretation of it (an environment promise)—application is the composition's inherent interpretation (see "Overview of structural forms" above), and "type values are applicable" is the environment's extension promise for that interpretation; no new categories are introduced;
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent");
- Necessity: necessary—type declarations (see "Type declaration" above) produce type values; without instance construction, type values could only appear at declaration positions and could not produce values of their type.

### 9. Failure capture: try / catch, a structural form

- Form:
  ```
  try {
    10 / 0;
  } catch (e) {
    print("failed");
  }
  ```
  —try is **structure** (a Java convention): inside the block is a structure sequence (semicolon-terminated); `catch (e)` is the capture branch (e binds the diagnostic value);
- Fourfold grounding: try is an **operation** (an atomic operation provided by the language environment), acting on (the protected block structure, the capture branch structure); the diagnostic value is a **value** (see *Semantics Design* "The unified response to operation failure") and can be bound;
- **Handling dynamic diagnostics**: the structures in the try block are interpreted in order—a structure in the block fails to interpret (a diagnostic value is produced; see *Semantics Design* "The unified response to operation failure") → the remaining structures of the block are not interpreted, and the diagnostic value is passed to catch: e binds the diagnostic value, and the catch block's structures are interpreted in order; if the whole block interprets successfully → catch is not interpreted;
- **Product: none**—a structural form (a Java convention), appearing in sequential structures / method bodies; no longer used as a value structure;
- **Qualification as a special operation**: selective interpretation (the same family as if; see "Conditional: if" above);
- Connection with the type system: diagnostic values do not participate in type checking (a failure signal is not a type);
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent");
- Necessity: necessary—dynamic interpretation failures (division by zero, out of range, etc.) must be disposable inside structures; otherwise diagnostic values could only propagate all the way to the decision-maker;
- finally can be refined in later versions.

### 10. Return: return, terminating the interpretation of a body

- Form: `return v;`—return is an operation name (a name), and the structure ends with a semicolon (see *Syntax Design* "The text of operation names");
- Fourfold grounding: return is an **operation** (an atomic operation provided by the language environment), acting on the value structure v;
- Interpretation: terminate the interpretation of the current method body / lambda body and produce v's interpretation result—structures after return are not interpreted;
- **Qualification as a special operation**: a terminating interpretation operation (the same family as if / try);
- A method / function with a return type: the body must produce via return (the body finishes without return → a diagnostic value; see *Semantics Design* "The unified response to operation failure"); without a return type: the body needs no return (each structure is interpreted in order; the product is nothing);
- Relation to the sequential structure: the sequential structure interprets in order and produces the last structure's result (see "Sequential structure" above); return appearing in it = early termination and production;
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent");
- Necessity: necessary—methods / functions must be able to explicitly produce a result and terminate early (a Java convention).

### 11. Loops: while / for, repeated interpretation

- Forms: `while (cond) { ... }`; enhanced for: `for (int x : numbers) { ... }` (iterate over array elements);
- Fourfold grounding: while / for are **operations** (atomic operations provided by the language environment), acting on (condition/parameter declaration structure, body structure);
- **Interpretation of while**: repeatedly interpret the condition structure—cond interprets to boolean true → interpret the body, judge cond again, until cond is false; each interpretation of the body happens in the current environment (name reassignments are visible);
- **Interpretation of enhanced for**: acts on (parameter declaration `int x`, array value)—element by element in written order: for each element, establish a local binding (x → that element) and interpret the body;
- **Qualification as special operations**: repeated interpretation (the same family as if / try / return; see "Conditional: if" / "Failure capture" / "Return" above);
- Connection with the type system: while's cond is of type boolean; enhanced for's x type and the array element type are judged per operation invocation (see *Type System Design* "What is checked", category 2);
- Mode-independent (see *Type System Design* "Operation interpretation rules are mode-independent");
- Necessity: necessary—iterating over arrays (with `a[n]` and `a.length`; see *Semantics Design* "Arrays") must be able to repeatedly interpret structures;
- The counting for (`for (int i = 0; i < n; i = i + 1) { }`) can be refined in later versions.

## Design principles (carried over from existing documents)

- Minimal necessity: nothing defined needs no proof; only once something is defined must it prove why it must exist;
- Explicit over implicit;
- Structure preserved, environment responds;
- Operations as adapters: new capabilities grow in the form of environment-provided operations; no independent categories are introduced;
- Every new operation passes the fourfold self-check before discussing its semantics.
