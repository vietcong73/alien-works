# Naughty Dog / Uncharted Lisp-Family Architecture Review

**Status:** EVIDENCE_REVIEW_COMPLETE  
**Scope:** Public Naughty Dog / Uncharted Lisp-family architecture evidence  
**Audit branch:** `audit/lisp-foundation-reserve`  
**Purpose:** Architectural comparison input for the separate Lisp Foundation Reserve  
**Decision impact:** Informative only; this document does not decide STREAMLINE vs REBUILD.

---

## 1. Review question

This review asks a bounded question:

> What did Naughty Dog demonstrably use Lisp-family technology for in the Uncharted production stack, what architectural strengths and weaknesses are visible in the public evidence, and which patterns are transferable into a general-purpose Lisp-centric reserve?

This is **not** a reconstruction of proprietary Naughty Dog source code.

The review distinguishes:

1. **Primary public evidence** from Naughty Dog / GDC material.
2. **Corroborating public evidence** from reverse-engineering work.
3. **Architectural interpretation** made for the reserve project.

---

## 2. Evidence base

### Primary sources

1. **Dan Liebgold, GDC 2008 — _Adventures in Data Compilation: Uncharted: Drake's Fortune_**  
   https://media.gdcvault.com/gdc08/slides/S6636i1.pdf

2. **Jason Gregory, GDC 2009 — _State-Based Scripting in Uncharted: Drake's Fortune and Uncharted 2: Among Thieves_**  
   https://www.gameenginebook.com/resources/gdc09-statescripting-uncharted2.pdf

3. **GDC Vault session page — State-Based Scripting in UNCHARTED**  
   https://www.gdcvault.com/play/1730/State-Based-Scripting-in-UNCHARTED

4. **Uncharted 2 postmortem / production-scale material**  
   https://media.gdcvault.com/GD_Mag_Archives/GDM_March_2010.pdf

### Corroborating source

5. **icemesh/dc — public reverse-engineering reference for the DC/state-script format**  
   https://github.com/icemesh/dc

The reverse-engineering source is treated only as corroboration. Claims about Naughty Dog's production architecture are grounded primarily in the public GDC material.

---

## 3. Confirmed architectural facts

### 3.1 Naughty Dog retained a substantial Lisp lineage

The GDC 2008 material explicitly describes a studio history involving multiple Lisp-family systems, including Common Lisp, GOAL, GOOS, GOOL, scripting systems, and animation-related tools.

The significance is not historical trivia. Naughty Dog's later Scheme work was influenced by prior experience with Lisp-family language design and code/data integration.

**Assessment:** CONFIRMED.

---

### 3.2 PLT Scheme was used for both data definition and runtime scripting

The GDC 2009 material states that Naughty Dog made heavy use of:

- data definition;
- runtime scripting;

and that both were based on **PLT Scheme**.

This is the central architectural fact.

Scheme was not present merely as an external editor helper or small scripting convenience. It occupied a production layer between authored definitions and native engine execution.

**Assessment:** CONFIRMED.

---

### 3.3 Scheme occupied the code/data middle layer

The GDC 2008 material describes Scheme as particularly useful for the "in-between" region where information is:

- too structural or dynamic to be treated comfortably as passive static data;
- too declarative or content-oriented to justify direct hard-coding in C++.

The public material shows Scheme being used to define types and instances and then export or connect those definitions into the native C++ runtime.

This is a strong example of Lisp-family code/data duality being used as an architectural mechanism rather than merely as language syntax.

**Assessment:** CONFIRMED.

---

### 3.4 Publicly identified Scheme-driven domains

The GDC material explicitly identifies uses including:

- particle definitions;
- animation states and blend trees;
- gameplay scripting;
- gameplay tuning;
- scripted in-game cinematics;
- weapon tuning;
- sound / voice setup;
- overall game sequencing and control.

These uses collectively show that the Scheme layer had broad semantic responsibility across content and runtime behavior.

**Assessment:** CONFIRMED.

---

## 4. Runtime state-script architecture

### 4.1 State-based model

The runtime scripting model is state-oriented.

Public material describes scripts containing:

- attributes / script-level data;
- states;
- begin behavior;
- update behavior;
- end behavior;
- event handling;
- transitions.

This provides an explicit state/event/transition semantics rather than a loose sequence of arbitrary callback fragments.

**Assessment:** CONFIRMED.

---

### 4.2 Multiple deployment modes

State scripts could be used in several distinct roles.

#### Native-object extension

A script could be associated with a native C++ game object to extend or alter behavior.

#### Trigger-region behavior

A script could be associated with a trigger region and respond to entry, exit, or occupancy events.

#### Stand-alone director

A script process could exist independently of a single entity and coordinate multiple game objects.

#### Task / checkpoint orchestration

Scripts could be associated with task or checkpoint structures and were used to coordinate:

- AI encounters;
- player objectives;
- sequence progression.

This distinction is architecturally important. The stand-alone director pattern separates **coordination logic** from the objects being coordinated.

**Assessment:** CONFIRMED.

---

## 5. Runtime execution model

### 5.1 Scheme authoring did not imply unrestricted Scheme execution

The runtime language was deliberately constrained.

The authoring layer used Scheme-family syntax and metaprogramming, but runtime behavior was compiled or reduced into a smaller execution model.

This matters because the production design did not expose unrestricted language authority to every script author.

**Assessment:** CONFIRMED.

---

### 5.2 Custom VM and bytecode

The GDC 2009 material describes runtime tracks compiled into bytecode blocks referred to as lambdas.

The runtime execution state included mechanisms such as:

- current lambda;
- instruction position;
- register bank;
- call stack.

**Assessment:** CONFIRMED.

---

### 5.3 Continuation-style suspension and resume

The runtime used continuation-style behavior to support operations such as waiting and later resuming execution.

This allows script authors to express temporal sequences in a straightforward style while the runtime manages suspension and continuation.

For orchestration-heavy systems, this is a particularly transferable concept.

**Assessment:** CONFIRMED.

---

### 5.4 Native hooks into C++

Most primitive runtime operations ultimately called into native engine functionality.

Examples include operations around:

- animation;
- object lookup;
- spawning;
- task interaction;
- other engine services.

Therefore the execution authority can be summarized as:

```text
PLT Scheme authoring
        ↓
domain/state DSL
        ↓
compiled runtime representation / VM
        ↓
explicit native hooks
        ↓
C++ engine services
```

Scheme owned substantial orchestration semantics but did not replace the native engine substrate.

**Assessment:** CONFIRMED.

---

## 6. Representative production patterns

### 6.1 Breakable-object behavior

Public examples show state scripts coordinating:

- object state;
- animation;
- particles;
- events;
- transitions;
- completion logic.

The value is not the specific object. It demonstrates that a concise state definition can coordinate multiple engine capabilities while keeping object behavior legible.

---

### 6.2 Multi-track cinematic orchestration

The bus-crash example demonstrates coordinated tracks handling multiple concerns such as:

- vehicle animation;
- player animation;
- temporary player-control changes;
- NPC spawning;
- NPC behavior / death;
- dialogue;
- synchronization using waits and signals.

The key architectural property is that concurrency is presented to authors as comprehensible sequential tracks rather than exposed as low-level engine concurrency primitives.

This is better described as a **temporal orchestration DSL** than as a conventional "game script".

---

## 7. Production scale

The Uncharted 2 postmortem reports approximately:

- 1.11 million source lines of code;
- 3,703 script files;
- approximately 708,951 script lines, including render settings and particles;
- a large production team.

### Important caveat

The postmortem does **not** establish that every reported script line was Scheme.

Therefore this review does not equate the total script-line count with Scheme LOC.

The scale evidence supports only the narrower claim that the scripting/data-definition layer was a major production subsystem rather than a toy or isolated experiment.

**Assessment:** CONFIRMED WITH CAVEAT.

---

## 8. What the architecture did well

### 8.1 It gave Lisp-family technology a clear semantic job

The strongest feature is not "using Scheme".

The strongest feature is that Scheme was placed where its language properties were unusually useful:

- code/data duality;
- macros;
- symbolic definitions;
- compact domain syntax;
- fast iteration;
- declarative behavior with executable semantics.

This avoided using Lisp merely for ideological language uniformity.

---

### 8.2 It created bounded domain languages

Naughty Dog did not simply give all users arbitrary access to the entire engine.

Instead, Scheme was used to create constrained languages and definitions for specific domains.

This is a major transferable lesson:

> A powerful Lisp host can expose smaller purpose-built languages rather than exposing raw implementation authority everywhere.

---

### 8.3 It separated orchestration from primitive execution

The Lisp-family layer expressed:

- what should happen;
- when it should happen;
- what state should transition;
- which entities should be coordinated.

The C++ engine performed most primitive heavy operations.

This is a clean separation between **semantic orchestration** and **execution capability**.

---

### 8.4 The director pattern is highly reusable

A coordinator that is not itself one of the domain objects is useful far outside games.

Potential mappings include:

| Uncharted concept | General reserve mapping |
|---|---|
| stand-alone director | workflow / mission / experiment supervisor |
| state script | domain state machine |
| native engine hook | capability adapter |
| task/checkpoint | workflow stage / mission phase |
| trigger | event source |
| AI encounter orchestration | multi-agent / subsystem coordination |
| objective control | policy / goal supervision |

This is one of the strongest patterns to carry into the reserve.

---

### 8.5 Temporal logic remained readable

Tracks, waits, signals, state transitions, and continuation-style execution allowed complex temporal choreography to remain relatively readable.

This is transferable to:

- experimental procedures;
- automation workflows;
- mission sequencing;
- asynchronous device control;
- agent coordination.

---

## 9. What the architecture did poorly or constrained deliberately

### 9.1 Lisp remained subordinate to the native engine ontology

Scheme had broad behavioral authority but did not own the full application ontology.

If a capability was not represented or exposed through the native engine, the scripting layer could not independently redefine the engine substrate.

For the future reserve, this is too restrictive as a master architecture.

---

### 9.2 Cross-language boundaries increase debugging cost

Potential failure locations include:

- Scheme authoring layer;
- DSL semantics;
- compiler / translator;
- VM;
- native binding;
- C++ implementation;
- authored content.

This creates a multi-layer debugging tax.

A reserve adopting this pattern must provide strong provenance and boundary diagnostics.

---

### 9.3 Production empowerment creates authority risk

The public presentation explicitly emphasizes the need for strong debugging and tooling.

Script authors can create severe production failures, including:

- broken progression;
- timing errors;
- sequencing errors;
- invalid object assumptions;
- synchronization failures;
- build-breaking content;
- crashes.

This is a direct argument for:

- capability containment;
- validation;
- static or pre-runtime checks where possible;
- clear ownership boundaries;
- runtime diagnostics.

---

### 9.4 Race and lifecycle problems remain

State/event systems do not remove temporal hazards.

Public material identifies problems such as:

- events arriving too early or too late;
- target objects not yet spawned;
- target objects already destroyed;
- tracks becoming unsynchronized.

A state DSL can make logic clearer, but it does not abolish distributed-state and lifecycle problems.

---

### 9.5 Runtime expressiveness was intentionally limited

The bounded VM improved predictability and authorability, but it reduced:

- unrestricted reflection;
- architecture-level redefinition;
- live ontology changes;
- arbitrary metaprogramming at runtime.

For a general Lisp reserve, this restriction should apply to **domain DSL users**, not necessarily to the governed Lisp host itself.

---

## 10. Transferable design decisions for the Lisp Foundation Reserve

### KEEP

- Lisp-family code/data middle layer.
- Macro-defined domain-specific languages.
- Explicit state / event / transition semantics.
- Stand-alone director / supervisor process.
- Temporal tracks with wait / signal style coordination.
- Compile / validate before runtime execution.
- Explicit native capability hooks.
- Strong tooling and debugging requirements.
- Authority containment for domain authors.

### ADAPT

- The custom VM concept should be adapted only if a smaller runtime representation is actually required.
- Continuation-style scheduling should be evaluated against Common Lisp-native control mechanisms, coroutines/tasks, and explicit schedulers.
- Native-hook lookup should become a typed/capability-aware adapter registry rather than a game-engine command table.

### DO NOT COPY

- Naughty Dog proprietary source.
- Game-specific object models.
- Game-specific commands or assets.
- The assumption that Lisp must remain subordinate to a C++ master ontology.
- A custom bytecode VM merely because Uncharted used one.

---

## 11. Implication for the reserve architecture

The strongest synthesis is:

```text
Common Lisp host authority
        ↓
governed semantic core
        ↓
bounded domain DSLs
        ↓
state / event / director orchestration
        ↓
explicit capability adapters
        ↓
native execution substrates
```

This differs from Uncharted in one crucial way:

> The future reserve should keep Common Lisp as the sovereign host while using Uncharted-style bounded DSLs as containment layers.

This preserves the strongest Lisp properties:

- REPL;
- introspection;
- macros;
- live redefinition;
- CLOS;
- symbolic structures;

without giving every domain script unrestricted authority over the whole system.

---

## 12. Revised assessment versus the initial hypothesis

The initial discussion understated the depth of the Uncharted Lisp-family layer.

It is inaccurate to describe it as merely a peripheral scripting system.

A better classification is:

> **bounded semantic control layer over a native engine substrate**

Evidence supporting that classification includes:

- type/instance definition;
- code/data compilation;
- runtime state semantics;
- stand-alone directors;
- task/checkpoint control;
- AI encounter orchestration;
- player objective control;
- cinematic and multi-object temporal sequencing.

At the same time, it was not a Lisp-sovereign engine architecture.

Therefore:

```text
Not:
    "Scheme as a small script language"

Not:
    "Uncharted written in Scheme"

Better:
    "A production-scale Scheme-derived semantic definition and orchestration layer
     embedded in a native AAA engine."
```

---

## 13. Evidence confidence

| Claim | Confidence |
|---|---|
| PLT Scheme used for data definition | HIGH |
| PLT Scheme used for runtime scripting | HIGH |
| Lisp heritage influenced architecture | HIGH |
| Code/data duality was an explicit design reason | HIGH |
| State/event/transition scripting | HIGH |
| Stand-alone director scripts | HIGH |
| AI encounter / objective orchestration | HIGH |
| Custom bytecode VM | HIGH |
| Continuation-style wait/resume | HIGH |
| C++ native hooks | HIGH |
| 708,951 reported script LOC all being Scheme | NOT SUPPORTED |
| Reverse-engineered DC repo matching production implementation exactly | NOT ESTABLISHED |

---

## 14. Review verdict

**Disposition:** ARCHITECTURALLY VALUABLE / ADOPT PATTERNS, NOT IMPLEMENTATION

The Naughty Dog evidence is sufficient to use Uncharted as a serious architectural reference for the Lisp Foundation Reserve.

The most valuable transferable ideas are:

1. **code/data middle-layer design;**
2. **bounded Lisp-family DSLs;**
3. **state/event/transition semantics;**
4. **stand-alone director processes;**
5. **temporal coordination through waits/signals;**
6. **explicit native capability hooks;**
7. **validation/debugging as mandatory infrastructure.**

The main design constraint to reject is the assumption that Lisp must remain subordinate to a native master ontology.

For the reserve, the preferred direction remains:

> **Lisp sovereignty with Uncharted-style containment.**

---

## 15. Next audit block

The next evidence block should inspect the actual `alien-works` source tree at file/subsystem level and classify components as:

- KEEP CORE
- KEEP OPTIONAL PROFILE
- ADAPT
- REPLACE
- DROP GAME-SPECIFIC
- DEFER

Only after that audit should the project decide whether the reserve should be a streamlined derivative or a clean rebuild.
