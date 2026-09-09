# Origin Language Comprehensive Examples

> Positioning: acceptance samples written in the settled syntax—complete programs covering all features. Each example comes with a "walkthrough record": the covered points and the problems found during the walkthrough.
>
> Syntax baseline: `int x = 1;`, `Person { ... }`, `B + A { ... }`, `Box<T> { ... }`, constructors, `f(args)`, infix operations, `[1, 2, 3]`, `a[n]` / `a.length`, `s.at(n)` / `s.length`, `if / while / for / try-catch / return`, `mode` / `expect` / `load`, `main`.

## Example 1: Composition and substitutability (interfaces + substitutability)

```
mode(strong);

Animal {
  String name;
  String speak();
}

Dog + Animal {
  String speak() { return "dog " + this.name; }
}

Cat + Animal {
  String speak() { return "cat " + this.name; }
}

run(Animal a) {
  print(a.speak());
}

main(String[] args) {
  run(Dog("wang"));
  run(Cat("miao"));
}
```

**Walkthrough record**:
- `Animal` contains the bodyless method `speak();` → an interface method; not instantiable under strong typing (it is not constructed) ✓;
- `Dog + Animal` = composition reuse + a nominal declaration that Dog implements Animal (speak has an implementation) → substitutability qualification ✓;
- `run(Dog("wang"))`: Dog used at an Animal position → statically judged (composition declaration + all bodyless methods implemented) ✓; at runtime `a.speak()` finds the Dog implementation via the instance's type value ✓;
- `Dog("wang")`: Dog has no explicit constructor → field-order construction; parameters correspond to fields in declaration order—Dog declares no fields of its own; the `name` promoted from Animal is its field, and the parameter `"wang"` corresponds to `name` ✓;
- `"dog " + this.name`: String + String concatenation ✓.

## Example 2: Generics + arrays + branching/loops + failure capture

```
mode(strong);

Box<T> {
  T value;
  T get() { return this.value; }
}

int sum(int[] a) {
  int s = 0;
  for (int x : a) {
    s = s + x;
  }
  return s;
}

main(String[] args) {
  int[] nums = [1, 2, 3, 4];
  print(sum(nums));

  Box(int) b = Box(int)(42);
  print(b.get());

  Box(String) s = Box(String)("hi");
  print(s.get());

  try {
    print(nums[10]);
  } catch (e) {
    print("out of bounds");
  }

  int i = 0;
  while (i < nums.length) {
    print(nums[i]);
    i = i + 1;
  }
}
```

**Walkthrough record**:
- `Box(int)(42)`: type instantiation + instance construction (a type value as the parameter → type construction; an ordinary value → instance construction) ✓;
- Enhanced for + while + element access `nums[i]` + `nums.length` ✓;
- `nums[10]` out of range → a diagnostic value → captured by the try/catch structure (structures inside the block are semicolon-terminated) ✓;
- `print(nums)` outputs the whole array (print holds for any value; presentation is decided by the environment) ✓.

## Example 3: Multiple composition + constructor overloading + mode regions

```
mode(strong);

A {
  int id;
  int show() { return this.id; }
}

B {
  String name;
  String label() { return this.name; }
}

C + A + B {
  int id;
  String name;
  C(int id, String name) {
    this.id = id;
    this.name = name;
  }
  C(int id) {
    this.id = id;
    this.name = "";
  }
}

C c1 = C(1, "one");
C c2 = C(2);
print(c1.show());
print(c2.label());

mode(weak) {
  print(c1);
}

main(String[] args) {
  print("done");
}
```

**Walkthrough record**:
- `C + A + B` multiple composition: member promotion (show / label usable) ✓;
- Overriding same-type fields (id, name)—values shared directly; constructor parameters take only sub-structure fields ✓;
- Constructor overloading matched by parameter declarations (`C(1, "one")` / `C(2)`) ✓; with explicit constructors present, field-order construction no longer applies ✓;
- `c1.show()`: the promoted method body `this.id` resolves in the merged member table to C's overridden id (int) ✓, types consistent;
- `mode(weak) { ... }` region-level declaration (a weak-typing region under a strong-typed project) ✓; `print(c1)` weak-typing presentation ✓.

## Example 4: A multi-file project (load + the main entry + two execution modes)

The project consists of three files; the entry file is designated by the loading environment (see *Engineering Layer Design* "Entry file"):

`app.o` (the entry file):
```
mode(strong);
load("math.o");
load("greet.o");

main(String[] args) {
  int n = int(args[0]);
  String name = args[1];
  print(add(n, 10));
  print(greet(name));
}
```

`math.o`:
```
int add(int a, int b) {
  return a + b;
}
```

`greet.o`:
```
String greet(String name) {
  return "hello " + name;
}
```

**Walkthrough record**:
- `mode(strong);` = the project-level declaration as the first value at the entry file's top level; the whole project is strong-typed (see *Type System Design* "Project-level declaration") ✓;
- `load("math.o")` single-parameter form: the relative link resolves relative to the location of the file containing that load (app.o) → the same directory ✓; no policy passed → the priority chain falls through to script-internal declaration / project-level declaration (math.o / greet.o have no internal declarations → interpreted under the project-level declaration strong)—the whole project is interpreted under strong typing mode, consistent with "Project-level declaration"'s "other files do not repeat the declaration"; interpretation under the project-level declaration does not constitute penetration, and main's direct calls to add / greet hold under strong typing checking ✓;
- The loaded files' top levels are interpreted in order: `add` / `greet` bindings enter the global binding table (see *Engineering Layer Design* "Namespace") → directly callable in main ✓;
- Idempotence: within one run, the same file is loaded and interpreted only once—a second `load("math.o")` inside main does not re-interpret (no repeated effects; see *Engineering Layer Design* "load semantics") ✓;
- Compiled execution: after top-level interpretation completes (loading + declarations), `main(command-line argument sequence)` is applied automatically—`args[0]` / `args[1]` are String array elements passed in by the launcher; `int(args[0])` parses an int from a String (see *Semantics Design* "Conversion") ✓;
- Interpreted execution: not applied automatically; main is an ordinary name—executing it = explicit `main(["5", "Jack"])` (see *Engineering Layer Design* "Two execution modes") ✓;
- Position constraints: a loaded file containing `mode(P);` → a diagnostic (a non-entry file)—the loaded files in this example have none ✓; a main in a loaded file (if any) = an ordinary name ✓;
- The `load(link, weak)` passed-in policy form and the expect boundary are in *Type System Design* "Passed-in invocation" / "The expect assertion operation" (not used in this example).
