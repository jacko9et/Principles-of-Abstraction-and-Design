# Origin Language Core Abstract Design

> Core positioning: on top of a minimal abstraction, keep structures stable, and allow environments to keep growing capabilities without breaking the fundamental relations.

## Document guide

- This document is the application of *Principles of Abstraction and Design* to programming language design. The principles answer "what it is and why"; this document answers "what that means for language design".
- `Origin` is a minimal abstraction framework, not a complete ready-made language definition.
- It cares about the relations among values, operations, compositions, environments, and source code (as a form of composition), and why those relations must remain minimal, explicit, and interpretable.
- It does not promise to freeze any concrete implementation; it only promises to establish a sufficiently stable abstraction anchor for later environments to extend.
- `Origin` is in fact a minimal axiom system of a computational ontology.

This document answers: which basic abstractions in `Origin` cannot be deleted; what minimal closed relation they form among themselves; the relation among atomicity, observation layers, and environments; and which content belongs to the core and which is left to concrete environments to grow.

This document does **not define** concrete syntax, type systems, branching and loops, module systems, compilers, runtimes, standard libraries, or engineering systems. Those can be built on top of the core, but they cannot in turn become premises of the core.

> Reading reminder: none of the concepts in this document are physical entities at fixed levels. When you read any concept, first ask yourself: from which environment's viewpoint am I standing right now? If you find yourself judging the behavior of one environment by the rules of another, pause—that may be exactly the signal that you need to switch viewpoints.

## Design philosophy

### 1. Acknowledging facts

`Origin`'s design philosophy first requires acknowledging facts: separating "existing facts" from "desired capabilities". Physical laws directly describe facts in nature; mathematics is an abstract language that can describe nature; programming is born from mathematics. Therefore, `Origin` understands "laws" themselves as abstract objects—this shows in: the final power of interpretation is handed to environments, along with the atomic values and atomic operations environments provide.

Many people naturally think of security, permissions, and the like—but that is exactly mixing facts with capabilities. Security must be built on facts; therefore those capabilities should be added later as needed.

### 2. Minimal necessity

> **What is not defined needs no proof of its reasonableness; only once something is defined must it prove why it must exist.**

The core abstraction keeps only the minimum required for relational closure. A concept cannot enter the core merely because it "might be useful later"; whether it belongs to the core is judged by the deletion experiment: delete it, and see whether the core relations still close. This turns the core's design goal from "fewest features" into: **the minimum required for relational closure**. We need not decide in advance what the future should look like—as long as the basic abstractions are stable enough, new capabilities can keep growing from the relations among them.

### 3. No presupposed concrete implementation

The core does not prescribe:

- how data is stored;
- whether values have addresses;
- how operations are encoded;
- how structures are represented;
- that source code must be text;
- whether a compiler exists;
- whether an interpreter exists;
- whether a traditional runtime exists.

Host languages, syntax, parsers, IR, runtimes, platform backends, and self-hosting paths are all parts that grow later.

Therefore:

> **Rules can be abstracted away, abstractions can be inherited, and concrete rules are decided by environments.**

What the core must establish is a sufficiently small, sufficiently stable abstraction anchor that can connect with reality.

## Why do it this way

### 1. Assemblable architecture and extensibility

Decoupling unnecessary capabilities brings extensibility—at the source code level, the core looks like nothing at all, just some concepts.

Facing an outlandish requirement, there are in fact only two possibilities: the current environment or architecture does not support it (the rules do not allow it), or it is very difficult (a subjective refusal).

If we do not put unnecessary rules into the core, the architecture stays assemblable: what really limits a capability is no longer a boundary prescribed in advance by the core, but the rules of the current environment itself; and `Origin`'s "acknowledging facts" means exactly acknowledging that those rules do exist.

Therefore, the meaning of a minimal core is not to have the architecture refuse every requirement, but to separate "the core must be this way" from "a concrete environment wants it that way"—the former is facts and basic relations; the latter continues to be constructed inside environments. Only after this separation is made will the architecture not mistake some concrete implementation for the only possible world.

### 2. A logical anchor in the AI era

AI tends to be accommodating: first of all it does not refuse, but looks for ways to realize the user's demand. But today's IT world is too fragmented, AI's thinking is fragmentary, and there is no logical anchor to tell AI "this touches a rule". So the user falls into an endless loop—depending on AI, while AI keeps spinning in place: it has not refused you, but it just cannot give you what you want.

## The core fourfold and its irreducibility

The `Origin` core contains four irreducible basic dimensions:

- **Value**: a distinguishable existence;
- **Operation**: an effect;
- **Composition**: the structure formed by operations acting on values;
- **Environment**: provides interpretation and prescribes observation boundaries.

They are not four parallel, unrelated objects, but four necessary positions in a minimally closed system.

```text
                  Environment
                /     \
               /       \
   prescribes observation   provides interpretation
   boundaries              |
             ↓             ↓
          Value ⇄ Operation
             \   /
              \ /
            Composition
               │
               │ switch observation layer
               ↓
              Value
               │
               └────→ Operation → Composition → …
```

This is not a program execution flow, but the core's **conceptual relation diagram**: values need relations and effects to show distinguishability in their differences from other existences; operations need existences as the objects they act on. The two need each other—that is exactly what "Value ⇄ Operation" means. After operations act on values, compositions form; compositions need environments to provide interpretation; and the atomic values and atomic operations environments provide are precisely values and operations. The four positions connect end to end, forming a closed ring.

To say they are irreducible, test with the deletion experiment: delete any one position and see whether the whole relation can still stay closed.

**Delete `Value`.** Operations lose the objects they can act on, relate to, and distinguish; compositions lose their participants; environments lose the basic existences they can interpret. If concepts like "object" or "entity" are reintroduced to take over these duties, that is essentially redefining value. Therefore value cannot be deleted.

**Delete `Operation`.** Distinguishability is the first property of value, and it can only show itself through relations and effects. With operations deleted, the distinguishability of values cannot be recognized—value loses its meaning as a distinguishable existence. If concepts like "change" or "action" are introduced to restore this capability, that is essentially reintroducing operation. Therefore operation cannot be deleted.

**Delete `Composition`.** Values and operations can exist separately, but the structure they form has no independent carrying position; more importantly, there is no way to explain how a structure becomes a value again, as a whole, at another observation layer. Restoring this capability with concepts like "relation" or "structure" only renames composition. Therefore composition cannot be deleted.

**Delete `Environment`.** The definition of atoms loses its basis; the meaning of operations would have to be prescribed by the core itself; compositions could not obtain interpretation. Structure and interpretation would then be forced into one position—and the entire design exists precisely to separate the two. Therefore environment cannot be deleted.

The conclusion of the deletion experiment: none of the fourfold positions can be deleted. This is not because the four concepts are favored, but because deleting any one leaves a gap in the closed relation—and whatever fills the gap will only be itself.

The meaning of "switch observation layer" in the diagram—a composition being acknowledged as atomic, as a whole, at another observation layer and re-entering operation and composition—will be explained in the Composition chapter.

## Value

Value is a distinguishable existence. By what means distinguishability is achieved is not prescribed here; the distinction strategy is provided by environments. What language design needs to add:

**A value is not a concrete data representation.** Everything in a computer can ultimately appear as data, but a value is first of all an abstraction. An existence can become a value because it can be distinguished from other existences.

Distinguishability can be understood with the help of "position"—not a physical address, but a position in an abstract sense:

- a memory address can be understood as the position of data in memory;
- an entity ID can be understood as the position of an entity in a database;
- the two `x`s in `x = x + 1` have the same name but occupy different positions and carry different relations.

These examples are not meant to prescribe that values must have addresses, but to help understand: a value must be distinguishable.

A value does not prescribe: storage location, memory address, data structure, concrete type, physical representation. Those belong to concrete environments. But an environment must provide some operable uniqueness anchor; otherwise operations cannot happen.

## Operation

An operation is an effect with directionality and connectedness; the core of an operation is the effect itself, not the result; an operation is also a value. What language design needs to add:

When an operation is taking effect, it occupies the operation position; when it is viewed as a whole, it occupies the value position. The two positions do not occur simultaneously—an operation taking effect is an operation; a viewed operation is a value.

**Example: 1 + 2 = 3**

In a calculator environment, `1 + 2` is a composition; the `+` operation both expresses connectedness and describes the connection relation. The addition is not executed immediately; only when the `=` operation acts on the `1 + 2` structure is the value `3` returned.

## Composition

Composition is the structure formed by operations acting on values; the product of a composition is also a value; a composition is only responsible for keeping relations in place, with effects interpreted by environments. What language design needs to add:

Assembly code is interpreted by CPU instruction sets; C code is interpreted layer by layer by lower-level environments. A composition itself does not carry the answer to how it ultimately produces effects—it is first of all just a structure.

Atomicity is not an absolute property but relative to environments. Suppose environment `E₁` treats `A` and `O` as atoms:

```text
E₁:
A → value
O → operation
```

When they form `C = O(A)`, `C` is a composition to `E₁`. If another environment `E₂` recognizes `C` as a whole:

```text
E₂:
C → value
```

then `C` gains value status in `E₂`. "Being a composition" and "being a value" are not absolute mutual exclusions; they can belong to different observation layers.

The fourfold relation is therefore not a one-way chain, but a structure that can keep nesting itself between observation layers:

```text
┌──────────────────────────────┐
│ Environment 1               │
│                              │
│   value + operation          │
│          ↓                   │
│       composition            │
└──────────┬───────────────────┘
           │ view as a whole
           ↓
┌──────────────────────────────┐
│ Environment 2               │
│                              │
│      composition = value     │
│              ↓               │
│           operation          │
│              ↓               │
│          composition         │
└──────────┬───────────────────┘
           │
           ↓
          ……
```

This "loop" is not an infinite recursion of definitions, but allows the same structure to gain different atomicity statuses at different observation layers. This is exactly the important foundation on which `Origin` keeps structures stable while environments keep growing.

The indelibility of composition is already proven by the deletion experiment. Its deeper reason is exactly the looping structure above: composition is the structural carrier of holification and re-atomization between observation layers—without composition, a structure formed by multiple units of the current layer could not become a value again at a higher observation layer, and thus could not enter new operations and compositions.

## Environment

An environment provides interpretation and prescribes observation boundaries; an environment provides atomic values, atomic operations, and interpretation; an environment is also a value. What language design needs to add:

**An environment is a necessary condition for abstraction to connect with reality.** Why do CPU instructions produce certain behaviors? Why do C language operations ultimately produce such machine behaviors? These questions all need environments. Discussing concrete behavior without an environment loses meaning in itself.

An environment can also be understood as an abstraction. A CPU can provide a code execution environment; the CPU itself is first of all an abstract concept, and different CPU models can be seen as concrete realizations of that abstraction. Assembly language is similar: it can serve as an environment, providing rules and atomic operations for higher layers; but from another level, assembly language itself is data—a value. Values, operations, and environments are not completely isolated things; they are different abstractions of the same reality at different observation levels.

Viewing an environment as a value does not require the environment to prove its own existence at runtime. When an environment works as an interpreter, it is only responsible for interpreting the current structure and does not need to interpret itself at the same time; only when a lower-level environment actively treats it as operable data does it show the property of a value at that observation layer. The interpreter view and the observed view do not occur on the same layer at the same time, so no recursive dependency forms.

The core role of an environment is to provide atomic values, atomic operations, and interpretation rules. The "atoms" mentioned here are boundary declarations relative to the current environment: an operation indivisible within that environment counts as atomic; it may be composed of multiple operations in a lower-level environment, but that does not affect its atomicity in the upper environment.

From the bottom, the CPU provides an instruction set; above the CPU, assembly language can form another environment layer; further up, the C language can form yet another layer. **CPU → Assembly → C** helps understand the inheritance and construction relations among environments. But the "layers" here are mainly a way of understanding: environments need not form a strict tree; inheritance, composition, nesting, and other interpretation relations can exist among environments. What really matters is: the environment decides how a structure is interpreted. The same structure entering different environments does not necessarily produce exactly the same effects.

An environment carries two basic roles: **first, providing the atomic units of the current layer**—the environment provides or acknowledges the current layer's atomic values and atomic operations; **second, interpreting structures**—when values and operations form compositions, the environment interprets them according to its own atomic rules and observation boundaries. The "interpretation" here is not a fifth basic entity beyond the fourfold: interpretation itself falls on the "operation" element—it is an operation provided by the environment, acting on a composition and producing a value. How that interpretive operation is established at a lower level belongs to another observation layer's question.

This allows the same structure to gain different concrete meanings in different environments without modifying the structure itself. The "environment" here should not be narrowly understood as some runtime, process, virtual machine, or operating system—those are only implementation forms of concrete environments. The core's "environment" is more abstract: it is the necessary basis for an abstract structure to obtain concrete interpretation, and at the same time it prescribes the boundaries of the current observation layer.

**The landing of uninterpreted and refused interpretations.** When an environment cannot interpret a composition, or explicitly refuses to interpret it, the fourfold relation still needs a landing—otherwise, once an environment meets an unprocessable structure, the core relation breaks at the point of failure. The landing is still a value: the environment can produce a value carrying the fact of "not interpreted", such as the `Unknown` anchor. It is not a crash, nor a special "non-value", but an ordinary value whose carried semantics is "this interpretation has not happened yet, or has been refused by the current environment".

```text
Composition → Environment → Value (an ordinary result, or a value carrying "uninterpreted / refused")
```

The `Unknown` here is only one concrete example of an "uninterpreted value", not an additional basic entity. The Lua example later will further explain, from the value side, why it can serve as a public anchor.

## Observation layers, exclusivity, and penetration

**Exclusivity of interpretation.** The same value, at the same moment, is interpreted by only one environment.

At another moment, another environment can interpret it too—two environments interpret the same structure one after another, and each interpretation belongs wholly to one of them. This is exactly "structure preserved, environment grows": the structure is unchanged, and the interpretation differs with the environment.

Even when other environments' interpretations are allowed into the current environment, this does not change: the content of an interpretation is still borne by the environment it belongs to.

**Penetration.** Various relations can exist among environments: one environment interprets the structure produced by another; one environment views another as a whole value; one environment constructs another. This chapter discusses one of them: penetration.

Must an environment use only the interpretations it directly provides? Not necessarily. If the current environment allows it, other environments' interpretations can enter the current environment. That is penetration.

> **Penetration is the act of mapping other environments' interpretations into the current environment.**

Whether penetration is allowed is decided by the environment currently interpreting the structure. An environment may allow or forbid it.

For example, in environment relations like CPU, Assembly, and C: if the relevant environments all allow direct use of CPU instructions, then C can penetrate its own environment and directly use the CPU instruction set. Likewise, if C allows direct use of assembly, assembly code that can penetrate the C environment may appear in C.

This example only shows how other environments' interpretations are allowed into the current environment; it does not mean the current environment's observation boundary is cancelled, nor that all other environments' interpretations automatically enter the current environment. Therefore an environment is not an absolutely closed container; it can decide which other environments' interpretations may enter.

When penetration occurs, the current environment interprets only the mapping itself—recognizing it as an interpretation from other environments; the mapped interpretation's content is still borne by the environment it belongs to.

What penetration is: a concrete relation among environments, not a new basic concept to be introduced here. The environments, interpretations, and mappings penetration uses are all already inside the fourfold; it is only a concrete combination of existing elements and needs no new concepts. Therefore an environment can, without breaking the fourfold relations, gain through penetration interpretive capabilities it does not itself have.

## Origin, the decision-maker, and environments

`Origin` is not a final observer standing above all environments, but first exists within some given environment. For `Origin`, the current environment is a given condition: `Origin` cannot directly decide the rules of the environment it is currently in; it can create environments, but that happens only after creation. Therefore:

> **In the current environment, `Origin` is passive; only after creating new environments can `Origin` become a constructor of environments.**

**Semantic sovereignty belongs to the decision-maker.** First give the decision-maker a position: the decision-maker falls on the "environment" element—it is a role of the environment: the party that interprets active behavior. The interpretive work in an environment divides into two parts: active behavior is interpreted by the decision-maker; passive behavior is interpreted by the rest of the environment; which behaviors count as active and which as passive is defined by the environment. First comes the environment's division, then the decision-maker's bearing of active behavior's interpretation—therefore the scope of the decision-maker's semantic sovereignty is delimited by the environment.

Landing the position onto `Origin`, three things need distinguishing: the core prescribes irreducible basic relations; environments interpret structures by their own rules; the decision-maker constructs source code and chooses environments. Whether source code is submitted to some environment for interpretation, and whether the interpretation result that environment gives is accepted, is also decided by the decision-maker. These are exactly the decision-maker's active behaviors, interpreted by the decision-maker itself.

This is not pushing responsibility outside the system, but acknowledging a boundary: the decision-maker's goals are not decided for it by the core. Forcing this external decision into the core would instead mistake some particular wish for a rule of existence itself.

For example, security permissions can be added to an environment because of external demands, but they are not properties naturally owned by the `Origin` core. Whether security permissions are needed and what permission rules to adopt are first of all choices the decision-maker makes in the face of reality; only afterwards do they become functions in some concrete environment.

## Source code

### 1. The minimal source code is one composition

When we need to describe composition relations, source code appears.

Therefore:

> **The minimal source code is one composition.**

Source code describes values, operations, and the relations formed among them. From the core's abstract viewpoint, the minimal source code is this composition itself; in real environments, when such a relational structure is recorded and can be submitted to some environment for further observation or interpretation, it gains the meaning of source code.

Thus source code is not a fifth core abstraction, but the form **composition** takes when described and migrated across environments. It is still composition—only a composition that has become a structure that can be saved, passed, and re-observed by different environments.

### 2. Source code is both interpreted by environments and able to construct environments

Besides describing compositions, source code can in turn participate in constructing environments. The above said how environments interpret source code; the following discusses how source code forms new environments through compositions and operations in the current environment.

Because computer languages themselves are also constructed from source code.

Assembly language has source code;

C can construct virtual machines;

the Java virtual machine is itself an environment, and it can be constructed by C.

Therefore:

> **Source code is both interpreted by environments and able to construct new environments.**

This makes the relation between source code and environments a two-way one.

Environments interpret source code;

source code constructs environments.

> Two kinds of environment changes must be distinguished:
> (1) Construction—source code runs in the current environment and creates a new logical environment through compositions and operations; that environment then exists as a sub-layer or neighboring layer of the current environment;
> (2) Selection/switching—the decision-maker submits the same source code (or structures within it) to another already-existing environment for interpretation; this is a deployment act and is not decided by the source code itself.
> The former is part of the computational process; the latter is part of sovereign decisions. The `Origin` core only concerns itself with the former and does not take responsibility for the latter.

### 3. The cross-environment nature of source code

If source code is always inside some environment, then why can the same source code exist in different environments?

For example, the same C source code can run on CPUs of different architectures.

The reason is not that different CPUs must have exactly identical implementations.

Rather:

> **The structure described by the source code can be preserved, while environments can change how they interpret that structure.**

Environments of different architectures can, through different lower-level environments, convert the same source code structure into forms they can interpret.

Therefore, source code has, at some level, environment independence.

But this environment independence does not mean source code is completely unaffected by environments.

Source code is always inside an environment and always affected by it.

What truly stays unchanged is:

> **The structure.**

### 4. Abstract source code is not a requirements list

Therefore, abstract source code, more precisely, is:

> **A mathematical description of a set of relational structures.**

It is not a requirements list demanding capabilities from environments.

It does not need to declare:

> I require the environment to have capabilities X, Y, Z.

It only describes:

> Which values exist, which operations exist, and what relations have formed among those values and operations.

For example:

> Operation A takes B as input and produces C.

What is described first is a structure.

As for this structure:

- in which environment it is activated;
- by what rules it is interpreted;
- what physical meaning it is given;
- what effects it ultimately produces;

—those are all the environment's own business.

Therefore, different environments being able to interpret the same source code structure does not mean they must implement a common interface.

It is because:

> **Their respective atomic rules can correspond with that structure.**

### 5. Structure unchanged, environment responds

This is also one of the most important values of `Origin`'s abstract design: the core preserves structures, while environments can keep growing without breaking the core relations.

Today, environment A's atomic rules can interpret some structure.

In the future, environment D's rules, having been enhanced, may also be able to interpret the same structure.

The source code itself need not change.

What changes is:

> **The environment's response policy for that structure.**

Therefore:

> **Structures remain unchanged; environments can keep growing.**

This is what `Origin` calls "capability growth".

New capabilities do not necessarily require modifying the original abstraction.

As long as environments keep adding their own atomic rules, new interpretive capabilities may appear.

Therefore, what `Origin` pursues is not:

> Defining all possible capabilities in advance.

But rather:

> **Establishing a structure small enough that new capabilities can emerge naturally from the relations between environments and structures.**

## Compiler

The above discussed the abstract relation between environments and structures. Entering concrete implementations, a compiler can be one way to realize that relation, but it is not the core relation itself.

A compiler is not a concept the `Origin` core must prescribe in advance.

Most of the time, what a compiler does is:

> **Convert the structures described by abstract source code into data that lower-level environments can interpret.**

Hence:

**Abstract structure → compilation transformation → data interpretable by lower-level environments**

But this is only one of many possible implementations.

The `Origin` core does not depend on any particular compiler structure.

## What the Core does not define

Whether some content belongs to the core is judged by the deletion experiment: delete it, and see whether the core still closes. "Useful" is not a core membership qualification; "indeletable" is.

The following content can exist in `Origin`'s concrete environments or upper-level languages, but is not a necessary abstraction of the core:

### 1. Syntax

- Java-style syntax;
- parentheses, semicolons, symbols;
- lambda syntax;
- method declaration syntax;
- indentation rules;
- textual source code formats.

### 2. Type systems

- strong typing / weak typing;
- static typing / dynamic typing;
- generics;
- type inference;
- type checking;
- type conversion;
- concrete type systems like `Option` / `Result`.

### 3. Branching and loop structures

- `if`;
- `while`;
- `for`;
- `return`;
- `break` / `continue`;
- `try` / `catch` / `finally`.

### 4. Data structures

- arrays;
- List;
- Map;
- Set;
- Object / Class;
- Enum;
- Record.

### 5. Engineering structures

- Module;
- Namespace;
- Package;
- Build;
- Configuration;
- Dependency Management;
- project directory structures.

### 6. Implementation technologies

- Compiler;
- Interpreter;
- AST;
- IR;
- VM;
- JIT;
- GC;
- Native Backend;
- Self-hosting.

### 7. Capability policies

- Security;
- Permission;
- concurrency models;
- networking;
- IO;
- standard libraries;
- debuggers;
- IDEs.

> **These contents can all be very important, but their importance is not enough to prove they belong to the core.**

## The overall relation of Origin

Therefore, `Origin` does not prescribe a complete language world from the start. The relation chain below is a linear unfolding of the conceptual relations above—not a program execution order, nor a temporal order among the fourfold.

It starts from a very small starting point:

**Value**

↓

Values can be distinguished.

**Operation**

↓

Operations make values take effect.

**Environment**

↓

Environments provide atomic values, atomic operations, and interpretation rules.

**Composition**

↓

Values and operations form relations.

**Source code (a form of composition)**

↓

Source code describes those relations.

Meanwhile:

**Source code → constructs environments**

And the new environments can again:

**Interpret → source code**

Thus a closed loop forms that can keep growing.

Finally:

> **Structures can be preserved and environments can grow; source code can describe structures and can also participate in constructing new environments.**

That is the core of `Origin`.

It does not try to define a complete world in advance.

Rather, it defines a starting point small enough that new worlds can keep being constructed from above it.

The final abstraction diagram:

```text
                        Environment
                    ┌────────┴────────┐
                    │                 │
          current observation     concrete
             boundaries         interpretation
                    │                 │
                    ↓                 ↑
               Value ⇄ Operation
                    │       │
                    └───┬───┘
                        ↓
                    Composition
                        │
                        │ view as a whole
                        ↓
                       Value
                        │
                        ↓
                     Operation
                        │
                        ↓
                    Composition
                        │
                        ↓
                       …
```

This diagram does not mean "the program executes top to bottom"; it means:

> **Atomic existences and effects in one observation layer form structures; a structure can be observed again as a whole at another observation layer; after regaining value status, it can enter new operations and compositions. Environments decide each layer's observation boundaries and provide concrete interpretation for structures.**

## A minimal core example of Origin expressed in Lua

Note: this is only an example. The `interpret` function in it is not a complete implementation of a compiler or interpreter, but a direct mapping of `Origin`'s core relations: it hands a structure (input) to an operation (operation), which decides how that structure is interpreted in the current environment. `interpret` itself does no conversion, no optimization, and attaches no rules—it is only a convergence point that stitches the two core relations together: "structure" and "the operation acting on structure". Interpretive behavior is not defined in advance; it grows inside environments. This code shows the minimal form of that growth's starting point.

```lua
-- Unknown is first of all an ordinary value: it carries the semantics of "uninterpreted / primordial",
-- rather than being a container or base class of values.
-- It contains no other values, and no other values inherit from it.
-- It merely declares a fact: in this abstract space, values hold as distinguishable existences.
-- All values are on the same plane as Unknown; there is no hierarchy or derivation.
-- Unknown's declaration looks "empty", but that emptiness is not meaningless—it is a state
-- carrying no presupposed semantics.
-- When an environment has no type system, Unknown can stand for "a value of unknown meaning";
-- when returning empty values, handling null pointers, gradual typing, or type conversion,
-- the environment can grant it other roles.
-- Unknown itself does not define those roles; it only provides a distinguishable,
-- operable anchor for them.
-- In concrete implementations, Unknown should remain a public anchor recognizable across
-- environments, not a private internal mark of each environment.
Unknown = {}

-- interpret does no conversion or optimization.
-- It does only one thing: hand the structure (input) to an operation (operation),
-- letting the environment decide how to interpret it.
-- This is the direct mapping of the core relation "environments interpret structures".
-- The environment does not appear explicitly here—it lives in the operation's
-- interpretive behavior.
function interpret(input, operation)
    return operation(input)
end
```
