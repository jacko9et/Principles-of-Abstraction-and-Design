# Origin Language: Implementer's Guide

> Design goal: turn the core abstract design into implementable environment design principles, helping implementers decide atomic values, atomic operations, interpretation rules, and boundary behavior.

## Document guide

- This document answers "how to use it": how to construct a concrete environment based on the `Origin` abstraction.
- It does not prescribe one unique standard answer, but provides the key questions an implementer must face and reasonable design directions.
- The core requirement is: keep the core abstraction relations undistorted while allowing different environments to interpret the same structure in different ways.

## Foreword

This document is the companion guide to *Origin Language Core Abstract Design*.

The core abstract design document defines `Origin`'s **"what"**—it describes a sufficiently small core abstraction and explains the relations among values, operations, environments, compositions, and source code.

This guide tries to answer **"how to use it"**—if you are to construct a concrete environment based on `Origin`'s abstraction (a compiler, an interpreter, a virtual machine, a DSL, or a runtime middle layer), what practical problems you face and how the `Origin` core expects you to handle them.

Like the core abstract design, this guide offers no "standard answers". It will not dictate whether you must use an LR(1) parser or PEG, your bytecode format, or your memory model. It does only two things:

1. **Remind you which problems an implementer must face;**
2. **Provide reasonable handling directions within the Origin abstraction framework, for you to choose or borrow.**

> Every "direction" is only a suggestion, not a constraint. If your environment has sufficient reasons to deviate from a suggestion, that is a reasonable design choice as long as the core abstraction relations are respected.

## From abstraction to concreteness: the first choice

When you decide to construct an environment based on `Origin`, the first problem you face is not "how to write code", but:

**What will my environment take as its atomic starting point?**

The "values" and "operations" in the core abstract design are descriptions at the philosophical level. In a concrete implementation, you must give them a landing.

### 1. Optional landing directions

- **The physical direction**: take CPU instruction sets, memory addressing, and system calls as the atomic starting point. Suitable for builders of operating systems and embedded runtimes.
- **The logical direction**: take a host language's object model, function calling conventions, and exception handling mechanisms as the atomic starting point. Suitable for implementers constructing DSLs or interpreters on top of an existing runtime.
- **The hybrid direction**: take a set of operations on external services (databases, networks, file systems) as the atomic starting point. Suitable for building domain-specific distributed computing environments.

Whichever direction you choose, `Origin` asks only one thing of you:

> **In your environment, every atomic operation must have a clear boundary of "can take effect". The environment cannot provide vague basic actions whose behavior depends on external guessing.**

### 2. Details worth thinking about

- How are your atomic values distinguished? Memory addresses? Handles? URIs? Hashes of immutable content?
- After an atomic operation acts on values, how are side effects perceived or isolated?
- If an atomic operation fails in its underlying environment (say a system call returns an error), how do you map that reality back into `Origin`'s abstraction relations?

## Concretizing "structure"

In the core abstract design, source code is defined as the carrier that "describes composition relations", and "composition" itself is a relational structure.

When implementing, you **must** choose a concrete representation for this relational structure. This is one of the most important decisions an implementer makes.

### 1. Common representation forms

- **S-expressions / AST**: the structure appears as a nested tree. The advantage is human readability and easy traversal. The disadvantage is that a tree is not direct enough for expressing cross-references (such as GOTO and control flow graphs).
- **Directed graphs / data flow graphs**: the structure appears as nodes (values/operations) and edges (data flow or control dependencies). The advantage is stronger expressiveness, suitable for compiler back-end optimization. The disadvantage is that the description form is more complex and unfriendly to humans writing source code; a front-end syntax tree is usually converted into this form.
- **Text lines + conventional indentation**: the structure is implied by whitespace and newlines. The advantage is extreme minimality. The disadvantage is that parsing rules easily become ambiguous and demand more of the environment's interpretive ability.

### 2. Trade-offs in choosing

- If the goal is for **humans to write source code directly**, the structural representation should be as clear, consistent, and easy to learn as possible.
- If the goal is for **machines or toolchains (IDEs, analyzers)** to process source code, the structural representation should have a clear formal definition (enumerable node types, definite nesting rules).
- One environment can accept multiple structural representations (e.g. supporting both binary AST and readable text). `Origin` does not forbid it, but the implementer must handle the mapping between them well.

> The Origin core does not prescribe the representation of structures, but it requires: **the same structural description, when interpreted repeatedly in the current environment, should produce consistent results** (unless the environment itself has undergone intentional, explainable changes over time). This is an implicit "determinism" requirement.

## The positioning of Unknown: your attitude as an environment

In the core abstract design, `Unknown` is defined as the "primordial abstraction mark". In concrete implementations it appears in many scenarios:

- As a placeholder for **null values / empty types**;
- As a mark when **type information is missing** and the environment guesses or postpones type checking;
- As an anchor in **gradual type systems** for the transition from "dynamic" to "static";
- As a safety valve when **data crosses environments** but its format or semantics cannot be determined.

As an environment implementer, you need not give `Unknown` the same handling in all scenarios. But `Origin` expects you to do one of the following two things:

1. **Clarify your environment's interpretation policy for `Unknown`** and make that policy part of the environment's behavior, so the decision-maker (the programmer) can anticipate it;
2. **Treat `Unknown` as a run-time error trigger**—that is, if in the current environment an `Unknown` cannot be deterministically interpreted as some operable value, the environment should report clearly rather than silently convert it into an arbitrary value.

> The second strategy is the safer one. If you choose the first strategy (giving `Unknown` a specific semantics), make sure the grant is traceable and explicitly declared by the environment.

## Design considerations for penetration

Penetration is the most "powerful" relation in `Origin`. It allows other environments' rules to enter the current environment.

The core abstract design only defines penetration's **existence** and **control ownership** (decided by the environment currently interpreting the structure). In a concrete implementation, you need to supply the following information:

### 1. The effective scope of penetration

- **Global penetration**: the environment allows other environments' rules to enter when interpreting all structures. The advantage is convenience; the disadvantage is blurred security boundaries.
- **Explicitly marked penetration**: the source code uses a specific marker (such as a keyword or special syntax) to declare "penetration is allowed here". The advantage is auditability; the disadvantage is the need for extra syntax conventions.
- **Type-driven penetration**: when a value's type is undefined in the current environment but defined in other environments, penetration triggers automatically. The advantage is smoothness; the disadvantage is excessive transparency—the decision-maker may be unable to anticipate it.

### 2. Behavior when penetration fails

When the penetrated environment cannot interpret the current structure (for example, that environment lacks the corresponding atomic operation), how should the current environment respond?

- Fall back to the current environment's own interpretation path and give a warning;
- Report an error directly, asking the decision-maker to modify the source code or re-select an environment;
- Attempt relay interpretation through a third environment (if one exists).

> The Origin core does not prescribe a failure policy, but suggests: **when penetration fails, the environment should give clear information locatable to the concrete structure and the concrete source rule**, rather than silent failure or vague errors.

## Capability growth: designing extensible atomic rules

One of `Origin`'s core promises is that "capabilities can grow out of the relations between environments and structures". This statement is self-consistent at the abstract design level, but at the implementation level you must actively design "growable" mechanisms.

### 1. Suggested design patterns

- **Modular atomic rule registration**: treat atomic operations as registrable units. The environment loads a set of core atomic rules at startup; later, new atomic rules can be injected through source code construction or external configuration. In this way, "growth" becomes a continuation of "registration" rather than a modification of the environment itself.
- **Rule priority chains**: when the same structure can be interpreted by multiple atomic rules (for example, the same `+` can be integer addition, floating-point addition, or string concatenation), the environment needs to define a priority or disambiguation policy. The priority chain itself can be adjusted dynamically as "capabilities grow".
- **Meta atomic operations**: provide a set of operations on the atomic rules themselves (for example, "query which atomic operations the current environment has registered", "replace one atomic operation with another"). This gives the environment the ability of "self-examination" and is the foundation for the self-hosting path.

> The introduction of meta atomic operations should be restrained. Introduce them only when your environment needs to support the explicit requirement of "modifying its own interpretation rules at run time". Otherwise, keeping atomic rules static or semi-static makes the environment more stable and easier to verify.

## Errors and boundary cases

The core abstract design does not define error handling, because errors are the environment's responsibility. But in a concrete implementation, errors are unavoidable.

### 1. Boundary cases that should be handled at minimum

- **Operating on a value that does not exist**: if the environment cannot locate the value the decision-maker specified, how should the environment inform the decision-maker?
- **Ambiguous structures**: if the same source code structure has more than one legal interpretation in the current environment (for example, `a - b - c` could be left-associative or understood as right-associative), the environment needs to choose one interpretation path or ask the decision-maker for disambiguation information.
- **Resource exhaustion**: if the environment's internal resources (memory, file handles, network connections) are exhausted and an atomic operation cannot complete, how much context should the environment preserve for the decision-maker to locate the problem?

### 2. A direction worth considering

Treat errors as **a special value the environment returns to the decision-maker**, rather than an "environment crash" or "undefined behavior". This special value can be captured or passed by higher-level operations in the structure, thereby bringing error handling into the description range of composition relations rather than excluding it.

## Commitments of the document

Finally, `Origin` is not a mandatory standard. If your environment adopts some suggestions from this guide and deviates from others, it is a qualified `Origin` implementation as long as it can clearly answer the following three questions:

1. **What are your atomic values and atomic operations?**—The decision-maker should be able to know before starting to write source code.
2. **How does your environment interpret composition relations?**—Given a structure, the behavior the environment produces should be predictable, or at least traceable.
3. **How do you handle environment boundaries (penetration, selection/switching)?**—The decision-maker needs to know under what conditions their source code will be accepted by the current environment, and under what conditions it will be rejected or produce unpredictable behavior.

The document that answers these three questions is your environment's own "implementer's guide".

## Conclusion

This guide cannot exhaust the needs of all concrete environments. It exists so that you feel less lost and have more structured directions of thought on the way from "core abstract design" to "concrete implementation".

If, during implementation, you find an insurmountable obstacle in `Origin`'s core abstract design itself—that is not your fault; it means the core abstract design needs to be supplemented or corrected. The origin can move, but the motive for moving must be real-world need, not theoretical deduction.

May your environment grow into what it should be on top of `Origin`.

# Appendix: a minimal example environment

This appendix presents a tiny example environment constructed from `Origin`'s abstract concepts. Its purpose is not to serve as `Origin`'s reference implementation or recommended paradigm, but to demonstrate **a mapping path from abstraction to concreteness**—that is, how the core concepts of the core abstract design (values, operations, environments, compositions, source code) map onto real code.

Implementers may reference this path but need not follow it at all. Your environment may use any language, any data structure, and any interpretation strategy to realize its own design choices.

> This example uses JavaScript, because it is conceptually simple enough to run and understand without extra tooling. If you prefer another language, the same structure can easily be migrated.

## Environment definition

`MiniEnv` is a minimal environment. It contains two parts:
- **An atomic operation registry** (`ops`): a set of callable functions.
- **An interpreter** (`evaluate`): receives structures, interprets recursively, and returns results.

```javascript
// ============================================================
// 0. Public anchor definition (shareable across environments)
// ============================================================

// Unknown is the primordial abstraction mark of values.
// Any environment constructed on Origin can recognize it.
// It carries no semantics itself, but as an anchor it can keep
// carrying uninterpretable structures in a distinguishable,
// passable form.
const Unknown = {};

// ============================================================
// 1. Atomic operation registry (the core of the environment)
// ============================================================

const MiniEnv = {
  ops: {
    // Addition: adds two operands
    add: (a, b) => a + b,
    // Subtraction: subtracts two operands
    sub: (a, b) => a - b,
    // Conditional: if the first operand is truthy, return the second, otherwise the third
    if: (cond, thenVal, elseVal) => cond ? thenVal : elseVal,
    // Print: outputs the value and returns it (a side-effecting operation)
    print: (val) => { console.log(val); return val; }
  },

  // ============================================================
  // 2. Interpreter: turns structures into actions
  // ============================================================
  evaluate: function(struct) {
    // If the structure is not an array, it is an atomic value (number, string, etc.)
    if (!Array.isArray(struct)) {
      return struct;
    }

    // The structure is an array: [operation name, argument 1, argument 2, ...]
    const [opName, ...args] = struct;

    // Interpret all arguments recursively (arguments are evaluated first)
    const evaluatedArgs = args.map(arg => this.evaluate(arg));

    // Look up the atomic operation
    const operation = this.ops[opName];
    if (typeof operation === 'function') {
      // The operation takes effect (the core: the action executes here)
      return operation(...evaluatedArgs);
    } else {
      // Return a structure carrying the Unknown mark
      // Unknown is a public anchor that any Origin environment can recognize
      return {
        anchor: Unknown,   // the public anchor
        op: opName,         // context information (optional)
        args: evaluatedArgs // context information (optional)
      };
    }
  }
};
```

## Usage examples

Example 1: simple addition

```javascript
// Source code structure: describes relations, carries no execution rules
const source = [
  'add', 1, 2
];

// The environment interprets the structure
const result = MiniEnv.evaluate(source);
console.log(result); // output: 3
```

Example 2: nested composition (conditional + side effect)

```javascript
const source2 = [
  'if',
  ['add', 1, 2],    // condition: 3 (truthy)
  ['print', 'yes'], // then: prints "yes"
  ['print', 'no']   // else: prints "no"
];

MiniEnv.evaluate(source2);
// output: yes
// returns: "yes"
```

Example 3: undefined operation → Unknown

```javascript
const source3 = [
  'multiply', 3, 4
];

const result3 = MiniEnv.evaluate(source3);
console.log(result3);
// output: { anchor: {}, op: 'multiply', args: [3, 4] }

// Check whether it is Unknown
if (result3.anchor === Unknown) {
  console.log('This is an Unknown; the current environment cannot interpret it');
}
```

Example 4: atomic operations can also be passed as values

```javascript
// Passing operation names as values
const source4 = [
  'add',
  ['add', 1, 2],   // the first argument is 3
  ['sub', 5, 3]    // the second argument is 2
];

console.log(MiniEnv.evaluate(source4)); // output: 5
```

## Concept mapping table

| Origin abstraction | Its counterpart in this example |
|---|---|
| **Value** | The numbers `1`, `2`, the string `"yes"`, and the `Unknown` marker object. All values are distinguishable (value comparison for primitives, reference comparison for objects; the example uniformly uses ===). |
| **Operation** | `add`, `sub`, `if`, `print`. They are functions registered in the `ops` table. When executed they "take effect" (compute, output, branch-select). |
| **Environment** | The `MiniEnv` object itself. It holds the atomic operation registry (`ops`) and the interpretation rules (`evaluate`). |
| **Composition** | The array structure `['add', 1, 2]`. It describes the relation between the "operation name" and the "arguments". The array itself does not execute until processed by `evaluate`. |
| **Source code** | The variables `source`, `source2`, `source3`, etc. They are carriers of composition structures. |
| **Atomicity (relativity)** | `add` is an atomic operation in `MiniEnv`. But underneath it may be composed of multiple micro-operations of the JS engine. `MiniEnv` does not care about those; it only declares: "within my boundary, `add` is indivisible". |
| **Penetration** | This example does not enable penetration. `MiniEnv` only uses the operations in its own `ops` table. To enable penetration, one could let `evaluate`, when it cannot find an operation, try requesting a same-named function from an underlying environment (e.g. the JS global object). |
| **Unknown** | When an operation name does not exist, a structure containing the `Unknown` public anchor is returned. This is a clear, distinguishable mark, rather than silently returning `undefined` or throwing an exception. The decision-maker can decide the follow-up behavior by checking `result.anchor === Unknown`. |
| **Environment construction** | If the decision-maker wants to extend `MiniEnv`, they can create a new object that inherits or merges `MiniEnv.ops` and add their own atomic operations. The new environment has all of `MiniEnv`'s capabilities plus its own extensions. |
| **Structure preserved** | The same `source` structure, if interpreted by another environment with the same operation names, may produce different results (for example, `add` defined as string concatenation in another environment). The structure itself is unchanged; the environment's response differs. |

## Limitations of the example environment

This example intentionally stays minimal, so it does **not** include the following features:

- A type system or type checking;
- Variable binding or scoping (all values are immediately evaluated literals);
- Explicit control of memory management or garbage collection;
- Concurrency, asynchronous, or parallel execution models;
- A complete implementation of penetration;
- Error recovery mechanisms (returning `Unknown` for unknown operations is one strategy, but not the only one);
- A text parser for source code (in this example "source code" is given directly as arrays, omitting the parsing step).

These omissions are not flaws, but **design choices**. If a real environment needs these features, they should be "grown" step by step by that environment's implementer according to concrete needs, rather than inherited all at once from the core. This is exactly what Origin's "capability growth" idea means at the concrete implementation level.

## Extension exercises

If readers want to explore further based on this example, here are a few possible extension directions:

1. **Add variable binding**: add an `env` object to the environment to store the mapping from variable names to values, and add `let` and `get` operations.
2. **Implement penetration**: modify `evaluate` so that, when an operation is not found in `ops`, it tries calling `globalThis[opName]` (or a similar mechanism), letting JavaScript native functions enter as atomic operations of other environments.
3. **Support custom Unknown behavior**: turn the Unknown return policy into a configurable callback, letting the decision-maker customize the behavior for "undefined operations" (for example, throwing an exception, returning a default value, logging, etc.).
4. **Provide a text parser**: write a simple textual syntax for array structures (for example `"(add 1 2)"`), letting structures be parsed from text strings.

These extensions are none of them requirements of the Origin core. Their existence only says one thing: **once an environment is constructed, capabilities can keep growing from within it.**

---

*This appendix is part of the Origin Implementer's Guide. It aims to demonstrate the mapping path from abstraction to code, not to prescribe an implementation.*

**End of document**
