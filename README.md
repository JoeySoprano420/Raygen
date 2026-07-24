# Raygen Programming Language

## Unified Normative and Systems Specification

### Raygen 1 — Maximum-Throughput Consolidated Edition

**Document Identifier:** RG-UNIFIED-1.0
**Language Edition:** Raygen 1
**Revision:** 2026-07-23
**Specification Status:** Authoritative Normative Core
**Official Name:** Raygen
**Meaning:** Ray Generator
**Source Extension:** `.rg1`
**Primary Compilation Mode:** Ahead-of-time native compilation
**Optional Execution Mode:** Verified Raygen VM
**Primary Design Goal:** Maximum deterministic native throughput without sacrificing statically provable correctness

---

# Part 1 — Authority, Identity, Design Foundation, and Conflict Resolution

## 1.1 Language Identity

Raygen is a hardware-aware, execution-oriented, statically typed systems programming language designed for:

* native machine-code generation;
* predictable low-level behavior;
* explicit ownership and memory intent;
* deterministic control flow;
* high-throughput scalar and vector execution;
* auditable concurrency;
* compile-time deduction;
* direct mapping between source intent and generated operations;
* minimal mandatory runtime support.

A geometric ray begins at a known origin, proceeds in a defined direction, and reaches a traceable destination.

Raygen applies that model to program execution:

> Every operation begins from a known state, proceeds through an explicit direction, and produces a traceable result.

The ray is Raygen’s semantic execution model.

It is not:

* a mandatory runtime object;
* an allocation header;
* a pointer chain;
* a linked execution node;
* a heap representation;
* a required metadata structure;
* a physical machine abstraction imposed upon every value.

A conforming compiler SHALL represent programs using whichever internal form produces the best correct code.

The ray model constrains meaning, not physical storage.

---

## 1.2 Foundational Principle

The unified foundation of Raygen is:

> Meaning comes first.
> Safety constrains execution.
> Ownership constrains access.
> Concurrency constrains visibility.
> Layout constrains representation.
> Deduction removes only proven work.
> Optimization changes implementation, never meaning.
> Explicit programmer intent outranks compiler preference.
> Hardware specialization is permitted wherever semantics remain unchanged.

This foundation resolves the central tension between Raygen’s safety model and its maximum-performance objective.

Raygen does not obtain safety by forcing every operation through a runtime abstraction.

Raygen obtains safety primarily through:

* static typing;
* ownership analysis;
* lifetime analysis;
* provenance analysis;
* alias analysis;
* concurrency-rights verification;
* compile-time deduction;
* layout validation;
* explicit unsafe boundaries.

When correctness is proven statically, the corresponding runtime mechanism SHALL be removable.

---

## 1.3 Document Authority

This document is the sole authoritative specification for Raygen 1.

It consolidates and supersedes:

* the original Raygen language proposal;
* the hardware-aware professional language specification;
* the expanded execution-oriented specification;
* the Raygen 1 Normative Core Parts 1 through 10;
* later grammar corrections;
* later ownership and concurrency corrections;
* later conformance and diagnostic requirements;
* performance clarifications;
* VM and native-compilation clarifications;
* all prior examples, design notes, and amendments.

Earlier versions remain useful as historical and explanatory material.

They SHALL NOT override this specification.

Where an earlier version describes a feature absent from another version, that feature is retained when it can be integrated without violating:

1. semantic consistency;
2. type safety;
3. ownership correctness;
4. concurrency correctness;
5. deterministic failure behavior;
6. ABI correctness;
7. the maximum-throughput design objective.

Where two earlier rules conflict, the resolution defined by this document is normative.

---

## 1.4 Consolidation Rule

The unified language uses an **inclusive consolidation rule**.

A previously described feature SHALL be retained when all of the following are true:

* its semantics can be stated deterministically;
* it does not duplicate another feature with incompatible meaning;
* it can be type-checked or explicitly confined to unsafe execution;
* it does not force unnecessary runtime overhead upon unrelated programs;
* it can participate in separate compilation;
* it can be lowered to native code without requiring a universal runtime object model.

A feature SHALL be revised rather than discarded when its intent is valuable but its earlier definition is incomplete.

A feature SHALL be removed only when:

* it is irreconcilably ambiguous;
* it duplicates a stronger canonical mechanism;
* it fundamentally contradicts memory safety;
* it requires hidden global runtime behavior;
* it prevents deterministic native lowering;
* it would weaken the language’s throughput model for all programs.

---

## 1.5 Canonical Conflict-Resolution Order

When multiple rules apply, the following precedence order SHALL govern interpretation:

1. **Lexical and grammatical validity**
2. **Declared language semantics**
3. **Static type correctness**
4. **Ownership, borrowing, and lifetime correctness**
5. **Control-flow and failure-flow correctness**
6. **Concurrency rights and memory-model correctness**
7. **Explicit semantic policies**
8. **Explicit safety policies**
9. **Mandatory ABI contracts**
10. **Mandatory representation and layout contracts**
11. **Pointer provenance and alias legality**
12. **Validated derivation dependencies**
13. **Validated deduction results**
14. **Required optimization contracts**
15. **Preferred optimization policies**
16. **Profile-guided decisions**
17. **Target-specific cost models**
18. **Compiler heuristics**

No lower-precedence rule may invalidate a higher-precedence rule.

Examples:

* Vectorization SHALL NOT create an ownership violation.
* A forced layout SHALL NOT violate a declared foreign ABI.
* Deduction SHALL NOT remove a bounds check unless the relevant range proof remains valid after alias and layout analysis.
* Task fusion SHALL NOT weaken synchronization visibility.
* Inlining SHALL NOT change destruction order.
* A target cost model SHALL NOT change overflow behavior.
* Profile-guided dispatch SHALL NOT remove a required mapping case.
* Native lowering SHALL NOT change denial propagation.
* A safety profile SHALL NOT silently change a valid program’s mathematical result.
* A diagnostic preference SHALL NOT suppress a mandatory language error.

---

## 1.6 Resolution of the Layout-and-Policy Precedence Conflict

Earlier Raygen material placed layout and semantic policies in different relative positions.

The unified rule is:

### Semantic meaning precedes representation.

Therefore:

* arithmetic behavior;
* failure behavior;
* observable ordering;
* concurrency visibility;
* numerical reproducibility;
* destruction;
* externally visible side effects

are resolved before representation preferences.

However, mandatory foreign ABI and stable-layout declarations constrain which representations are legal.

The final ordering is therefore:

> Semantic policy determines what the operation means.
> Mandatory ABI and layout determine which representations may express that meaning.
> Optimization chooses the fastest legal representation.

A layout declaration cannot change the value-level meaning of a program.

A semantic policy cannot require a representation forbidden by a mandatory ABI without causing a compilation error.

---

## 1.7 Resolution of the 256-Bit Architecture Conflict

Raygen retains native 256-bit language support without forcing every value into 256-bit storage.

The following are distinct:

### Exact-width types

Examples:

```rg1
u8
u16
u32
u64
u128
u256

i8
i16
i32
i64
i128
i256

f16
f32
f64
f128
```

These types possess their declared widths.

### Native scalar types

```rg1
uint
int
real
address
usize
isize
```

These use the efficient native representation defined by the active target profile.

### Rayword

```rg1
rayword
```

`rayword` is Raygen’s preferred wide computational unit.

Its logical width is 256 bits.

A target may implement it using:

* one native 256-bit register;
* multiple scalar registers;
* a vector register;
* paired 128-bit registers;
* a compiler-managed software sequence;
* another semantically equivalent representation.

### Vector types

```rg1
vector<T, Lanes>
vector_bits<T, Width>
```

Vectors are distinct from `rayword`.

A `rayword` is a general wide value.

A vector is a lane-structured computational type.

A conforming implementation SHALL NOT widen every scalar to 256 bits merely because Raygen supports `rayword`.

This prevents:

* excessive stack use;
* register pressure;
* cache waste;
* unnecessary memory bandwidth;
* poor calling-convention compatibility;
* degraded scalar performance.

The compiler SHALL use the narrowest representation consistent with the declared type and applicable policy.

---

## 1.8 Performance Doctrine

Raygen’s maximum-throughput model is based on the following rule:

> Pay only for semantics the program actually uses.

A conforming production compiler SHALL attempt to eliminate:

* proven bounds checks;
* redundant null checks;
* redundant ownership metadata;
* redundant retain or release operations;
* unnecessary temporary objects;
* unnecessary VM dispatch;
* unnecessary task boundaries;
* unnecessary synchronization;
* unnecessary abstraction dispatch;
* redundant denial tagging;
* redundant layout conversions;
* dead derivation nodes;
* duplicate range checks;
* unnecessary scalarization;
* unnecessary widening;
* unnecessary memory clearing;
* unused runtime services.

Such removal is legal only when program meaning remains unchanged.

The following facilities are designed to be zero-cost when statically resolved:

* generics;
* traits using static dispatch;
* ownership;
* borrowing;
* lifetimes;
* derivation;
* deduction;
* direct maps;
* fixed arrays;
* slices;
* regions;
* access rights;
* exhaustive selections;
* denial propagation;
* compile-time contracts;
* stable layouts;
* ordered layers whose order is statically known.

---

## 1.9 Maximum Raw Throughput Requirements

A professional Raygen implementation SHALL provide an optimization pipeline capable of:

* constant folding;
* constant propagation;
* dead-code elimination;
* dead-store elimination;
* copy propagation;
* move elision;
* return-value optimization;
* scalar replacement of aggregates;
* stack allocation through escape analysis;
* region allocation;
* loop invariant code motion;
* loop fusion;
* loop fission;
* loop interchange;
* loop unrolling;
* automatic vectorization;
* explicit vector lowering;
* software pipelining;
* instruction scheduling;
* branch folding;
* branchless selection where profitable;
* direct-map specialization;
* jump-table generation;
* perfect-hash generation;
* devirtualization;
* generic monomorphization;
* cross-module inlining;
* whole-program optimization;
* link-time optimization;
* profile-guided optimization;
* cache-aware layout transformation where permitted;
* structure-of-arrays transformation where permitted;
* task fusion;
* work partitioning;
* deterministic parallel reduction;
* target-specific intrinsic selection;
* register allocation;
* tail-call optimization where semantically legal;
* native instruction selection;
* elimination of unused runtime facilities.

An implementation may provide additional optimizations.

No optimization is mandatory when it would increase execution cost on the selected target.

“Optimization support” does not mean blindly applying every transformation.

It means selecting the fastest legal implementation according to:

* target architecture;
* cache hierarchy;
* register file;
* instruction latency;
* instruction throughput;
* branch behavior;
* data layout;
* profile information;
* programmer policies;
* code-size restrictions.

---

## 1.10 Native Execution Is Canonical

Raygen’s canonical high-performance execution mode is ahead-of-time native compilation.

The canonical production pipeline is:

```text
Raygen source
    ↓
lexical token stream
    ↓
parsed semantic structure
    ↓
typed ownership-aware representation
    ↓
validated derivation graph
    ↓
deduction and proof facts
    ↓
optimized target-independent representation
    ↓
assembly serial or equivalent low-level representation
    ↓
symbol and relocation resolution
    ↓
native object code
    ↓
linked executable or library
```

The named historical pipeline remains valid:

```text
source
    → token.map
    → asm.serial
    → tag.assign
    → native code
```

These stage names describe externally understandable compiler artifacts.

They do not require the compiler to use text files between stages.

A production compiler may retain these representations entirely in memory.

---

## 1.11 Raygen VM Role

The Raygen VM is retained as an optional verified execution target.

It is intended for:

* development;
* rapid startup of compiler implementations;
* sandboxing;
* portable bytecode distribution;
* deterministic testing;
* compile-time execution;
* educational use;
* debugging;
* environments without a native backend;
* staged or tiered compilation.

The VM SHALL NOT be mandatory for native executables.

A native Raygen program may contain no VM dispatcher and no VM bytecode.

The VM uses a typed register-and-stack hybrid model and may provide:

* native-width scalar registers;
* exact-width registers;
* 256-bit rayword registers;
* typed vector registers;
* call frames;
* region frames;
* a constant pool;
* symbolic tag tables;
* case scheduling;
* layer execution;
* denial-state transport;
* ownership metadata only where runtime enforcement is required.

The VM SHALL NOT force every value into a 256-bit slot.

Production implementations MAY:

* interpret bytecode;
* translate bytecode ahead of time;
* tier from interpretation to native code;
* remove unused VM facilities;
* bypass the VM completely.

---

## 1.12 Runtime Model

Raygen does not mandate a universal garbage collector.

Raygen does not mandate a universal exception runtime.

Raygen does not mandate reflection metadata.

Raygen does not mandate dynamic dispatch tables.

Raygen does not mandate a scheduler when concurrency is unused.

Raygen does not mandate a heap when allocation is unused.

Raygen does not mandate unwinding when the selected denial ABI uses explicit return transport.

Runtime facilities are linked according to actual program requirements.

A minimal freestanding Raygen executable may consist of:

* generated machine code;
* static data;
* the target entry point;
* explicitly selected intrinsic support.

A hosted program may additionally use:

* allocation;
* I/O;
* threading;
* channels;
* filesystem support;
* networking;
* dynamic libraries;
* runtime reflection;
* asynchronous scheduling.

---

## 1.13 Explicitness Without Forced Verbosity

Raygen preserves its explicit execution-oriented syntax.

Its core instructions include:

```rg1
let
var
set
call
emit
return
read
write
move
copy
compare
jump
select
sort
merge
split
spawn
wait
stop
```

Function calls are expressions.

Therefore:

```rg1
calculate(value)
```

is canonical.

The explicit form:

```rg1
call calculate(value)
```

is also retained.

It is an instruction form emphasizing that the result is intentionally discarded or that invocation itself is the operation.

The two forms are resolved as follows:

```rg1
let result = calculate(value)
```

uses a call expression.

```rg1
call perform_job()
```

uses a call statement and discards a `void` result.

A non-`void` result SHALL NOT be silently discarded unless:

* the return type permits discard;
* an explicit `dismiss` operation is used;
* a diagnostic policy authorizes intentional discard.

This preserves both earlier forms without ambiguity.

---

## 1.14 Canonical Directives

Raygen retains the full directive system.

Core directives are:

```rg1
retain
store
dismiss
free
allow
deny
align
map
derive
deduce
order
layer
case
```

Additional standardized directives may include:

```rg1
region
policy
layout
atomic
foreign
direct
vector
parallel
merge
require
assume
verify
```

Instructions perform operations.

Directives constrain or organize how operations are interpreted, validated, stored, ordered, or lowered.

A directive SHALL NOT secretly execute unrelated runtime behavior.

---

## 1.15 Memory-Intent Resolution

The original memory-intent verbs are retained and assigned distinct canonical meanings.

### `retain`

Extends a value’s ownership or storage eligibility according to a valid ownership mechanism.

It does not automatically create reference counting.

### `store`

Places a value into a declared destination, storage class, region, stash, field, or persistent location.

### `dismiss`

Explicitly abandons a value, result, temporary, denial, or resource where its type permits dismissal.

Destruction still occurs when required.

### `free`

Terminates an explicitly managed allocation or region resource.

`free` is legal only for storage whose ownership contract permits explicit release.

These directives SHALL NOT all mean “deallocate.”

Their separation preserves the original memory model while preventing overlapping semantics.

---

## 1.16 Ownership and Performance

Every owned runtime value has one active owner unless its type declares another standardized ownership model.

Ordinary safe ownership is implemented through compile-time legality, not mandatory runtime reference counting.

The compiler SHALL elide ownership bookkeeping when:

* ownership is statically known;
* destruction points are statically known;
* no dynamic ownership facility is used;
* no debugging or safety profile requires runtime tracking.

Borrowing SHALL normally compile to an ordinary address, register reference, or no runtime entity at all.

Lifetimes SHALL not exist at runtime unless required by:

* dynamic verification;
* reflection;
* debugging;
* foreign interfaces;
* a selected profile.

---

## 1.17 Failure-Flow Resolution

Raygen defines one canonical fallible type:

```rg1
outcome<Allowed, Denied>
```

The preferred surface form is:

```rg1
allow<T> deny<E>
```

The following are equivalent:

```rg1
function load() -> outcome<Data, IoError>
```

```rg1
function load() -> allow<Data> deny<IoError>
```

Construction uses:

```rg1
return allow value
return deny error
```

A successful void result uses:

```rg1
return allow
```

The language does not use hidden exception propagation for ordinary denial flow.

An implementation may represent an outcome through:

* registers;
* condition flags;
* tagged values;
* spare-bit encodings;
* separate return channels;
* ABI-specific multiple returns;
* branch specialization.

The representation SHALL be chosen for maximum performance while preserving the declared result semantics.

Where the compiler proves that denial is impossible, it may eliminate the denial tag and branch.

Where the caller immediately propagates denial, the compiler may fuse the callee and caller control flow.

---

## 1.18 Arithmetic Resolution

Raygen supports these arithmetic policies:

```rg1
deny
wrap
saturate
clamp
report
widen
```

Their meanings are fixed:

* `deny` rejects statically proven overflow and produces defined denial behavior for unresolved runtime overflow.
* `wrap` performs modular arithmetic.
* `saturate` clamps to the numeric type’s minimum or maximum.
* `clamp` clamps to an explicitly declared programmer range.
* `report` returns an outcome describing success or arithmetic failure.
* `widen` evaluates in a defined wider type.

No implicit numeric widening, narrowing, or signedness conversion exists in ordinary typed expressions.

However, constant literals begin as abstract compile-time numbers and may be assigned directly when representable.

This gives exact arithmetic semantics without forcing unnecessary conversion instructions.

---

## 1.19 Derivation Resolution

Raygen retains two distinct systems that earlier material sometimes placed under the same word.

### Semantic derivation

A semantic derivation declares value dependencies:

```rg1
derive final_price from base_price, tax_rate, discount
{
    taxed = base_price * tax_rate
    final_price = base_price + taxed - discount
}
```

The compiler constructs a dependency graph.

A cycle is illegal unless the declaration explicitly selects a supported dynamic fixed-point or feedback mechanism.

### Trait or implementation derivation

A type may request standardized generated behavior:

```rg1
record Point
derive Equality, Hash
{
    x: i32
    y: i32
}
```

This generates trait implementations.

These systems share declarative intent but not semantic machinery.

They SHALL be diagnosed and resolved independently.

---

## 1.20 Static and Dynamic Derivation

Static derivation is resolved during compilation whenever the dependency structure is known.

Static derivation may be:

* inlined;
* constant-folded;
* cached;
* incrementally recomputed;
* fused with consumers;
* removed when unused.

Dynamic derivation is permitted when dependencies are established at runtime.

Dynamic derivation SHALL use an explicit runtime graph or scheduler only for the dynamic portion.

A program using only static derivation SHALL NOT pay for a dynamic dependency engine.

Cycles in static derivation produce:

```text
RG-DERIVE-001:
Cyclic derivation detected.
```

Dynamic cycles SHALL follow an explicitly selected policy such as:

```rg1
deny_cycle
fixed_point
bounded_iterations
feedback_delay
```

No implicit infinite recomputation is permitted.

---

## 1.21 Deduction Resolution

`deduce` performs bounded compile-time reasoning.

It may establish facts concerning:

* numeric ranges;
* non-nullness;
* alignment;
* alias separation;
* branch reachability;
* mapping completeness;
* loop bounds;
* ownership state;
* denial impossibility;
* derivation stability;
* vector legality;
* memory-order redundancy;
* task independence.

Deduction is not an unrestricted theorem prover.

A failed deduction SHALL NOT make an otherwise valid program invalid unless the programmer used `require`.

Example:

```rg1
deduce index in 0 ..< values.length
```

If proven, a bounds check may be removed.

If not proven:

* the compiler retains the runtime check;
* uses a fallback;
* or denies compilation only when the proof was explicitly required.

The forms are:

```rg1
deduce condition
```

Best-effort proof.

```rg1
require deduce condition
```

Mandatory proof.

```rg1
assume condition
```

Unsafe programmer assertion unless verified by the active profile.

---

## 1.22 Direct Mapping Resolution

Direct mapping remains a core Raygen feature:

```rg1
map command direct
{
    "start" -> start_engine
    "stop"  -> stop_engine
    "reset" -> reset_engine
    other   -> unknown_command
}
```

A direct map declares semantic mapping, not a mandatory implementation strategy.

The compiler SHALL choose among legal strategies including:

* direct jump table;
* compressed jump table;
* perfect hash;
* ordinary hash dispatch;
* binary search;
* decision tree;
* computed branch;
* SIMD comparison;
* branchless lookup;
* profile-guided hot/cold partitioning.

The compiler may combine strategies.

The selected strategy SHALL preserve:

* key equality semantics;
* exhaustiveness;
* case priority where priority is defined;
* failure behavior;
* observable side effects.

This feature therefore provides direct control topology without locking Raygen to a single hardware-hostile representation.

---

## 1.23 Concurrency Resolution

Raygen supports:

* tasks;
* `spawn`;
* `wait`;
* joins;
* channels;
* locks;
* atomic storage;
* explicit memory orderings;
* cases;
* case groups;
* ordered layers;
* parallel layers;
* barriers;
* deterministic merges;
* cancellation;
* isolated execution paths.

These facilities form one unified concurrency system.

### Tasks

Tasks are general independent execution contexts.

### Cases

Cases are rights-declared concurrent branches.

A case explicitly defines:

* inputs;
* outputs;
* ownership transfers;
* borrowed rights;
* cancellation behavior;
* completion behavior.

### Layers

Layers define execution dependencies between groups of operations.

Operations within a parallel layer may execute concurrently when proven independent.

Layers are ordered relative to one another.

### Isolated paths

An isolated path is a task or case with no shared mutable access except through declared channels, atomics, or synchronized capabilities.

No construct weakens the ownership system.

Parallelism is explicit or compiler-generated only under a policy that permits proven semantic equivalence.

---

## 1.24 Determinism Levels

Raygen defines multiple determinism levels because requiring identical schedules for every program would unnecessarily reduce throughput.

### `result`

The program must produce the same defined result for the same inputs, but internal scheduling may vary.

### `merge`

Parallel partial results must be combined in a declared deterministic order.

### `schedule`

Task and layer execution order must conform to a declared schedule.

### `bitwise`

Numerical and memory-visible results must be bit-for-bit reproducible across conforming executions within the target profile.

### `none`

The program accepts permitted scheduling and numerical variation while remaining data-race-free and memory-safe.

The default hosted profile uses `result`.

Performance-sensitive applications may select `none` or `merge`.

Safety-critical and reproducible-science profiles may select `schedule` or `bitwise`.

---

## 1.25 Evaluation Order and Optimization

The source-language evaluation order is left to right for operations whose order is observably relevant.

The compiler may reorder operations only when it proves that the reordering cannot alter:

* returned values;
* denial behavior;
* volatile access;
* atomic behavior;
* synchronization;
* I/O;
* destruction;
* foreign calls;
* observable memory state;
* required numerical reproducibility.

Pure independent expressions may therefore be:

* reordered;
* vectorized;
* fused;
* executed concurrently;
* evaluated at compile time.

This preserves deterministic meaning without unnecessarily serializing pure computation.

---

## 1.26 Layout Freedom

Types use one of these layout classes:

```rg1
implementation
optimized
stable
packed
foreign
vector
device
```

### `implementation`

The compiler has ordinary representation freedom.

### `optimized`

The compiler may reorder or transform representation for performance when observable semantics remain unchanged.

### `stable`

The representation is fixed by the Raygen stable-layout specification.

### `packed`

Padding is reduced or removed according to declared alignment rules.

Misaligned access SHALL be lowered safely for the target.

### `foreign`

The representation follows a named external ABI.

### `vector`

The representation is selected for vector processing.

### `device`

The representation follows device-memory and address-space rules.

Ordinary safe types default to `implementation`.

Internal data used only within one optimization unit may use `optimized`.

This allows cache-aware representation without silently breaking ABI or persistence.

---

## 1.27 Abstract Structure Versus Physical Layout

Raygen retains its first-class structural vocabulary:

```rg1
record
unit
choice
list
array
nest
node
branch
tree
graph
table
slice
buffer
region
stash
```

These names define semantic structures.

They do not always dictate one physical layout.

For example, a `tree<T>` may be represented through:

* pointers;
* indices;
* flat arrays;
* arena offsets;
* compact nodes;
* structure-of-arrays storage;
* cache-oblivious layouts.

The compiler may choose an alternative representation only when:

* layout is not externally fixed;
* address identity is not observably required;
* pointer stability is preserved where required;
* complexity contracts remain satisfied;
* unsafe code does not depend on the original representation.

This resolution preserves Raygen’s rich structures while avoiding pointer-heavy mandatory layouts.

---

## 1.28 Sorting and Tables

Sorting remains a built-in semantic operation.

Raygen may provide:

```rg1
sort stable
sort rapid
sort parallel
sort vector
sort radix
sort key
sort multi_key
sort compile_time
```

Names describe required properties or preferred strategies.

`stable` requires stability.

`rapid` requests the fastest legal general strategy but does not guarantee a specific algorithm.

The compiler or library may select based upon:

* element type;
* key width;
* count;
* target cache;
* vector support;
* parallel threshold;
* stability requirement;
* available memory.

Tables support:

```rg1
where
order
group
select
join
index
```

A compiler may fuse table operations into one pipeline.

Intermediate tables SHALL be removable when not observably required.

---

## 1.29 Unsafe Resolution

Safe Raygen programs SHALL NOT exhibit undefined behavior.

Unsafe operations are confined to explicit boundaries:

```rg1
unsafe
{
    ...
}
```

Unsafe code remains obligated to preserve:

* valid object lifetime;
* valid pointer provenance;
* valid alignment;
* valid aliasing;
* valid initialization;
* valid synchronization;
* valid ABI use;
* valid ownership transitions.

Violation results in undefined behavior.

Safety checks not relevant to the unsafe operation remain active.

An unsafe block is not a global optimization-disable switch and is not permission for the compiler to invent behavior.

---

## 1.30 Profiles

Raygen defines cumulative conformance profiles.

### Raygen Core

Includes:

* lexical grammar;
* syntax;
* scalar types;
* functions;
* control flow;
* outcomes;
* basic composites;
* modules;
* deterministic evaluation.

### Raygen Native

Adds:

* native-code generation;
* exact layouts;
* regions;
* raw memory;
* FFI;
* ABI conformance;
* atomics;
* direct hardware facilities.

### Raygen Concurrent

Adds:

* tasks;
* cases;
* layers;
* channels;
* locks;
* barriers;
* cancellation;
* memory ordering;
* deterministic merges.

### Raygen Professional

Adds:

* generics;
* traits;
* interfaces;
* semantic derivation;
* implementation derivation;
* deduction;
* advanced policies;
* vectorization;
* profile-guided optimization;
* cross-module optimization;
* full diagnostic records.

### Raygen Safety-Critical

Requires a restricted, auditable configuration including:

* bounded allocation where required;
* bounded recursion;
* complete denial handling;
* restricted unsafe execution;
* deterministic scheduling;
* reproducible builds;
* audited foreign interfaces;
* conformance evidence;
* stable diagnostic categories;
* validated toolchain configuration.

A profile may restrict features.

A profile SHALL NOT reinterpret the meaning of retained features.

---

## 1.31 Bootstrap Compiler Versus Complete Language

Raygen preserves the goal of a rapidly implementable compiler without pretending that the complete production compiler fits into the same scope.

A focused bootstrap compiler may be constructed from:

1. lexer;
2. block parser;
3. expression parser;
4. symbol table;
5. basic type checker;
6. token-map generator;
7. assembly-serial emitter;
8. local tag resolver;
9. bytecode writer;
10. minimal VM or direct backend.

The bootstrap subset may support:

* `raygen 1`;
* modules;
* native scalar types;
* exact-width integers;
* immutable and mutable bindings;
* arithmetic;
* comparisons;
* `if`;
* `while`;
* `repeat`;
* functions;
* calls;
* returns;
* fixed arrays;
* simple outcomes;
* basic memory directives;
* small direct maps;
* VM output;
* simple native output.

The full language additionally requires professional passes for:

* ownership;
* lifetime analysis;
* alias analysis;
* provenance;
* generics;
* traits;
* deduction;
* vectorization;
* concurrency;
* advanced layouts;
* whole-program optimization;
* production diagnostics;
* safety certification.

The language remains architecturally approachable even though the production toolchain is substantial.

---

## 1.32 Compiler Phase Model

The normative semantic order is:

1. Source decoding
2. Lexical analysis
3. Parsing
4. Edition validation
5. Module and import resolution
6. Name resolution
7. Declaration collection
8. Type resolution
9. Generic and trait resolution
10. Ownership analysis
11. Borrow and lifetime analysis
12. Control-flow analysis
13. Failure-flow analysis
14. Concurrency-rights analysis
15. Mandatory layout and ABI validation
16. Alias and provenance analysis
17. Derivation graph construction
18. Deduction
19. Required runtime-guard selection
20. Optimization-policy resolution
21. Target-independent optimization
22. Vector and parallel legality analysis
23. Target-specific optimization
24. Directive lowering
25. Assembly-serial or low-level IR generation
26. Symbol, tag, and relocation assignment
27. Register allocation
28. Instruction scheduling
29. Machine-code or VM-code emission
30. Link-time validation
31. Final conformance verification

An implementation may combine, split, repeat, or internally reorder passes.

Its result SHALL be semantically equivalent to this model.

Optimization passes may run iteratively.

Any transformation invalidating a prior proof SHALL either:

* recompute that proof;
* preserve sufficient proof metadata;
* restore the removed guard;
* abandon the transformation.

---

## 1.33 Required Optimization Conflict Behavior

A required optimization is a contractual build requirement.

Example:

```rg1
policy vectorization optimization
{
    require width 256
}
```

When the optimization cannot be performed legally, the compiler SHALL deny the build.

It SHALL NOT weaken safety or silently ignore the requirement.

A preferred optimization permits fallback:

```rg1
policy vectorization optimization
{
    prefer width 256
    fallback scalar
}
```

A guarded optimization permits versioning:

```rg1
policy vectorization optimization
{
    prefer width 256
    guard aliasing
    fallback scalar
}
```

The compiler may emit:

* a fast vector path;
* a runtime legality guard;
* a scalar fallback.

This preserves maximum throughput in the common case without producing unsafe code.

---

## 1.34 Stable Diagnostic Identity

Raygen diagnostics have stable semantic categories and codes.

Representative categories include:

```text
RG-LEX
RG-PARSE
RG-NAME
RG-MODULE
RG-TYPE
RG-GENERIC
RG-TRAIT
RG-OWN
RG-LIFE
RG-MEM
RG-FLOW
RG-DENY
RG-CASE
RG-LAYER
RG-ATOMIC
RG-DERIVE
RG-DEDUCE
RG-POLICY
RG-LAYOUT
RG-ABI
RG-FFI
RG-OPT
RG-UNSAFE
RG-CONFORM
```

Exact prose may differ between implementations.

The code’s semantic condition SHALL remain stable within the edition.

Example:

```text
RG-DERIVE-001
Cyclic semantic derivation.
```

A compiler SHALL NOT reuse that code for an unrelated error.

---

## 1.35 Normative Versus Informative Material

Normative material defines mandatory language or implementation behavior.

Informative material explains:

* motivation;
* examples;
* implementation strategies;
* performance implications;
* historical context;
* migration;
* recommended practice.

Examples are informative unless explicitly marked normative.

An example cannot override a grammar or semantic rule.

Performance claims are informative unless expressed as a specific compiler conformance obligation.

---

## 1.36 Core Guarantee

A conforming safe Raygen program receives these guarantees:

* deterministic static typing;
* no implicit narrowing;
* no use-after-move;
* no use-after-free;
* no double destruction;
* no dangling safe references;
* no mutable aliasing outside defined synchronization;
* no data races in safe code;
* deterministic destruction;
* defined failure propagation;
* explicit overflow semantics;
* explicit concurrency visibility;
* explicit unsafe boundaries;
* preservation of ABI contracts;
* preservation of declared evaluation semantics;
* optimization without semantic weakening.

The compiler remains free to erase every supporting mechanism that is no longer needed after these guarantees have been proven.

That freedom is the basis of Raygen’s raw throughput.

---

# Part 1 Normative Conclusion

Raygen 1 is therefore defined as:

> A statically verified, hardware-aware, execution-oriented native systems language whose abstractions are removable, whose concurrency is rights-based, whose memory intent is explicit, whose failures are typed, whose data representation remains optimizable, and whose compiler is authorized to generate the fastest machine implementation that preserves declared meaning.

The language does not choose between safety and speed.

It establishes safety early enough that speed does not need to carry unnecessary checks afterward.



## *** ##



# Part 2 — Lexical Grammar, Tokens, Keywords, Source Structure, and Statement Boundaries

## 2.1 Purpose

This Part defines how Raygen 1 source text is converted into lexical tokens.

It establishes:

* source encoding;
* source-file structure;
* edition declarations;
* whitespace;
* line boundaries;
* comments;
* identifiers;
* Unicode normalization;
* keyword classes;
* literals;
* operators;
* punctuation;
* statement termination;
* lexical continuation;
* generic-token handling;
* lexical diagnostics.

Lexical analysis occurs before parsing, type resolution, ownership analysis, deduction, or optimization.

A conforming lexer SHALL:

* accept every lexically valid Raygen 1 source file;
* reject every lexically invalid source file;
* preserve accurate source locations;
* produce a deterministic token stream;
* avoid dependence upon semantic type information except where explicitly defined as contextual token interpretation.

---

# 2.2 Normative Grammar Notation

This specification uses an extended Backus–Naur form.

The notation is:

```text
production = expression ;
```

Alternatives use:

```text
|
```

Optional content uses:

```text
[ ... ]
```

Zero or more repetitions use:

```text
{ ... }
```

One or more repetitions use:

```text
item, { item }
```

Literal source text appears in quotation marks:

```text
"raygen"
```

Character ranges use:

```text
"A" ... "Z"
```

A lexical production describes characters.

A syntactic production describes tokens.

Grammar notation itself is not Raygen source syntax.

---

# 2.3 Source Encoding

A conforming implementation SHALL accept UTF-8 source files.

UTF-8 source files MAY begin with a Unicode byte-order mark.

When present, the byte-order mark:

* is permitted only at the beginning of the source file;
* SHALL NOT produce a token;
* SHALL NOT be interpreted as part of the edition declaration.

A byte-order mark appearing elsewhere is a lexical error.

Implementations MAY support additional source encodings.

Additional encodings SHALL:

* be explicitly selected;
* decode into the same Unicode scalar-value sequence;
* preserve identical Raygen semantics;
* be documented as implementation-defined input facilities.

The canonical portable source encoding is UTF-8 without a byte-order mark.

---

# 2.4 Invalid Encodings

A source file containing an invalid encoding sequence SHALL be rejected.

A conforming implementation SHALL NOT silently:

* replace malformed input;
* discard invalid bytes;
* reinterpret malformed UTF-8 using a local code page;
* normalize malformed byte sequences into valid identifiers.

The minimum diagnostic is:

```text
RG-LEX-001:
Invalid source encoding.
```

The diagnostic SHOULD identify:

* the byte offset;
* the source location recoverable before the error;
* the expected encoding.

---

# 2.5 Unicode Scalar Values

Raygen source text is processed as Unicode scalar values.

Surrogate code points are not Unicode scalar values and SHALL NOT occur in decoded source text.

The null scalar value `U+0000` is prohibited in ordinary source text.

Its appearance SHALL produce:

```text
RG-LEX-002:
Null character is not permitted in Raygen source.
```

A null value may be represented within a character or text literal through an escape sequence.

---

# 2.6 Canonical Line Endings

The lexer SHALL recognize these line-ending forms:

```text
LF
CRLF
CR
```

All three forms have identical lexical meaning.

A `CRLF` pair represents one newline.

Implementations SHOULD normalize line endings internally while preserving correct original byte and source locations.

A source file need not end with a physical newline.

End-of-file acts as a potential statement boundary under the same completeness rules as a newline.

---

# 2.7 Source-File Structure

A Raygen compilation unit has the canonical structure:

```text
source-file =
    optional-shebang,
    edition-declaration,
    { file-attribute },
    [ module-declaration ],
    { use-declaration },
    { top-level-declaration },
    end-of-file ;
```

The required edition declaration SHALL be the first Raygen language construct.

Comments and whitespace may precede it.

A permitted shebang may precede it.

No ordinary declaration, attribute, literal, or expression may precede the edition declaration.

---

# 2.8 Optional Shebang

Hosted tool profiles MAY recognize a shebang line.

A shebang is permitted only when:

* the first two source bytes are `#!`;
* it begins at byte offset zero;
* it ends at the first newline.

Example:

```rg1
#!/usr/bin/env raygen
raygen 1
```

The shebang line is ignored by the Raygen lexer.

A `#!` sequence occurring elsewhere is not a shebang and SHALL be tokenized according to the ordinary grammar or rejected.

Freestanding implementations are not required to support shebang execution, but SHALL diagnose a valid initial shebang consistently when the hosted-source profile is not enabled.

---

# 2.9 Edition Declaration

Every Raygen 1 compilation unit SHALL contain exactly one edition declaration.

The grammar is:

```text
edition-declaration =
    "raygen",
    horizontal-space,
    edition-number,
    terminator ;

edition-number =
    nonzero-decimal-digit,
    { decimal-digit } ;
```

For this specification, the only valid edition number is:

```rg1
raygen 1
```

The edition number is a language identifier, not an ordinary numeric literal.

Therefore, the following are invalid:

```rg1
raygen 01
raygen 1u8
raygen 1:u8
raygen 0x1
raygen 1_0
raygen +1
raygen 1.0
```

An unsupported edition SHALL produce:

```text
RG-PARSE-EDITION-001:
Unsupported Raygen language edition.
```

A missing edition declaration SHALL produce:

```text
RG-PARSE-EDITION-002:
Every Raygen source file must begin with an edition declaration.
```

---

# 2.10 Canonical Header

A canonical Raygen source file begins as follows:

```rg1
raygen 1

module application.main

use core::io::{print, println}
use data::table as tables
```

The minimal executable form is:

```rg1
raygen 1

origin main
{
    return
}
```

The `origin` entry declaration is syntactic and semantic material and is specified in later Parts.

---

# 2.11 Significant and Insignificant Source Text

The lexer distinguishes:

### Significant source text

Source text that produces tokens or statement boundaries.

### Insignificant source text

Source text ignored between tokens.

Insignificant source text includes:

* horizontal whitespace;
* comments;
* non-terminating newlines;
* permitted formatting characters where explicitly defined.

Insignificant text SHALL NOT merge two tokens that would otherwise be distinct.

For example:

```rg1
letvalue
```

is one identifier.

The following contains two keyword or identifier tokens:

```rg1
let value
```

---

# 2.12 Horizontal Whitespace

Horizontal whitespace includes:

```text
U+0009  CHARACTER TABULATION
U+0020  SPACE
```

Implementations MAY recognize additional Unicode spacing characters as whitespace outside literals.

However:

* non-ASCII spacing SHOULD be diagnosed under secure or strict profiles;
* invisible formatting SHALL NOT silently alter identifier spelling;
* line-separator characters SHALL be handled as newlines or diagnosed consistently.

The recommended portable formatting characters are ASCII space and horizontal tab.

---

# 2.13 Tabs and Source Columns

A tab is insignificant whitespace outside literals.

The language does not assign semantic indentation meaning to tabs or spaces.

Implementations SHALL document the display width used for diagnostic columns.

A compiler SHALL preserve a stable byte offset even when visual tab width differs between tools.

---

# 2.14 Comments

Raygen supports:

* line comments;
* nested block comments;
* documentation line comments;
* documentation block comments.

Comments behave as whitespace except where documentation metadata is explicitly retained.

Comments SHALL NOT terminate or merge adjacent tokens incorrectly.

---

# 2.15 Line Comments

A line comment begins with:

```text
//
```

and extends to, but does not include, the next newline or end-of-file.

Example:

```rg1
let width = 256 // preferred vector width
```

The newline following a line comment remains available as a statement terminator.

A line comment at end-of-file does not create an additional source line.

---

# 2.16 Nested Block Comments

A block comment begins with:

```text
/*
```

and ends with the matching:

```text
*/
```

Block comments are nestable.

Example:

```rg1
/*
    Outer comment.

    /*
        Nested comment.
    */

    Outer comment resumes.
*/
```

The lexer SHALL maintain nesting depth.

An opening delimiter inside a block comment increases depth.

A closing delimiter decreases depth.

The comment ends when depth reaches zero.

An unterminated block comment SHALL produce:

```text
RG-LEX-003:
Unterminated block comment.
```

The diagnostic SHOULD identify the location of the unmatched opening delimiter.

---

# 2.17 Comment Contents

Inside an ordinary block comment:

* string delimiters have no special meaning;
* character delimiters have no special meaning;
* line-comment delimiters have no special meaning;
* nested block-comment delimiters remain significant.

Therefore:

```rg1
/* "not a string" */
```

is one comment.

The following contains a nested comment:

```rg1
/* outer /* inner */ outer */
```

---

# 2.18 Documentation Comments

A documentation line comment begins with:

```text
///
```

A documentation block comment begins with:

```text
/**
```

and ends with its matching block-comment delimiter.

Documentation block comments are nested according to ordinary block-comment rules.

Documentation comments attach to the immediately following eligible declaration when no intervening non-attribute declaration exists.

Example:

```rg1
/// Computes the sum of two exact-width values.
function add(a: u64, b: u64) -> u64
{
    return a + b
}
```

Documentation comments:

* do not change runtime semantics;
* may be retained as compiler metadata;
* may participate in documentation generation;
* may be available to compile-time reflection where enabled;
* SHALL NOT grant access to private declarations;
* SHALL NOT suppress diagnostics.

---

# 2.19 Newlines Inside Block Comments

A newline inside a block comment contributes to source-line counting.

It does not automatically terminate a statement.

Consider:

```rg1
let total = base /*
    explanation
*/ + tax
```

This is one continued statement because the expression remains syntactically incomplete before the comment.

A lexer or parser SHALL preserve enough information to apply newline-completeness rules correctly.

---

# 2.20 Tokens

Raygen tokens belong to these lexical classes:

```text
identifier
keyword
integer-literal
real-literal
character-literal
text-literal
byte-literal
byte-text-literal
operator
delimiter
attribute-marker
newline
end-of-file
```

Comments and ordinary horizontal whitespace do not produce ordinary language tokens.

Documentation comments may produce metadata tokens in implementation pipelines, but are not grammar tokens unless explicitly consumed by a documentation grammar.

---

# 2.21 Longest Valid Token Rule

When multiple tokens can begin at the same source position, the lexer SHALL choose the longest valid token unless another rule explicitly overrides it.

Examples:

```text
<    before <=
>    before >=
-    before ->
<    before <-
|    before ||
|    before |>
.    before ..
..   before ..<
:    before ::
```

The longest-token rule does not permit the lexer to combine generic closing brackets into a shift token where doing so would make generic parsing ambiguous.

Generic closing-token behavior is defined separately in Section 2.61.

---

# 2.22 Identifiers

An identifier names a language entity.

The lexical form is conceptually:

```text
identifier =
    identifier-start,
    { identifier-continue } ;
```

An identifier start may be:

* `_`;
* a Unicode character with the `XID_Start` property.

An identifier continuation may be:

* `_`;
* a Unicode character with the `XID_Continue` property.

A decimal digit may not begin an ordinary identifier.

Examples:

```rg1
value
_private
vector_width
taxRate
Δtime
café
表格
```

The exact supported Unicode version SHALL be documented by the implementation and SHALL remain stable within a compiler release series.

A portability profile MAY require ASCII identifiers.

---

# 2.23 Identifier Normalization

Identifiers SHALL be normalized to Unicode Normalization Form C before comparison.

Name resolution uses normalized spelling.

Source locations and original spelling SHALL be preserved for diagnostics and tools.

Consequently, two source spellings that normalize to the same sequence denote the same identifier within the same namespace.

A duplicate declaration caused by normalization SHALL be rejected.

Example diagnostic:

```text
RG-NAME-UNICODE-001:
Identifier duplicates an existing declaration after Unicode NFC normalization.
```

---

# 2.24 Identifier Case

Raygen identifiers are case-sensitive.

These are distinct identifiers:

```rg1
packet
Packet
PACKET
```

Keywords use their exact lowercase spelling.

Therefore:

```rg1
function
```

is a keyword, while:

```rg1
Function
```

may be an identifier.

Style tools SHOULD reserve leading uppercase names for types and traits, but this is not a lexical requirement.

---

# 2.25 Secure Identifier Diagnostics

A secure diagnostic profile SHOULD diagnose:

* mixed writing systems within one identifier;
* identifiers visually confusable with keywords;
* identifiers visually confusable with names in the same scope;
* invisible format characters;
* bidirectional-control characters;
* suspicious normalization differences;
* unusual use of combining characters.

A safety-critical profile MAY prohibit these forms.

Ordinary Unicode identifiers remain semantically valid unless prohibited by the active profile.

Diagnostics SHALL NOT silently rename identifiers.

---

# 2.26 Raw Identifiers

A hard keyword may be used as an external or generated name through a raw identifier.

The canonical form is:

```text
r#identifier
```

Examples:

```rg1
let r#type = foreign_record.type
foreign function r#select(...)
```

A raw identifier:

* denotes the identifier following `r#`;
* does not include `r#` in name identity;
* SHALL contain a lexically valid identifier;
* may escape a hard keyword;
* SHALL NOT escape an invalid character sequence;
* SHALL NOT redefine language punctuation.

Thus:

```rg1
r#function
```

denotes the ordinary name `function`.

A raw identifier and an unescaped identifier with the same normalized name conflict in the same namespace.

Raw identifiers exist primarily for:

* FFI;
* generated code;
* schema import;
* cross-language tooling.

They SHOULD be uncommon in hand-written Raygen code.

---

# 2.27 Reserved Identifier Forms

The following identifier forms are reserved to the implementation or language:

```text
__name
_rg_name
```

More precisely:

* identifiers beginning with two underscores are reserved globally;
* identifiers beginning with `_rg_` are reserved globally;
* identifiers beginning with one underscore followed by an uppercase letter are reserved at module scope;
* names explicitly listed as compiler intrinsics are reserved in intrinsic namespaces.

User declarations SHALL NOT use globally reserved forms.

Implementations SHALL NOT reserve arbitrary ordinary identifiers outside these rules.

---

# 2.28 Keyword Classification

Every language-defined word belongs to exactly one of four categories:

1. hard keyword;
2. contextual keyword;
3. predeclared identifier;
4. standard-library name.

A word SHALL NOT simultaneously belong to multiple categories.

Classification is normative.

---

# 2.29 Hard Keywords

Hard keywords are reserved in every ordinary identifier context.

The Raygen 1 hard-keyword set is:

```text
alias
allow
as
atomic
break
case
cases
choice
const
continue
deduce
deny
derive
direct
dismiss
each
else
emit
ensures
false
foreign
free
function
if
implementation
in
interface
late
layer
layers
let
loop
map
module
move
never
origin
own
policy
private
public
raygen
read
record
region
repeat
require
requires
retain
return
select
set
spawn
static
stop
store
trait
true
type
unit
unsafe
use
var
void
wait
where
while
write
```

These spellings cannot be used as ordinary identifiers unless escaped through the raw-identifier syntax.

The hard-keyword set may be expanded only by a new edition or a formally compatible reservation mechanism.

---

# 2.30 Contextual Keywords

Contextual keywords are ordinary identifiers outside the grammar contexts that activate them.

The Raygen 1 contextual-keyword set includes:

```text
acquire
acquire_release
align
ascending
automatic
balanced
bitwise
clamp
copy
descending
device
diagnostic
dynamic
effectful
experimental
fallback
forced
guard
implementation
inlining
isolated
locked
merge
module
native
none
optimization
optimized
packed
parallel
prefer
profiled
pure
rapid
relaxed
release
report
result
safety
saturate
schedule
semantic
sequentially_consistent
stable
target
then
vector
widen
wrap
```

Where a spelling is listed both as a hard and contextual candidate in historical material, the unified classification in this section controls.

In particular:

* `implementation` is a hard keyword when introducing a trait implementation or layout class;
* `module` is a hard keyword when introducing a module;
* contextual uses of those spellings are parsed from their hard-keyword token.

A parser SHALL activate contextual meaning only in a defined production.

Contextual keywords allow the language to expand specialized grammar without unnecessarily occupying the entire identifier namespace.

---

# 2.31 Predeclared Identifiers

Predeclared identifiers are available without import but are not lexically reserved.

They include fundamental type names such as:

```text
bool
byte
char
text
bytes
address
usize
isize
int
uint
real
rayword
```

They also include exact-width numeric types:

```text
u8
u16
u32
u64
u128
u256

i8
i16
i32
i64
i128
i256

f16
f32
f64
f128
```

Because these are predeclared identifiers rather than hard keywords, a narrow scope may technically shadow one where the applicable profile permits shadowing.

Compilers SHOULD issue a warning for shadowing a fundamental type name.

Safety-critical profiles MAY prohibit such shadowing.

---

# 2.32 Standard-Library Names

Standard-library names are never reserved solely because they appear in the standard library.

Examples may include:

```text
print
println
vector
slice
buffer
table
tree
channel
mutex
option
outcome
```

Their availability depends upon imports, prelude configuration, and active profile.

A standard-library package SHALL NOT change the lexical classification of an ordinary identifier.

---

# 2.33 Boolean Literals

The boolean literals are:

```rg1
true
false
```

They have type `bool`.

Boolean literals are hard keywords and cannot be redefined.

---

# 2.34 Numeric Literal Overview

Raygen supports:

* decimal integer literals;
* binary integer literals;
* octal integer literals;
* hexadecimal integer literals;
* decimal real literals;
* hexadecimal real literals where enabled;
* explicit type suffixes;
* digit separators.

Numeric literal syntax SHALL preserve exact source value before contextual typing.

No overflow occurs merely while lexing an integer literal.

Representability is checked during semantic analysis.

---

# 2.35 Decimal Integer Literals

The grammar is:

```text
decimal-integer =
    "0"
    | nonzero-decimal-digit,
      { decimal-digit | "_" } ;
```

Examples:

```rg1
0
1
42
1_000
18_446_744_073_709_551_615
```

A separator:

* SHALL appear only between digits;
* SHALL NOT appear first;
* SHALL NOT appear last;
* SHALL NOT appear twice consecutively.

Invalid examples:

```rg1
_42
42_
4__2
```

---

# 2.36 Based Integer Literals

Binary literals use:

```text
0b
0B
```

Octal literals use:

```text
0o
0O
```

Hexadecimal literals use:

```text
0x
0X
```

Examples:

```rg1
0b1010_0110
0o755
0xFF_EC_20
```

At least one valid digit SHALL follow the radix prefix.

A digit invalid for the selected radix is a lexical error when it is part of the same apparent numeric token.

---

# 2.37 Integer Type Suffixes

An integer literal may include an exact suffix:

```text
u8
u16
u32
u64
u128
u256
i8
i16
i32
i64
i128
i256
uint
int
usize
isize
rayword
```

The canonical compact form is:

```rg1
255u8
1024u64
-5i32
0xFFFFu16
```

The explicit annotation form is also permitted:

```rg1
255:u8
1024:u64
```

The two forms have identical type intent.

A style profile MAY prefer one form.

The edition declaration SHALL NOT use either literal-suffix form.

Unary negation is not part of the literal token.

Therefore:

```rg1
-128i8
```

is tokenized as:

```text
"-"
"128i8"
```

Representability is checked after unary negation according to the active arithmetic policy.

---

# 2.38 Abstract Integer Type

An unsuffixed integer literal initially has the compile-time abstract type:

```text
integer
```

Context selects a concrete type.

Example:

```rg1
let count: u32 = 12
```

The literal is directly representable as `u32`.

When no contextual type exists, the default concrete type is:

```text
int
```

The compiler SHALL reject an unrepresentable contextual conversion.

This contextual literal rule is not implicit conversion between concrete numeric types.

---

# 2.39 Decimal Real Literals

Decimal real literals include:

```rg1
0.0
3.14159
1.
.5
1e9
1.5e-3
6.022_140_76e23
```

A conforming lexer SHALL distinguish:

```rg1
1..10
```

as an integer literal followed by a range operator, not as a malformed real literal.

The preferred style uses a digit on both sides of the decimal point:

```rg1
0.5
1.0
```

---

# 2.40 Real Type Suffixes

A real literal may use:

```text
f16
f32
f64
f128
real
```

Examples:

```rg1
1.0f32
3.141592653589793f64
2.0:real
```

An unsuffixed real literal initially has the abstract compile-time type:

```text
real-literal
```

When no contextual type exists, it becomes the predeclared type `real`.

A compiler SHALL round the literal according to the declared floating-point policy when converting it to a concrete format.

---

# 2.41 Hexadecimal Real Literals

The Native and Professional profiles SHALL support hexadecimal floating-point literals.

Example:

```rg1
0x1.8p+1f64
```

This denotes an exact binary-scaled value.

The grammar requires:

* a hexadecimal significand;
* `p` or `P`;
* a signed or unsigned decimal exponent.

Hexadecimal real literals are recommended for exact binary floating-point constants.

A Core-only bootstrap implementation MAY diagnose them as unsupported profile syntax.

---

# 2.42 Character Literals

A character literal is enclosed by single quotation marks:

```rg1
'A'
'λ'
'\n'
'\u{1F642}'
```

A character literal SHALL represent exactly one Unicode scalar value after escape processing.

Invalid examples include:

```rg1
''
'ab'
'\u{D800}'
```

The type is `char`.

A newline may not appear unescaped within an ordinary character literal.

---

# 2.43 Byte Literals

A byte literal uses the prefix:

```text
b
```

Example:

```rg1
b'A'
b'\n'
b'\xFF'
```

A byte literal SHALL represent exactly one value in `0 ... 255`.

A direct non-ASCII source character is not permitted in a byte literal unless its encoded byte interpretation is explicitly defined by a selected compatibility profile.

The portable form uses ASCII or `\xNN`.

The type is `byte`.

---

# 2.44 Text Literals

An ordinary text literal is enclosed in double quotation marks:

```rg1
"Raygen"
"maximum throughput"
"line one\nline two"
```

An ordinary text literal may not contain an unescaped physical newline.

Its semantic value is a sequence of Unicode scalar values.

The standard text encoding is defined by the `text` type specification, not by the lexical token alone.

---

# 2.45 Byte-Text Literals

A byte-text literal uses:

```text
b"..."
```

Example:

```rg1
b"RG1"
b"\x52\x47\x31"
```

Its value is a sequence of bytes.

Non-ASCII source characters are prohibited unless the active profile explicitly defines source-to-byte encoding.

Portable byte-text literals SHALL use ASCII source characters and byte escapes.

---

# 2.46 Raw Text Literals

A raw text literal begins with `r`, followed by zero or more `#` characters and a quotation mark.

It ends with a quotation mark followed by the same number of `#` characters.

Examples:

```rg1
r"no escapes here"

r#"This contains "quotation marks"."#

r##"This contains "# and quotes."##
```

Within a raw text literal:

* escape sequences are not processed;
* physical newlines are permitted;
* quotation marks terminate the literal only when followed by the matching number of `#` characters.

Raw text literals are useful for:

* regular expressions;
* generated code;
* paths;
* embedded schemas;
* assembly fragments;
* structured documents.

---

# 2.47 Multiline Text Literals

A multiline text literal uses triple quotation marks:

```rg1
"""
First line
Second line
"""
```

The opening delimiter may be followed immediately by content or by a newline.

Indentation trimming, when requested, SHALL use an explicit modifier or library operation.

The lexer SHALL NOT silently remove indentation from an ordinary multiline literal.

A canonical stripped form may be provided through:

```rg1
text.strip_indent("""
    first
    second
""")
```

This keeps lexical behavior simple and predictable.

---

# 2.48 Escape Sequences

Ordinary character, text, byte, and byte-text literals support defined escape sequences.

Standard escapes include:

```text
\\       backslash
\'       single quotation mark
\"       double quotation mark
\n       line feed
\r       carriage return
\t       horizontal tab
\0       null value
\b       backspace
\f       form feed
\v       vertical tab
\xNN     byte value
\u{H...} Unicode scalar value
```

A Unicode scalar escape contains one to six hexadecimal digits and SHALL denote a valid Unicode scalar value.

Unknown escape sequences are lexical errors.

Implementations SHALL NOT silently preserve an unknown escape as ordinary text.

---

# 2.49 Literal Concatenation

Adjacent text literals are not implicitly concatenated by the lexer.

The following is invalid without an operator or defined macro facility:

```rg1
"first" "second"
```

Explicit concatenation uses the language or library operation defined for the involved types.

This rule prevents whitespace-sensitive accidental merging and simplifies token processing.

Compile-time concatenation remains optimizable to one constant.

---

# 2.50 Delimiters

Raygen defines these primary delimiters:

```text
( )
[ ]
{ }
, ;
:
::
.
```

Their ordinary purposes include:

```text
( )   calls, grouping, tuples, parameters
[ ]   indexing, arrays, attributes where applicable
{ }   blocks, records, selections, mappings
,     element and argument separation
;     explicit statement termination
:     type annotation and labels
::    qualified-name separation
.     member access and decimal notation
```

Delimiter meaning is determined by the syntactic grammar.

---

# 2.51 Operators and Compound Tokens

Raygen 1 recognizes these core operator tokens:

```text
+
-
*
/
%
**
&
|
^
~
<<
>>
&&
||
!
==
!=
<
<=
>
>=
<=>
=
<-
->
=>
|>
?
??
?.
..
..<
...
@
#
```

Not every token is valid in every expression context.

Specific semantics and precedence are defined in the expression grammar.

The lexer SHALL distinguish:

```text
=
```

used in initialization or derivation equations from:

```text
<-
```

used for explicit mutation.

The parser and semantic context determine the exact legal use.

---

# 2.52 Mutation Token

The canonical mutation operator is:

```text
<-
```

Example:

```rg1
set counter <- counter + 1
```

The token communicates directional state change.

Ordinary mutation SHALL NOT use plain `=`.

Plain `=` is reserved for contexts including:

* initialization;
* named arguments where specified;
* default values;
* pattern association;
* declarative derivation equations;
* compile-time metadata assignment.

This distinction is central to Raygen’s execution-oriented grammar.

---

# 2.53 Arrow Tokens

Raygen distinguishes:

```text
->    return type or directional relation
=>    expression body or branch result
<-    mutation or inward transfer
|>    pipeline
```

Examples:

```rg1
function width() -> u32
    => 256

set width <- 512

values
    |> where value.active
    |> order value.rank descending
```

These tokens SHALL NOT be decomposed when the compound token is valid at the current lexical position.

---

# 2.54 Range Tokens

Raygen recognizes:

```text
..     inclusive range where defined
..<    half-open range
...    context-defined spread or variadic token
```

The preferred iteration range is half-open:

```rg1
each index in 0 ..< count
{
    ...
}
```

A parser SHALL correctly tokenize:

```rg1
1..10
```

without treating the first period as part of a real literal.

Whitespace is not required around a range token, though it is recommended.

---

# 2.55 Attribute Marker

The `@` token introduces an attribute in the canonical syntax:

```rg1
@inline
@align(64)
@deprecated("Use process_fast.")
function process(...)
```

Attributes are parsed before the declaration or construct they modify.

Unknown standard attributes are diagnosed according to the active profile.

User-defined attributes do not alter core semantics unless consumed by an authorized compile-time facility.

---

# 2.56 Statement Categories

Statements are classified as:

### Simple statements

Simple statements require a terminator.

Examples include:

* bindings;
* mutations;
* returns;
* expression statements;
* explicit calls;
* memory instructions;
* emitted outcomes;
* simple directives.

### Compound statements

Compound statements are self-delimiting.

Examples include:

* blocks;
* `if`;
* `while`;
* `repeat`;
* `each`;
* `select`;
* `case`;
* `cases`;
* `layer`;
* `layers`;
* `derive`;
* `deduce`;
* `direct`;
* policy bodies.

A compound statement does not require a semicolon after its closing brace.

A semicolon following a compound statement MAY be accepted as an empty statement only in compatibility mode.

Strict Raygen 1 SHOULD diagnose it as unnecessary.

---

# 2.57 Statement Terminators

The statement terminators are:

```text
newline
;
end-of-file
```

However, a newline or end-of-file terminates a statement only when the statement is syntactically complete.

A semicolon explicitly terminates a simple statement whenever the surrounding grammar permits termination.

A closing brace is not itself a statement terminator.

The block grammar makes a compound statement complete.

---

# 2.58 Newline Significance

A newline terminates the current simple statement when all of the following are true:

* the current delimiter nesting permits termination;
* no token requires a following operand or clause;
* the construct parsed so far forms a complete statement;
* the newline is not explicitly escaped;
* the grammar is not inside a token or literal.

Example:

```rg1
let width = 256
let height = 144
```

This contains two statements.

---

# 2.59 Newlines That Do Not Terminate

A newline SHALL NOT terminate a statement:

* inside unmatched `(` and `)`;
* inside unmatched `[` and `]`;
* inside an expression-oriented `{` and `}` construct before its grammar is complete;
* after an infix operator;
* after a prefix operator requiring an operand;
* after a comma;
* after `.`;
* after `::`;
* after `=`;
* after `<-`;
* after `->`;
* after `=>`;
* after `|>`;
* while parsing generic arguments;
* before a required `else`;
* before a required denial branch;
* where a contextual continuation keyword requires another clause.

Example:

```rg1
let total =
    subtotal
    + tax
    - discount
```

This is one statement.

---

# 2.60 Continuation Tokens

The following tokens suppress statement termination when they appear at the end of a physical line and the grammar expects continuation:

```text
=
<-
->
=>
|>
,
.
::
+
-
*
/
%
**
&
|
^
&&
||
<
<=
>
>=
==
!=
<<
>>
..
..<
?
??
```

Contextual continuation words include:

```text
else
allow
deny
then
where
requires
ensures
from
uses
fallback
```

A continuation word suppresses termination only where the active production expects its continuation meaning.

---

# 2.61 Generic Angle-Bracket Tokenization

The lexer SHALL emit individual `<` and `>` tokens.

It SHALL NOT require semantic knowledge to emit a special generic-open or generic-close token.

Nested generic types therefore tokenize as:

```rg1
Outer<Inner<T>>
```

with two individual closing `>` tokens.

The expression parser forms shift operators from adjacent angle tokens when the grammar admits a shift expression.

Thus:

```rg1
value >> amount
```

is parsed as a right shift.

The same character sequence inside generic arguments is parsed as two generic closers.

This rule eliminates the historical `>>` generic ambiguity.

Whitespace does not alter semantic interpretation:

```rg1
Outer<Inner<T>>
Outer<Inner<T> >
```

are equivalent.

---

# 2.62 Explicit Line Continuation

A backslash immediately followed by a newline may be supported as an explicit line continuation under the compatibility profile.

Canonical Raygen source SHALL NOT require it.

The preferred continuation style is to leave the construct grammatically incomplete:

```rg1
let result =
    first
    + second
```

Strict Raygen 1 MAY reject explicit backslash continuation to prevent invisible trailing-space errors.

---

# 2.63 Semicolon Use

A semicolon may terminate multiple simple statements on one physical line:

```rg1
let x = 1; let y = 2; let z = x + y
```

The preferred style uses one statement per line except for compact generated code.

A semicolon does not permit omission of required syntax.

The following remains invalid:

```rg1
let value = ;
```

A semicolon inside delimiters terminates only where the enclosed grammar permits a statement.

---

# 2.64 Empty Statements

An isolated semicolon represents an empty statement only in compatibility or generated-source profiles.

Strict Raygen 1 SHOULD diagnose empty statements:

```text
RG-PARSE-STATEMENT-001:
Empty statement has no effect.
```

A blank line is not an empty statement.

Multiple blank lines are insignificant.

---

# 2.65 End-of-File Termination

End-of-file terminates a syntactically complete simple statement.

Therefore this file is valid without a final physical newline:

```rg1
raygen 1
origin main
{
    return
}
```

End-of-file SHALL NOT repair an incomplete construct.

Examples that remain invalid:

```rg1
let value =
```

```rg1
function run(
```

```rg1
/* unterminated
```

---

# 2.66 Function-Call Lexical Form

Function calls use ordinary expression syntax:

```rg1
calculate(value)
perform_job()
```

Raygen additionally retains the explicit call instruction:

```rg1
call perform_job()
```

`call` is recognized as a contextual instruction word in a statement-leading position.

The unified rule is:

* ordinary calls are expressions;
* explicit `call` emphasizes statement-level invocation;
* `call` does not create a separate calling convention;
* the called function’s return and effect contracts remain unchanged.

A strict compiler SHALL reject silent discard of a non-discardable result.

Example:

```rg1
call calculate(value)
```

is legal only when:

* the result is `void`;
* the result type is marked discardable;
* an explicit discard policy applies.

Otherwise:

```rg1
dismiss calculate(value)
```

is required.

---

# 2.67 Qualified Names

The namespace separator is:

```text
::
```

Example:

```rg1
core::io::print
math::vector::dot
```

The period token is reserved for member access:

```rg1
packet.header.length
```

This distinction prevents ambiguity between:

* module qualification;
* value-member access.

A qualified name consists of identifiers and `::` separators.

Raw identifiers may appear as components.

---

# 2.68 Canonical Module Declaration

The canonical module declaration is:

```rg1
module application::main
```

A dotted compatibility form:

```rg1
module application.main
```

MAY be accepted by migration tools but is not canonical Raygen 1.

The unified specification uses `::` consistently for namespace paths.

A module declaration is terminated by newline or semicolon.

Only one module declaration is permitted per compilation unit.

---

# 2.69 Canonical Use Declaration

Imports use:

```rg1
use core::io::print
use core::io::{print, println}
use data::table as tables
```

Wildcards are excluded from strict Raygen 1:

```rg1
use core::io::*
```

A profile or extension MAY support them with explicit opt-in.

Explicit imports improve:

* deterministic name resolution;
* compile speed;
* dependency auditing;
* dead-code elimination;
* tooling;
* stable public interfaces.

---

# 2.70 Canonical Top-Level Ordering

The recommended source order is:

1. shebang, when used;
2. edition declaration;
3. file attributes;
4. module declaration;
5. imports;
6. aliases and constants;
7. types;
8. traits and interfaces;
9. implementations;
10. functions;
11. origin declaration.

Only the edition declaration and relative grammar restrictions are mandatory.

A compiler SHALL NOT change program meaning based solely on declaration order except where initialization or explicit order semantics require it.

---

# 2.71 File Attributes

File attributes apply to the compilation unit.

Example:

```rg1
raygen 1

@profile(native)
@deny(unused_result)

module application::main
```

A file attribute SHALL appear:

* after the edition declaration;
* before the module declaration, imports, or ordinary declarations.

An attribute placed later applies to the immediately following eligible construct rather than the file.

---

# 2.72 Token-Map Representation

The historical `token.map` stage is a canonical inspectable representation of the lexical stream.

A token-map entry SHOULD contain:

* token category;
* canonical token spelling;
* original source spelling where different;
* byte start;
* byte end;
* source line;
* source column;
* source file identity;
* normalized identifier value where applicable;
* literal payload or payload reference;
* trivia attachment where tooling requires it.

Example conceptual token map:

```text
EDITION_KEYWORD    "raygen"   1:1
EDITION_NUMBER     "1"        1:8
NEWLINE                       1:9
ORIGIN_KEYWORD     "origin"   3:1
IDENTIFIER         "main"     3:8
LBRACE             "{"        4:1
RETURN_KEYWORD     "return"   5:5
RBRACE             "}"        6:1
EOF                           6:2
```

A compiler need not materialize `token.map` as a text file.

It SHALL be capable of exposing equivalent information under a diagnostic or tooling mode.

---

# 2.73 Trivia Preservation

Compilers may discard ordinary whitespace and comments after tokenization.

Tools that rewrite, format, index, or refactor source SHOULD preserve trivia.

Documentation comments SHALL be preserved when documentation metadata is requested.

Loss of trivia SHALL NOT alter compilation semantics.

A formatter SHALL NOT change:

* token identity;
* statement boundaries;
* literal payloads;
* raw-text delimiters;
* attribute attachment;
* normalized identifier identity.

---

# 2.74 Lexical Error Recovery

A compiler MAY continue after a lexical error to report additional diagnostics.

Recovery SHALL NOT cause the compiler to accept or emit executable code for a lexically invalid compilation unit.

Recovery techniques may include:

* skipping to a newline;
* skipping to a matching delimiter;
* synthesizing a closing literal delimiter for diagnostics only;
* replacing an invalid token with an error token.

Synthesized recovery tokens SHALL be marked as non-source tokens.

---

# 2.75 Required Lexical Diagnostics

A conforming implementation SHALL diagnose at least:

* invalid source encoding;
* prohibited null source characters;
* invalid Unicode scalar values;
* invalid identifiers;
* invalid raw identifiers;
* reserved user identifiers;
* malformed numeric literals;
* invalid radix digits;
* invalid numeric separators;
* malformed suffixes;
* unterminated character literals;
* unterminated text literals;
* invalid escape sequences;
* invalid Unicode escapes;
* invalid byte values;
* unterminated block comments;
* unmatched literal delimiters;
* unsupported edition declarations;
* lexical tokens not recognized by the active edition.

Diagnostics SHOULD identify the smallest source range responsible for the failure.

---

# 2.76 Stable Lexical Diagnostic Codes

The minimum stable lexical codes include:

```text
RG-LEX-001   Invalid source encoding
RG-LEX-002   Prohibited null source character
RG-LEX-003   Unterminated block comment
RG-LEX-004   Unterminated text literal
RG-LEX-005   Unterminated character literal
RG-LEX-006   Invalid escape sequence
RG-LEX-007   Invalid Unicode scalar escape
RG-LEX-008   Malformed numeric literal
RG-LEX-009   Invalid digit for numeric radix
RG-LEX-010   Invalid numeric separator
RG-LEX-011   Invalid identifier
RG-LEX-012   Reserved identifier
RG-LEX-013   Unsupported source character
RG-LEX-014   Mismatched raw-text delimiter
RG-LEX-015   Invalid byte literal
```

Implementations may provide more specific subcodes.

The meaning of an existing code SHALL NOT be reassigned within Raygen 1.

---

# 2.77 Lexical Security Requirements

A secure or safety-critical compiler profile SHALL provide configurable diagnostics for:

* bidirectional text controls;
* visually confusable names;
* mixed-script identifiers;
* hidden non-spacing characters;
* noncanonical line separators;
* source characters that display differently across common tools;
* misleading comment placement;
* escaped versus literal identifier confusion.

A compiler MAY render escaped forms of suspicious characters in diagnostics.

The original source bytes SHALL remain available for audit.

---

# 2.78 Lexical Performance Requirements

Lexical safety SHALL NOT require slow tokenization.

A production lexer SHOULD support:

* single-pass UTF-8 decoding;
* vectorized ASCII scanning;
* fast paths for ASCII identifiers;
* block scanning for whitespace;
* SIMD delimiter detection;
* efficient nested-comment depth tracking;
* zero-copy source slices for tokens;
* deferred literal decoding;
* interned normalized identifiers;
* incremental re-lexing for development tools;
* parallel file-level lexing.

Unicode normalization may be deferred until an identifier contains non-ASCII source characters.

ASCII identifiers are already NFC-normalized.

String and numeric payloads may be decoded lazily when semantic analysis requires them.

These strategies preserve full lexical correctness while keeping source processing overhead negligible relative to later compilation stages.

---

# 2.79 Deterministic Tokenization

Given:

* identical source bytes;
* identical declared source encoding;
* identical language edition;
* identical enabled lexical extensions,

a conforming implementation SHALL produce the same semantic token sequence.

Diagnostic wording may differ.

Tokenization SHALL NOT depend upon:

* target architecture;
* optimization level;
* processor count;
* current locale;
* operating-system language;
* filesystem case behavior;
* runtime scheduling.

Numeric decimal separators are always `.`.

Keyword recognition is locale-independent.

Case conversion based on the user’s locale SHALL NOT participate in lexing.

---

# 2.80 Extension Tokens

Compiler extensions MAY introduce new lexical forms only when:

* the extension is explicitly enabled;
* the form does not alter standard tokenization when disabled;
* the extension declares its edition compatibility;
* source using the extension is marked nonportable;
* the extension does not reinterpret an existing valid Raygen token sequence silently.

An implementation SHALL NOT make ordinary Raygen source mean something different merely because an unrelated extension is installed.

---

# 2.81 Lexical Conformance

A lexer conforms to Raygen 1 when it:

* accepts UTF-8;
* recognizes canonical line endings;
* requires the edition declaration;
* implements deterministic tokenization;
* implements Unicode identifier normalization;
* implements nested block comments;
* distinguishes hard and contextual keywords;
* recognizes all standard literal forms required by its profile;
* applies longest-token rules correctly;
* preserves individual generic closing brackets;
* implements newline-sensitive statement boundaries;
* reports mandatory lexical failures;
* preserves source locations;
* does not make tokenization target-dependent.

A compiler SHALL NOT claim Raygen 1 lexical conformance while:

* treating nested block comments as flat;
* comparing Unicode identifiers without required normalization;
* silently repairing malformed literals;
* permitting locale-dependent keywords;
* merging generic closing brackets irreversibly into shift tokens;
* terminating incomplete statements merely because a newline occurred;
* interpreting ordinary comments as executable language content.

---

# Part 2 Normative Conclusion

Raygen source is designed to remain visually explicit, mechanically simple, and fast to tokenize.

Its lexical model combines:

* mandatory edition identity;
* deterministic UTF-8 processing;
* Unicode-capable names;
* secure normalization;
* nested comments;
* exact literal syntax;
* directional mutation tokens;
* explicit namespace separators;
* predictable newline termination;
* unambiguous nested generics;
* a directly inspectable token-map stage.

The lexer establishes a stable, lossless semantic boundary between source text and parsing.

No ownership object, safety table, dependency graph, runtime tag, or virtual-machine structure is created merely to tokenize a Raygen program.

The result is a grammar foundation suitable both for a compact bootstrap compiler and for an aggressively optimized professional toolchain.




## *** ##



# Part 3 — Syntactic Grammar, Declarations, Statements, Control Flow, Directives, Generics, and Operator Precedence

## 3.1 Purpose

This Part defines the canonical syntactic grammar of Raygen 1.

It establishes:

* compilation-unit structure;
* modules and imports;
* declaration grammar;
* origins and entry points;
* functions;
* parameters;
* generic parameters;
* contracts;
* blocks;
* bindings;
* initialization;
* mutation;
* expression statements;
* calls;
* returns;
* denial flow;
* conditional execution;
* selections and pattern matching;
* loops;
* ranges;
* control transfer;
* direct maps;
* derivations;
* deductions;
* memory directives;
* policies;
* cases;
* layers;
* unsafe blocks;
* operator precedence;
* associativity;
* pipeline syntax.

This Part defines syntactic validity.

Later Parts define:

* type legality;
* ownership behavior;
* lifetime validity;
* failure compatibility;
* concurrency rights;
* layout legality;
* runtime and optimization semantics.

A syntactically valid program may still be rejected during semantic analysis.

---

# 3.2 Grammar Organization

The Raygen grammar is divided into four coordinated layers:

```text
Raygen lexical grammar
Raygen declaration and statement grammar
Raygen expression and type grammar
Raygen structural and privileged-syntax grammar
```

The lexical grammar is defined in Part 2.

This Part defines the ordinary declaration, statement, and expression grammar.

Structural declarations such as tables, trees, storage regions, and device mappings use ordinary Raygen grammar wherever practical.

Compiler-recognized standard-library syntax SHALL be specified separately and SHALL lower into ordinary typed semantic operations.

No syntax may remain meaningful solely because one implementation happens to recognize it.

---

# 3.3 Compilation Unit

The canonical grammar is:

```text
compilation-unit =
    edition-declaration,
    { file-attribute },
    [ module-declaration ],
    { use-declaration },
    { top-level-declaration },
    end-of-file ;
```

A compilation unit contains exactly one edition declaration.

A compilation unit belongs to exactly one module.

When no module declaration is present, the module identity is supplied by the package manifest or build system.

A source file SHALL NOT define more than one module body.

---

# 3.4 Top-Level Declarations

A top-level declaration is one of:

```text
top-level-declaration =
      alias-declaration
    | constant-declaration
    | static-declaration
    | record-declaration
    | unit-declaration
    | choice-declaration
    | structural-declaration
    | trait-declaration
    | interface-declaration
    | implementation-declaration
    | function-declaration
    | origin-declaration
    | derivation-declaration
    | deduction-declaration
    | storage-declaration
    | policy-declaration
    | foreign-declaration
    | submodule-declaration ;
```

A declaration may be preceded by:

```text
{ attribute }
[ visibility ]
```

The canonical ordering is:

```rg1
@attributes
public function calculate(...)
{
    ...
}
```

---

# 3.5 Visibility

The visibility grammar is:

```text
visibility =
      "private"
    | "module"
    | "package"
    | "public" ;
```

When omitted, top-level declarations default to:

```text
module
```

Nested declarations default to the narrowest containing visibility.

Visibility is part of declaration semantics, not type identity.

---

# 3.6 Module Declaration

The grammar is:

```text
module-declaration =
    "module",
    module-path,
    terminator ;

module-path =
    identifier,
    { "::", identifier } ;
```

Example:

```rg1
module analytics::production
```

A module declaration SHALL appear after file attributes and before imports or ordinary declarations.

---

# 3.7 Submodule Declaration

A module may declare a child module:

```text
submodule-declaration =
    [ visibility ],
    "module",
    identifier,
    block ;
```

Example:

```rg1
public module encoding
{
    public function encode(value: read Packet) -> bytes
    {
        ...
    }
}
```

A submodule declaration creates a namespace.

It does not create a runtime object.

---

# 3.8 Use Declarations

The canonical forms are:

```text
use-declaration =
      "use", qualified-name, terminator
    | "use", qualified-name, "as", identifier, terminator
    | "use", qualified-name, "::", import-group, terminator ;

import-group =
    "{",
    import-item,
    { ",", import-item },
    [ "," ],
    "}" ;

import-item =
    identifier,
    [ "as", identifier ] ;
```

Examples:

```rg1
use core::io::print
use core::io::println as write_line
use core::io::{print, println}
use data::table::{Table, Row as TableRow}
```

Wildcard imports are not part of strict Raygen 1.

Imports are compile-time namespace operations.

They do not execute runtime initialization merely by being lexically present.

---

# 3.9 Aliases

A type alias is declared as:

```text
alias-declaration =
    [ visibility ],
    ( "alias" | "type" ),
    identifier,
    [ generic-parameter-list ],
    "=",
    type-expression,
    terminator ;
```

Canonical form:

```rg1
type SampleId = u256
```

Generic alias:

```rg1
type Pair<T> = record
{
    first: T
    second: T
}
```

An alias does not create a nominally distinct type unless explicitly declared through a nominal-type form defined in Part 4.

---

# 3.10 Constants

The grammar is:

```text
constant-declaration =
    [ visibility ],
    "const",
    identifier,
    [ ":", type-expression ],
    "=",
    constant-expression,
    terminator ;
```

Example:

```rg1
public const VECTOR_WIDTH: u32 = 256
```

A constant SHALL have a compile-time-evaluable initializer.

A constant has no mutable storage identity unless explicitly materialized by the compiler.

---

# 3.11 Static Declarations

The grammar is:

```text
static-declaration =
    [ visibility ],
    [ "atomic" ],
    "static",
    identifier,
    ":",
    type-expression,
    "=",
    expression,
    terminator ;
```

Example:

```rg1
static process_count: atomic<u64> = 0
```

Static initialization rules are defined by the module and ownership Parts.

A mutable static declaration requires:

* atomic storage;
* a lock capability;
* an unsafe declaration;
* or another language-defined synchronization mechanism.

---

# 3.12 Record Declarations

A record is a named product type.

```text
record-declaration =
    [ visibility ],
    "record",
    identifier,
    [ generic-parameter-list ],
    [ where-clause ],
    record-body ;

record-body =
    "{",
    { record-member },
    "}" ;

record-member =
      field-declaration
    | associated-declaration
    | nested-type-declaration ;
```

Example:

```rg1
record Sample
{
    id: SampleId
    value: f64
    timestamp: u64
}
```

Fields are declared as:

```text
field-declaration =
    { attribute },
    [ visibility ],
    identifier,
    ":",
    type-expression,
    [ "=", expression ],
    terminator ;
```

Field initialization syntax is not mutation.

---

# 3.13 Unit Declarations

A `unit` is a tightly controlled structural aggregate.

```text
unit-declaration =
    [ visibility ],
    "unit",
    identifier,
    [ generic-parameter-list ],
    [ layout-clause ],
    record-body ;
```

Example:

```rg1
@layout(packed)
unit NetworkHeader
{
    version: u8
    flags: u8
    length: u16
}
```

A unit may impose:

* packed layout;
* exact field order;
* explicit alignment;
* restricted constructors;
* value semantics.

The declaration itself does not imply unsafe access.

---

# 3.14 Choice Declarations

A `choice` is a tagged alternative type.

```text
choice-declaration =
    [ visibility ],
    "choice",
    identifier,
    [ generic-parameter-list ],
    [ where-clause ],
    choice-body ;

choice-body =
    "{",
    choice-variant,
    { choice-variant },
    "}" ;

choice-variant =
    identifier,
    [ choice-payload ],
    terminator ;

choice-payload =
      "(", type-list, ")"
    | record-body ;
```

Example:

```rg1
choice LoadResult
{
    ready(Data)
    unavailable
    denied(IoError)
}
```

A choice is exhaustive by construction.

Representation is selected according to its layout category and optimization policy.

---

# 3.15 Structural Declarations

Raygen recognizes standardized structural declaration forms:

```text
structural-declaration =
      array-declaration
    | list-declaration
    | nest-declaration
    | node-declaration
    | branch-declaration
    | tree-declaration
    | graph-declaration
    | table-declaration
    | buffer-declaration ;
```

Most applications use generic structural types directly:

```rg1
array<f64, 8>
list<Packet>
tree<Node>
table<Sample>
buffer<byte>
```

Named structural declarations are provided when the structure requires:

* schema fields;
* keys;
* indexes;
* layout policies;
* invariants;
* specialized operations.

Example:

```rg1
table Samples
{
    id: SampleId key
    category: text index
    value: f64
    timestamp: u64 index
}
```

Structural modifiers such as `key` and `index` are contextual grammar whose semantic meaning is defined by the relevant structural specification.

---

# 3.16 Generic Parameter Lists

The grammar is:

```text
generic-parameter-list =
    "<",
    generic-parameter,
    { ",", generic-parameter },
    [ "," ],
    ">" ;

generic-parameter =
      type-parameter
    | constant-parameter ;

type-parameter =
    identifier,
    [ ":", trait-bound-list ],
    [ "=", type-expression ] ;

constant-parameter =
    "const",
    identifier,
    ":",
    integer-type,
    [ "=", constant-expression ] ;
```

Examples:

```rg1
record Pair<T>
{
    first: T
    second: T
}
```

```rg1
record PacketBuffer<T, const N: usize>
{
    values: array<T, N>
}
```

Generic parameter names SHALL be unique within the declaration.

---

# 3.17 Trait Bounds

The grammar is:

```text
trait-bound-list =
    trait-reference,
    { "+", trait-reference } ;
```

Example:

```rg1
function sort_values<T: Comparable + Movable>(
    values: write slice<T>
) -> void
{
    ...
}
```

The `+` token in a trait-bound list denotes conjunction, not arithmetic addition.

Every listed bound SHALL be satisfied.

---

# 3.18 Where Clauses

The grammar is:

```text
where-clause =
    "where",
    where-requirement,
    { ",", where-requirement } ;

where-requirement =
      type-expression, ":", trait-bound-list
    | constant-expression
    | lifetime-requirement ;
```

Example:

```rg1
function merge<T>(
    left: read slice<T>,
    right: read slice<T>
) -> list<T>
where T: Comparable + Copy
{
    ...
}
```

A newline may occur after `where` without terminating the declaration.

---

# 3.19 Trait Declarations

The grammar is:

```text
trait-declaration =
    [ visibility ],
    "trait",
    identifier,
    [ generic-parameter-list ],
    [ where-clause ],
    trait-body ;

trait-body =
    "{",
    { trait-member },
    "}" ;
```

Trait members may include:

* required functions;
* provided functions;
* associated types;
* associated constants.

Example:

```rg1
trait Comparable
{
    function compare(
        left: read Self,
        right: read Self
    ) -> Ordering
}
```

A trait contains behavior contracts, not instance storage.

---

# 3.20 Interface Declarations

The grammar parallels traits:

```text
interface-declaration =
    [ visibility ],
    "interface",
    identifier,
    [ generic-parameter-list ],
    [ where-clause ],
    trait-body ;
```

An interface explicitly supports runtime interface objects where declared.

Example:

```rg1
interface Stream
{
    function read(
        output: write slice<byte>
    ) -> allow<usize> deny<IoError>
}
```

Interface implementation is explicit.

---

# 3.21 Implementation Declarations

The grammar is:

```text
implementation-declaration =
    "implementation",
    [ generic-parameter-list ],
    implementation-target,
    [ where-clause ],
    implementation-body ;

implementation-target =
      type-expression
    | trait-reference, "for", type-expression ;

implementation-body =
    "{",
    { associated-declaration },
    "}" ;
```

Examples:

```rg1
implementation Point
{
    function magnitude(self: read Self) -> f64
    {
        ...
    }
}
```

```rg1
implementation Equality for Point
{
    function equal(
        left: read Point,
        right: read Point
    ) -> bool
    {
        return left.x == right.x &&
               left.y == right.y
    }
}
```

Specialization is prohibited in Raygen 1.

---

# 3.22 Function Declarations

The grammar is:

```text
function-declaration =
    { attribute },
    [ visibility ],
    [ function-modifier-list ],
    "function",
    identifier,
    [ generic-parameter-list ],
    parameter-list,
    [ return-contract ],
    { function-contract },
    function-body ;
```

Function modifiers include:

```text
function-modifier =
      "pure"
    | "effectful"
    | "unsafe"
    | "foreign"
    | "const"
    | "static"
    | "async"
    | "vector"
    | "direct" ;
```

Not every combination is legal.

Modifier legality is defined semantically.

---

# 3.23 Parameter Lists

The canonical grammar is:

```text
parameter-list =
    "(",
    [ parameter, { ",", parameter }, [ "," ] ],
    ")" ;

parameter =
    identifier,
    ":",
    [ access-mode ],
    type-expression,
    [ "=", expression ] ;
```

Examples:

```rg1
input: read slice<f32>
output: write slice<f32>
packet: own Packet
state: locked SharedState
counter: read atomic<u64>
```

The parameter name precedes the colon.

The access mode follows the colon and precedes the type.

This is the sole canonical Raygen 1 parameter ordering.

Earlier forms placing the access mode before the parameter name are noncanonical compatibility syntax.

---

# 3.24 Access Modes

The parameter access modes are:

```text
access-mode =
      "own"
    | "read"
    | "write"
    | "locked" [ "(", expression ")" ]
    | "atomic" ;
```

Meanings:

```text
own       ownership transfers into the callee
read      shared immutable borrow
write     exclusive mutable borrow
locked    access requires the declared lock capability
atomic    access requires atomic storage semantics
```

`move` is not an access mode.

Ownership transfer in a parameter uses:

```rg1
packet: own Packet
```

A move expression uses:

```rg1
move packet
```

This resolves the earlier conflict where `move` appeared both as a parameter qualifier and an operation.

---

# 3.25 Receiver Parameters

Methods may declare an explicit receiver:

```rg1
self: read Self
self: write Self
self: own Self
```

Example:

```rg1
function clear(self: write Buffer) -> void
{
    ...
}
```

No implicit receiver access mode is inferred when ownership behavior would be ambiguous.

A shorthand `self` MAY be supported for an immutable receiver, but canonical professional source SHOULD write the access mode explicitly.

---

# 3.26 Default Parameters

A parameter may have a default expression:

```rg1
function connect(
    address: text,
    timeout: Duration = seconds(30)
) -> allow<Connection> deny<ConnectError>
{
    ...
}
```

Parameters with defaults SHALL follow required parameters unless named-argument rules explicitly permit another ordering.

Default expressions are type-checked in the declaration context.

They are evaluated at the call site unless a specification defines equivalent precomputation.

---

# 3.27 Return Contracts

The grammar is:

```text
return-contract =
    "->",
    return-type ;

return-type =
      type-expression
    | fallible-return-type
    | "void"
    | "never" ;

fallible-return-type =
    "allow", "<", type-expression, ">",
    "deny", "<", type-expression, ">" ;
```

Examples:

```rg1
-> u64
-> void
-> never
-> allow<Data> deny<IoError>
```

The order is fixed:

```text
allow<T> deny<E>
```

The reverse order is invalid.

The canonical underlying type is:

```rg1
outcome<T, E>
```

---

# 3.28 Void Fallible Returns

A function that succeeds without a payload may use:

```rg1
-> allow<void> deny<IoError>
```

The shorthand is:

```rg1
-> void deny<IoError>
```

where permitted by the parser.

Canonical professional declarations SHOULD use:

```rg1
-> allow<void> deny<IoError>
```

when maximum clarity is required.

A successful return is:

```rg1
return allow
```

---

# 3.29 Function Contracts

Function contracts are:

```text
function-contract =
      requires-clause
    | ensures-clause ;

requires-clause =
    "requires",
    expression ;

ensures-clause =
    "ensures",
    expression ;
```

Example:

```rg1
function divide(
    value: i256,
    divisor: i256
) -> i256
requires divisor != 0
{
    return value / divisor
}
```

Multiple contracts are permitted:

```rg1
function normalize(
    values: write slice<f64>
) -> void
requires values.length > 0
ensures sum(values) approximately 1.0
{
    ...
}
```

Contract expressions must satisfy the restrictions defined in the contracts Part.

---

# 3.30 Function Bodies

A function body is:

```text
function-body =
      block
    | "=>", expression, terminator ;
```

Block-bodied function:

```rg1
function add(a: u64, b: u64) -> u64
{
    return a + b
}
```

Expression-bodied function:

```rg1
function add(a: u64, b: u64) -> u64
    => a + b
```

An expression-bodied function performs an implicit ordinary return.

Fallible expression bodies SHALL return an expression compatible with the full outcome type unless explicit propagation syntax is used.

---

# 3.31 Origins

An `origin` is an externally enterable execution root.

The grammar is:

```text
origin-declaration =
    { attribute },
    [ visibility ],
    "origin",
    identifier,
    [ parameter-list ],
    [ return-contract ],
    { function-contract },
    block ;
```

Example:

```rg1
origin main
{
    emit "Hello, Raygen."
}
```

An origin may represent:

* the program entry point;
* a service endpoint;
* an interrupt entry;
* a device kernel entry;
* a test entry;
* a VM entry;
* a foreign-exported entry.

The active package profile determines which origin forms are legal.

---

# 3.32 Main Origin

An executable package SHALL define exactly one selected main origin.

Canonical form:

```rg1
origin main
{
    ...
}
```

A hosted main origin may accept standardized arguments:

```rg1
origin main(
    arguments: read slice<text>
) -> allow<void> deny<ExitError>
{
    ...
}
```

The precise hosted entry ABI is defined by the execution profile.

A library package need not define an origin.

---

# 3.33 Origin Versus Function

An origin differs from an ordinary function because it may establish:

* an external entry boundary;
* an initial execution region;
* an initial denial boundary;
* a scheduler root;
* a device execution context;
* process-level policy scope;
* an ABI-visible symbol.

Inside the language, an origin body uses ordinary statements and expressions.

An origin is not a mandatory heap object or runtime ray structure.

---

# 3.34 Blocks

The grammar is:

```text
block =
    "{",
    { statement },
    [ final-expression ],
    "}" ;
```

A block creates:

* a lexical scope;
* an ownership scope;
* a destruction boundary;
* a potential expression result.

A block containing no final expression has type `void` or unit according to context.

Example:

```rg1
{
    let first = calculate_first()
    let second = calculate_second()
    first + second
}
```

When used as an expression, the final expression has no terminator before the closing brace.

---

# 3.35 Statement Grammar

A statement is one of:

```text
statement =
      binding-statement
    | mutation-statement
    | expression-statement
    | call-statement
    | emit-statement
    | return-statement
    | allow-control-statement
    | deny-control-statement
    | if-statement
    | select-statement
    | while-statement
    | repeat-statement
    | each-statement
    | loop-statement
    | break-statement
    | continue-statement
    | stop-statement
    | direct-map-statement
    | derivation-declaration
    | deduction-declaration
    | memory-statement
    | region-declaration
    | policy-declaration
    | case-declaration
    | cases-declaration
    | layer-declaration
    | layers-declaration
    | unsafe-block
    | nested-declaration
    | block ;
```

A simple statement requires a terminator.

A compound statement is self-delimiting.

---

# 3.36 Immutable Bindings

The grammar is:

```text
binding-statement =
    binding-kind,
    identifier,
    [ type-annotation ],
    [ initializer ],
    terminator ;

binding-kind =
      "let"
    | "var"
    | "late" ;

type-annotation =
    ":",
    type-expression ;

initializer =
    "=",
    expression ;
```

An immutable binding uses:

```rg1
let count: u64 = 10
```

`let` creates a binding that cannot be reassigned.

The stored value may still contain interior mutability when its type explicitly permits it.

---

# 3.37 Mutable Bindings

A mutable binding uses:

```rg1
var count: u64 = 0
```

`var` permits assignment through `set`.

Example:

```rg1
set count <- count + 1
```

Mutability is attached to the binding or storage location.

It does not weaken ownership or concurrency restrictions.

---

# 3.38 Late Initialization

A late binding is declared without an immediate value:

```rg1
late result: Calculation
```

It SHALL be initialized exactly once before use:

```rg1
set result <- calculate()
```

After its first initialization, a `late` binding behaves as an immutable `let` binding unless declared:

```rg1
late var result: Calculation
```

where that extended form is enabled.

The compiler SHALL prove definite initialization before every read.

A late binding SHALL NOT be destroyed unless initialization completed.

---

# 3.39 Type Inference

The type annotation may be omitted when the initializer determines exactly one valid type:

```rg1
let width = 256u32
```

When an unsuffixed literal lacks another context:

```rg1
let width = 256
```

the literal uses its default concrete type.

Inference SHALL NOT silently invent an ownership mode, numeric promotion, or denial conversion.

Public declarations SHOULD include explicit types where omission would weaken interface clarity.

---

# 3.40 Initialization Operator

The `=` token initializes a new binding or declarative member:

```rg1
let total = subtotal + tax
```

Initialization establishes the binding’s first value.

It is distinct from mutation.

This distinction allows the compiler and reader to identify state changes immediately.

---

# 3.41 Mutation Statements

The grammar is:

```text
mutation-statement =
    "set",
    assignable-expression,
    "<-",
    expression,
    terminator ;
```

Examples:

```rg1
set count <- count + 1
set packet.header.length <- new_length
set values[index] <- replacement
```

The left side SHALL denote a writable storage location.

The right side is evaluated before the mutation becomes observable, subject to the evaluation rules.

Mutation does not produce an ordinary expression value.

This prevents accidental assignment within conditions.

---

# 3.42 Compound Mutation

Raygen may provide explicit compound-mutation syntax:

```rg1
set count <-+ 1
set total <-* factor
```

However, the canonical portable form is:

```rg1
set count <- count + 1
set total <- total * factor
```

A compound form, when supported, SHALL:

* evaluate the destination exactly once;
* use the same arithmetic policy as the expanded form;
* preserve atomicity only when explicitly applied to atomic storage;
* not introduce a distinct semantic result.

The minimum Raygen 1 core does not require compound mutation tokens.

---

# 3.43 Assignable Expressions

An assignable expression may be:

```text
assignable-expression =
      mutable-identifier
    | member-access
    | index-access
    | pointer-dereference
    | writable-view ;
```

Legality depends upon:

* mutability;
* ownership;
* access mode;
* lifetime;
* aliasing;
* atomicity;
* layout;
* unsafe status.

A syntactically assignable expression may still be semantically read-only.

---

# 3.44 Expression Statements

An expression may appear as a statement:

```text
expression-statement =
    expression,
    terminator ;
```

An expression statement is legal only when:

* the expression has an observable permitted effect;
* its result type is discardable;
* the result is explicitly ignored by policy.

Pure unused expressions SHOULD be diagnosed.

Fallible outcomes SHALL NOT be silently discarded.

---

# 3.45 Explicit Call Statements

The grammar is:

```text
call-statement =
    "call",
    call-expression,
    terminator ;
```

Example:

```rg1
call flush_logs()
```

An explicit call statement emphasizes invocation for effect.

It does not alter:

* parameter evaluation;
* ownership transfer;
* calling convention;
* failure type;
* synchronization semantics.

A non-`void`, non-discardable return SHALL be handled explicitly.

---

# 3.46 Dismissing Results

A result may be intentionally discarded through:

```rg1
dismiss calculate_statistics()
```

This form is preferred over an unused expression when the discard is semantically intentional.

A fallible result may be dismissed only when:

* the denial type permits dismissal;
* the active policy permits unchecked denial;
* or explicit denial handling occurs inside the dismissal form.

Safety profiles SHOULD prohibit unchecked dismissal of fallible outcomes.

---

# 3.47 Emit Statements

The grammar is:

```text
emit-statement =
    "emit",
    [ emit-target, ":" ],
    expression,
    terminator ;
```

Simple form:

```rg1
emit "Processing complete."
```

Targeted form:

```rg1
emit log: message
emit output: result
emit event_bus: notification
```

`emit` expresses delivery to an output capability or effect channel.

It is not restricted to console printing.

The target and effect type are resolved semantically.

---

# 3.48 Return Statements

The grammar is:

```text
return-statement =
      "return", terminator
    | "return", expression, terminator
    | "return", "allow", [ expression ], terminator
    | "return", "deny", expression, terminator ;
```

Examples:

```rg1
return
return value
return allow result
return allow
return deny error
```

`return allow` is legal only for a successful void outcome.

`return deny` requires a denial value unless the denial type has a unique default constructor explicitly selected by the program.

---

# 3.49 Allow Propagation Expression

Raygen’s canonical propagation expression is:

```text
allow-expression =
    "allow",
    expression ;
```

Example:

```rg1
let data = allow load_data(path)
```

Behavior:

* if the expression is allowed, its allowed payload becomes the expression value;
* if denied, the denial propagates from the enclosing fallible function.

This use of `allow` is distinct from:

```rg1
return allow value
```

The parser distinguishes them by context.

---

# 3.50 Explicit Outcome Matching

A fallible result may be matched directly:

```rg1
allow load_data(path) as data
{
    process(data)
}
deny as error
{
    handle(error)
}
```

The canonical grammar is:

```text
allow-control-statement =
    "allow",
    expression,
    [ "as", pattern ],
    block,
    [ deny-branch ] ;

deny-branch =
    "deny",
    [ "as", pattern ],
    block ;
```

When the expression can deny, omitting the denial branch is legal only when denial propagates automatically under the surrounding construct’s defined semantics.

Strict professional source SHOULD make the behavior explicit.

---

# 3.51 If Statements

The grammar is:

```text
if-statement =
    "if",
    expression,
    block,
    { "else", "if", expression, block },
    [ "else", block ] ;
```

Example:

```rg1
if temperature > 100
{
    emit "critical"
}
else if temperature > 80
{
    emit "high"
}
else
{
    emit "normal"
}
```

The condition SHALL have type `bool`.

No implicit numeric truth conversion exists.

---

# 3.52 If Expressions

An `if` construct may produce a value:

```rg1
let category =
    if temperature > 100
    {
        "critical"
    }
    else
    {
        "normal"
    }
```

All reachable value-producing branches SHALL have compatible types.

An `if` expression without `else` has type `void` unless the omitted branch is statically unreachable.

---

# 3.53 Select Statements and Expressions

The grammar is:

```text
select-statement =
    "select",
    expression,
    "{",
    select-arm,
    { select-arm },
    "}" ;

select-arm =
    "case",
    pattern,
    [ guard-clause ],
    block ;

guard-clause =
    "where",
    expression ;
```

Example:

```rg1
select status
{
    case success
    {
        emit "Completed"
    }

    case denied
    {
        emit "Access denied"
    }

    case unavailable
    {
        emit "Unavailable"
    }
}
```

The canonical catch-all pattern is:

```rg1
case other
{
    ...
}
```

The identifier `other` is contextual within a selection pattern and does not create a binding unless explicitly written as a binding pattern.

---

# 3.54 Selection Exhaustiveness

Every value-producing selection SHALL be exhaustive.

Statement selections SHALL also be exhaustive for:

* choices;
* booleans;
* finite enums;
* outcomes;
* statically bounded domains

unless the selection explicitly declares:

```rg1
partial select value
{
    ...
}
```

where the partial-selection feature is permitted.

Strict Raygen 1 prefers exhaustive selection.

The compiler SHALL diagnose unreachable or duplicate cases.

---

# 3.55 Pattern Forms

Patterns include:

```text
pattern =
      wildcard-pattern
    | literal-pattern
    | binding-pattern
    | variant-pattern
    | record-pattern
    | tuple-pattern
    | range-pattern
    | alternative-pattern
    | guarded-pattern ;
```

Examples:

```rg1
case _
case 0
case allowed(value)
case Point { x, y }
case (left, right)
case 0 ..< 16
case red | green | blue
```

Pattern syntax does not execute arbitrary side effects.

Guards are evaluated only after structural matching succeeds.

---

# 3.56 While Loops

The grammar is:

```text
while-statement =
    [ loop-label ],
    "while",
    expression,
    block ;

loop-label =
    "loop",
    identifier ;
```

Examples:

```rg1
while count < 100
{
    set count <- count + 1
}
```

Named form:

```rg1
loop outer while active
{
    ...
}
```

The condition SHALL have type `bool`.

---

# 3.57 Repeat Loops

Raygen supports two repeat forms.

### Counted repeat

```text
repeat-statement =
    [ loop-label ],
    "repeat",
    expression,
    [ "as", identifier ],
    block ;
```

Examples:

```rg1
repeat 10
{
    emit "Ray"
}
```

```rg1
repeat 10 as index
{
    emit index
}
```

The expression SHALL produce a nonnegative integral count.

The indexed form iterates from zero to one less than the count.

### Postcondition repeat

The form is:

```rg1
repeat
{
    ...
}
while condition
```

Its grammar is:

```text
postcondition-repeat =
    [ loop-label ],
    "repeat",
    block,
    "while",
    expression,
    terminator ;
```

This form executes at least once.

The two forms are syntactically distinct.

---

# 3.58 Each Loops

The grammar is:

```text
each-statement =
    [ loop-label ],
    "each",
    pattern,
    "in",
    expression,
    block ;
```

Examples:

```rg1
each item in values
{
    emit item
}
```

```rg1
each (key, value) in table
{
    emit key
}
```

Iteration order is defined by the iterable type.

The loop variable’s ownership mode is derived from the iterable and pattern.

---

# 3.59 Range Expressions

Canonical range forms are:

```text
range-expression =
      expression, "..", expression
    | expression, "..<", expression
    | expression, "..=", expression
    | expression, ".."
    | "..", expression
    | "..<", expression
    | "..=", expression ;
```

Meanings:

```text
a .. b     inclusive range, canonical compact form
a ..= b    explicit inclusive compatibility spelling
a ..< b    half-open range
a ..       range beginning at a
.. b       inclusive range ending at b
..< b      half-open range ending before b
```

The unified canonical recommendation is:

* use `..<` for iteration and indexing;
* use `..=` when explicit inclusive intent improves clarity;
* retain `..` as the general inclusive range constructor.

Earlier Raygen examples used both `..` and `..=` for inclusive ranges. Both are retained, but `..=` is preferred in contexts where the upper-bound inclusion might otherwise be visually unclear.

---

# 3.60 Infinite Loops

An unconditional loop uses:

```rg1
loop
{
    ...
}
```

Grammar:

```text
loop-statement =
    "loop",
    [ identifier ],
    block ;
```

An infinite loop has type `never` only when no reachable `break` exits it.

Named infinite loop:

```rg1
loop event_cycle
{
    ...
}
```

The identifier is available as a break or continue target.

---

# 3.61 Break Statements

The grammar is:

```text
break-statement =
    "break",
    [ identifier ],
    [ expression ],
    terminator ;
```

Examples:

```rg1
break
break outer
break result
break search with found_value
```

Canonical value-breaking form:

```rg1
break result
```

A named loop target is resolved lexically.

A loop used as an expression may accept a break value.

All reachable break values SHALL have compatible types.

---

# 3.62 Continue Statements

The grammar is:

```text
continue-statement =
    "continue",
    [ identifier ],
    terminator ;
```

Examples:

```rg1
continue
continue outer
```

The target SHALL be an enclosing loop.

Cleanup for the current iteration occurs before continuation.

---

# 3.63 Stop Statements

The grammar is:

```text
stop-statement =
    "stop",
    [ expression ],
    terminator ;
```

`stop` terminates the active execution root or runtime context according to the current profile.

Examples:

```rg1
stop
stop ExitCode.failure
```

`stop` is distinct from:

* `return`, which exits the current function;
* `break`, which exits a loop;
* `deny`, which produces a typed failure;
* task cancellation, which is cooperative.

A library profile MAY prohibit `stop`.

---

# 3.64 Direct Maps

The grammar is:

```text
direct-map-statement =
    "map",
    expression,
    "direct",
    [ "<", type-expression, ">" ],
    direct-map-body ;

direct-map-body =
    "{",
    direct-map-entry,
    { direct-map-entry },
    "}" ;

direct-map-entry =
    direct-key,
    "->",
    direct-target,
    terminator ;

direct-key =
      constant-expression
    | pattern
    | "other" ;

direct-target =
      qualified-name
    | call-expression
    | block
    | expression ;
```

Example:

```rg1
map command direct
{
    "start" -> start_engine
    "stop"  -> stop_engine
    "reset" -> reset_engine
    other   -> unknown_command
}
```

Typed key form:

```rg1
map opcode direct<u8>
{
    0x01 -> load
    0x02 -> store
    0x03 -> add
    0x04 -> multiply
    other -> invalid_opcode
}
```

---

# 3.65 Direct-Map Semantics

A direct map declares a fixed semantic mapping.

It does not require a particular branch implementation.

The compiler may lower it to:

* a jump table;
* a perfect hash;
* a binary tree;
* a vector comparison;
* a compressed table;
* a direct computed branch;
* a profile-guided hybrid.

Every key expression SHALL be compile-time-resolvable unless the selected direct-map class explicitly permits runtime patterns.

Duplicate keys are compile-time errors.

Required exhaustiveness depends upon the mapped domain.

---

# 3.66 Semantic Derivations

The grammar is:

```text
derivation-declaration =
    "derive",
    [ derivation-modifier-list ],
    identifier-list,
    "from",
    expression-list,
    [ derivation-contract-list ],
    derivation-body ;

derivation-body =
    "{",
    { derivation-equation | statement },
    "}" ;

derivation-equation =
    identifier,
    "=",
    expression,
    terminator ;
```

Example:

```rg1
derive final_price from base_price, tax_rate, discount
{
    taxed = base_price * tax_rate
    final_price = base_price + taxed - discount
}
```

Within a derivation body, `=` declares a dependency equation.

It is not mutation.

Derived names become available according to dependency order rather than merely textual order.

---

# 3.67 Derivation Modifiers

Standard derivation modifiers include:

```text
retain
dynamic
incremental
eager
lazy
parallel
stable
```

Example:

```rg1
derive retain final_price from base_price, tax_rate
{
    final_price = base_price * (1.0 + tax_rate)
}
```

Modifiers are declarative constraints.

A modifier SHALL NOT weaken dependency correctness.

Conflicting modifiers are diagnosed.

---

# 3.68 Derivation Dependencies

The `from` list declares external dependencies.

The compiler also derives internal dependencies from equations.

Example:

```rg1
derive summary from source
{
    count = source.length
    average = source.total / count
    summary = Summary(count, average)
}
```

The dependency graph is:

```text
source → count
source → average
count  → average
count  → summary
average → summary
```

A static cycle is a compile-time error:

```text
RG-DERIVE-001:
Cyclic semantic derivation.
```

---

# 3.69 Implementation Derivation

Trait or implementation generation uses a declaration-attached form:

```rg1
@derive(Equality, Hash, Copy)
record Point
{
    x: i32
    y: i32
}
```

This is distinct from semantic derivation.

The compiler SHALL NOT confuse:

```rg1
@derive(Equality)
```

with:

```rg1
derive result from source
{
    ...
}
```

The former generates implementations.

The latter creates a semantic dependency graph.

---

# 3.70 Deduction Declarations

The grammar is:

```text
deduction-declaration =
    "deduce",
    deduction-name-or-expression,
    [ "from", expression-list ],
    deduction-body ;

deduction-name-or-expression =
      identifier
    | expression ;

deduction-body =
    "{",
    { deduction-statement },
    "}" ;
```

Example:

```rg1
deduce safe_division from divisor
{
    require divisor != 0
}
```

Direct proof request:

```rg1
deduce index in 0 ..< values.length
```

A direct proof request is a simple deduction statement and requires a terminator.

---

# 3.71 Deduction Statements

Deduction bodies may contain:

```text
deduction-statement =
      deduction-binding
    | require-statement
    | verify-statement
    | assume-statement
    | derivation-equation
    | conditional-deduction ;
```

Example:

```rg1
deduce buffer_size
{
    source = element_count * size_of<Element>()
    require source <= MAX_BUFFER
}
```

`require` makes proof mandatory.

`verify` requests validation and diagnostic evidence.

`assume` introduces an unsafe or externally trusted fact according to profile.

---

# 3.72 Require Statements

The grammar is:

```text
require-statement =
    "require",
    expression,
    terminator ;
```

A `require` statement may appear:

* in a deduction body;
* in a policy;
* in a direct block;
* in an unsafe contract;
* where explicitly permitted by another grammar production.

Failure to prove or enforce the condition denies compilation.

---

# 3.73 Memory Statements

The grammar is:

```text
memory-statement =
      retain-statement
    | store-statement
    | dismiss-statement
    | free-statement ;
```

These statements retain their distinct semantics.

---

# 3.74 Retain Statements

The grammar is:

```text
retain-statement =
    "retain",
    expression,
    [ retain-clause ],
    terminator ;

retain-clause =
      "until", expression
    | "in", identifier
    | "as", identifier ;
```

Examples:

```rg1
retain session
retain session until origin_end
retain session in process_region
```

Declaration shorthand:

```rg1
retain var session: Session = create_session()
```

is grammatical sugar for:

```rg1
var session: Session = create_session()
retain session
```

The compiler may fuse the operations.

---

# 3.75 Store Statements

The grammar is:

```text
store-statement =
    "store",
    [ "copy" | "move" ],
    expression,
    "in",
    expression,
    terminator ;
```

Examples:

```rg1
store profile in user_cache
store copy profile in user_cache
store move profile in user_cache
```

When neither `copy` nor `move` is written, the operation follows the value type’s canonical storage-transfer behavior.

Professional code SHOULD write the mode explicitly when ownership is not obvious.

After `store move`, the source is unavailable.

---

# 3.76 Dismiss Statements

The grammar is:

```text
dismiss-statement =
    "dismiss",
    expression,
    terminator ;
```

Examples:

```rg1
dismiss temporary
dismiss calculate_optional_statistics()
```

Dismissal:

* ends the binding’s usable lifetime;
* destroys the value when required;
* explicitly ignores a discardable result;
* does not free unrelated backing storage;
* does not bypass a type’s destructor.

---

# 3.77 Free Statements

The grammar is:

```text
free-statement =
    "free",
    expression,
    terminator ;
```

Examples:

```rg1
free allocation
free temporary_region
```

`free` is legal only for explicitly releasable resources.

Ordinary stack and region-owned values normally use deterministic destruction and do not require `free`.

A second free is a compile-time error when detectable and undefined behavior only within an unsafe violation not statically detected.

---

# 3.78 Region Declarations

The grammar is:

```text
region-declaration =
    "region",
    identifier,
    [ region-parameter-list ],
    region-body ;

region-body =
    "{",
    { region-property | statement },
    "}" ;
```

Example:

```rg1
region request_memory
{
    capacity = 32:mb
    alignment = 64
    release = origin_end

    let buffer = allocate<byte>(4096)
}
```

Region properties use declarative `=` equations.

They are not ordinary mutations.

A region establishes an allocation and destruction scope.

---

# 3.79 Storage Declarations

The grammar is:

```text
storage-declaration =
    [ visibility ],
    "storage",
    identifier,
    [ storage-type-clause ],
    storage-body ;
```

Example:

```rg1
storage user_cache
{
    capacity = 64:mb
    alignment = 64
    policy = retained
}
```

Storage declarations define named storage capabilities.

They do not necessarily create memory until used.

---

# 3.80 Policy Declarations

The grammar is:

```text
policy-declaration =
    "policy",
    identifier,
    policy-class,
    policy-body ;

policy-class =
      "semantic"
    | "safety"
    | "optimization"
    | "diagnostic"
    | "deployment" ;

policy-body =
    "{",
    { policy-entry },
    "}" ;

policy-entry =
    policy-path,
    [ policy-operation ],
    [ expression-list ],
    terminator ;
```

Example:

```rg1
policy arithmetic semantic
{
    overflow wrap
    rounding nearest_even
}
```

Example:

```rg1
policy vectorization optimization
{
    prefer width 256
    guard aliasing
    fallback scalar
}
```

---

# 3.81 Policy Operations

Standard policy operations include:

```text
allow
deny
require
prefer
fallback
enable
disable
warning
note
target
mode
guard
```

Example:

```rg1
policy diagnostics diagnostic
{
    warning unused_binding
    deny implicit_copy
    note scalar_fallback
}
```

Policy grammar is intentionally declarative.

A policy entry SHALL NOT contain an arbitrary runtime side effect.

---

# 3.82 Case Declarations

A case is a rights-declared concurrent branch.

The grammar is:

```text
case-declaration =
    "case",
    identifier,
    [ case-header ],
    case-body ;

case-header =
    "(",
    [ case-parameter, { ",", case-parameter } ],
    ")" ;

case-body =
    "{",
    { case-contract | statement },
    "}" ;
```

Case contracts include:

```text
use read expression
use write expression
use own expression
output identifier [ ":" type-expression ]
rights expression
cancel expression
```

Example:

```rg1
case write_binary
{
    use read summaries
    output binary_status

    set binary_status <-
        write summaries to "summary.rgd"
}
```

---

# 3.83 Case Groups

The grammar is:

```text
cases-declaration =
    "cases",
    identifier,
    [ case-group-modifiers ],
    "{",
    case-declaration,
    { case-declaration },
    [ case-merge ],
    "}" ;
```

Example:

```rg1
cases reporting
{
    case write_binary
    {
        use read summaries
        output binary_status
        ...
    }

    case write_text
    {
        use read summaries
        output text_status
        ...
    }
}
```

Cases in a group are eligible for concurrent execution subject to validated rights and dependencies.

---

# 3.84 Wait Statements

The grammar is:

```text
wait-statement =
    "wait",
    expression,
    terminator ;
```

Examples:

```rg1
wait reporting
wait worker_task
wait channel.ready
```

`wait` is an instruction and may be used inside appropriate execution contexts.

The awaited expression determines synchronization semantics.

---

# 3.85 Spawn Statements and Expressions

The grammar is:

```text
spawn-expression =
    "spawn",
    expression ;

spawn-statement =
    spawn-expression,
    terminator ;
```

Examples:

```rg1
let worker = spawn process(move packet)
spawn refresh_cache()
```

An unbound spawned task is legal only when the active policy permits detached execution.

Ownership transfer remains explicit.

---

# 3.86 Layer Declarations

A layer is an ordered execution group.

The grammar is:

```text
layer-declaration =
    "layer",
    identifier,
    [ layer-modifiers ],
    block ;
```

Example:

```rg1
layer verify
{
    parallel verify(binary_status)
    parallel verify(text_status)
    parallel verify(index_status)
}
```

The `parallel` contextual instruction marks operations eligible for concurrent execution.

Unmarked statements execute according to ordinary block sequencing.

---

# 3.87 Layer Groups

The grammar is:

```text
layers-declaration =
    "layers",
    identifier,
    [ layer-group-modifiers ],
    "{",
    layer-declaration,
    { layer-declaration },
    [ merge-declaration ],
    "}" ;
```

Example:

```rg1
layers finalization
{
    layer verify
    {
        parallel verify(binary_status)
        parallel verify(text_status)
    }

    layer commit
    {
        parallel commit_files()
        parallel commit_index()
    }

    merge
    {
        emit "Finalization complete."
    }
}
```

Layers execute in declared dependency order.

Operations within a layer may run concurrently when legal.

---

# 3.88 Merge Declarations

The grammar is:

```text
merge-declaration =
    "merge",
    [ merge-policy ],
    block ;
```

A merge combines outputs from cases, tasks, or parallel layer operations.

Examples:

```rg1
merge
{
    return combine(left_result, right_result)
}
```

```rg1
merge deterministic by index
{
    ...
}
```

Merge policy controls ordering and reproducibility.

No implicit nondeterministic reduction is permitted when the active determinism level requires a stable result.

---

# 3.89 Unsafe Blocks

The grammar is:

```text
unsafe-block =
    "unsafe",
    [ unsafe-contract ],
    block ;
```

Example:

```rg1
unsafe
requires pointer_is_valid(raw)
{
    let value = read_pointer(raw)
}
```

An unsafe block enables only operations requiring unsafe authority.

It does not suspend:

* parsing;
* ordinary type checking;
* visibility;
* unrelated ownership rules;
* ordinary lexical scope;
* declared arithmetic policies.

Unsafe code remains responsible for preserving all low-level invariants.

---

# 3.90 Direct Blocks

A direct block exposes low-level operations under explicit policy.

The grammar is:

```text
direct-block =
    "direct",
    [ direct-target ],
    block ;
```

Examples:

```rg1
direct machine
{
    ...
}
```

```rg1
direct x86_64
{
    ...
}
```

Direct blocks may contain:

* intrinsics;
* assembly forms;
* device commands;
* explicit registers;
* exact memory operations.

They are subject to:

* deployment policy;
* target validation;
* unsafe rules;
* ABI restrictions;
* signature or audit requirements where selected.

A direct block SHALL NOT silently appear through ordinary optimization.

---

# 3.91 Expression Grammar Overview

Expressions include:

```text
expression =
      literal-expression
    | identifier-expression
    | grouped-expression
    | tuple-expression
    | array-expression
    | record-expression
    | block-expression
    | if-expression
    | select-expression
    | unary-expression
    | binary-expression
    | call-expression
    | member-expression
    | index-expression
    | cast-expression
    | move-expression
    | copy-expression
    | range-expression
    | pipeline-expression
    | outcome-expression
    | spawn-expression
    | closure-expression ;
```

Every expression has exactly one static type.

Mutation is not a general expression.

---

# 3.92 Primary Expressions

Primary expressions include:

```text
primary-expression =
      literal
    | qualified-name
    | "self"
    | grouped-expression
    | tuple-expression
    | array-expression
    | record-construction
    | block-expression
    | closure-expression ;
```

Grouped expression:

```rg1
(a + b)
```

Single-element tuple:

```rg1
(value,)
```

Empty tuple or unit value:

```rg1
()
```

---

# 3.93 Postfix Expressions

Postfix operations include:

```text
postfix-expression =
    primary-expression,
    {
          call-suffix
        | index-suffix
        | member-suffix
        | optional-member-suffix
        | generic-argument-suffix
    } ;
```

Examples:

```rg1
calculate(value)
values[index]
packet.header.length
optional?.field
factory<T>(value)
```

Postfix operations have the highest ordinary expression precedence.

---

# 3.94 Calls

The grammar is:

```text
call-expression =
    callable-expression,
    argument-list ;

argument-list =
    "(",
    [ argument, { ",", argument }, [ "," ] ],
    ")" ;

argument =
      expression
    | identifier, "=", expression ;
```

Examples:

```rg1
calculate(value)
connect(address, timeout = seconds(30))
```

Named arguments use `=` because they associate a parameter name with a call value; they do not mutate storage.

Arguments are evaluated left to right unless a call is proven pure and reordering is unobservable.

---

# 3.95 Generic Arguments

The grammar is:

```text
generic-argument-list =
    "<",
    generic-argument,
    { ",", generic-argument },
    [ "," ],
    ">" ;

generic-argument =
      type-expression
    | constant-expression ;
```

Examples:

```rg1
array<u8, 64>
cast<u32>(value)
create_buffer<Packet, 256>()
```

The parser determines whether a generic argument is a type or constant according to the referenced declaration.

No runtime generic argument resolution is required.

---

# 3.96 Member Access

Member access uses:

```rg1
receiver.member
```

Qualified namespace access uses:

```rg1
module::member
```

The two forms SHALL remain distinct.

Optional member access uses:

```rg1
receiver?.member
```

where the receiver type supports optional access.

---

# 3.97 Indexing

Indexing uses:

```rg1
container[index]
```

Multiple-dimensional indexing may use:

```rg1
matrix[row, column]
```

or nested indexing:

```rg1
matrix[row][column]
```

depending on the type’s indexing contract.

Bounds behavior is determined by:

* static deduction;
* active safety policy;
* safe versus unsafe context.

---

# 3.98 Unary Expressions

Unary operators include:

```text
+
-
!
~
*
&
move
copy
read
write
allow
spawn
```

Their availability depends on operand type and context.

Examples:

```rg1
-value
!ready
~mask
move packet
copy sample
allow load()
spawn worker()
```

Pointer dereference and address operations are restricted by safe or unsafe type rules.

---

# 3.99 Explicit Casts

Canonical casts use functions recognized by the language:

```rg1
cast<T>(value)
checkcast<T>(value)
bitcast<T>(value)
view<T>(value)
```

Meanings:

```text
cast       explicit semantic conversion
checkcast  checked conversion producing defined failure
bitcast    representation reinterpretation under strict legality
view       non-owning typed view
```

`bitcast` and certain views require unsafe or representation-proof conditions.

Casts occupy the postfix/conversion precedence level because their operand is explicitly delimited.

---

# 3.100 Binary Operators

Core binary operators include:

```text
**
*
/
%
+
-
<<
>>
&
^
|
<
<=
>
>=
<=>
==
!=
&&
||
??
..
..=
..<
|>
```

Operator applicability is determined by type contracts.

No operator silently promotes concrete numeric operands.

---

# 3.101 Comparison Chaining

Raygen does not implicitly chain relational comparisons.

The following:

```rg1
a < b < c
```

is invalid unless a specialized type defines the intermediate comparison meaning.

Write:

```rg1
a < b && b < c
```

This avoids evaluating a boolean as a numeric operand and makes evaluation explicit.

---

# 3.102 Equality and Identity

Equality uses:

```rg1
==
!=
```

Storage or object identity, where supported, uses an explicit operation:

```rg1
identity(left, right)
```

or a standardized identity operator defined by the relevant pointer profile.

Ordinary equality SHALL NOT silently become address identity for records or values.

---

# 3.103 Logical Operators

Logical operators are:

```rg1
&&
||
!
```

They require boolean operands.

`&&` and `||` short-circuit left to right.

Bitwise operators are distinct:

```rg1
&
|
^
~
```

No implicit conversion exists between boolean and integer bitwise operations.

---

# 3.104 Null and Optional Coalescing

Raygen safe references are non-null.

Optional values may use:

```rg1
optional_value ?? fallback
```

Optional member access may use:

```rg1
optional_value?.member
```

The exact optional type is provided through the core type system or standard prelude.

These operators do not introduce null into ordinary reference types.

---

# 3.105 Pipeline Expressions

The pipeline operator is:

```rg1
|>
```

Example:

```rg1
let active_users =
    users
    |> where row.active
    |> order row.name ascending
```

A pipeline stage receives the previous stage’s result as its input.

A stage may lower to:

* a function call;
* an iterator adapter;
* a table operation;
* a compiler-recognized intrinsic;
* a fused data-processing stage;
* a parallel or vector pipeline.

Pipeline syntax SHALL preserve left-to-right transformation order.

Intermediate values may be eliminated when not observably required.

---

# 3.106 General Pipeline Call Expansion

The general expansion rule is:

```rg1
value |> function(arguments)
```

equivalent to:

```rg1
function(value, arguments)
```

unless the function declares a named pipeline input position.

Example:

```rg1
data
    |> filter(is_valid)
    |> transform(normalize)
```

may expand conceptually to:

```rg1
transform(filter(data, is_valid), normalize)
```

The compiler is encouraged to fuse compatible stages.

---

# 3.107 Structural Pipeline Stages

Contextual pipeline stages include:

```text
where
order
then
group
select
calculate
join
index
merge
split
sort
```

Example:

```rg1
filtered
    |> group row.category
    |> calculate
    {
        count = count(rows)
        average = average(rows.value)
    }
```

These forms are compiler-recognized only when defined by the structural interface of the input type.

They are not universal keywords with arbitrary meaning.

---

# 3.108 Conditional Expressions

Raygen supports ordinary block-form conditional expressions:

```rg1
let result =
    if condition
    {
        first
    }
    else
    {
        second
    }
```

A compact conditional operator MAY be defined as:

```rg1
condition ? first : second
```

The block form is canonical.

The compact form SHALL preserve:

* left-to-right condition evaluation;
* single-branch evaluation;
* compatible branch types;
* ownership correctness.

---

# 3.109 Closures

The grammar is:

```text
closure-expression =
    [ capture-clause ],
    "|",
    [ closure-parameter-list ],
    "|",
    closure-body ;

capture-clause =
    "[",
    capture-item,
    { ",", capture-item },
    "]" ;

capture-item =
      identifier
    | "read", identifier
    | "write", identifier
    | "move", identifier ;
```

Examples:

```rg1
|value| value * 2
```

```rg1
[move packet] |channel| channel.send(packet)
```

```rg1
[read configuration] |request|
{
    process(request, configuration)
}
```

Capture mode may be inferred only when unambiguous and semantically safe.

---

# 3.110 Expression Evaluation Order

Unless otherwise specified:

* receiver expressions evaluate before arguments;
* function arguments evaluate left to right;
* binary operands evaluate left to right;
* short-circuit operators may skip the right operand;
* only the selected conditional branch evaluates;
* pipeline stages evaluate left to right;
* each expression evaluates exactly once.

The compiler may reorder only when observable behavior cannot change.

---

# 3.111 Operator Precedence

Raygen expression precedence, from highest to lowest, is:

| Level | Operators or forms                                                                            | Associativity   |
| ----: | --------------------------------------------------------------------------------------------- | --------------- |
|     1 | Primary grouping, literals, names                                                             | —               |
|     2 | Member access `.`, optional access `?.`, indexing `[]`, calls `()`, generic application       | Left            |
|     3 | Explicit casts, checked casts, views, postfix operations                                      | Left            |
|     4 | Power `**`                                                                                    | Right           |
|     5 | Unary `+ - ! ~`, `move`, `copy`, `read`, `write`, `allow`, `spawn`, dereference/address forms | Right           |
|     6 | Multiplication `*`, division `/`, remainder `%`                                               | Left            |
|     7 | Addition `+`, subtraction `-`                                                                 | Left            |
|     8 | Shifts `<< >>`                                                                                | Left            |
|     9 | Bitwise AND `&`                                                                               | Left            |
|    10 | Bitwise XOR `^`                                                                               | Left            |
|    11 | Bitwise OR `\|`                                                                               | Left            |
|    12 | Relational `< <= > >= <=>`                                                                    | Non-associative |
|    13 | Equality `== !=`                                                                              | Non-associative |
|    14 | Logical AND `&&`                                                                              | Left            |
|    15 | Logical OR `\|\|`                                                                             | Left            |
|    16 | Optional coalescing `??`                                                                      | Right           |
|    17 | Ranges `.. ..= ..<`                                                                           | Non-associative |
|    18 | Pipelines `\|>`                                                                               | Left            |
|    19 | Conditional expression `? :`                                                                  | Right           |
|    20 | Declaration-specific and association forms                                                    | Contextual      |

Examples:

```rg1
a + b * c
```

means:

```rg1
a + (b * c)
```

```rg1
value & mask == expected
```

means:

```rg1
(value & mask) == expected
```

```rg1
source |> filter(valid) |> transform(normalize)
```

groups from left to right.

---

# 3.112 Power and Unary Negation

Power binds more strongly than unary negation.

Therefore:

```rg1
-2 ** 2
```

means:

```rg1
-(2 ** 2)
```

Use:

```rg1
(-2) ** 2
```

for a negative base.

This matches mathematical convention and removes ambiguity.

---

# 3.113 Range Precedence

Range operators bind more weakly than arithmetic and logical operations but more strongly than pipelines.

Therefore:

```rg1
start + offset ..< end - reserve
```

means:

```rg1
(start + offset) ..< (end - reserve)
```

The following:

```rg1
values |> select 0 ..< limit
```

is interpreted as a pipeline stage receiving the range expression according to the stage grammar.

Parentheses are recommended where structural pipeline syntax could reduce clarity.

---

# 3.114 Mutation Is Not an Expression Operator

`<-` does not participate in ordinary expression precedence.

It appears only in grammar productions that explicitly authorize mutation or directional storage transfer.

Therefore, this is invalid:

```rg1
if set count <- 5
{
    ...
}
```

Write:

```rg1
set count <- 5

if count == 5
{
    ...
}
```

This prevents assignment-condition mistakes and keeps state changes visible.

---

# 3.115 Initialization Is Not General Assignment

Plain `=` is not a general assignment operator.

It appears in contexts including:

* new binding initialization;
* field defaults;
* named arguments;
* declaration properties;
* derivation equations;
* deduction equations;
* policy properties;
* record construction fields.

The parser SHALL determine legality from the surrounding production.

Existing mutable storage changes only through `set ... <- ...` or another explicit mutation instruction.

---

# 3.116 Record Construction

A record may be constructed as:

```rg1
let sample = Sample
{
    id = sample_id
    value = reading
    timestamp = now()
}
```

Field association uses `=` because it initializes a new record value.

A compact constructor may use:

```rg1
Sample(sample_id, reading, now())
```

only when the type declares a positional constructor.

Named construction is preferred for stable source clarity.

---

# 3.117 Collection Literals

An array literal uses:

```rg1
[1, 2, 3, 4]
```

A repeated element form may use:

```rg1
[0; 256]
```

A list or other structural collection may use a constructor:

```rg1
list
{
    first
    second
    third
}
```

The precise resulting type is determined by context or an explicit constructor.

Collection literals SHALL NOT silently mix incompatible concrete numeric types.

---

# 3.118 Table and Query Blocks

Table transformations may use structural blocks:

```rg1
let summary =
    samples
    |> where row.value >= 0.0
    |> order row.category ascending
    |> then row.timestamp ascending
    |> group row.category
    |> calculate
    {
        count = count(rows)
        minimum = minimum(rows.value)
        maximum = maximum(rows.value)
        average = average(rows.value)
    }
```

Inside `calculate`, `=` declares output fields.

It does not mutate existing bindings.

A compiler may lower the full chain into one fused pipeline.

---

# 3.119 Sort Statements

The grammar is:

```text
sort-statement =
    "sort",
    [ sort-mode-list ],
    expression,
    [ sort-key-clause ],
    terminator ;
```

Examples:

```rg1
sort stable values
sort rapid values
sort parallel records by row.key ascending
```

A value-returning form is:

```rg1
let sorted = sort stable copy values
```

An in-place sort requires writable access and explicit mutation intent.

Sorting semantics and algorithmic guarantees are defined separately.

---

# 3.120 Named Loops and Control Targets

A loop label is declared by:

```rg1
loop outer while active
{
    each item in items
    {
        if item.stop
        {
            break outer
        }
    }
}
```

A label:

* exists only within the loop body;
* cannot shadow an active loop label in the same control chain;
* may be targeted by `break` or `continue`;
* is not a runtime string.

The compiler may lower labels directly into control-flow graph edges.

---

# 3.121 Declaration Placement

Raygen permits nested:

* functions where profile-enabled;
* types;
* constants;
* derivations;
* deductions;
* regions;
* policies with block scope.

A nested declaration’s visibility cannot exceed its containing scope.

Declarations that require stable external symbols SHALL occur at module scope.

Nested declarations may be fully inlined or eliminated.

---

# 3.122 Semicolon and Newline Grammar

Every simple statement ends with:

```text
terminator =
      newline
    | ";"
    | end-of-file-when-complete ;
```

Compound statements are complete at their closing brace.

A newline belongs to the grammar only when the preceding construct is syntactically complete.

Examples:

```rg1
let result =
    first
    + second
```

one statement.

```rg1
let first = 1
let second = 2
```

two statements.

---

# 3.123 Grammar Error Recovery

A parser MAY recover after syntax errors to report additional diagnostics.

Recovery SHALL NOT make an invalid program executable.

Recovery points SHOULD include:

* terminators;
* closing braces;
* declaration-leading keywords;
* `case`;
* `else`;
* `deny`;
* `layer`;
* `origin`;
* end-of-file.

Synthesized recovery nodes SHALL be marked invalid.

Semantic analysis SHALL not treat them as ordinary source declarations.

---

# 3.124 Required Syntax Diagnostics

A conforming parser SHALL diagnose:

* missing edition declarations;
* duplicate module declarations;
* malformed import groups;
* incomplete generic lists;
* missing parameter types;
* invalid access-mode placement;
* use of `move` as a parameter qualifier;
* reversed fallible return syntax;
* missing statement terminators;
* mutation using plain `=`;
* invalid assignment targets;
* unmatched delimiters;
* malformed direct maps;
* duplicate direct-map keys where syntactically discoverable;
* malformed derivation equations;
* missing derivation dependencies;
* malformed deduction blocks;
* invalid loop targets;
* missing selection cases;
* misplaced `else`;
* misplaced `deny`;
* invalid policy class;
* invalid declaration placement.

---

# 3.125 Stable Syntax Diagnostic Codes

The minimum stable syntax codes include:

```text
RG-PARSE-001              Unexpected token
RG-PARSE-002              Expected token missing
RG-PARSE-003              Unmatched delimiter
RG-PARSE-004              Invalid declaration
RG-PARSE-005              Invalid statement
RG-PARSE-006              Invalid expression
RG-PARSE-007              Invalid type expression
RG-PARSE-008              Invalid generic argument list
RG-PARSE-009              Invalid parameter declaration
RG-PARSE-010              Invalid return contract
RG-PARSE-011              Invalid mutation syntax
RG-PARSE-012              Invalid control target
RG-PARSE-013              Invalid direct-map entry
RG-PARSE-014              Invalid derivation equation
RG-PARSE-015              Invalid policy declaration
RG-PARSE-016              Invalid origin declaration
RG-PARSE-EDITION-001      Unsupported edition
RG-PARSE-EDITION-002      Missing edition declaration
RG-PARSE-STATEMENT-001    Empty or ineffective statement
```

An implementation may refine these codes.

It SHALL NOT reuse them for unrelated conditions.

---

# 3.126 Syntactic Performance Requirements

A professional Raygen parser SHOULD support:

* linear-time parsing for ordinary source;
* bounded lookahead;
* Pratt or precedence-climbing expression parsing;
* deterministic generic-bracket handling;
* incremental parsing;
* syntax-tree reuse;
* parallel parsing of independent compilation units;
* compact immutable syntax nodes;
* lazy parsing of inactive conditional source where extensions permit;
* direct lowering from syntax nodes into typed semantic nodes.

The grammar SHALL NOT require general parser backtracking.

Contextual keywords SHALL be resolved through bounded syntactic context.

A bootstrap compiler may use:

* a hand-written recursive-descent parser;
* a Pratt expression parser;
* a block parser;
* a compact symbol prepass.

No parser generator is mandatory.

---

# 3.127 Syntax Conformance

A parser conforms to Raygen 1 when it:

* implements every required declaration form;
* recognizes ordinary and explicit calls;
* uses canonical parameter ordering;
* treats access modes consistently;
* distinguishes initialization from mutation;
* distinguishes semantic derivation from implementation derivation;
* parses direct maps without fixing their lowering;
* supports all core control-flow constructs;
* implements exhaustive selection grammar;
* recognizes count and postcondition repeat forms;
* parses named loops;
* preserves individual generic closers;
* applies the normative precedence table;
* handles newline termination deterministically;
* produces stable source ranges;
* rejects grammatically invalid programs.

A parser SHALL NOT claim conformance while:

* interpreting `=` as unrestricted mutation;
* interpreting `move` as both access mode and operation;
* allowing arbitrary outcome-return ordering;
* requiring `call` for every function invocation;
* treating direct maps as ordinary repeated `if` syntax;
* allowing derivation equations to mutate arbitrary storage;
* making operator precedence implementation-defined;
* changing parsing according to optimization level or target.

---

# Part 3 Normative Conclusion

Raygen’s unified syntax expresses execution direction directly:

```rg1
let value = create()
set value <- transform(value)
store move value in destination
```

It expresses fallibility visibly:

```rg1
let data = allow load(path)
return deny error
```

It expresses fixed control topology visibly:

```rg1
map command direct
{
    "start" -> start_engine
    other   -> reject_command
}
```

It expresses dependencies visibly:

```rg1
derive result from source
{
    result = calculate(source)
}
```

It expresses proof requirements visibly:

```rg1
deduce index in 0 ..< values.length
```

It expresses concurrency rights and ordering visibly:

```rg1
cases reporting
{
    ...
}

layers finalization
{
    ...
}
```

The grammar remains suitable for a compact compiler because:

* mutation has one explicit form;
* expressions use fixed precedence;
* blocks are self-delimiting;
* newlines terminate only complete statements;
* generic brackets remain individually tokenized;
* declarations begin with recognizable keywords;
* advanced constructs lower into ordinary typed control and data structures.

At the same time, the syntax gives a professional compiler enough explicit information to produce:

* branch-specialized control flow;
* zero-copy ownership transfer;
* fused pipelines;
* statically ordered derivations;
* eliminated checks;
* vectorized loops;
* parallel case execution;
* direct native machine code.

Raygen source remains explicit where explicitness improves correctness, and flexible where compiler freedom improves raw throughput.




## *** ##



# Part 4 — Unified Type System

## 4.1 Purpose

This Part defines the complete Raygen type system.

It establishes:

* type identity;
* scalar types;
* integer hierarchy;
* floating-point hierarchy;
* rayword;
* vectors;
* booleans;
* text;
* bytes;
* addresses;
* records;
* units;
* choices;
* tuples;
* arrays;
* lists;
* trees;
* tables;
* views;
* slices;
* references;
* ownership-qualified types;
* generic types;
* associated types;
* trait constraints;
* type inference;
* conversions;
* representation;
* ABI stability;
* compile-time type deduction.

The type system exists to define program meaning.

Optimization may change implementation.

Optimization SHALL NOT change type meaning.

---

# 4.2 Design Principles

Every Raygen type obeys the following principles.

## Meaning Before Representation

The type defines semantics.

Storage layout is chosen afterward.

---

## Explicit Conversions

No silent narrowing.

No silent ownership transfer.

No silent lifetime extension.

No silent denial conversion.

---

## Deterministic Identity

Every type has one canonical identity.

Identity never depends upon:

* optimization level
* compiler vendor
* operating system
* processor architecture

---

## Layout Independence

Except where explicitly requested, a type does not define memory layout.

Programs reason about values.

The compiler reasons about layout.

---

## Zero-Cost Abstractions

Every abstraction SHALL lower to machine code equivalent to an expert manual implementation.

No runtime metadata is required merely because a type is generic, owns data, or implements traits.

---

# 4.3 Categories of Types

Raygen recognizes seven primary categories.

```text
Scalar Types

Compound Types

Structural Types

Behavioral Types

Ownership Types

Reference Types

Compile-Time Types
```

Each category has independent semantics.

---

# 4.4 Scalar Types

Fundamental scalar types are:

```text
bool

byte

char

text

address

rayword

usize
isize

u8
u16
u32
u64
u128
u256

i8
i16
i32
i64
i128
i256

f16
f32
f64
f128
```

No implementation may remove one of these types while claiming Raygen 1 conformance.

---

# 4.5 The Native Integer

The canonical integer type is:

```text
int
```

It aliases:

```text
i256
```

Likewise,

```text
uint
```

aliases

```text
u256
```

Therefore,

```rg1
let value = 12
```

defaults to

```text
int
```

which is

```text
i256
```

This removes architecture dependence from ordinary integer programming.

---

# 4.6 Exact Width Types

Exact-width integers have guaranteed sizes.

```text
u8
u16
u32
u64
u128
u256

i8
i16
i32
i64
i128
i256
```

The compiler SHALL reject an implementation unable to represent them.

---

# 4.7 Machine Index Types

Machine-dependent sizes use

```text
usize

isize
```

These exist solely for

* indexing
* pointer arithmetic
* ABI interaction
* platform APIs

General arithmetic SHOULD prefer

```text
int

uint
```

---

# 4.8 rayword

The defining primitive of Raygen is

```text
rayword
```

A rayword represents one native execution register width.

On Raygen 1

```text
rayword == u256
```

Future editions may redefine rayword while preserving language compatibility through editioning.

Programs needing architectural portability SHOULD prefer

```text
rayword
```

Programs needing exact widths SHOULD choose

```text
u256
```

---

# 4.9 Floating Types

Floating hierarchy

```text
f16

f32

f64

f128
```

Default floating literal type:

```text
real
```

which aliases

```text
f64
```

Professional profiles may configure

```text
real = f128
```

through project policy.

The language semantics remain unchanged.

---

# 4.10 Boolean

```text
bool
```

contains only

```text
true

false
```

No integer implicitly converts to bool.

No bool implicitly converts to integer.

---

# 4.11 Character

```text
char
```

contains exactly one Unicode scalar value.

It is never UTF-8.

It is never UTF-16.

It is a Unicode scalar.

---

# 4.12 Text

```text
text
```

represents immutable Unicode text.

Its storage encoding is implementation-defined.

Its observable behavior is not.

Implementations may store text using

* UTF-8
* UTF-16
* UTF-32
* compressed forms

provided observable semantics remain identical.

---

# 4.13 Bytes

```text
bytes
```

represents immutable byte sequences.

```text
buffer<byte>
```

represents mutable byte storage.

The two are intentionally distinct.

---

# 4.14 Address

```text
address
```

is an opaque machine address.

Arithmetic upon

```text
address
```

is prohibited except through explicitly defined address operations.

This prevents accidental pointer arithmetic.

---

# 4.15 Compound Types

Compound types include

```text
record

unit

choice

tuple

array

list

tree

graph

table

buffer

slice
```

Each has well-defined semantics independent of layout.

---

# 4.16 Record Identity

Records are nominal.

Example

```rg1
record Point
{
    x: f64
    y: f64
}
```

and

```rg1
record Coordinate
{
    x: f64
    y: f64
}
```

are distinct types despite identical fields.

This preserves API evolution.

---

# 4.17 Unit Identity

Units are also nominal.

Unlike records,

units additionally constrain representation.

Units are intended for

* protocols
* binary layouts
* hardware interfaces
* packed data

---

# 4.18 Choice Types

Choices represent tagged alternatives.

Example

```rg1
choice ParseResult
{
    success(Value)

    syntax(Error)

    eof
}
```

Choices are exhaustive.

The compiler always knows every possible variant.

---

# 4.19 Tuples

Tuple types are structural.

```text
(u32, bool)

(u32, u32, text)
```

Tuple identity depends solely on element order and element types.

---

# 4.20 Arrays

```text
array<T,N>
```

Length is compile-time constant.

Storage is contiguous.

No hidden heap allocation occurs.

---

# 4.21 Lists

```text
list<T>
```

Length is dynamic.

Representation is implementation-selected.

Semantics remain identical.

---

# 4.22 Trees

```text
tree<T>
```

Represents hierarchical ownership.

Traversal order is explicit.

---

# 4.23 Tables

```text
table<Row>
```

Represents relational data.

Tables possess semantic operations

```text
where

order

group

calculate

merge
```

These are language-recognized operations.

They are not SQL.

---

# 4.24 Buffers

```text
buffer<T>
```

Mutable contiguous storage.

Capacity and length are distinct.

---

# 4.25 Slices

```text
slice<T>
```

Non-owning contiguous view.

A slice never owns storage.

Destroying a slice never destroys elements.

---

# 4.26 Views

```text
view<T>
```

General non-owning typed projection.

Views may represent

* slices
* mapped memory
* device memory
* transformed storage

Ownership never transfers.

---

# 4.27 Ownership-Qualified Types

Ownership belongs to bindings and references rather than changing nominal type identity.

Canonical ownership forms are:

```text
own T
read T
write T
locked T
atomic T
```

These are access contracts, not new nominal types.

Thus:

```text
Packet
```

and

```text
own Packet
```

refer to the same underlying type with different access semantics.

---

# 4.28 Generic Types

Every generic specialization is a distinct concrete type.

Example:

```text
array<u32,16>
```

and

```text
array<u64,16>
```

are different types.

Likewise:

```text
list<Point>
```

and

```text
list<Coordinate>
```

are different.

Generics are monomorphized by default, enabling full specialization and maximum optimization.

Implementations may share code only when doing so preserves identical semantics and performance guarantees.

---

# 4.29 Type Identity

Type identity follows these rules:

* Nominal types (`record`, `unit`, `choice`, named aliases when declared nominal) are equal only by declaration identity.
* Structural types (`tuple`, `array`, `slice`, function signatures) are equal by structure.
* Ownership qualifiers do not create new nominal identities.
* Type aliases do not create new identities unless explicitly declared nominal.

---

# 4.30 Conversions

Raygen defines four conversion classes:

```text
Implicit
Explicit
Checked
Representation
```

Only widening, lossless conversions may be implicit.

Potentially lossy conversions require `cast`.

Conversions that may fail require `checkcast`.

Bit reinterpretation requires `bitcast` and appropriate safety conditions.

---

# 4.31 Representation Independence

Except for explicitly layout-constrained types (`unit`, packed layouts, ABI-declared structures), the compiler may reorder, pad, vectorize, or otherwise represent values differently so long as observable semantics remain unchanged.

Programs reason about values.

The compiler reasons about bits.

---

# 4.32 Compile-Time Type Deduction

The compiler performs type deduction before optimization.

Every expression has exactly one resolved static type.

Ambiguous deductions are compile-time errors.

No optimization pass may reinterpret a resolved type.

---

# 4.33 SIMD and Vector Types

Raygen includes first-class vector types:

```text
vector<T, N>
```

Examples:

```text
vector<f32, 8>
vector<i32, 16>
vector<u8, 64>
```

These map to the most efficient available hardware implementation while preserving deterministic semantics.

Vector width is part of the type identity.

---

# 4.34 Outcome Types

Fallible computations use:

```text
outcome<T, E>
```

The surface syntax:

```rg1
allow<T> deny<E>
```

is syntactic sugar for the underlying `outcome<T, E>` type.

This keeps the grammar expressive while giving the compiler a single canonical representation.

---

# 4.35 Optional Types

Optional values are represented by:

```text
option<T>
```

An `option<T>` contains either:

* `some(T)`
* `none`

It is distinct from `outcome<T, E>` because absence is not failure.

---

# 4.36 ABI Stability

Types participating in public ABI boundaries must have deterministic layouts.

The compiler SHALL reject ABI-exposed declarations whose layout cannot be guaranteed.

---

# 4.37 Type-System Conformance

A conforming implementation SHALL:

* implement every required scalar type;
* preserve nominal and structural identity rules;
* implement explicit conversion semantics;
* support 256-bit native integers;
* treat `rayword` as the edition-defined execution word;
* monomorphize generics by default;
* preserve representation independence except where explicitly constrained;
* resolve every expression to one static type before optimization.

---

# Part 4 Normative Conclusion

The Raygen type system is designed to make every value, every conversion, and every ownership relationship explicit while leaving maximal freedom for optimization.

Programs describe **what a value means**.

The compiler determines **how that value is represented**.

This separation allows aggressive inlining, specialization, vectorization, register allocation, and layout optimization without weakening correctness.

The result is a type system that is simultaneously:

* deterministic,
* high-performance,
* ownership-aware,
* representation-independent,
* ABI-stable where required,
* and capable of generating machine code that matches hand-written low-level implementations while preserving the safety and clarity expected of a modern systems language.



