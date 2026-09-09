# Principles of Abstraction and Design

## Preface and reading notes

The title says "principles", not "rules", and that is itself a statement: this document offers an explanation and a declaration, not a constraint on behavior.

This document originates from a computer software practitioner's summary of knowledge and information encountered across long-term practice and study. The summary involves vocabulary from mathematics, physics, and computing: mathematics is an abstract science invented by humans, and it can describe the world; physics can verify itself against nature and help us understand; computers provide the most direct observation ground for this set of regularities. Therefore, this set of regularities should apply equally to aspects beyond software.

What can these principles be used for? Within the scope this document can determine, the most direct use is to guide the design and development of computer software, so that we need not be forever trapped in a ball of yarn.

This document first explains the core concepts, then unfolds the relations among them through step-by-step explanation, and finally lets the reader grasp the whole picture of the basic principles of abstraction.

The core concepts of this document are arguable at the philosophical level. This document uses examples to aid understanding, but will not argue them one by one—argumentation is not the point of this document.

### How to read this document

This document is best read from the beginning. The opening states the stance, the middle gives the core concepts and laws, then the methods of judgment, and finally the applicability and boundaries. The concepts are introduced layer by layer: later text uses concepts defined earlier, and keeps growing on top of them. Therefore, if while skipping around you meet a concept you have not seen, go back to where it is defined.

### Terminology conventions

This document is self-contained and does not presuppose that the reader has read any other material. Every concept that appears is defined at its first occurrence; the same word keeps the same meaning throughout the document. Some everyday terms (such as "structure", "relation") have stricter usage here—follow the definitions in this document, and do not substitute everyday habits. This document uses no concept from outside itself that has not been defined within it.

## Part One　Origins and stance

### 1. The starting point of the problem

In software development, one can often observe this: concrete technologies keep appearing and keep going out of date; a practice widely adopted yesterday is seen as a burden today. This raises a question: beneath these constantly changing concrete contents, are there some unchanging basic relations? If they exist, where are they? If they do not exist, does every accumulation lose its meaning as concrete technologies are replaced?

This document's answer is: they exist. These basic relations are few in number, but they can support concrete contents that keep growing. Finding them and stating them clearly is what this document sets out to do.

### 2. Feasibility guarantee: acknowledging facts

Facts exist beforehand. All our observation, thinking, and construction take place within facts, not before them. Without acknowledging this, everything that follows has no foundation. This is this document's first guarantee: acknowledging facts.

There is only one relation between us and facts: first acknowledge them, then act within them. Denying a fact cannot cancel the fact—it can only cancel the basis of one's own action. Therefore, acknowledging facts is not passive concession, but confirmation of possibility: only by first seeing where the facts are can we know which things can be done and which things need another path.

The direct application of this guarantee is to separate "existing facts" from "desired capabilities". Desired capabilities can be proposed, discussed, and changed; existing facts cannot be cancelled by discussion. Mixing the two together is exactly why many designs are doomed to entanglement before they even begin.

### 3. Growth guarantee: no definition, no proof needed

This document's second guarantee is: what is not defined needs no proof of its reasonableness; only once something is defined must it prove why it must exist.

This guarantee sets both the lower bound and the upper bound of this document.

The lower bound: this document defines only the fewest things. Any concept that is not necessary for the whole system does not enter this document. A concept cannot be admitted merely because it "might be useful later"—"might be useful" is not proof of its existence.

The upper bound: this document is not responsible for what it does not define. For anything this document does not define, it makes no promises and does not presuppose its form. What will grow on top of these basic concepts in the future is not prescribed in advance here.

The two guarantees are one: acknowledging facts, so nothing is fabricated; defining only the necessary, so nothing swells.

## Part Two　The core axiom system: the fourfold

The two guarantees were established above: acknowledge facts, and define only the necessary. Now comes the main subject: under these two guarantees, this document needs to define four concepts in total. These four concepts are all the axioms of this document; everything that follows is built on them. They are: value, operation, composition, environment.

### 1. Value

**A value is a distinguishable existence. An existence without distinguishability is meaningless.**

A value is first of all an abstraction, not some concrete representation. An existence can become a value because it can be distinguished from other existences. That is distinguishability—the first property of value.

What does "distinguishable" mean? Imagine two existences. If there is no way at all to tell them apart, then the claim "there are two" itself does not hold: two existences that cannot be distinguished do not hold conceptually. Therefore, a value must be distinguishable—a value must, in some sense, "be this one and not that one". For example, 3 and 4 are two values that can be distinguished from each other.

By what means distinguishability is achieved is not prescribed here. Position, name, moment—anything that can say "this is this one" will do. This document establishes only one thing: a value must be distinguishable; as for what specific means the distinction relies on, that is the environment's question to answer.

At the same time, a value does not prescribe many things. A value does not prescribe where it is stored, in what form it appears, how it is constructed internally—those contents are not what the "value" abstraction cares about. The "value" abstraction cares about only one thing: that it exists and can be distinguished. This restraint is consistent with the guarantees above: this document defines only the necessary and presupposes nothing else.

### 2. Operation

**An operation is an effect; this effect has directionality and connectedness. An operation is itself also a value.**

If there were only mutually isolated values, nothing would ever happen among them. There must be effects for relations to arise among values. Therefore, an operation should first be understood as an effect.

Directionality means the effect departs from somewhere and points somewhere—"who acts on whom" is determinate. Connectedness means the effect links the things involved.

One may use force in physics to aid understanding: gravity acts on objects; it has direction; and it links things within a certain range. This example only aids understanding; it does not equate operations with force.

The core of an operation is the effect itself, not the result it produces. An operation must really have taken effect—even if, viewed from the result, the effect changed nothing. What result the effect produces is not in the definition of operation.

An operation is itself also a value. Because as long as an existence can be distinguished, it can occupy the value position; and operations can be distinguished from each other—this operation and that operation are different existences.

Two positions need separating here: when an operation is taking effect, it occupies the operation position; when it is viewed as a whole, it occupies the value position. The two positions do not occur simultaneously—an operation taking effect is an operation; a viewed operation is a value.

Note: an operation promises the effect, not the result.

### 3. Composition

**A composition is the structure formed by an operation acting on values.**

Two mutually isolated values will not form a composition by themselves—there is no effect between them. There must be an operation for isolated values to establish contact.

A composition first describes relations: after an operation acts on values, the relational structure formed among the values. For example, a sentence: words are organized in a sequential relation and form the structure of a sentence—the sentence is a composition.

A composition itself does not carry the answer to how it ultimately produces effects. How this relational structure is understood and what effects it produces are not in the composition—they are what the environment interprets. A composition is responsible for only one thing: keeping the relations in place.

Hence, composition and value are not mutually exclusive categories: the product of a composition is constituted by the current layer's atomic values and atomic operations; it has distinguishability, so it is itself also a value. The two positions each emphasize one side—value emphasizes distinguishable existence; composition emphasizes the relational structure formed by operations acting on values. What really changes with the observation layer is atomicity; this will be unfolded later when levels of observation are explained.

### 4. Environment

**An environment provides interpretation and prescribes observation boundaries. The environment's interpretation necessarily contains a determination of atomicity.**

What does an operation mean, after all? After an operation takes effect, what result is produced? Such questions cannot be answered apart from a concrete environment. Therefore, the environment is a necessary condition for abstraction to connect with reality.

An environment provides three things: atomic values, atomic operations, and interpretation. The so-called atom is the indivisible basic unit in an environment: it is indivisible in this environment, but may consist of finer structures in another environment. This determination is made by the environment. The environment's interpretation decides how structures are understood and what effects they produce.

An environment also prescribes observation boundaries. Observation boundaries prescribe what the current observation layer can see and understand: structures inside the boundary the environment can interpret; contents outside the boundary are not within the current layer's discussion.

An environment does not refer to any one concrete thing. Any existence that provides interpretation and prescribes observation boundaries can occupy the environment position. For example, a board game: it provides pieces (atomic values), moves (atomic operations), and winning/losing judgment (interpretation), and prescribes what can and cannot be talked about in this game.

An environment is itself also a value. An environment not only provides interpretation for other structures; it can itself be treated as an existence: viewed from another observation layer, the environment as a whole can occupy the value position. A factory provides a working environment for its workers; and in an industry directory, this factory is a distinguishable entry. The same existence, in the former relation, occupies the environment position; in the latter relation, the value position.

These two positions likewise do not occur simultaneously: when an environment works as an interpreter, it is only responsible for interpreting and does not need to interpret itself at the same time; only when another observation layer views it as a whole does it show the property of a value.

Earlier it was said that by what means distinguishability is achieved is not prescribed here. Now answer another question: who provides the distinction strategy? The environment. Whether an existence can be viewed by the current environment as an independent value depends on whether it can show difference within the relations and effects the current environment provides. Thus, values provide distinguishable existences; environments and operations make that distinction observable and usable.

### 5. The minimal closed relation and irreducibility

The four concepts have each been stated. Now answer a holistic question: what is their relation to each other? Value, operation, composition, environment—they are not four parallel, unrelated objects, but four necessary positions in a minimally closed system.

First look at how they implicate each other. Values need relations and effects in order to show distinguishability in their differences from other existences; operations need existences as the objects they act on. The two need each other, and neither can absorb the other—this is exactly what "Value ⇄ Operation" means. After operations act on values, compositions form; compositions need environments to provide interpretation; and the atomic values and atomic operations environments provide are precisely values and operations. The four positions connect end to end, forming a closed ring.

To say they are irreducible, what test to use? The deletion experiment: delete any one position, and see whether the whole relation can still stay closed.

Delete `Value`. Operations lose the objects they can act on, relate to, and distinguish; compositions lose their participants; environments lose the basic existences they can interpret. If concepts like "object" or "entity" are reintroduced to take over these duties, that is essentially redefining value. Therefore value cannot be deleted.

Delete `Operation`. Distinguishability is the first property of value, and it can only show itself through relations and effects. With operations deleted, the distinguishability of values cannot be recognized—value loses its meaning as a distinguishable existence. If concepts like "change" or "action" are introduced to restore this capability, that essentially reintroduces operation. Therefore operation cannot be deleted.

Delete `Composition`. Values and operations can exist separately, but the structure they form has no independent carrying position; more importantly, there is no way to explain how a structure becomes a value again, as a whole, at another observation layer. Restoring this capability with concepts like "relation" or "structure" only renames composition. Therefore composition cannot be deleted.

Delete `Environment`. The determination of atoms loses its basis; the meaning of operations would have to be prescribed by this document itself; compositions could not obtain interpretation. Structure and interpretation would then be forced into one position—and the entire design of this document exists precisely to separate the two. Therefore environment cannot be deleted.

The conclusion of the deletion experiment: none of the fourfold positions can be deleted. This is not because this document favors these four concepts, but because deleting any one leaves a gap in the closed relation—and whatever fills the gap will only be itself.

The relations among the fourfold can be surveyed with the following diagram:

```text
                  Environment
                /     \
               /       \
    prescribes observation  provides interpretation
       boundaries            |
             ↓               ↓
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

This diagram does not depict a sequential process, but the relations among the core concepts. The meaning of "switch observation layer" in the diagram—a composition being acknowledged as atomic, as a whole, at another observation layer and re-entering operation and composition—will be unfolded later when levels of observation are explained.

## Part Three　Levels of observation

The four axioms were given above. Two points among them need further unfolding: first, how "atom" is actually determined; second, what "switch observation layer" in the core relation diagram means. This part answers both. Their common point is: observation has levels.

### 1. Atomicity is relative to the environment

Earlier it was said that environments determine atoms. Here the further question: is an atom absolute?

No. "Atom" has no absolute definition apart from the observing environment. Whether something is an atom depends on the current environment's observation boundaries.

For example, a wooden board. In a carpentry environment, it is an atomic value: it is used as a whole, and carpentry does not care about its interior. In a wood-processing environment, it is a composition: a structure formed by fibers and glued layers. The same board is an atom in one environment and a structure in another.

Which environment is right? Neither is "truly correct"—they simply stand at different observation layers. Atomicity is relative to the environment, not an absolute property of some existence itself.

Earlier it was said that a composition is itself also a value: "being a value" and "being a composition" are not mutually exclusive categories—value emphasizes distinguishable existence; composition emphasizes relational structure. What the observation layer changes is not the identity of "value", but the status of atomicity: a structure that is not an atom in the current layer can be acknowledged as an atom, as a whole, in another layer. This acknowledgment is determinate within one observation layer: once a value is acknowledged as an atom in the current layer, it is indivisible within that layer; whoever takes it apart must be another observation layer. Within the current layer, its identity is complete.

The other side of this fact is the exclusivity of interpretation: the same value, at the same moment, is interpreted by only one environment.

At another moment, another environment can interpret it too—two environments interpret the same structure one after another, and each interpretation belongs wholly to one of them. This is exactly "structure preserved, environment grows": the structure is unchanged, and the interpretation differs with the environment.

Even when other environments' interpretations are allowed into the current environment, this does not change: the content of an interpretation is still borne by the environment it belongs to. This will be unfolded in the penetration section. The division of interpretive labor never collides over the same value.

### 2. Observation-layer transition and the looping structure

Since atomicity is relative to the observation layer, how the same structure changes status between different observation layers deserves a complete walkthrough. This process is called observation-layer transition.

Take one complete process as an example. In a carpentry environment: boards and strips are atomic values, and "nailing" is an atomic operation. Nailing acts on boards and strips and forms a tabletop—the tabletop is a composition, not an atom in the carpentry environment.

Switch to an interior environment: the tabletop is recognized as a whole—the tabletop gains atomic status in the interior environment. Then the tabletop can join new operations: together with chairs and lamps, it forms the layout of a room—a new composition.

This process can repeat: every newly formed composition can potentially be acknowledged as an atom, as a whole, at another observation layer, and re-enter new operations and compositions. The fourfold relation is therefore not a one-way chain, but a structure that can keep nesting itself between observation layers—this is the looping structure.

```text
┌──────────────────────────────┐
│ Observation layer one:       │
│ carpentry                    │
│   board (atomic value)       │
│   + nailing (operation)      │
│              ↓               │
│      tabletop (composition)  │
└──────────┬───────────────────┘
           │ view as a whole
           ↓
┌──────────────────────────────┐
│ Observation layer two:       │
│ interior                     │
│   tabletop (atomic value)    │
│   + arranging (operation)    │
│              ↓               │
│   room layout (composition)  │
└──────────┬───────────────────┘
           │
           ↓
          ……
```

Two points must be stated clearly.

First, the loop is not infinite recursion. It does not demand that an environment forever contains another environment, nor that every environment must be interpreted down to a lower level. It says: the same structure gains different atomicity statuses at different observation layers. Nothing more.

Second, the step that makes the transition happen—"viewing as a whole"—is itself also an operation. Viewing-as-a-whole is an operation provided by the environment; it acts on a composition and produces a value—a value acknowledged as an atom in the new observation layer. Therefore, the transition introduces nothing beyond the fourfold; it is only one operation occurring when switching observation layers.

Taken together, these two points answer an important question: by what can structures stay stable while environments keep growing? The answer is the looping structure—the structure does not change with the observation layer; what changes is the atomicity status it gains at different observation layers. This will be unfolded later.

### 3. Penetration

What was discussed above is how one environment interprets structures. But environments are more than one—what relations do they have among themselves? This chapter discusses that question.

Various relations can exist among environments: one environment interprets the structure produced by another; one environment views another as a whole value; one environment constructs another. These relations are many; this chapter discusses only one of them: penetration.

The question is this. Must an environment use only the interpretations it directly provides? Not necessarily. If the current environment allows it, other environments' interpretations can enter the current environment. That is penetration.

**Penetration is the act of mapping other environments' interpretations into the current environment.**

Whether penetration is allowed is decided by the environment currently interpreting the structure. An environment may allow it, or forbid it.

Use an example to aid understanding. A company prescribes workflows for its departments; the company is an environment. A certain department already has a mature set of practices, sitting at a lower level. The company can choose: allow this department to keep using those practices directly—that is allowing penetration; or require that all practices be redefined according to the company's workflows—that is forbidding penetration. Both choices are the company's own decisions.

This example only shows how other environments' interpretations are allowed into the current environment; it does not mean the current environment's observation boundaries are cancelled, nor that all other environments' interpretations automatically enter the current environment.

Therefore, an environment is not an absolutely closed container. It can decide which other environments' interpretations may enter it.

Finally, state what penetration is: it is a concrete relation among environments, not a new basic concept introduced by this document. When penetration occurs, the current environment interprets only the mapping itself—recognizing it as an interpretation from another environment; the mapped interpretation's content is still borne by the environment it belongs to. The environments, interpretations, and mappings penetration uses are all already within the fourfold; it is only a concrete combination of existing elements and needs no new concepts. Therefore an environment can, without breaking the fourfold relations, gain through penetration interpretive capabilities it does not itself have.

## Part Four　Extended concepts

On top of the four axioms, this chapter introduces three concepts: result, test, decision-maker. They are not a fifth kind of basic entity, but concepts grown out of the fourfold; hence they are called extended concepts.

Why are they not basic concepts? Test with the deletion experiment: delete them, and the fourfold still closes—the mutual implication of value, operation, composition, environment does not depend on any of them. Therefore, they are not new members of the system, but exemplary unfoldings of the fourfold in concrete use. This document introduces them only to display that unfolding. The following explains them one by one, pointing out which of the fourfold each falls on.

### 1. Result

**A result is what is produced after an operation takes effect; it is also a value.**

The result falls on the "value" element. What is produced after an operation takes effect is the result; by the definition of value, it is also a value. This is consistent with the earlier conclusion: the products of operations and compositions necessarily have distinguishability.

The result has another role: it is the decision-maker's basis of judgment—this will be explained in the decision-maker section.

### 2. Test

**A test is an operation that verifies the result of an operation.**

After any operation takes effect, a result is produced. A test is precisely an operation acting on the result: it verifies this result and produces a conclusion. The test falls on the "operation" element—it is not a new kind beyond the fourfold, but one kind of operation. The conclusion a test produces is also a value. Note: the result does not presuppose concrete content—after an operation takes effect, something may be produced, or nothing; producing nothing is also a result.

### 3. Decision-maker

**Decision-maker: active behavior is interpreted by the decision-maker, while passive behavior is interpreted by the rest of the environment. The determination of active and passive behavior is interpreted by the environment.**

The decision-maker falls on the "environment" element—it is not a new kind beyond the fourfold, but a role of the environment: the party in the environment that interprets active behavior. The interpretive work in an environment divides into two parts: active behavior is interpreted by the decision-maker, and passive behavior by the rest of the environment. Two layers of relation need attention here. The first is the division of interpretive labor—the decision-maker and the rest of the environment each interpret the behavior they are responsible for. The second is the drawing of the division—which behaviors count as active and which as passive is not up to the decision-maker itself, but determined by the environment. The so-called active and passive behaviors do not describe properties of the behaviors themselves, but only the environment's division: what is assigned to the decision-maker to interpret is active behavior; what is assigned to the environment is passive behavior.

The relation between the decision-maker and the result: the result is the decision-maker's basis of judgment—the decision-maker judges on the basis of the result; this is exactly the other role the result bears.

The relation between the decision-maker and the fourfold is equally clear: the decision-maker is a distinguishable existence, so it is also a value.

## Part Five　Laws derived from the axioms

The axioms and levels of observation given above answer "what it is". This part derives two laws from the axioms, answering "what will happen": structures can be preserved and environments can grow; compositions are both objects of interpretation and participants in constructing new environments.

### 1. Structure preserved, environment grows

**Structures can be preserved, and environments can grow.**

This law is not a new axiom, but is derived from two already-established facts.

The first fact: a composition itself does not carry the answer to how it ultimately produces effects; effects are given by the environment's interpretation. The same structure entering different environments may receive different interpretations.

The second fact: observation-layer transition changes the structure's atomicity status, not the structure itself. The same structure gains different statuses at different observation layers, and the structure is not thereby changed.

Putting the two facts together yields this law: structure and interpretation are separate. Structures can stay stable; environments can grow amid change—new environments keep appearing, giving new interpretations of the same structure, while the structure does not change with the environment.

For example, a musical score. The same score, performed by different orchestras, produces different music. The score is unchanged; the performance differs with the orchestra.

What needs precise delimiting: what is "preserved" is the structure itself, not its concrete effects in some environment. The concrete effects in some environment differ with the environment—this is not a loss to the structure, but exactly the expression of the environment growing.

The significance of this law: what does not change and what does change can henceforth be managed separately. The later statement of this document's scope of application will use it as a fulcrum.

### 2. Compositions are both objects of interpretation and participants in constructing new environments

**Compositions are both objects of interpretation and participants in constructing new environments.**

This law is likewise not a new axiom. Its first half is a direct result of the environment's definition: environments provide interpretation, and compositions are structures awaiting interpretation. This half needs nothing new.

The second half is obtained by two steps of derivation. Step one: an environment is itself also a value—viewed from another observation layer, the environment as a whole can occupy the value position. Step two: the product of a composition is itself also a value. Hence, a composition, as a value, can become a constituent when constructing a new environment.

Here precise delimiting is likewise needed: "participating in construction", not "directly constructing". For a new environment to hold, it needs its own atomic values and atomic operations, and its own interpretation; compositions provide the building materials. Compositions participate; the new environment holds on that basis.

For example, a group of carpenters found a new carpentry workshop following the same blueprint. The blueprint is a composition; it participates in the construction of a new environment—the workshop; after the new workshop holds, it can interpret new structures.

Viewed together, the two laws answer questions in two directions: structure preserved answers "what does not change"; participating in constructing environments answers "where change comes from". Thus a closed loop of relations forms: compositions are interpreted, producing new compositions; new compositions participate in constructing new environments; new environments continue to interpret new compositions. This is not a sequential process, but the unfolding of the fourfold relations in growth.

## Part Six　Methods of judgment

The axioms and laws were established above. But one practical question remains: if new concepts are proposed in the future, how to judge whether they should enter this document's basic conceptual system? This part gives an operable method. It is the concretization of the growth guarantee—defined, then proof of why it must exist; this method is the process of that proof.

### 1. The concept review process and the five review questions

Faced with a new concept, no need to rush to decide. This document's approach is to pose it five questions in order.

**Question one: does it describe an existing fact, or a desired capability?**

If it is only a desired capability, by default it does not enter this document. This document describes only existing facts; desired capabilities can be built on facts, but they are not themselves this document's basic concepts.

**Question two: if it is deleted, does this document's basic conceptual system still close?**

If the system still closes after deletion, then it is most likely not necessary.

**Question three: if, after deletion, some equivalent concept must be reintroduced to restore the system, is it only the original under a new name?**

If so, the original concept still has indelibility—changing the name does not mean deleting it.

**Question four: does it bear an independent observation-layer role?**

If it is only a concrete form of some basic concept, it does not enter this document.

**Question five: does it depend on some concrete environment?**

If it does, it belongs to that environment, not to this document's basic conceptual system.

The five questions are posed in order. The conclusion: "useful" is not a qualification for entering this document; "indeletable" is.

### 2. The deletion experiment

Questions two and three of the five review questions unfold into a method usable on its own: the deletion experiment.

The deletion experiment proves a concept's irreducibility for this system, not that this system is the only possible ontology.

The procedure of the deletion experiment: imagine deleting the concept, then observe two things. First, whether the system still closes—if it still closes, the concept is not necessary. Second, if a gap appears after deletion, see whether whatever fills the gap is only this concept under another name—if so, the original concept is still indeletable.

The deletion experiment has been used above: Part Two proved that value, operation, composition, and environment cannot be deleted, and Part Four used it to explain why extended concepts are not basic concepts. Those uses displayed the results; this chapter establishes it as a method—from now on, whenever a concept is proposed, it can be tested with this.

What needs stating clearly is what the deletion experiment tests: it tests necessity, not usefulness. A concept can be very useful, but as long as the system still closes after its deletion, it is not this document's basic concept. "Might be useful later" cannot serve as its reason for entering this document—this was already settled in the growth guarantee.

## Part Seven　Applicability of the program

The above completed all the theoretical content of this document: axioms, levels of observation, extended concepts, laws, and methods of judgment. This part explains where this set of things can be used. The most direct use is guiding the design and development of computer software; further, as extended understanding, it also helps with aspects beyond software.

### 1. Guiding software design

The preface said this document's most direct use is guiding the design and development of computer software. Here is how it guides.

First, take apart entangled problems. In software design, many arguments have persisted for a long time, one reason being that "existing facts" and "desired capabilities" are discussed in the same dimension. Using question one of the five review questions, the two can first be separated: which are facts, which are capabilities. Facts cannot be cancelled by discussion; capabilities can be proposed, compared, and chosen. Once separated, many arguments no longer need to happen.

Second, keep attention on the unchanging basic relations. Concrete technologies keep being replaced—that was the predicament described in the preface. This document's answer: beneath the replacement of technologies, the closed relation of value, operation, composition, and environment does not change; concrete technologies are concrete forms of environments. When designing, keep structures on the basic relations, and structures will not lapse as concrete technologies become obsolete—this is exactly what "structure preserved, environment grows" directly means in software design.

Third, review new concepts with the methods of judgment. Whenever a new concept or practice is proposed in a design, test it with the five review questions and the deletion experiment: does it describe facts or capabilities? If deleted, does the system still close? "Might be useful later" is not a reason for admission. In this way the design is kept at the minimum necessary.

Fourth, put test, result, and decision-maker in their proper places. The test falls on the operation element, verifying the result of operations; the result falls on the value element, serving as the decision-maker's basis of judgment; the decision-maker falls on the environment element, judging on the basis of results. With the three in place, design verification gains a clear skeleton.

This chapter ends here: it only states directions of application, not concrete practices. Concrete practices are the follow-up work of those who use this document; they do not belong to this document.

### 2. Beyond software: extended understanding

The preface also said this set of regularities should apply equally to aspects beyond software. This chapter takes responsibility for that sentence, in the same manner as the whole document: only using examples to aid understanding, not arguing one by one.

Example one, physics. Nature is an environment: it provides physical facts as the basis of interpretation; elementary particles are atomic values, interactions are operations, and material structures are compositions. What this document acknowledges as facts is exactly the existence of this kind of environment.

Example two, board games. A board game is an environment: pieces are atomic values, moves are atomic operations, and winning/losing judgment is interpretation. The same pieces and board, interpreted by different environments, become different games—structure preserved, environment grows.

Example three, organizations. A company is an environment; departments and employees are values, collaboration and processes are operations, and team structures are compositions; the company can allow mature practices from lower levels to penetrate upward.

These examples all show the same thing: observing a domain with value, operation, composition, and environment reveals that domain's basic relations. This document does not claim those domains contain knowledge beyond this document; this document provides a way of observing. As for each domain's concrete contents, they belong to that domain itself.

What this document can do is now fully stated. Finally, state what this document cannot do.

## Part Eight　Boundaries of the program

### 1. Principles, not rules

This document's title is "principles", not "rules". The difference between principles and rules: rules prescribe "must be so"; principles explain "is so".

This document is an explanation and a declaration, not a set of constraints on behavior. This document commands no one to do anything; it provides a way of understanding things. Those who use this document may use it to observe, design, and judge—or not use it.

This positioning is consistent with the two guarantees: the feasibility guarantee determines that this document says "is so"—it describes existing facts; the growth guarantee determines that this document does not prescribe "must be so"—it defines only the necessary and presupposes nothing unnecessary. Principles only explain; they do not command.

### 2. What the program does not define

This document defines the fourfold, displays the extended concepts, and gives the laws and methods of judgment. Beyond that, what this document does not define can be grouped into three classes.

First, this document does not prescribe the contents of any concrete environment. What the atomic values and atomic operations concretely are, how interpretation is given, and where observation boundaries are drawn all belong to concrete environments, not to this document. This document only fixes their necessary positions, not their concrete contents.

Second, this document does not prescribe any concrete form. How structures are represented, stored, and passed is not presupposed here. The right to decide forms is in the hands of those who use this document and of concrete environments.

Third, this document does not prescribe practices in any concrete domain. Software design practices and the concrete contents of other domains are not within this document—the earlier text has declared each of these.

How to judge whether some content belongs to this document? Use the five review questions: whatever question five judges as "depending on a concrete environment" belongs to the environment, not to this document. The boundary is not a line this document draws for others to see, but the result drawn by the methods of judgment themselves.

Here, all the content of this document has been stated. This document starts from two guarantees, defines four axioms; explains levels of observation; gives extended concepts, two laws, and methods of judgment; and finally states applicability and boundaries. All the rest is left to environments and users to grow.
