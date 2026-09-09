# Origin Language Engineering Layer Design

## Positioning: The engineering layer is the environment layer where the decision-maker organizes a collection of multi-file source code

- Fourfold grounding: the engineering layer **introduces no new category beyond the fourfold**—a file link is a String value; loading (`load`) is an operation (provided by the environment; see *Type System Design* "Passed-in invocation"); engineering policies (mode / precision) are environment construction parameters (see *Type System Design* "Project-level declaration" / "Atomic type values"); the "entry" is the environment's response to a load request.
- Minimal principle: the minimal skeleton of the engineering layer contains only three substantive items—**entry file designation, the main entry, and load**; the minimal namespace = a global name pool (see "Namespace" below); the minimal project configuration = load-time environment parameters (see "Project configuration" below).
- Do not invent: modules, packages, configuration file syntax, build artifact formats—these are absorbed in future extensions (see "Future extensions" below).

## Entry file

- Designation: **specified by the loading environment**—the launcher (the loading environment) passes in the file link of the project entry; that file is the entry file. The engineering layer does not invent configuration file syntax (an existing convention in *Syntax Design* "Project-level declaration", formally settled in this section).
- What makes the entry file special: it is the only file that can carry a project-level declaration—`mode(P);` appearing in a non-entry file produces a diagnostic (see *Type System Design* "Project-level declaration" / *Syntax Design* "Project-level declaration").
- A project (one launch) has exactly one entry file; other files are loaded via `load(link, ...)` (see "load semantics" below).

## The main entry

- Forms (consistent with *Syntax Design* "Top level"):
  - A function declaration: `main(String[] args) { ... }`—no return = the return type is omitted;
  - Or a lambda binding: `main = (String[] args) -> { ... };`;
- Form constraints (checked under compiled execution): a single parameter, of type `String[]`, with no return type annotation—violations (e.g. `main()` or `int main(int x)`) produce a diagnostic under compiled execution; interpreted execution does not check (main is an ordinary name);
- A lambda-bound main (`main = (String[] args) -> { ... };`) is subject to the same form constraints: the body **must not return a value** (synonymous with the function declaration's "no return")—a body with return produces a diagnostic under compiled execution;
- Parameters: `args` is the **command-line argument sequence** (a String array) passed in by the launcher; no command-line arguments = an empty array;
- Rebinding: multiple top-level bindings of main follow the rebinding rule—the last binding takes effect;
- Position: main is an ordinary top-level binding; it need not be the last top-level value;
- Boundary of specialness: main is special **only in the entry file and under compiled execution**—a main in a non-entry file, or a main under interpreted execution, is always an ordinary name.

## Two execution modes

- The same source code structure can be executed in either mode (**structure preserved, environment responds**)—the execution mode is a choice of the engineering layer (the loading environment) and does not change the source code structure:
  - **Compiled execution**: the entry file's top level is interpreted in order to construct the project environment (mode declarations, type declarations, function declarations, bindings—all declarations take effect), and the main binding is recognized as the entry; after top-level interpretation completes, **`main(command-line argument sequence)` is applied automatically**—main is the program's execution start;
  - **Interpreted execution**: the top level is interpreted in order (see *Syntax Design* "Top level"); main is an ordinary name (an ordinary binding—not special, not automatically invoked); to execute it, apply it explicitly: `main(["a", "b"])`;
- The only difference between the two modes = **whether main is applied automatically after top-level interpretation completes**; the interpretation of each top-level value (including the effects that occurred) is identical;
- Compiled execution fails to find main in the entry file → a diagnostic.

## load semantics

- Fourfold grounding: load is an **operation** (provided by the type system environment; see *Type System Design* "Passed-in invocation"); link is a String value (a file link); the policy is a value; loading = the environment's interpretation;
- Position: load is an ordinary operation, applicable both at the top level and inside structures—one operation, one semantics; no "top-level only" position rule is invented;
- Link resolution: relative links resolve **relative to the location of the file containing that load**; absolute links are used as-is; the link form = a file path string;
- Load semantics:
  1. The environment finds the text pointed to by link, interprets it as a structure, and interprets it under the interpretation policy (priority chain: passed-in at load > script-internal declaration > project-level declaration > default weak typing; see *Type System Design* "Passed-in invocation");
  2. The loaded file's top level is interpreted in order (the same effect as *Syntax Design* "Top level"), and its top-level bindings enter **the same environment binding table** (the global name pool; see "Namespace" below)—after loading completes, the caller can see those bindings;
  3. The product of a load application = the interpretation result of the loaded file's top level (the result of interpreting the last value);
  4. Idempotence: within one project run, the same file is loaded and interpreted only once—a later load resolving to the same file does not re-interpret; it produces the already-loaded product (the minimal termination mechanism for cyclic loading);
- The policy parameter is optional: `load(link, weak)` = pass a policy at load time; `load(link)` = pass no policy—resolution falls through the priority chain to script-internal declaration / project-level declaration / default weak typing (consistent with the priority chain in *Type System Design* "Passed-in invocation");
- Relation to main: a main in a loaded file is an ordinary name (see "The main entry" above); a project-level declaration `mode(P);` in a loaded file produces a diagnostic (see "Entry file" above);
- Load failure semantics: the link cannot be found, or the loaded text fails to interpret → a diagnostic value (see *Semantics Design* "The unified response to operation failure"), propagated upward along the composition hierarchy;
- Boundary: a loaded file obtains its policy via the priority chain (passed-in at load > script-internal declaration > project-level declaration > default weak typing); **when interpreted under the project-level declaration it shares the project's mode, and this does not constitute penetration**; explicitly passing a policy at load, or a script-internal declaration differing from the project's mode, constitutes P11 explicit penetration, and the boundary check back into strong typing goes through `expect(T) { ... }` (see *Type System Design* "The expect assertion operation");
- Implementation extension: continue extending the implementation according to this form and semantic skeleton.

## Namespace (minimal: a global name pool)

- Loaded files share **the same environment binding table**—top-level bindings are globally visible;
- Name collisions follow the rebinding rule (`=` overwrites; no isolation mechanism);
- Modules / name isolation / prefix qualification → future extensions (see "Future extensions" below).

## Project configuration (minimal: load-time environment parameters)

- Project-level parameters such as the int precision policy are **passed in by the loading environment** (isomorphic with "environment sensing" in *Type System Design* "Atomic type values"), following the same priority-chain family as `mode`: passed-in at load > declared inside the project > default;
- The form in which the decision-maker declares project configuration explicitly in source code (configuration file syntax) → future extensions (see "Future extensions" below).

## Future extensions

- The load implementation (continue extending according to the form and semantics of "load semantics" above and *Type System Design* "Passed-in invocation");
- Module systems and namespace isolation;
- Configuration file forms, build / packaging artifacts;
- The diagnostic policy of "declarations only" at the entry file's top level (if stricter compilation discipline is needed, grow it as a diagnostic policy; it does not affect the minimal semantics).
