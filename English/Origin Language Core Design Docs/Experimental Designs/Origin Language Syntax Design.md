# Origin Language Syntax Design

> Positioning: the syntax layer = **how text is interpreted by the environment as structure**. Structural forms are defined by *Language Structure Design*; this layer defines only the textual forms.

## Positioning and scope

- Syntax is the interpretation of "text → structure", provided by the text environment—the environment decides how structures are interpreted (see *Origin Language Core Abstract Design*, the "Environment" chapter); structural forms (see *Language Structure Design* "Overview of structural forms") are upstream; this layer does not change structures, it only defines text;
- Syntax is **not unique**: the same structure can have multiple textual forms (growing on demand in the future); this layer first settles a minimal set;
- Connections with existing documents:
  - Textual forms of composition, name reference, and atomic values—corresponding to *Language Structure Design* "Overview of structural forms";
  - Textual symbols of operation names (`=` / if, etc.)—corresponding to *Language Structure Design* "The minimal operation set";
  - Textual forms of mode declarations and expect—landing *Type System Design* "The two-level forms of mode declaration" / "The expect assertion operation" (type annotation is retired—types are written directly at declaration positions; see *Type System Design* "The structural-layer skeleton of type positions");
  - The textual form of project-level declarations—*Type System Design* "Project-level declaration" (`mode(strong);`).

## Lexics: values and operations in the text environment

> General principle: everything in lexics falls within the fourfold ontology—no new concept like "rule" is introduced. Everything here is either a **value**, an **operation** (provided by the environment; operations are also values), or an **environment interpretation** (the environment decides how values are interpreted; see *Origin Language Core Abstract Design*, the "Environment" chapter).

### 1. Whitespace is an operation: connecting (boundary establishment)

- The source text is a string of characters; every character is a value;
- **Whitespace is an operation** (operations are also values): the environment interprets it as "connecting"—it acts on the text on its left and right, establishes the boundary between them, and makes them two distinguishable values;
- An atomic value in the text environment = a continuous run of characters established by a boundary (that value is indivisible within this environment);
- Newlines, tabs, and `;` are other textual forms of the same connecting operation, for multi-line writing and explicit separation; a connecting operation with no text on one side (line start / line end) has no effect;
- `,` is the **list-separating** textual form of the connecting operation: it separates elements of the same level—array text `[1, 2, 3]`, parameter lists `(int a, int b)`, arguments `f(a, b)`;
- **Comments**: text between `//` and the end of line, or between `/*` and `*/`, is interpreted by the environment as producing no value (comments are text, not structure)—`//` and `/* ... */` are comment-delimiting operations (a Java convention);

### 2. Parentheses and braces are operations: delimiting

- `(` is an operation whose effect is to open a composition; `)` is an operation whose effect is to close a composition;
- The parenthesis operations act on the content between them, making the content a composition—nesting follows: the content of a composition can be a composition;
- Three roles of parentheses: ① prefix composition—content is an "operation name + parameters" sequence, synonymous with the `f(args)` application form (syntax is not unique; see "Positioning and scope" above); ② **grouping**—when the content is an infix structure (e.g. `(x + y)`), that structure is fixed as one value participating in the outer infix sequence (see "How the text environment interprets these values" below, item 6); ③ **parameter parentheses**—parentheses immediately after a value / name (`f(a)`, `Person("Jack")`, `s.at(n)`); the parenthesized content is the application's parameters;
- `{` is an operation whose effect is to open a sequential structure; `}` is an operation whose effect is to close a sequential structure;
- `[` is an operation whose effect is to open array text; `]` is an operation whose effect is to close array text—the values between (separated by `,`) are aggregated into an array value (see *Semantics Design* "Arrays");
- The brace operations act on the content between them: the values inside are interpreted in order—the textual landing of sequential structure (see *Language Structure Design* "Sequential structure"; the block operation name is retired); braces can nest: the content of a sequential structure can be a sequential structure;
- **Type body distinction**: for a brace-delimited structure whose immediately preceding value is a name, the environment first looks up that name's binding—**unbound → type declaration** (type name + type body; see *Language Structure Design* "Type declaration"); **bound to an operation value → body-carrying application** (the operation acts on the brace body, e.g. `try { ... }`; see *Language Structure Design* "Failure capture"); when braces do not directly follow a name, it is still a sequential structure;
- **Composition declaration form**: `Name + Name { body }`—`+` in the type declaration form = composition connection (the same operation name as arithmetic `+`; the environment interpretation differs at declaration positions; see *Language Structure Design* "Type declaration"); multiple composition: `C + A + B { ... }`;
- **Type parameter form**: `Name<T> { body }`—`<T>` in the type declaration form = type parameter declaration (see *Language Structure Design* "Type declaration", type parameters); distinguished from comparison infixes by position (declaration form vs value position);
- **Type instantiation**: a type name immediately followed by parentheses whose content is a type value (`Box(int)`) = type constructor application (produces a type value); content is an ordinary value (`Person("Jack")`) = instance construction (see *Language Structure Design* "Instance construction and method invocation");
- **Conversion method form**: `Tparent __fieldName(Tchild fieldName) { ... }` and `Tchild __fieldName(Tparent fieldName) { ... }`—two-directional view adapters: the method name is `__` + the field name, the parameter name equals the field name, and parameter / return types distinguish the direction (covered in *Language Structure Design* "Type declaration");
- **The "operation name (parameters) { body }" form**: a name immediately followed by parameter parentheses and then braces (`if (cond) { ... }`, see *Language Structure Design* "Conditional: if"; `while (cond) { ... }`, see *Language Structure Design* "Loops"; `mode(strong) { ... }`, see *Type System Design* "Region-level declaration"; `expect(T) { ... }`, see *Type System Design* "The expect assertion operation")—the environment interpretation = that operation acts on (parameters, brace body);
- **Enhanced for form**: `for (Type name : arrayValue) { body }`—the parenthesized content of for = a parameter declaration + `:` + an array value (see *Language Structure Design* "Loops");
- **Function declaration distinction**: in the `[return type] Name(parameter declarations...) { body }` form, when the parenthesized content starts with a type value immediately followed by a name (e.g. `int a`) → **function declaration** (declaration is binding; see *Language Structure Design* "Operation construction: lambda"); otherwise → body-carrying application;
- **Constructor declaration distinction**: inside a type body, `Name(parameter declarations...) { body }` with **Name = the type's name** (no return type) → **constructor declaration** (overloadable; see *Language Structure Design* "Type declaration"); Name ≠ the type's name → method declaration (no return type);

### 3. How the text environment interprets these values

Once the above values are established by boundaries, the text environment interprets them as follows:

1. Values starting with a digit: interpreted as an integer (digit sequence) or a float (digit sequence + one decimal point + digit sequence); the type attribution of integers and floats (int / double) is interpreted by the type system environment (see *Type System Design* "Atomic type values");
2. Quote-delimited values: quotes are **delimiting operations** isomorphic with parentheses (opening / closing); their content is interpreted as a string;
3. Other values: interpreted as **names**; the environment's response = binding lookup (see *Language Structure Design* "Overview of structural forms")—except `true` / `false` (item 8);
4. The dot (`.`) is a **connecting operation** (operations are also values): it acts on the value structure on its left and the member structure on its right, expressing the "value → inner member" connectedness; within values starting with a digit, the decimal point remains part of the float form (item 1);
   - `v.name` (member name with no immediately adjacent parenthesized combination) → the environment's interpretation = **field access**: the field-name operation acts on the value v, producing the field's value (see *Language Structure Design* "Field access");
   - `v.name(args)` (member name with an immediately adjacent parenthesized combination) → the environment's interpretation = **method invocation**: the member-name operation acts on the value v, takes the method out via the type value carried by v, binds the receiver, and applies it to the parameters in the parenthesized combination (see *Language Structure Design* "Instance construction and method invocation");
   - "Immediately adjacent" = no connecting operation between the member name right of the dot and the parenthesized combination (no whitespace / `;` / newline; see "Whitespace is an operation" above); with a connecting operation, the two are two values and do not form a member invocation;
   - Members **exist through values** and do not enter the global binding table (writing `at` alone finds no binding).
5. `=` is a **binding operation name** (a symbolic name): the binding structure form `[Type] name = valueStructure;` (the type may be omitted)—`;` is a connecting operation (see "Whitespace is an operation" above): a structure ends with `;`, multiple structures are separated and connected by `;` (the final structure's `;` has no text on one side); the environment's interpretation = the binding operation acts on (name, type value (if given), the interpretation result of the value structure), binding the name to that value (see *Language Structure Design* "Binding"); `=` and the comparison operation `==` are two names (two bindings), unrelated to each other.
6. Infix operations and interpretation order: the symbolic operation names `+` `-` `*` `/` `==` `<` `>` are **infix connecting operations**—they act on the value structures on both sides (`x + y`, `a == b`); the environment's interpretation order for infix sequences (highest to lowest): ① `.` member access (including immediately adjacent parameter parentheses); ② `*` `/`; ③ `+` `-`; ④ `<` `>`; ⑤ `==`; ⑥ `=` binding; ⑦ `->` lambda arrow; same level left to right; parenthesis grouping (see "Parentheses and braces are operations" above, ②) changes the interpretation order; the lambda form of `->` = `(parameter list) -> body` (see *Language Structure Design* "Operation construction: lambda").
7. `[]` is a **type construction operation name** (postfix form): a type value immediately followed by `[]` (e.g. `int[]`) = an array type—the environment's interpretation = the type construction operation acts on the type value, producing an array type value (the homogeneous-repetition form; see *Type System Design* "Type construction operations"); distinguished from array text: `[` before (`[1, 2, 3]`) = an array value; `[]` after (`int[]`) = an array type; a type value followed by `[N]` (`int[10]`) = an array type + a **capacity hint** N (see *Semantics Design* "Arrays"; the hint may be omitted);
8. `true` / `false`: the environment interprets them directly as **boolean atomic values** (direct textual interpretation, no name binding lookup)—the same family as number text (item 1);
9. **Array element access**: `v[n]`—an array value in value position immediately followed by square brackets = take the n-th element in written order (n starts at 0; see *Semantics Design* "Arrays"); an array value followed by `.length` = the length field (element count); the distinction from the capacity hint uses **position** as the main criterion: **`T[N]` at a declaration position (before `=`) = capacity hint; `v[n]` at a value position = element access**.

### 4. Why there is no "rule"

"Rule" is not one of the fourfold. All of the above falls within the fourfold: characters are **values**, and boundary-established text is also **values**; whitespace, commas, parentheses, braces, square brackets, quotes, the dot, `=`, and the infix symbols are **operations**; the interpretation of these values is **the text environment's interpretation**. Introducing "rule" as an independent concept has no necessity, so it is not introduced.

## From text to structure: the interpretation

### 1. The text landing of composition and application

- `(` opens, `)` closes; the values between parentheses form a composition;
- Composition interpretation (if the value at the first position, after environment interpretation, is an operation value, it acts on the values at the remaining positions) is already defined in *Language Structure Design* "Overview of structural forms";
- **Application form**: parentheses immediately after a value / name = application—`f(a, b)` (operation value application), `Person("Jack")` (type value application = construction / conversion), `s.at(n)` (member method); `f(args)` and `(f args)` are two textual forms of the same application structure (syntax is not unique; see "Positioning and scope" above);
- **Grouping**: when the parenthesized content is an infix structure (e.g. `(x + y)`), that structure is fixed as one value participating in the outer infix sequence (see "How the text environment interprets these values" above, item 6);
- Nesting: values between parentheses may contain compositions delimited by parentheses; the environment's interpretation proceeds recursively.

### 2. The text of operation names

- `=` (binding), if / try / catch / while / for / mode / expect / load / return are all **names** (see "How the text environment interprets these values" above, item 3); the environment obtains the operation value through binding lookup; lambda and block are retired—lambda = the `(parameter list) -> body` arrow form (same as above, item 6); sequential = the brace structure (see "Parentheses and braces are operations" above);
- Member access (field `p.name`, method invocation `s.at(n)`) resolves through the **dot connecting operation** (same as above, item 4): the name left of the dot is looked up to obtain the instance value; the member name right of the dot takes out the field, or takes the method out via the type value carried by the instance, binds the receiver, and applies it to the parameters in the immediately adjacent parenthesized combination (see *Language Structure Design* "Field access" / "Instance construction and method invocation");
- There are no "keywords": operation names are no different from ordinary names—`else` is also a name (the false-branch connection of if; see *Language Structure Design* "Conditional: if").

### 3. The text of mode declarations

- Project-level: `mode(strong);`—application form (a name immediately followed by parameter parentheses) + semicolon, as the first value at the entry file's top level (see "Project-level declaration" below);
- Region-level: `mode(strong) { ... }`—the "operation name (parameters) { body }" form (see "Parentheses and braces are operations" above);
- expect: `expect(T) { ... }`—the same-family form (see "Parentheses and braces are operations" above);
- Passed-in at load: `load(link, weak)`—application form, loading an external script and passing in the interpretation mode (see *Type System Design* "Passed-in invocation"; this mechanism can be extended in later implementations).

### 4. Top level: the whole text = a sequence of values

- The whole text is interpreted by the environment as a sequence of values adjacent in order; the environment **interprets them in order**, producing the interpretation result of the last value;
- The top-level in-order interpretation and the sequential structure (see *Language Structure Design* "Sequential structure") are two landings of the same effect (ordering): the sequential structure is the explicit form inside a structure; top-level in-order interpretation is the text environment's interpretation of the whole text;
- A value failing to interpret during in-order interpretation: the diagnostic reaches the decision-maker (the top level has no higher environment to pass it to)—failure semantics: see "Failure semantics" below;
- **Entry (main)**: the top-level-bound name `main` (an operation value) is the program's entry convention—its form may be a function declaration `main(String[] args) { ... }` (no return = return type omitted) or a lambda binding `main = (String[] args) -> { ... };` (the lambda form likewise requires the body to have no return; see *Engineering Layer Design* "The main entry")—**compiled execution**: after top-level interpretation completes, the environment recognizes main and **applies it automatically** (parameters = the command-line argument sequence)—main is the program's execution start; **interpreted execution**: the top level is interpreted in order, and main is an ordinary name (an ordinary binding—not special, not automatically applied); the two execution modes are chosen by the engineering layer, and the only difference = whether main is applied automatically after top-level interpretation completes (see *Engineering Layer Design* "Two execution modes"); the source code structure is unchanged (**structure preserved, environment responds**).

### 5. Project-level declaration

- Form: `mode(P);`—application form + semicolon, located as **the first value at the entry file's top level**;
- Distinguished from the region-level declaration: a mode with a brace body = region-level declaration (`mode(P) { ... }`; see *Type System Design* "Region-level declaration"); a mode with a single parameter and a semicolon = project-level declaration—the two are distinguished by form; no extra convention is needed;
- Interpretation: when the environment interprets the top-level values in order, if the first value is `mode(P);`, it interprets it as a project-level declaration (environment construction; produces no value); once the policy takes effect, subsequent values are interpreted under that policy (P12: it cannot be switched during a run);
- Default: no declaration = weak typing mode (see *Type System Design* P4);
- Position constraint: a project-level declaration in a non-entry file or at a non-first position—produces a diagnostic; the "entry file" is designated by the loading environment (see *Engineering Layer Design* "Entry file");
- A project-level declaration produces no value: the product of top-level in-order interpretation = the interpretation result of the last value.

## Failure semantics

- For text it cannot interpret, the text environment produces a **diagnostic value**: a diagnostic is the environment's feedback to the decision-maker (the composition / operation / value threefold view; see *Type System Design* "Rejection and diagnostics"), not exclusive to the type system;
- Cases that cannot be interpreted: a value starting with a digit that does not fit the integer / float form (e.g. `123abc`); unbalanced parentheses; unbalanced quotes;
- Mode-independent: syntax failure occurs during the text → structure interpretation stage—the structure has not yet formed and the type system has not yet stepped in; behavior is identical in weak/strong typing modes;
- A value failing to interpret during top-level in-order interpretation: the diagnostic reaches the decision-maker (the top level has no higher environment to pass it to; see "Top level" above).

## Design principles (carried over from existing documents)

- Structure preserved, environment responds: syntax is only responsible for the interpretation of text and does not change structural semantics;
- Minimal necessity: do not introduce forms absent from the structure layer; extra textual forms are desired capabilities, not part of the minimal skeleton;
- Do not introduce concepts outside the fourfold: everything in lexics and syntax is either a value, an operation (provided by the environment), or an environment interpretation (the environment decides how values are interpreted)—"rule" is not an independent concept;
- Every new value / operation passes the fourfold self-check.
