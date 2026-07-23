** NEW UPDATED VERSION OF RAYGEN **

# Raygen 1 Normative Core
## Part 1 — Authority & Document Integrity (Revision 2026-07-22)

### 1. Document Authority

This document is the sole normative specification for the Raygen 1 programming language.

Document Identifier: RG-NCORE-1.0

Revision: 2026-07-22

All compiler conformance, language interpretation, and future amendments SHALL reference this document identifier and revision.

Any implementation claiming conformance SHALL implement the language defined herein.

---

## 2. Supersession

This edition supersedes every previous Raygen language description where any conflict exists.

Earlier drafts remain historical reference material only.

The following materials are non-authoritative whenever they conflict with this document:

- preliminary language proposals
- design notes
- experimental drafts
- archived specifications
- examples published before Revision 2026-07-22

Only the current Normative Core establishes required compiler behavior.

---

## 3. Historical Editions

Earlier language documents MAY be retained for archival purposes.

Historical editions SHALL NOT modify or reinterpret the meaning of this specification.

A historical document SHALL clearly state that it has been superseded.

Historical examples SHALL NOT be treated as normative language rules.

---

## 4. Separation of Normative and Informative Material

This specification distinguishes normative requirements from informative material.

Normative material defines language behavior and compiler obligations.

Informative material exists only to aid understanding.

The following SHALL be considered informative unless explicitly stated otherwise:

- examples
- explanatory notes
- implementation guidance
- rationale
- optimization discussion
- historical commentary
- migration guidance
- performance discussion

Marketing, adoption, popularity, or competitive claims SHALL NOT appear within normative sections.

---

## 5. Conformance Language

The following terminology has normative meaning.

SHALL

A mandatory language requirement.

SHALL NOT

A prohibited language behavior.

REQUIRED

A mandatory implementation obligation.

MUST

Equivalent to SHALL.

MUST NOT

Equivalent to SHALL NOT.

SHOULD

A recommended behavior that may be omitted only with documented justification.

MAY

An optional implementation behavior.

No other wording carries normative force.

---

## 6. Normative Precedence

Whenever multiple language rules appear applicable, the following precedence order SHALL apply.

1. Language semantics
2. Type safety
3. Ownership and lifetime correctness
4. Concurrency and memory-model correctness
5. Explicit semantic policies
6. Explicit safety policies
7. Mandatory ABI and representation contracts
8. Mandatory layout requirements
9. Derived semantic facts
10. Optimization requirements
11. Optimization preferences
12. Target-specific heuristics

No lower-precedence rule may invalidate a higher-precedence rule.

Optimization SHALL preserve all higher-precedence guarantees.

---

## 7. Compilation Ordering

A conforming implementation SHALL establish correctness before optimization.

The minimum semantic ordering is:

1. lexical analysis
2. parsing
3. name resolution
4. type checking
5. ownership and lifetime analysis
6. alias and provenance analysis
7. concurrency analysis
8. integer-width determination
9. foreign-interface validation
10. semantic deduction
11. legality verification
12. optimization
13. lowering
14. code generation

Implementations MAY internally reorder phases provided that the observable semantics remain identical to this ordering.

Semantic deduction SHALL be considered valid only after all prerequisite semantic analyses have completed.

---

## 8. Section References

Every normative section SHALL possess a unique section number.

Normative references SHALL resolve unambiguously to a single section within this document.

No duplicated section numbering is permitted.

Future amendments SHALL preserve stable section references whenever practical.

---

## 9. Amendment Rules

Future revisions SHALL identify every modified section.

Each amendment SHALL specify:

- affected section numbers
- replacement text
- effective revision
- compatibility impact

Partial amendments SHALL NOT redefine unaffected sections.

---

## 10. Compiler Conformance

A compiler conforms to Revision 2026-07-22 when it:

- implements every mandatory rule defined by this specification;
- rejects every prohibited construct identified herein;
- preserves the required semantic guarantees;
- reports non-conforming programs as diagnostics; and
- does not rely upon superseded editions when interpreting source programs.

Conformance SHALL be determined solely against this revision of the Normative Core.



# Raygen 1 Normative Core
## Part 2 — Grammar Completeness and Syntax Consistency

### 2.1 Purpose

This section defines the normative grammar requirements for Raygen 1.

A conforming parser SHALL accept every syntactically valid Raygen 1 program and SHALL reject every syntactically invalid program according to this specification.

Informative examples SHALL NOT override the grammar.

---

# 2.2 Grammar Package

The Raygen 1 grammar consists of the following normative components:

- Lexical grammar
- Tokenization rules
- Unicode rules
- Comment grammar
- Declaration grammar
- Statement grammar
- Expression grammar
- Type grammar
- Generic grammar
- Policy grammar
- Concurrency grammar
- Memory-directive grammar

A conforming implementation SHALL implement every component.

No grammar production referenced by another production may remain undefined.

---

# 2.3 Language Edition Declaration

Every source file SHALL begin with a language edition declaration.

version-declaration =     "raygen",     edition-number,     terminator ;  edition-number =     decimal-digit,     { decimal-digit } ;

For Raygen 1 the only accepted edition number is:

1

The following forms are invalid edition declarations:

1:u8 0x01 01 1_0

Edition numbers are language identifiers and SHALL NOT use numeric literal syntax.

---

# 2.4 Statement Categories

Statements are divided into two categories.

## Simple Statements

Simple statements terminate with a statement terminator.

Examples include:

- declarations
- assignments
- return statements
- outcome statements
- expression statements

## Compound Statements

Compound statements are self-delimiting.

Examples include:

- blocks
- if
- while
- repeat
- each
- select
- cases
- layers
- direct blocks

Compound statements SHALL NOT require an implicit semicolon after the closing brace.

---

# 2.5 Statement Terminators

The only statement terminators are:

newline ";"

A closing brace is not a statement terminator.

Instead, the grammar defines compound statements as complete syntactic units.

---

# 2.6 Function Return Grammar

A function return type SHALL use one of the following forms.

return-type =       type-expression     | fallible-return-type     | "void"     | "never" ;

fallible-return-type =     "allow",     "<",     type-expression,     ">",     "deny",     "<",     type-expression,     ">" ;

The ordering is fixed.

allow<T> deny<E>

is valid.

Every other ordering is invalid.

The abbreviated form

return allow

is valid only when the allowed type is void.

---

# 2.7 Parameter Grammar

Parameters SHALL use the following syntax.

parameter =     identifier,     ":",     [ access-mode ],     type-expression,     [ "=",       expression ] ;

Examples:

input: read slice<f32>  output: write slice<f32>  packet: own Packet  state: locked SharedState

Access qualification forms part of the parameter contract.

It does not modify the declared type itself unless otherwise specified by the type system.

---

# 2.8 Derivation Grammar

A derivation body defines declarative output equations.

derivation-equation =     identifier,     "=",     expression,     terminator ;

Within a derivation body,

=

defines a derived semantic relationship.

It SHALL NOT be interpreted as:

- initialization
- mutation
- reassignment

Outside derivation bodies, mutation SHALL continue to use the normal assignment syntax defined elsewhere.

---

# 2.9 Function Calls

Function calls are expressions.

perform_job()  calculate(x)

The language does not define a mandatory call statement.

Implementations MAY accept call only as deprecated compatibility syntax when operating outside strict Raygen 1 conformance mode.

---

# 2.10 Newline Significance

A newline terminates a statement only when the preceding construct is syntactically complete.

A newline SHALL NOT terminate a statement:

- within unmatched delimiters,
- immediately following an operator,
- immediately following a continuation token,
- while parsing generic argument lists.

Continuation operators include:

= <- |> , . :: + - * / % & | ^ && || < <= > >= == !=

The following contextual continuation keywords also suppress termination:

else allow deny then where

---

# 2.11 Generic Parsing

The lexer SHALL always emit individual < and > tokens.

Shift operators are formed during expression parsing.

Nested generic arguments therefore parse as:

Outer<Inner<T>>

using two closing angle tokens.

The lexer SHALL NOT emit a >> token while parsing generic argument lists.

---

# 2.12 Unicode Identifiers

Identifiers SHALL be normalized using Unicode NFC before comparison.

Name resolution operates on normalized identifiers.

Secure diagnostic profiles SHOULD diagnose:

- mixed-script identifiers
- visually confusable identifiers

Normalization SHALL NOT modify preserved source locations used for diagnostics.

---

# 2.13 Block Comments

Raygen 1 block comments are nested.

Every conforming lexer SHALL count nested opening and closing delimiters.

An unterminated block comment SHALL produce a lexical error.

Nested block comments are mandatory language behavior.

---

# 2.14 Keyword Classes

Every keyword SHALL belong to exactly one category.

## Hard Keywords

Reserved in every syntactic context.

## Contextual Keywords

Reserved only within specific grammar productions.

## Predeclared Identifiers

Defined by the language but not reserved.

## Standard Library Names

Provided by the standard library and never reserved by the grammar.

A word SHALL NOT appear simultaneously in more than one category.

---

# 2.15 Parser Conformance

A parser conforms to Raygen 1 when it:

- implements the complete normative grammar;
- classifies every example as normative or informative;
- accepts every valid grammar construct;
- rejects every invalid construct deterministically;
- performs Unicode normalization as specified; and
- parses edition declarations according to this section.

No implementation may claim parser conformance while relying upon undefined grammar productions or implementation-specific parsing behavior.



# Raygen 1 Normative Core
## Part 3 — Type System and Failure-Flow Consistency

### 3.1 Purpose

This section defines the normative Raygen type system.

The type system establishes:

- static type identity,
- ownership,
- borrowing,
- lifetime legality,
- failure-flow,
- numeric legality,
- generic substitution, and
- representation independence.

A conforming implementation SHALL reject any program whose type correctness cannot be established.

---

# 3.2 Fundamental Principles

The Raygen type system is governed by the following rules.

1. Every expression has exactly one static type.

2. Type checking is deterministic.

3. No implicit narrowing conversion exists.

4. No implicit signedness conversion exists.

5. Ownership is part of semantic analysis.

6. Failure-flow is statically typed.

7. Layout never changes type identity.

---

# 3.3 Outcome Type

Raygen defines one canonical fallible type constructor.

text outcome<Allowed, Denied> 

This type contains exactly two variants:

text allowed(Allowed)  denied(Denied) 

The surface syntax

text allow<T> deny<E> 

is defined as canonical syntactic sugar for

text outcome<T,E> 

No alternative interpretation is permitted.

---

# 3.4 Outcome Construction

Successful completion constructs an allowed outcome.

text return allow value 

is equivalent to

text return allowed(value) 

Failure constructs a denied outcome.

text return deny error 

is equivalent to

text return denied(error) 

Both constructions are typed.

---

# 3.5 Outcome Matching

Outcome matching SHALL distinguish allowed and denied branches.

Example:

text allow result as value { ... }  deny result as error { ... } 

Both branches are statically typed.

A match is exhaustive only when every outcome variant is handled.

---

# 3.6 Outcome Propagation

Propagation using

text allow expression 

is permitted only when:

- the expression has type outcome<A,D>, and
- the enclosing function returns a compatible denied type.

The compiler SHALL verify denial compatibility.

---

# 3.7 Denial Compatibility

Propagation is legal only when one of the following holds.

### Exact identity

text deny<IoError>  ↓  deny<IoError> 

---

### Explicit conversion

text IoError  ↓  LoadError 

through a declared conversion.

---

### Declared variant inclusion

A denial type may explicitly include another denial type as a variant.

No implicit widening or narrowing of denial types exists.

---

# 3.8 Void Outcomes

The following declaration

text allow<void> deny<E> 

may be written as

text void deny<E> 

only where explicitly permitted by the grammar.

return allow

constructs

text allowed(()) 

where () denotes the unit value.

---

# 3.9 Access Qualification

Access qualifiers define semantic contracts.

They are not independent types.

The defined qualifiers are:

text own  read  write  locked  atomic 

Each qualifier has exactly one meaning.

---

## own

Ownership transfers into the callee.

The caller relinquishes ownership.

---

## read

A shared immutable borrow.

Mutation is prohibited.

---

## write

An exclusive mutable borrow.

No competing mutable access is permitted.

---

## locked

Access requires possession of the specified lock capability.

---

## atomic

The referenced object must already possess atomic storage semantics.

This qualifier does not transform ordinary storage into atomic storage.

---

# 3.10 Move

move is an expression-level transfer operation.

Example:

text move packet 

It is not an access qualifier.

Accordingly,

text packet: move Packet 

is not valid Raygen 1 syntax.

Ownership transfer in declarations SHALL use

text own Packet 

---

# 3.11 Atomic Types

Atomicity is primarily a storage property.

Examples:

text atomic<u32>  atomic<bool> 

Parameter contracts may require atomic access:

text counter: read atomic<u32> 

They SHALL NOT reinterpret

text u32 

as atomic storage.

---

# 3.12 Literal Types

Integer literals initially possess the abstract compile-time type

text integer 

Floating literals initially possess

text real 

Context determines their concrete type.

Without contextual expectation:

text integer → int  real → real 

---

# 3.13 Unary Negation

Unary negation is applied after literal parsing.

Thus

text -128:i8 

is parsed as

text -(128:i8) 

Representability is then checked.

---

# 3.14 Arithmetic Typing

Binary arithmetic requires identical concrete operand types.

Example:

Valid

text u16 + u16 

Invalid

text u8 + u16 

unless one operand is explicitly converted.

Raygen performs no implicit numeric promotion.

---

# 3.15 Numeric Conversion

Conversions SHALL be explicit.

Example:

text cast<u16>(small) 

Implementations SHALL reject any implicit narrowing conversion.

---

# 3.16 Constant Evaluation

Compile-time arithmetic follows the active arithmetic policy.

The policies are defined as follows.

## deny

Overflow is a compile-time error.

---

## wrap

Arithmetic is performed modulo 2ⁿ.

---

## saturate

Results clamp to the numeric minimum or maximum.

---

## clamp

Requires an explicit programmer-supplied range.

Without a declared range the operation is ill-formed.

---

## report

Produces an outcome-bearing result.

It never yields a plain scalar.

---

## widen

Uses a statically defined wider type.

If no wider type exists, compilation fails.

Compile-time and runtime evaluation SHALL produce identical observable behavior.

---

# 3.17 Floating-Point Semantics

Raygen adopts IEEE 754 semantics where supported.

Implementations lacking native support may provide conforming software emulation.

The specification defines:

- signed zero,
- infinities,
- NaNs,
- rounding,
- conversion,
- exceptional values,
- intermediate precision.

Profiles requiring deterministic execution SHALL additionally define canonical NaN behavior.

---

# 3.18 Generic Types

Raygen 1 supports:

- type parameters,
- constant integer parameters,
- conjunction trait bounds.

Raygen 1 excludes:

- specialization,
- higher-kinded types,
- implicit variance,
- higher-ranked lifetimes.

Generic semantics SHALL be independent of implementation strategy.

Whether code is monomorphized or shared SHALL NOT alter observable behavior.

---

# 3.19 Trait Coherence

A trait implementation is legal only when the implementation package defines either:

- the trait, or
- the implementing type.

For every concrete type/trait pair:

exactly one implementation SHALL exist.

Specialization is prohibited.

---

# 3.20 Target Legality

Every type belongs to one of three categories.

text Language-valid  Target-native  Target-emulated 

A type SHALL remain language-valid even if the current target requires emulation.

Deployment policies may prohibit emulated implementations.

---

# 3.21 Slice Types

A slice is a non-owning contiguous view.

A slice contains:

- provenance,
- length,
- access qualification.

Semantics:

text read slice<T> 

shared immutable borrow.

text write slice<T> 

exclusive mutable borrow.

Slices do not own their backing storage.

Ownership requires a separate owning buffer type.

---

# 3.22 Text Type

The language distinguishes:

text text  bytes  character 

Their concrete storage representation is implementation-independent.

The semantic interface specifies:

- Unicode encoding,
- ownership,
- borrowing,
- slicing,
- equality,
- hashing,
- ordering.

Implementations SHALL preserve semantic equivalence regardless of internal representation.

---

# 3.23 Type Conformance

A conforming implementation SHALL:

- implement the canonical outcome type,
- verify denial compatibility,
- reject implicit numeric promotion,
- enforce ownership qualifiers,
- distinguish storage from access permissions,
- implement deterministic generic semantics,
- enforce trait coherence,
- classify target support independently of language legality, and
- reject any program whose type correctness cannot be established.

No implementation may claim Raygen 1 type-system conformance while providing incompatible interpretations of allow, deny, ownership qualifiers, arithmetic typing, or generic substitution.



# Raygen 1 Normative Core
## Part 4 — Ownership, Borrowing, Lifetimes, Regions, and Destruction

### 4.1 Purpose

This section defines the normative ownership model for Raygen 1.

The ownership system guarantees:

- memory safety,
- deterministic destruction,
- absence of use-after-free,
- absence of double destruction,
- statically verified borrowing,
- deterministic lifetime analysis.

A conforming implementation SHALL enforce these rules during semantic analysis.

---

# 4.2 Ownership Principles

Every runtime object has exactly one ownership state.

An object is either:

- owned,
- borrowed immutably,
- borrowed mutably,
- moved, or
- destroyed.

No object may simultaneously exist in multiple ownership states.

---

# 4.3 Ownership Identity

An owned value possesses exclusive authority over its storage.

Ownership includes responsibility for:

- destruction,
- resource release,
- lifetime termination.

Ownership is transferred only through language-defined move operations.

Copying ownership is prohibited unless the type explicitly permits duplication.

---

# 4.4 Move Semantics

A move transfers ownership from one binding to another.

Example:

text let packet2 = move packet1 

After a successful move:

- the destination becomes the owner,
- the source becomes invalid.

Any subsequent use of the source SHALL be rejected.

Example:

text let b = move a  print(a) 

The final statement is ill-formed.

---

# 4.5 Copy Semantics

Types may define copy semantics.

Copying duplicates the value without transferring ownership.

Only types satisfying the language's copy requirements may be copied implicitly.

All other types require explicit movement.

---

# 4.6 Borrowing

Borrowing creates temporary access without transferring ownership.

Borrowing never extends ownership.

The original owner remains responsible for destruction.

Borrowing creates a lifetime constraint between:

- borrower,
- owner.

---

# 4.7 Immutable Borrow

An immutable borrow provides shared read-only access.

Example:

text read Packet 

Properties:

- unlimited concurrent immutable borrows,
- mutation prohibited,
- ownership unchanged.

---

# 4.8 Mutable Borrow

A mutable borrow provides exclusive write access.

Example:

text write Packet 

During an active mutable borrow:

- no immutable borrow may coexist,
- no second mutable borrow may exist,
- ownership remains unchanged.

---

# 4.9 Borrow Exclusivity

At every program point exactly one of the following is true:

- any number of immutable borrows exist, or
- exactly one mutable borrow exists.

Both conditions SHALL NEVER hold simultaneously.

Violation is a compile-time error.

---

# 4.10 Lifetime

Every reference has a statically determined lifetime.

A lifetime begins when the borrow is created.

A lifetime ends at the final legal use of that borrow.

Lifetime analysis SHALL use semantic usage rather than lexical scope whenever possible.

---

# 4.11 Non-Lexical Lifetimes

Implementations SHALL support non-lexical lifetime analysis.

A borrow terminates after its final observable use.

Example:

text read data  print(data)  /* borrow ends here */  modify(data_owner) 

The borrow does not remain active merely because the enclosing block continues.

---

# 4.12 Lifetime Validity

A borrow SHALL NOT outlive its owner.

Example:

text {     let value      return read value } 

The returned borrow exceeds the owner's lifetime.

The program is ill-formed.

---

# 4.13 Escape Analysis

References SHALL NOT escape beyond their verified lifetime.

Examples include:

- returning local borrows,
- storing short-lived references into long-lived storage,
- spawning asynchronous work using expired references.

The compiler SHALL reject every escaping borrow.

---

# 4.14 Region Model

Each allocation belongs to exactly one region.

Regions define:

- allocation scope,
- destruction scope,
- lifetime boundaries.

Objects from different regions remain independent unless explicitly connected through ownership transfer.

---

# 4.15 Region Nesting

Child regions terminate before parent regions.

The following invariant SHALL hold:

text child lifetime ⊂ parent lifetime 

References from a parent region into a child region are prohibited unless lifetime analysis proves safety.

---

# 4.16 Destruction

Every owned object is destroyed exactly once.

Destruction occurs when ownership terminates.

A conforming implementation SHALL guarantee:

- no double destruction,
- no missed destruction,
- deterministic ordering.

---

# 4.17 Destruction Order

Objects within the same scope are destroyed in reverse construction order.

Example:

text let first  let second 

Destruction proceeds as:

text destroy(second)  destroy(first) 

This ordering is normative.

---

# 4.18 Partial Initialization

Objects are considered initialized only after successful completion of initialization.

If initialization fails:

- initialized components are destroyed,
- uninitialized components are ignored.

The compiler SHALL preserve deterministic cleanup.

---

# 4.19 Destructors

A destructor executes automatically when ownership ends.

A destructor SHALL execute at most once.

Destructors SHALL NOT be invoked explicitly by user code except through language-defined unsafe mechanisms.

---

# 4.20 Destruction During Failure

If execution terminates because of denial propagation,

all active owned objects whose lifetimes end during stack unwinding SHALL be destroyed in reverse ownership order.

Cleanup SHALL remain deterministic.

---

# 4.21 Cyclic Ownership

Pure ownership graphs SHALL be acyclic.

Reference cycles that prevent deterministic destruction are prohibited.

Shared ownership mechanisms, if provided by future language editions, SHALL define independent cycle management semantics.

---

# 4.22 Pinning

Pinned objects possess stable storage addresses.

Pinned ownership prohibits relocation after pinning.

Move operations involving pinned values are illegal unless explicitly released.

---

# 4.23 Self-Referential Objects

Self-referential objects require stable addresses.

Construction is legal only after the object has obtained stable storage.

Implementations SHALL reject constructions that would invalidate internal references through relocation.

---

# 4.24 Aliasing

Aliasing is permitted only through immutable borrowing.

Mutable aliasing is prohibited unless explicitly authorized by the language's concurrency model.

Unsafe mechanisms may temporarily suspend aliasing guarantees but SHALL remain outside ordinary safe semantics.

---

# 4.25 Ownership Across Function Calls

Ownership behavior is determined by parameter qualification.

text own T 

Ownership transfers into the callee.

text read T 

Shared borrow.

text write T 

Exclusive mutable borrow.

Ownership SHALL behave exactly as specified by the declared parameter contract.

---

# 4.26 Return Ownership

Returning an owned value transfers ownership to the caller.

Returning a borrow transfers only the borrow.

The compiler SHALL verify that returned borrows remain valid.

---

# 4.27 Closures

A closure may capture variables by:

- immutable borrow,
- mutable borrow,
- ownership transfer.

The capture mode SHALL be inferred only when the result is unambiguous.

Otherwise an explicit capture declaration is required.

Captured lifetimes remain subject to ordinary borrow verification.

---

# 4.28 Concurrency Interaction

Ownership rules remain valid across concurrent execution.

No ownership rule is relaxed because execution becomes parallel.

Ownership transfer between concurrent execution contexts SHALL be explicit.

---

# 4.29 Unsafe Ownership Operations

Unsafe operations may temporarily bypass ownership verification.

Unsafe code SHALL remain responsible for preserving:

- lifetime validity,
- aliasing correctness,
- destruction correctness,
- ownership consistency.

Violation produces undefined behavior.

---

# 4.30 Ownership Conformance

A conforming implementation SHALL:

- enforce exclusive ownership,
- reject invalid moves,
- verify borrow lifetimes,
- implement non-lexical lifetime analysis,
- perform escape analysis,
- guarantee deterministic destruction,
- reject use-after-move,
- reject dangling references,
- reject mutable aliasing,
- preserve ownership semantics across concurrency boundaries.

No implementation may claim ownership conformance while permitting use-after-free, double destruction, invalid borrowing, or lifetime violations within safe Raygen programs.



# Raygen 1 Normative Core
## Part 5 — Concurrency, Tasks, Synchronization, Memory Model, and Determinism

### 5.1 Purpose

This section defines the normative concurrency model for Raygen 1.

The concurrency model establishes:

- task creation,
- task execution,
- synchronization,
- inter-task communication,
- memory visibility,
- atomic operations,
- data race prevention,
- cancellation semantics,
- deterministic behavior.

A conforming implementation SHALL enforce these rules for all concurrent programs.

---

# 5.2 Concurrency Principles

Raygen concurrency is based upon the following principles.

1. Concurrency is explicit.

2. Parallel execution is never inferred.

3. Shared mutable state requires synchronization.

4. Data races are prohibited within safe code.

5. Ownership remains valid across concurrent execution.

6. Memory visibility is defined solely by this specification.

---

# 5.3 Task

A task is an independent execution context.

Each task possesses:

- its own execution stack,
- its own local variables,
- its own lifetime,
- its own failure state.

Tasks execute concurrently according to the implementation scheduler.

---

# 5.4 Task Creation

A task is created using the language-defined task construction syntax.

Creating a task SHALL establish:

- a new execution context,
- an independent stack,
- an initial entry point.

Task creation SHALL NOT implicitly share mutable ownership.

---

# 5.5 Ownership Transfer

Objects transferred into a task SHALL move ownership.

Example (illustrative):

text spawn worker(move packet) 

Following transfer:

- the spawned task owns the object,
- the parent task no longer owns it.

Subsequent use by the originating task is ill-formed.

---

# 5.6 Borrowing Across Tasks

Borrowed references SHALL NOT outlive the creating task.

A borrowed reference may be shared with another task only when:

- its lifetime is proven valid,
- synchronization guarantees safe access,
- the reference satisfies the language's concurrency requirements.

Otherwise the program is ill-formed.

---

# 5.7 Shared State

Shared mutable state SHALL be protected by synchronization.

Permitted mechanisms include:

- locks,
- atomic objects,
- message passing,
- implementation-defined synchronization primitives conforming to this specification.

Unsynchronized mutable sharing is prohibited.

---

# 5.8 Data Race

A data race exists when:

- two or more tasks access the same memory,
- at least one access is a write,
- the accesses are unordered,
- synchronization is absent.

Data races are prohibited in safe Raygen programs.

A conforming implementation SHALL reject statically detectable races and SHALL define any remaining races within unsafe code as undefined behavior.

---

# 5.9 Happens-Before Relation

The language defines a happens-before relation.

If operation A happens-before operation B, then:

- every write performed by A is visible to B,
- memory effects preceding A are visible to B.

The happens-before relation is established only through language-defined synchronization.

Execution order alone does not establish visibility.

---

# 5.10 Synchronization Operations

The following operations establish synchronization:

- lock acquisition,
- lock release,
- atomic synchronization operations,
- task completion,
- message transfer,
- barrier synchronization.

Implementations SHALL preserve the visibility guarantees defined by these operations.

---

# 5.11 Locking

A lock provides mutually exclusive access.

At most one task may hold a lock simultaneously.

While a lock is held:

- protected mutable state may be accessed,
- competing lock acquisition SHALL block or fail according to the selected locking policy.

---

# 5.12 Lock Ordering

Programs SHOULD acquire multiple locks in a consistent order.

Implementations MAY diagnose inconsistent lock ordering.

The language does not guarantee deadlock prevention.

Deadlock avoidance remains the programmer's responsibility unless a higher-level synchronization abstraction is employed.

---

# 5.13 Atomic Objects

Atomic objects provide indivisible operations.

Supported operations include:

- load,
- store,
- exchange,
- compare-and-exchange,
- fetch-add,
- fetch-subtract,
- implementation-defined atomic primitives.

Atomicity applies only to atomic storage.

---

# 5.14 Memory Ordering

Raygen defines the following memory ordering modes:

text relaxed  acquire  release  acquire_release  sequentially_consistent 

Every atomic operation SHALL specify an ordering either explicitly or through a language-defined default.

The meaning of each ordering is normative.

---

## relaxed

Provides atomicity only.

No synchronization is established.

---

## acquire

Subsequent reads and writes SHALL observe memory effects synchronized with the corresponding release operation.

---

## release

All preceding writes become visible to a corresponding acquire operation.

---

## acquire_release

Combines acquire and release semantics.

---

## sequentially_consistent

Participates in a single global total order of sequentially consistent atomic operations.

---

# 5.15 Fences

A memory fence establishes ordering without directly accessing data.

Fence semantics SHALL follow the selected memory ordering.

Fences do not replace ownership verification.

---

# 5.16 Task Completion

Task completion synchronizes with any successful join operation.

After a join:

- all writes performed by the completed task become visible,
- ownership returned by the task transfers to the joining task.

---

# 5.17 Cancellation

Cancellation requests cooperative task termination.

Cancellation SHALL NOT immediately terminate execution.

A task may observe cancellation only at defined cancellation points.

Resources SHALL be released according to ordinary destruction rules.

---

# 5.18 Cancellation Points

Cancellation points include:

- explicit cancellation checks,
- blocking synchronization,
- task join,
- implementation-defined interruptible operations.

Implementations SHALL document additional cancellation points.

---

# 5.19 Message Passing

Message passing transfers ownership between tasks.

Messages SHALL be delivered according to the semantics of the selected communication primitive.

Ownership of transferred objects passes to the receiving task upon successful receipt.

---

# 5.20 Channels

Channels provide synchronized communication.

A channel defines:

- element type,
- capacity,
- blocking behavior,
- closure semantics.

Operations on closed channels SHALL produce behavior defined by the channel specification.

---

# 5.21 Barriers

A barrier synchronizes a fixed collection of participating tasks.

No participating task proceeds beyond the barrier until all required participants have arrived.

Barrier completion establishes a happens-before relationship among participating tasks.

---

# 5.22 Thread-Local Storage

Thread-local objects belong exclusively to one task.

Thread-local storage SHALL NOT be shared directly with other tasks.

Ownership transfer requires explicit movement into shared synchronization mechanisms.

---

# 5.23 Deterministic Execution

Unless otherwise specified, task scheduling is implementation-defined.

Programs SHALL NOT depend upon any particular scheduling order.

Observable behavior SHALL depend only upon properly synchronized interactions.

---

# 5.24 Unsafe Concurrency

Unsafe code may bypass certain concurrency guarantees.

Unsafe concurrent code SHALL remain responsible for preserving:

- memory safety,
- synchronization correctness,
- ownership validity,
- lifetime correctness.

Violation results in undefined behavior.

---

# 5.25 Memory Consistency

Every conforming implementation SHALL provide a memory model consistent with:

- the happens-before relation,
- atomic ordering semantics,
- synchronization visibility rules,
- ownership guarantees.

No implementation may expose behavior contradicting these guarantees.

---

# 5.26 Scheduler Independence

The observable semantics of a correct Raygen program SHALL be independent of:

- processor count,
- scheduling algorithm,
- execution timing,
- operating system thread assignment.

Implementation performance characteristics MAY differ, but language semantics SHALL remain identical.

---

# 5.27 Concurrency Conformance

A conforming implementation SHALL:

- implement explicit task creation,
- preserve ownership across task boundaries,
- prohibit data races in safe code,
- implement the defined happens-before relation,
- support the specified atomic memory orderings,
- synchronize task completion,
- enforce deterministic destruction during cancellation,
- preserve scheduler-independent language semantics.

No implementation may claim Raygen 1 concurrency conformance while permitting observable violations of the defined memory model, ownership rules, synchronization semantics, or atomic ordering guarantees within safe programs.



# Raygen 1 Normative Core
## Part 6 — Traits, Interfaces, Derivations, Generic Constraints, and Compile-Time Resolution

### 6.1 Purpose

This section defines the normative abstraction system for Raygen 1.

The abstraction system establishes:

- traits,
- interfaces,
- implementations,
- derivations,
- associated members,
- generic constraints,
- overload resolution,
- method dispatch.

A conforming implementation SHALL apply these rules uniformly during semantic analysis.

---

# 6.2 Design Principles

The abstraction system is governed by the following principles.

1. Every implementation is statically known.

2. Trait resolution is deterministic.

3. Exactly one applicable implementation shall exist.

4. Generic substitution preserves program semantics.

5. Derivations are declarative.

6. Runtime dispatch is explicit.

---

# 6.3 Trait Definition

A trait defines a collection of behavioral requirements.

A trait may contain:

- required functions,
- provided default functions,
- associated types,
- associated constants.

Traits do not contain storage.

---

# 6.4 Trait Members

Each member of a trait has a unique identifier.

A member SHALL NOT be overloaded solely by return type.

Every declared member has exactly one complete type signature.

---

# 6.5 Required Members

A required member has no implementation within the trait.

Every concrete implementation SHALL provide all required members unless satisfied by inherited defaults.

Failure to satisfy all required members is a compile-time error.

---

# 6.6 Default Members

A trait may provide default implementations.

An implementing type may override a default implementation.

If no override exists, the default implementation is used.

Default implementations participate in ordinary type checking.

---

# 6.7 Associated Types

Traits may declare associated types.

Example (illustrative):

text id="p7q9m1" type Item 

Each implementation SHALL bind every associated type exactly once.

Associated types become part of the implementation's public contract.

---

# 6.8 Associated Constants

Traits may declare associated constants.

Each implementation SHALL define every required associated constant.

Associated constants are compile-time values.

---

# 6.9 Trait Implementation

An implementation binds a concrete type to a trait.

Each implementation SHALL satisfy:

- all required members,
- all associated types,
- all associated constants,
- every declared constraint.

Incomplete implementations are ill-formed.

---

# 6.10 Coherence

For every concrete type and trait pair:

exactly one implementation SHALL exist.

Multiple applicable implementations are prohibited.

Implementations SHALL be unambiguous.

---

# 6.11 Orphan Rule

A package may define a trait implementation only if it defines:

- the trait, or
- the implementing type.

This rule preserves global coherence.

---

# 6.12 Specialization

Trait specialization is not part of Raygen 1.

An implementation SHALL NOT partially override another implementation based upon more specific type arguments.

Future language editions may define specialization independently.

---

# 6.13 Generic Parameters

Generic declarations may introduce:

- type parameters,
- constant integer parameters.

Each parameter has a unique identifier.

Parameter names SHALL be distinct within the same declaration.

---

# 6.14 Generic Constraints

Generic parameters may declare trait requirements.

Example (illustrative):

text id="k3f0r8" T where Comparable 

A generic argument SHALL satisfy every declared constraint.

Unsatisfied constraints are compile-time errors.

---

# 6.15 Constraint Satisfaction

Constraint satisfaction is determined entirely at compile time.

A type satisfies a constraint only if:

- an applicable implementation exists, and
- every required associated member is available.

No runtime constraint resolution occurs.

---

# 6.16 Generic Substitution

Generic substitution replaces formal parameters with concrete arguments.

Substitution SHALL preserve:

- type correctness,
- ownership semantics,
- lifetime correctness,
- failure-flow semantics.

No substitution may alter observable language behavior.

---

# 6.17 Monomorphization Independence

Implementations MAY generate:

- specialized code,
- shared generic code,
- hybrid strategies.

The chosen implementation strategy SHALL NOT alter program semantics.

---

# 6.18 Method Resolution

Method resolution proceeds in the following order:

1. Inherent members of the concrete type.

2. Trait members made available through satisfied constraints.

3. Explicitly qualified trait members.

If resolution remains ambiguous, compilation SHALL fail.

---

# 6.19 Fully Qualified Calls

When multiple member names are visible, the programmer SHALL explicitly identify the intended trait.

A fully qualified call uniquely specifies:

- the implementing type,
- the trait,
- the selected member.

---

# 6.20 Overload Resolution

Overload resolution considers:

- function name,
- parameter count,
- parameter types,
- generic substitutions,
- constraint satisfaction.

Return types SHALL NOT participate in overload resolution.

If multiple candidates remain equally applicable, the call is ambiguous.

Ambiguous calls SHALL be rejected.

---

# 6.21 Interface Definitions

An interface specifies externally observable behavior.

Interfaces define contracts rather than implementation inheritance.

Interfaces may require:

- functions,
- associated types,
- associated constants.

Interfaces SHALL NOT define storage layout.

---

# 6.22 Interface Satisfaction

A type satisfies an interface only by explicit declaration.

Structural conformance alone is insufficient.

Implicit interface implementation is prohibited.

---

# 6.23 Derivations

A derivation automatically generates language-defined implementations.

Derivations are declarative.

The compiler SHALL generate only implementations explicitly authorized by the specification.

No implementation may infer undeclared derivations.

---

# 6.24 Derivation Validity

A derivation is valid only when every participating field satisfies the derivation requirements.

Example requirements include:

- comparable fields,
- hashable fields,
- cloneable fields.

Failure of any participating member invalidates the derivation.

---

# 6.25 Derivation Order

Multiple derivations are independent.

The observable semantics of one derivation SHALL NOT depend upon the textual ordering of unrelated derivations.

Where dependencies exist, the specification defining that derivation SHALL describe them explicitly.

---

# 6.26 Compile-Time Evaluation

Trait resolution,

constraint verification,

generic substitution,

and derivation validation

are compile-time operations.

No runtime reflection is required for these processes.

---

# 6.27 Dynamic Dispatch

Raygen favors static dispatch.

Dynamic dispatch is permitted only through explicitly declared runtime interface objects.

Dynamic dispatch SHALL NOT occur implicitly.

---

# 6.28 Reflection

Compile-time reflection MAY be supported through future language editions.

Runtime reflection is outside the Raygen 1 normative core unless explicitly defined by another specification.

Programs SHALL NOT depend upon implementation-specific reflection behavior.

---

# 6.29 Name Lookup

Name lookup proceeds according to lexical scope.

Imported names participate only after successful import resolution.

Trait members become visible only after the associated trait participates in successful type resolution.

---

# 6.30 Diagnostic Requirements

When constraint resolution fails, implementations SHALL diagnose:

- the unsatisfied constraint,
- the affected declaration,
- the missing implementation,
- the location responsible for failure.

Diagnostics SHOULD identify candidate implementations when ambiguity exists.

---

# 6.31 Abstraction Conformance

A conforming implementation SHALL:

- implement deterministic trait resolution,
- enforce coherence,
- enforce the orphan rule,
- reject specialization,
- verify generic constraints,
- preserve substitution semantics,
- resolve overloads deterministically,
- generate only valid derivations,
- prohibit implicit runtime dispatch.

No implementation may claim Raygen 1 abstraction conformance while permitting ambiguous implementations, inconsistent generic substitution, multiple applicable trait implementations, or implementation-defined overload resolution.



# Raygen 1 Normative Core
## Part 7 — Modules, Packages, Imports, Visibility, Namespaces, and Compilation Units

### 7.1 Purpose

This section defines the normative program organization model for Raygen 1.

The module system establishes:

- compilation units,
- modules,
- packages,
- namespaces,
- imports,
- exports,
- visibility,
- dependency resolution,
- edition compatibility.

A conforming implementation SHALL enforce these rules during compilation.

---

# 7.2 Organizational Principles

The Raygen module system is governed by the following principles.

1. Every declaration belongs to exactly one module.

2. Every module belongs to exactly one package.

3. Every package defines a unique public namespace.

4. Name resolution is deterministic.

5. Visibility is explicit.

6. Circular module initialization is prohibited.

---

# 7.3 Compilation Unit

A compilation unit is the smallest independently parsed source file.

Each compilation unit SHALL contain:

- one edition declaration,
- zero or more imports,
- zero or more declarations.

Compilation units SHALL be parsed independently before semantic analysis.

---

# 7.4 Module

A module is a logical collection of declarations.

A module provides a namespace boundary.

Modules SHALL contain:

- type declarations,
- function declarations,
- trait declarations,
- constant declarations,
- submodules.

Modules SHALL NOT overlap.

Every declaration belongs to exactly one module.

---

# 7.5 Package

A package is the primary compilation artifact.

A package consists of:

- one root module,
- zero or more child modules,
- package metadata,
- dependency declarations.

Every package possesses a unique package identifier.

---

# 7.6 Package Identity

A package identity consists of:

- package name,
- edition,
- version.

The package identifier SHALL uniquely distinguish packages participating in compilation.

Version comparison SHALL follow the package specification defined by the build system.

---

# 7.7 Root Module

Every package SHALL define exactly one root module.

The root module establishes:

- package entry declarations,
- exported symbols,
- child module hierarchy.

Compilation begins from the root module.

---

# 7.8 Submodules

Modules may contain child modules.

Each child module possesses exactly one parent.

Module relationships form a rooted tree.

Cycles within the module hierarchy are prohibited.

---

# 7.9 Namespace

Each module defines an independent namespace.

Identifiers declared within one namespace SHALL NOT automatically appear in another namespace.

Visibility determines accessibility.

---

# 7.10 Declaration Scope

A declaration is visible only within its defining scope unless exported.

Scopes include:

- module scope,
- block scope,
- function scope,
- generic parameter scope.

Name lookup SHALL follow lexical scope.

---

# 7.11 Visibility Levels

Raygen defines the following visibility levels:

text private  module  package  public 

Each declaration possesses exactly one visibility level.

---

## private

Visible only within the enclosing declaration.

---

## module

Visible throughout the defining module.

---

## package

Visible to every module within the same package.

---

## public

Visible to importing packages.

---

# 7.12 Export

Only public declarations may be exported.

Exported declarations become part of the package's public interface.

Private implementation details SHALL NOT become visible through export.

---

# 7.13 Import

Imports introduce external declarations into the current compilation unit.

An import SHALL specify:

- the source package,
- the source module,
- the imported declaration.

Wildcard imports are not part of the Raygen 1 normative core unless explicitly enabled by an implementation profile.

---

# 7.14 Qualified Imports

A qualified import preserves the original namespace.

Example (illustrative):

text math::sqrt 

Qualified imports reduce identifier ambiguity.

---

# 7.15 Aliased Imports

Imports may define local aliases.

Example (illustrative):

text math as m 

Aliases affect only the importing compilation unit.

They SHALL NOT modify exported names.

---

# 7.16 Name Resolution

Name resolution proceeds in the following order:

1. Local declarations.

2. Block scope.

3. Module scope.

4. Explicit imports.

5. Package-visible declarations.

6. Fully qualified names.

If multiple candidates remain equally valid, resolution is ambiguous.

Ambiguous references SHALL be rejected.

---

# 7.17 Shadowing

Inner scopes may shadow outer declarations.

Shadowing SHALL NOT alter the meaning of declarations outside the shadowing scope.

Implementations SHOULD diagnose accidental shadowing when it is likely to cause confusion.

---

# 7.18 Re-Export

A package may explicitly re-export imported declarations.

Re-exported declarations become part of the package's public interface.

Re-export SHALL NOT modify the declaration's semantic identity.

---

# 7.19 Dependency Graph

Package dependencies form a directed graph.

Dependency edges are established through package imports.

The dependency graph SHALL be acyclic.

Circular package dependencies are prohibited.

---

# 7.20 Module Initialization

Modules initialize in dependency order.

A module SHALL NOT observe partially initialized state from another module.

Initialization order SHALL be deterministic.

---

# 7.21 Static Initialization

Static objects initialize before program execution.

Initialization SHALL occur exactly once.

If initialization fails, the package SHALL fail to initialize.

Partially initialized objects SHALL be destroyed according to the ownership rules defined in Part 4.

---

# 7.22 Entry Point

An executable package SHALL define exactly one program entry point.

The entry point signature SHALL conform to the execution specification.

Multiple entry points within a single executable package are prohibited.

Library packages SHALL NOT define executable entry points.

---

# 7.23 Separate Compilation

Modules may be compiled independently.

Separate compilation SHALL preserve:

- type correctness,
- ownership verification,
- trait coherence,
- visibility semantics.

Link-time processing SHALL NOT alter language semantics.

---

# 7.24 Symbol Identity

Each exported declaration possesses one canonical symbol identity.

Symbol identity is independent of:

- source filename,
- compilation order,
- object layout,
- optimization level.

---

# 7.25 Edition Compatibility

Each compilation unit declares exactly one language edition.

Compilation units belonging to different editions SHALL interact only through explicitly defined compatibility rules.

A Raygen 1 implementation SHALL reject unsupported edition combinations.

---

# 7.26 Standard Library

The standard library is provided as one or more packages.

Standard library declarations participate in ordinary visibility and import rules.

Standard library identifiers are not reserved keywords unless explicitly defined by the grammar.

---

# 7.27 Build Profiles

Build profiles may define:

- optimization level,
- diagnostics,
- target architecture,
- feature selection.

Build profiles SHALL NOT alter the normative language semantics.

A program accepted under one conforming profile SHALL preserve identical observable behavior under another conforming profile, except where implementation-defined performance characteristics differ.

---

# 7.28 Implementation-Defined Packages

Implementations may provide additional packages.

Implementation-defined packages SHALL NOT redefine normative language behavior.

Programs depending upon implementation-defined packages are not automatically portable across conforming implementations.

---

# 7.29 Diagnostic Requirements

When module resolution fails, implementations SHALL identify:

- the unresolved package,
- the unresolved module,
- the unresolved declaration,
- the importing location.

When dependency cycles are detected, diagnostics SHALL identify every participating package or module in the cycle.

---

# 7.30 Module System Conformance

A conforming implementation SHALL:

- implement deterministic module resolution,
- enforce explicit visibility,
- preserve namespace isolation,
- prohibit cyclic package dependencies,
- initialize modules deterministically,
- preserve symbol identity,
- enforce edition compatibility,
- support separate compilation without altering language semantics.

No implementation may claim Raygen 1 module-system conformance while permitting ambiguous imports, nondeterministic initialization, visibility violations, cyclic dependency graphs, or implementation-defined name resolution within the normative language.



# Raygen 1 Normative Core
## Part 8 — Expressions, Statements, Control Flow, Pattern Matching, and Evaluation Semantics

### 8.1 Purpose

This section defines the normative execution semantics for Raygen 1.

The execution model establishes:

- expression evaluation,
- statement execution,
- sequencing,
- branching,
- iteration,
- pattern matching,
- control transfer,
- constant evaluation,
- unreachable code detection.

A conforming implementation SHALL execute every well-formed program according to this section.

---

# 8.2 Evaluation Principles

Execution is governed by the following principles.

1. Every expression produces exactly one value or one denial.

2. Evaluation order is deterministic.

3. Expressions are free of undefined sequencing.

4. Side effects occur only through defined language constructs.

5. Control flow is explicit.

---

# 8.3 Expression Categories

Expressions include:

- literals,
- identifiers,
- unary expressions,
- binary expressions,
- function calls,
- constructor expressions,
- member access,
- indexing,
- assignment expressions,
- block expressions,
- conditional expressions,
- match expressions.

Every expression has exactly one static type.

---

# 8.4 Evaluation Order

Unless otherwise specified:

Expression operands SHALL be evaluated from left to right.

Example:

text f(a(), b(), c()) 

Evaluation proceeds as:

text a()  ↓  b()  ↓  c()  ↓  f(...) 

No implementation may reorder observable side effects.

---

# 8.5 Temporary Values

Temporary values exist until:

- the enclosing full expression completes, or
- ownership is transferred, or
- destruction occurs earlier as permitted by lifetime analysis.

Temporary destruction SHALL be deterministic.

---

# 8.6 Constant Expressions

A constant expression is fully evaluable during compilation.

Constant expressions SHALL NOT require:

- runtime memory allocation,
- runtime synchronization,
- runtime I/O,
- implementation-defined state.

Failure during constant evaluation is a compile-time error.

---

# 8.7 Block Expressions

A block evaluates to the value of its final expression.

Example:

text {     compute()     result } 

If no final expression exists, the block evaluates to the unit value.

---

# 8.8 Assignment

Assignment updates an existing storage location.

Assignment requires:

- mutable access,
- compatible type,
- valid ownership.

Assignment evaluates to the assigned value unless otherwise specified by the grammar.

---

# 8.9 Conditional Expressions

An if expression evaluates exactly one branch.

The condition SHALL evaluate to bool.

Example:

text if ready {     value } else {     fallback } 

Both branches SHALL produce compatible types.

---

# 8.10 Boolean Evaluation

Boolean operators evaluate left to right.

The following operators short-circuit:

text &&  || 

If the result is determined by the left operand, the right operand SHALL NOT be evaluated.

---

# 8.11 Comparison Expressions

Comparison operators return bool.

Comparison operands SHALL be comparable according to the type system.

Comparison SHALL NOT modify operand values.

---

# 8.12 Arithmetic Expressions

Arithmetic expressions follow the arithmetic policies defined in Part 3.

Evaluation SHALL preserve:

- operand order,
- overflow behavior,
- failure behavior.

No implicit numeric promotion occurs.

---

# 8.13 Function Calls

Function call evaluation proceeds in this order:

1. Evaluate the callable expression.

2. Evaluate arguments left to right.

3. Verify parameter compatibility.

4. Transfer ownership where required.

5. Invoke the function.

Every argument SHALL be evaluated exactly once.

---

# 8.14 Method Calls

Method invocation first resolves the method according to Part 6.

Receiver evaluation precedes argument evaluation.

Receiver ownership SHALL follow the declared parameter contract.

---

# 8.15 Member Access

Member access evaluates:

1. the receiver,

2. the selected member.

Member lookup SHALL be deterministic.

Access SHALL respect visibility rules.

---

# 8.16 Indexing

Index expressions evaluate:

1. the container,

2. the index.

Bounds checking follows the selected safety profile.

Out-of-range access in safe code SHALL produce defined behavior according to the active profile.

---

# 8.17 Statement Execution

Statements execute sequentially.

Execution continues until:

- normal completion,
- return,
- denial propagation,
- break,
- continue,
- unreachable termination.

---

# 8.18 Return

return immediately terminates the current function.

Before returning:

- owned temporaries are destroyed,
- ownership is transferred,
- outstanding borrows are verified.

Return type SHALL match the declared function type.

---

# 8.19 Denial Propagation

Propagation using

text allow expression 

evaluates the expression.

If the result is:

text allowed(value) 

evaluation continues using value.

If the result is:

text denied(error) 

the enclosing function immediately returns the denial.

Intermediate cleanup SHALL follow Part 4.

---

# 8.20 Break

break immediately exits the nearest enclosing loop.

Before exit:

- temporary objects are destroyed,
- ownership cleanup occurs.

Break SHALL target only enclosing loops unless explicitly labeled.

---

# 8.21 Continue

continue skips the remaining loop body.

Execution resumes with the next iteration.

Temporary cleanup SHALL occur before the next iteration begins.

---

# 8.22 Loop Semantics

Loop constructs evaluate according to their defined iteration rules.

Raygen defines:

- while,
- repeat,
- each.

Each loop has one entry point and one normal exit.

Infinite loops are permitted.

---

# 8.23 While Loop

The condition evaluates before every iteration.

If the condition evaluates to false, execution terminates normally.

---

# 8.24 Repeat Loop

A repeat loop executes its body before testing its termination condition.

The body therefore executes at least once.

---

# 8.25 Each Loop

An each loop iterates over an iterable value.

Iteration order SHALL follow the iterable's defined semantics.

Mutation during iteration SHALL obey ownership and borrowing rules.

---

# 8.26 Pattern Matching

Pattern matching compares a value against one or more patterns.

Evaluation proceeds top to bottom.

The first matching pattern is selected.

No later pattern is evaluated.

---

# 8.27 Pattern Exhaustiveness

Every match expression SHALL be exhaustive.

The compiler SHALL reject non-exhaustive matches unless an explicit wildcard pattern is present.

---

# 8.28 Pattern Binding

Bindings introduced by a successful pattern are visible only within the corresponding branch.

Bindings SHALL possess precisely determined types.

Bindings from unsuccessful patterns do not exist.

---

# 8.29 Guards

A pattern may define an optional guard.

The guard is evaluated only after successful structural matching.

If the guard evaluates to false, matching continues with the next pattern.

---

# 8.30 Unreachable Code

Code is unreachable if no execution path can reach it.

Examples include code following:

- return,
- unconditional denial propagation,
- unconditional break,
- unconditional continue.

Implementations SHALL diagnose unreachable code.

---

# 8.31 Diverging Expressions

Expressions of type never never complete normally.

Examples include:

- infinite loops,
- guaranteed denial,
- guaranteed termination.

A diverging expression may appear wherever any type is expected.

---

# 8.32 Evaluation Conformance

A conforming implementation SHALL:

- evaluate expressions left to right,
- preserve deterministic sequencing,
- short-circuit boolean operators,
- enforce exhaustive pattern matching,
- verify control-transfer legality,
- destroy temporaries deterministically,
- preserve ownership during evaluation,
- reject unreachable control-flow violations.

No implementation may claim Raygen 1 evaluation conformance while permitting implementation-defined sequencing, ambiguous evaluation order, incomplete pattern matching, or observable reordering of side effects within safe programs.



# Raygen 1 Normative Core
## Part 9 — Memory, Layout, ABI, Foreign Function Interface (FFI), and Unsafe Operations

### 9.1 Purpose

This section defines the normative low-level execution model for Raygen 1.

The low-level model establishes:

- object representation,
- alignment,
- layout,
- pointer semantics,
- provenance,
- foreign-function interoperability,
- calling conventions,
- unsafe operations,
- binary compatibility.

A conforming implementation SHALL preserve these guarantees.

---

# 9.2 Design Principles

The Raygen low-level model is governed by the following principles.

1. Safe code shall not depend upon object representation.

2. Layout is explicit.

3. Pointer provenance is preserved.

4. Unsafe operations are explicitly delimited.

5. Foreign interfaces shall not weaken language safety.

6. ABI behavior is deterministic.

---

# 9.3 Object Representation

Every object possesses:

- a static type,
- an alignment,
- a size,
- a storage location,
- provenance.

Representation is independent of semantic type identity.

Programs SHALL NOT infer semantic meaning solely from binary layout.

---

# 9.4 Object Lifetime

An object exists only during its valid lifetime.

Storage SHALL NOT be accessed:

- before initialization,
- after destruction,
- outside its lifetime.

Violation within unsafe code results in undefined behavior.

---

# 9.5 Alignment

Every object has a required alignment.

The implementation SHALL ensure that objects are stored at addresses satisfying their alignment requirements.

Access through misaligned pointers is undefined unless explicitly supported by the target ABI.

---

# 9.6 Object Size

Every concrete type has a compile-time size unless explicitly defined as dynamically sized.

The size of dynamically sized objects SHALL be determined through language-defined metadata.

The size of a type SHALL remain constant throughout program execution.

---

# 9.7 Layout Categories

Raygen defines three layout categories:

text implementation  stable  foreign 

---

## implementation

The compiler may freely choose representation.

Safe code SHALL NOT depend upon layout.

---

## stable

Field ordering, alignment, and padding are defined by this specification.

Stable layout is intended for binary persistence and deterministic interchange.

---

## foreign

Layout is determined by the foreign ABI.

The compiler SHALL preserve compatibility with the declared foreign representation.

---

# 9.8 Padding

Padding bytes are implementation artifacts.

Programs SHALL NOT assume any value for padding.

Reading padding through safe code is prohibited.

Writing padding through safe code is prohibited.

---

# 9.9 Pointer

A pointer identifies a storage location.

A pointer possesses:

- provenance,
- address,
- mutability,
- lifetime.

Pointer equality compares storage identity.

---

# 9.10 Provenance

Every pointer originates from exactly one valid allocation.

Pointer provenance SHALL be preserved across:

- borrowing,
- movement,
- reborrowing,
- permitted arithmetic.

Operations that fabricate provenance are undefined unless explicitly authorized within unsafe code.

---

# 9.11 Null Pointers

Safe Raygen references are never null.

Nullable pointers SHALL be represented using an explicit optional type.

Foreign interfaces may expose null pointers according to the declared ABI.

---

# 9.12 Pointer Arithmetic

Pointer arithmetic is permitted only within unsafe code.

Arithmetic SHALL remain within the bounds of the originating allocation, or one element beyond its end where permitted for comparison.

Dereferencing an out-of-bounds pointer is undefined behavior.

---

# 9.13 Aliasing

Aliasing rules defined in Part 4 remain applicable.

Unsafe pointer operations SHALL preserve aliasing correctness.

Violation results in undefined behavior.

---

# 9.14 Byte Representation

Objects may be viewed as bytes only through language-defined mechanisms.

Byte access SHALL preserve provenance.

Reinterpreting arbitrary bytes as unrelated object types is prohibited unless explicitly authorized.

---

# 9.15 Type Reinterpretation

Bit reinterpretation requires explicit unsafe operations.

The source and destination representations SHALL satisfy the representation requirements defined by this specification.

Programs SHALL NOT assume representation compatibility solely because object sizes are identical.

---

# 9.16 Calling Convention

Every callable entity has an associated calling convention.

The default Raygen calling convention is implementation-independent.

Foreign declarations SHALL explicitly identify any non-default calling convention.

Calling conventions determine:

- parameter passing,
- return value passing,
- stack discipline,
- register preservation.

---

# 9.17 Application Binary Interface (ABI)

The ABI specifies binary interoperability between independently compiled components.

The ABI defines:

- symbol naming,
- object layout,
- function calling,
- exception boundaries,
- alignment,
- parameter passing.

Conforming implementations SHALL preserve ABI compatibility within the same edition and target profile.

---

# 9.18 Foreign Function Interface

The Foreign Function Interface (FFI) enables interaction with external languages.

Foreign declarations SHALL explicitly identify:

- the foreign language or ABI,
- the calling convention,
- the imported symbol.

Foreign declarations participate in ordinary type checking.

---

# 9.19 Foreign Types

Foreign types SHALL be declared explicitly.

The compiler SHALL verify representation compatibility between Raygen and foreign declarations.

Representation mismatches are compile-time errors unless explicitly marked as unsafe.

---

# 9.20 Ownership Across FFI

Ownership transfer across foreign boundaries SHALL be explicit.

The declaration SHALL specify whether:

- ownership is transferred,
- ownership is borrowed,
- ownership is retained.

Implicit ownership transfer is prohibited.

---

# 9.21 Failure Across FFI

Foreign calls SHALL specify how failures are represented.

A foreign function SHALL NOT silently violate the Raygen failure-flow model.

If necessary, foreign failures SHALL be translated into the canonical outcome<Allowed,Denied> representation defined in Part 3.

---

# 9.22 Unsafe Block

Unsafe operations SHALL occur only within an explicit unsafe block.

Unsafe blocks suspend selected static safety checks.

All other language rules remain in force.

---

# 9.23 Unsafe Responsibilities

Unsafe code is responsible for preserving:

- memory safety,
- lifetime validity,
- ownership correctness,
- aliasing correctness,
- pointer provenance,
- ABI correctness.

Failure to satisfy these responsibilities results in undefined behavior.

---

# 9.24 Undefined Behavior

Undefined behavior exists only when a program violates the rules governing unsafe execution.

Safe Raygen programs SHALL NOT exhibit undefined behavior.

Examples include:

- dereferencing invalid pointers,
- use-after-free,
- violating provenance,
- invalid type reinterpretation,
- data races within unsafe code.

---

# 9.25 Volatile Access

Volatile operations provide observable access to externally changing storage.

Volatile access SHALL NOT establish synchronization.

Volatile operations SHALL NOT replace atomic operations for inter-task communication.

---

# 9.26 Memory-Mapped I/O

Memory-mapped device access SHALL occur only through explicitly designated APIs or types.

Ordinary references SHALL NOT be assumed to represent device memory.

Device access semantics are target-specific but SHALL preserve the language's ownership and lifetime guarantees.

---

# 9.27 Binary Compatibility

Binary compatibility requires agreement on:

- ABI,
- layout,
- alignment,
- calling convention,
- symbol identity.

Binary compatibility SHALL NOT be inferred solely from matching source declarations.

---

# 9.28 Optimization Constraints

Compiler optimizations SHALL preserve the observable behavior of well-formed programs.

Optimizations SHALL NOT:

- violate ownership,
- alter memory ordering,
- invalidate provenance,
- change ABI-visible behavior.

---

# 9.29 Diagnostic Requirements

When representation incompatibilities are detected, implementations SHALL diagnose:

- the conflicting declarations,
- the incompatible layouts,
- the affected ABI boundary.

Unsafe operations SHOULD produce diagnostics identifying potential violations of ownership, provenance, or lifetime rules.

---

# 9.30 Low-Level Conformance

A conforming implementation SHALL:

- preserve object lifetime,
- enforce alignment requirements,
- distinguish representation from type identity,
- preserve pointer provenance,
- require explicit unsafe blocks,
- verify foreign declarations,
- preserve ABI correctness,
- prohibit undefined behavior in safe code.

No implementation may claim Raygen 1 low-level conformance while permitting implementation-defined violations of provenance, lifetime, ownership, layout guarantees, or foreign-interface semantics within safe programs.



# Raygen 1 Normative Core
## Part 10 — Diagnostics, Contracts, Attributes, Reflection, Profiles, Deprecation, and Implementation Conformance

### 10.1 Purpose

This section defines the normative requirements governing:

- diagnostics,
- compile-time contracts,
- attributes,
- reflection,
- implementation-defined behavior,
- language profiles,
- deprecation,
- editions,
- conformance.

This Part specifies the obligations of both Raygen programs and conforming implementations.

---

# 10.2 General Principles

A conforming implementation SHALL:

- diagnose violations of mandatory language rules,
- preserve deterministic program meaning,
- identify implementation-defined behavior,
- distinguish errors from warnings,
- maintain edition compatibility according to this specification.

---

# 10.3 Diagnostic Categories

Diagnostics are classified as:

text error  warning  note  information 

---

## Error

An error indicates violation of a mandatory language rule.

Compilation SHALL fail if any error remains unresolved.

---

## Warning

A warning indicates potentially undesirable or unsafe program behavior.

Warnings SHALL NOT prevent successful compilation unless elevated by implementation configuration.

---

## Note

Notes provide supplemental explanatory information associated with another diagnostic.

Notes SHALL NOT independently affect compilation.

---

## Information

Informational diagnostics communicate implementation details that do not indicate incorrect programs.

---

# 10.4 Mandatory Diagnostics

The implementation SHALL diagnose violations involving:

- syntax,
- name resolution,
- type checking,
- ownership,
- borrowing,
- lifetime analysis,
- visibility,
- duplicate declarations,
- invalid generics,
- invalid trait implementations,
- pattern exhaustiveness,
- unreachable required return paths,
- invalid attributes,
- invalid contracts,
- profile violations.

---

# 10.5 Diagnostic Quality

Diagnostics SHOULD identify:

- source location,
- violated rule,
- relevant declarations,
- possible corrective actions.

Implementations MAY include additional explanatory information.

---

# 10.6 Compile-Time Contracts

Contracts express semantic requirements beyond ordinary type checking.

Contracts are evaluated during compilation whenever sufficient information is available.

Contracts SHALL NOT alter program semantics.

---

# 10.7 Preconditions

A precondition specifies requirements that SHALL hold before execution of an operation.

Violation of a compile-time-verifiable precondition SHALL produce a compilation error.

Runtime-verifiable preconditions SHALL produce deterministic failure according to the language failure model.

---

# 10.8 Postconditions

A postcondition specifies properties guaranteed after successful execution.

Implementations SHALL preserve observable behavior while verifying postconditions.

---

# 10.9 Invariants

An invariant defines a property that SHALL remain true throughout the lifetime of an object or abstraction.

Violation of an invariant constitutes program failure.

---

# 10.10 Compile-Time Assertions

Compile-time assertions evaluate constant expressions.

Failure SHALL produce a compilation error.

Compile-time assertions SHALL NOT generate runtime code.

---

# 10.11 Runtime Assertions

Runtime assertions evaluate during execution.

Assertion failure SHALL produce deterministic program failure.

Implementations MAY disable runtime assertions in designated optimization profiles.

---

# 10.12 Attributes

Attributes provide declarative metadata affecting compilation.

Attributes SHALL be explicitly recognized by the implementation.

Unknown attributes SHALL produce diagnostics unless explicitly permitted by the implementation profile.

---

# 10.13 Attribute Scope

Attributes may apply to:

- modules,
- declarations,
- types,
- functions,
- variables,
- fields,
- generic parameters,
- implementations.

Attributes SHALL NOT alter language semantics beyond those explicitly defined by this specification.

---

# 10.14 Standard Attributes

The Raygen standard library defines the normative attribute set.

Each standard attribute SHALL specify:

- valid targets,
- required arguments,
- semantic effect,
- diagnostic behavior.

---

# 10.15 User-Defined Attributes

Implementations MAY permit user-defined attributes.

User-defined attributes SHALL NOT modify the core semantics of the language.

---

# 10.16 Reflection

Reflection provides controlled compile-time inspection of program structure.

Reflection SHALL preserve deterministic compilation.

---

# 10.17 Reflection Scope

Compile-time reflection may inspect:

- declarations,
- attributes,
- types,
- trait implementations,
- generic parameters,
- visibility,
- documentation metadata.

Reflection SHALL NOT observe runtime state.

---

# 10.18 Runtime Reflection

Runtime reflection is optional.

If supported, runtime reflection SHALL preserve type safety.

Implementations SHALL explicitly identify unsupported reflection capabilities.

---

# 10.19 Metaprogram Safety

Reflection and metaprogramming facilities SHALL NOT bypass:

- ownership,
- visibility,
- lifetime analysis,
- type checking,
- safety rules.

Generated programs are subject to all ordinary language rules.

---

# 10.20 Implementation-Defined Behavior

Implementation-defined behavior is behavior explicitly left to the implementation.

Every implementation-defined behavior SHALL be documented.

Programs SHALL NOT depend upon undocumented implementation behavior.

---

# 10.21 Unspecified Behavior

Unspecified behavior permits multiple valid implementation choices.

Each execution SHALL remain well-defined.

Unspecified behavior SHALL NOT introduce undefined behavior.

---

# 10.22 Undefined Behavior

Undefined behavior exists only as specified in Part 9.

Implementations MAY assume undefined behavior never occurs within well-formed programs.

---

# 10.23 Language Profiles

Profiles define constrained subsets or extensions of Raygen.

Profiles SHALL preserve compatibility with the normative language unless explicitly designated experimental.

---

## Examples

Profiles may include:

- embedded
- hosted
- kernel
- safety-critical
- educational

---

# 10.24 Profile Requirements

Each profile SHALL specify:

- supported language features,
- prohibited features,
- runtime assumptions,
- standard library availability,
- optimization requirements.

---

# 10.25 Editions

An edition defines a stable evolution of the language.

Programs SHALL identify the edition against which they are compiled.

Editions SHALL preserve source compatibility whenever practical.

Breaking changes SHALL require a new edition.

---

# 10.26 Feature Stability

Language features are classified as:

- stable,
- experimental,
- deprecated,
- removed.

Stable features SHALL preserve their specified behavior throughout an edition.

---

# 10.27 Experimental Features

Experimental features SHALL require explicit opt-in.

Experimental features SHALL NOT be enabled implicitly.

Implementations MAY alter experimental features without preserving compatibility.

---

# 10.28 Deprecated Features

Deprecated features remain valid but are scheduled for future removal.

Use of deprecated features SHALL produce a warning.

Removal SHALL occur only in a subsequent edition.

---

# 10.29 Removed Features

Removed features SHALL produce compilation errors.

Implementations MAY provide migration diagnostics.

---

# 10.30 Standard Library Conformance

The standard library SHALL conform to all language rules.

Library facilities SHALL NOT require privileged access unavailable to ordinary programs unless explicitly designated intrinsic.

---

# 10.31 Intrinsics

Intrinsics provide implementation-defined operations that cannot be expressed in ordinary Raygen.

Intrinsics SHALL be explicitly documented.

Safe programs SHALL interact with intrinsics only through standardized interfaces.

---

# 10.32 Compiler Extensions

Implementations MAY provide extensions.

Extensions SHALL require explicit opt-in.

Extensions SHALL NOT silently alter the semantics of standard Raygen programs.

---

# 10.33 Reserved Identifiers

Identifiers reserved by this specification SHALL NOT be defined by user programs.

Violation SHALL produce diagnostics.

---

# 10.34 Documentation Comments

Documentation comments constitute implementation metadata.

Documentation comments SHALL NOT affect program semantics.

---

# 10.35 Source Encoding

A conforming implementation SHALL accept UTF-8 source files.

Additional encodings MAY be supported.

Implementations SHALL document supported encodings.

---

# 10.36 Source Locations

Every diagnostic SHALL identify an associated source location whenever available.

Generated code diagnostics SHOULD reference the originating source construct.

---

# 10.37 Conformance Levels

Implementations are classified as:

- conforming,
- profile-conforming,
- experimental.

Only conforming implementations may claim full Raygen 1 compatibility.

---

# 10.38 Requirements for Conforming Implementations

A conforming implementation SHALL:

- implement all mandatory language rules,
- preserve ownership guarantees,
- preserve lifetime guarantees,
- preserve type safety,
- preserve deterministic failure semantics,
- diagnose mandatory violations,
- document implementation-defined behavior,
- preserve edition compatibility,
- implement the standard library according to this specification.

---

# 10.39 Requirements for Conforming Programs

A conforming program SHALL:

- satisfy the lexical grammar,
- satisfy the syntactic grammar,
- satisfy the static semantics,
- avoid undefined behavior,
- respect ownership rules,
- respect lifetime rules,
- respect visibility,
- satisfy generic constraints,
- satisfy trait coherence,
- satisfy profile restrictions.

---

# 10.40 Normative Completion

Parts 1 through 10 collectively define the normative core of the Raygen 1 Language Specification.

No implementation may claim conformance while omitting mandatory requirements defined within this specification.

Future editions SHALL preserve these guarantees except where explicitly superseded through the edition mechanism defined herein.



## *** ##



** UPDATED VERSION 07/22/2026 **

# Raygen Programming Language

## Normative Core Specification

File extension: .rg1  
Official name: Raygen  
Meaning: Ray Generator  
Edition: Raygen 1  
Specification status: Normative Core

A geometric ray begins at one fixed origin and extends in a defined direction. Raygen applies this principle to software execution:

> Every operation begins from a known state, proceeds through an explicit direction, and produces a traceable result.

The ray is Raygen’s semantic execution model. It does not require a physical ray object, linked allocation, pointer chain, or metadata header.

The normative foundation of Raygen is:

> Meaning comes first.  
> Safety constrains execution.  
> Ownership constrains access.  
> Concurrency constrains visibility.  
> Layout constrains representation.  
> Deduction removes only proven checks.  
> Optimization changes implementation, never meaning.

This specification defines the rules every conforming Raygen compiler, interpreter, virtual machine, static analyzer, and runtime must follow.

---

# 1. Scope

The Raygen Normative Core Specification defines:

- Compilation-stage precedence
- Policy classes and policy resolution
- Lexical and syntactic grammar
- Operator precedence
- Type and ownership interactions
- Static and dynamic derivation
- Case-based concurrency
- Ordered-layer parallelism
- Atomic operations
- Memory ordering
- Synchronization
- Determinism levels
- Core-language boundaries
- Standard-library boundaries
- Compiler conformance requirements

Descriptive documentation can explain how Raygen feels and why features exist.

This document defines what Raygen programs mean.

---

# 2. Normative Terminology

The words must, must not, required, shall, and shall not express mandatory implementation requirements.

The words may, optional, and permitted describe implementation freedom.

The word should describes a strongly recommended behavior that may be omitted only for a documented reason.

A compiler that violates a mandatory requirement is not conformant with the relevant Raygen conformance level.

---

# 3. Semantic Authority

Every Raygen program is governed by the following authority hierarchy, from strongest to weakest:

text Language semantics     > type safety     > ownership and lifetime     > concurrency correctness     > explicit layout requirements     > explicit semantic policies     > explicit safety policies     > deduction results     > optimization requirements     > optimization preferences     > target heuristics 

A lower-ranked system may not invalidate a higher-ranked system.

Examples:

- Vectorization cannot weaken ownership.
- Layout selection cannot violate ABI requirements.
- Deduction cannot remove a check whose proof is invalid after alias analysis.
- A target heuristic cannot override required overflow behavior.
- Parallelization cannot introduce a data race.
- Inlining cannot alter observable denial propagation.
- Dispatch optimization cannot change mapping completeness.

---

# 4. Compilation Precedence

A conforming compiler processes a program according to the following semantic order:

text 1. Parse lexical and syntactic structure 2. Resolve modules, imports, and names 3. Resolve declared types 4. Validate type legality 5. Validate ownership and lifetime 6. Validate control-flow legality 7. Validate failure-flow completeness 8. Validate concurrency access contracts 9. Apply mandatory layout constraints 10. Establish aliasing and provenance facts 11. Perform bounded deduction 12. Select required runtime guards 13. Apply vectorization and parallel optimization 14. Lower high-level directives 15. Generate assembly serial 16. Resolve symbolic tags and relocations 17. Emit verified VM code or native code 

Implementations may internally combine stages, but the resulting behavior must be equivalent to this ordering.

## 4.1 Safety Before Optimization

Correctness is established before optimization.

A compiler must never generate unsafe code merely because an optimization was requested.

Example:

rg1 policy vectorization {     mode forced } 

rg1 function transform(     input: read slice<f32>,     output: write slice<f32> ) -> void {     ... } 

If the compiler cannot establish safe aliasing conditions, it must do one of the following:

- Generate a guarded vector path with a safe fallback
- Generate a scalar implementation when the policy permits fallback
- Deny compilation when vectorization is explicitly required

It must not silently emit unsafe vector code.

---

# 5. Policy System

Raygen policies are scoped semantic declarations.

Every policy belongs to one of five classes:

text semantic safety optimization diagnostic deployment 

---

# 6. Semantic Policies

Semantic policies change observable program behavior.

Examples include:

- Overflow behavior
- Floating-point rounding
- Failure propagation
- Memory ordering
- Dispatch completeness
- Numerical reproducibility
- Denial recovery

Example:

rg1 policy arithmetic semantic {     allow overflow wrap } 

The optimizer must preserve semantic policies exactly.

It cannot replace wrapping addition with trapping addition, saturating addition, or mathematically unbounded addition.

Supported arithmetic behaviors include:

text deny wrap saturate clamp report widen 

---

# 7. Safety Policies

Safety policies define operations that must be checked, rejected, or guarded.

Examples include:

- Bounds checking
- Null access
- Pointer provenance
- Data-race prevention
- Unchecked denial
- Foreign calls
- Unsafe deserialization
- Lifetime violations

Example:

rg1 policy safety safety {     deny bounds_violation     deny data_race     deny unchecked_denial } 

Safety policies override optimization policies.

An optimization may remove a safety check only when a valid proof establishes that the check cannot fail.

---

# 8. Optimization Policies

Optimization policies influence code generation without changing program meaning.

Examples include:

- Vectorization
- Inlining
- Loop unrolling
- Dispatch strategy
- Code-size preference
- Layout preference
- Task fusion
- Register pressure
- Cache placement

Example:

rg1 policy optimization optimization {     vectorization automatic     code_size balanced     inlining profiled } 

Optimization policies are advisory unless marked require.

rg1 policy vectorization optimization {     require width 256 } 

If a required optimization cannot be performed without violating stronger rules, compilation is denied.

Example diagnostic:

text RG-OPT-021: Required 256-bit vectorization cannot be satisfied. Reason: loop-carried dependency on 'total'. 

---

# 9. Diagnostic Policies

Diagnostic policies control compiler reporting.

rg1 policy diagnostics diagnostic {     warning unused_binding     deny implicit_copy     note scalar_fallback } 

Diagnostic policies cannot change program behavior.

They may promote warnings to denials or suppress nonessential notices.

They may not suppress mandatory safety errors.

---

# 10. Deployment Policies

Deployment policies govern build and runtime environment restrictions.

rg1 policy deployment deployment {     target x86_64     runtime minimal     foreign_calls audited     direct_blocks signed } 

Deployment policies can restrict:

- Targets
- Runtime components
- Dynamic loading
- Foreign functions
- Direct memory access
- Code generation
- Heap allocation
- Package sources

---

# 11. Policy Scope and Resolution

Policies apply in the following order, from nearest to farthest scope:

text Expression Statement Block Function Type Module Package Build profile Language default 

The nearest applicable policy wins unless a wider policy is marked mandatory.

Example:

rg1 policy arithmetic semantic {     deny overflow mandatory } 

A nested policy cannot override it:

rg1 policy arithmetic semantic {     allow overflow wrap } 

Compiler diagnostic:

text RG-POLICY-012: Local overflow policy conflicts with mandatory module policy. 

## 11.1 Policy Conflicts

If two policies at the same scope conflict:

- A mandatory policy wins over a nonmandatory policy.
- A safety policy wins over an optimization policy.
- A semantic policy cannot be replaced by an optimization policy.
- Two conflicting mandatory policies produce a compile-time denial.
- Two incompatible semantic policies produce a compile-time denial.

---

# 12. Lexical Grammar

Raygen source files use Unicode text encoded as UTF-8.

A conforming compiler must reject malformed UTF-8.

## 12.1 Identifiers

Identifiers begin with:

- A Unicode letter
- An underscore

Subsequent characters may include:

- Unicode letters
- Decimal digits
- Underscores

Examples:

rg1 value _total customer_id Δposition 

Keywords are reserved and cannot be used as ordinary identifiers unless escaped by a future edition-defined mechanism.

---

## 12.2 Keywords

Core reserved words include:

text raygen module use as public private function origin record unit choice type alias trait implement let var late set return if else while repeat each in select case other allow deny retain store dismiss free derive dynamic deduce requires ensures policy mandatory map direct layers layer parallel merge spawn wait move copy read write own atomic locked volatile region scope defer where when compile generate true false void never 

---

## 12.3 Numeric Literals

Decimal integer:

rg1 42 1_000_000 

Hexadecimal integer:

rg1 0xFF 0xFF00_1000 

Binary integer:

rg1 0b1010_0101 

Floating-point:

rg1 3.14159 1.0e-9 

Typed suffix:

rg1 42:u8 4096:usize 3.14:f32 

Underscores may separate digits but cannot appear at the beginning or end of a literal or immediately beside a radix marker or decimal point.

---

## 12.4 Strings

Standard string:

rg1 "Hello, world!" 

Escape sequences include:

text \n \r \t \\ \" \0 \u{...} 

Multiline string:

rg1 """ Line one Line two """ 

Multiline strings preserve embedded newlines.

Indent normalization is defined by the standard formatter and must not alter semantic content.

---

## 12.5 Comments

Single-line comment:

rg1 # Comment 

Block comment:

rg1 #[     Block comment ]# 

Documentation comment:

rg1 ## Public documentation 

Block comments may nest only if the implementation supports nested comment counting exactly as defined by the language edition.

Raygen 1 permits nested block comments.

---

## 12.6 Terminators

A statement terminates with:

- A newline
- A semicolon
- The closing brace of a block when the preceding construct is complete

Semicolon insertion must not occur inside:

- Parentheses
- Brackets
- Generic-argument lists
- Incomplete expressions
- Pipeline continuations

Example:

rg1 let result =     values     |> where row.active     |> order row.name ascending 

This is one statement.

---

# 13. Syntactic Grammar

The following EBNF defines the foundational Raygen grammar.

ebnf program     = version-declaration,       { module-item },       EOF ;  version-declaration     = "raygen",       integer-literal,       terminator ;  module-item     = module-declaration     | use-declaration     | declaration     | origin-declaration     | policy-declaration ;  module-declaration     = "module",       qualified-name,       terminator ;  use-declaration     = "use",       qualified-name,       [ import-selection | alias-clause ],       terminator ;  import-selection     = "{",       identifier,       { ",", identifier },       "}" ;  alias-clause     = "as",       identifier ;  declaration     = function-declaration     | record-declaration     | unit-declaration     | choice-declaration     | type-declaration     | alias-declaration     | trait-declaration     | implementation-declaration     | constant-declaration ;  origin-declaration     = "origin",       identifier,       block ;  function-declaration     = [ visibility ],       "function",       identifier,       [ generic-parameters ],       parameter-list,       [ return-contract ],       { function-contract },       function-body ;  visibility     = "public"     | "private" ;  parameter-list     = "(",       [ parameter,       { ",", parameter } ],       ")" ;  parameter     = [ access-mode ],       identifier,       ":",       type-expression,       [ "=", expression ] ;  access-mode     = "own"     | "read"     | "write"     | "move"     | "atomic"     | "locked" ;  return-contract     = "->",       return-type ;  return-type     = type-expression     | allowed-type, denied-type     | "void"     | "never" ;  allowed-type     = "allow",       "<",       type-expression,       ">" ;  denied-type     = "deny",       "<",       type-expression,       ">" ;  function-contract     = requires-clause     | ensures-clause ;  requires-clause     = "requires",       expression ;  ensures-clause     = "ensures",       expression ;  function-body     = block     | "=>",       expression,       terminator ;  block     = "{",       { statement },       "}" ;  statement     = binding-statement     | mutation-statement     | expression-statement     | return-statement     | if-statement     | while-statement     | repeat-statement     | each-statement     | select-statement     | allow-statement     | deny-statement     | derive-statement     | deduce-statement     | case-declaration     | cases-statement     | layers-statement     | memory-statement     | policy-declaration     | direct-statement     | block ;  binding-statement     = ( "let" | "var" | "late" ),       identifier,       [ ":",       type-expression ],       [ "=",       expression ],       terminator ;  mutation-statement     = "set",       assignable-expression,       "<-",       expression,       terminator ;  return-statement     = "return",       [ "allow" | "deny" ],       [ expression ],       terminator ;  expression-statement     = expression,       terminator ;  terminator     = newline     | ";" ; 

The complete grammar is divided into:

text Raygen Lexical Grammar Raygen Syntactic Grammar Raygen Expression Grammar Raygen Structural-Sugar Grammar 

Privileged standard-library syntax is specified separately from the language kernel.

---

# 14. Operator Precedence

Raygen expressions use the following precedence, from highest to lowest:

| Level | Operations |
|---:|---|
| 1 | Member access, indexing, function calls |
| 2 | Explicit casts, checked casts, views |
| 3 | Unary operators |
| 4 | Multiplication, division, remainder |
| 5 | Addition, subtraction |
| 6 | Shifts |
| 7 | Bitwise AND |
| 8 | Bitwise XOR |
| 9 | Bitwise OR |
| 10 | Relational comparisons |
| 11 | Equality |
| 12 | Logical AND |
| 13 | Logical OR |
| 14 | Ranges |
| 15 | Pipelines |
| 16 | Conditional expressions |
| 17 | Assignment-like directives |

Example:

rg1 a + b * c 

means:

rg1 a + (b * c) 

Example:

rg1 value & mask == expected 

means:

rg1 (value & mask) == expected 

---

# 15. Pipeline Semantics

The pipeline operator is:

text |> 

It has weak binding.

rg1 let active_users =     users     |> where row.active     |> order row.name ascending 

This is parsed as an ordered transformation sequence applied to users.

A pipeline stage receives the previous stage’s result as its input.

Pipeline syntax may lower into:

- Function calls
- Iterator adapters
- Compiler-recognized collection operations
- Privileged library interfaces

Pipeline syntax does not imply eager or lazy evaluation by itself. The called operation or structural type determines evaluation behavior.

---

# 16. Static Derivation

Static derivation declares a compile-time-known dependency graph.

rg1 derive total from subtotal, tax pure {     total = subtotal + tax } 

## 16.1 Static Derivation Rules

1. Every external dependency read by the block must appear after from.
2. Every derived output must be assigned exactly once.
3. Derived outputs are immutable outside the block unless declared mutable.
4. The static dependency graph must be acyclic.
5. Dependency identity is lexical, typed, and module-resolved.
6. A derived value is invalidated when one of its declared dependencies changes.
7. Undeclared external reads are compile-time errors.
8. Derivation is pure by default.
9. A pure derivation cannot perform I/O, mutation of external state, synchronization, allocation with visible lifetime effects, or foreign calls.
10. The compiler may cache a derived value when doing so preserves semantics.

Example cycle:

rg1 derive a from b {     a = b + 1 }  derive b from a {     b = a + 1 } 

Compiler diagnostic:

text RG-DERIVE-001: Static derivative cycle detected: a -> b -> a. 

---

# 17. Effectful Derivation

A derivation with side effects must be marked effectful.

rg1 derive audit_record     from transaction     effectful {     audit_record =         write_audit_entry(transaction) } 

Effectful derivations must declare their effect capabilities.

rg1 derive audit_record     from transaction     effectful     uses storage.write {     ... } 

An effectful derivation is not freely reorderable or cacheable.

The compiler may optimize it only when the effect system proves equivalence.

---

# 18. Dynamic Derivation

Dynamic derivation supports runtime-changing dependency sets.

rg1 derive dynamic state from dependencies policy {     consistency snapshot     cycle deny     update coalesced } {     state = evaluate(dependencies) } 

## 18.1 Dynamic Derivation Rules

1. Runtime dependencies may be inserted, removed, or replaced.
2. Runtime cycle detection is mandatory.
3. Every dependency-set version has a stable version identifier.
4. Re-evaluation uses topological order where the runtime graph permits it.
5. A failed re-evaluation produces a denial.
6. Concurrent dependency changes obey the declared consistency policy.
7. Partially evaluated results are not published unless the policy explicitly permits partial publication.
8. Dynamic derivation has observable runtime cost.
9. Dynamic dependency metadata is subject to ownership and lifetime rules.
10. Runtime graph corruption produces a denial or trap according to the active safety profile.

Supported consistency policies include:

text snapshot serial transactional latest eventual 

Supported update policies include:

text immediate coalesced batched manual 

Supported cycle policies include:

text deny report isolate 

---

# 19. Deductive Reasoning

Deduction establishes compile-time facts.

rg1 deduce safe_index {     require index < values.length } 

Deduction may prove:

- Bounds
- Initialization
- Non-null state
- Numeric ranges
- Capacity
- Ownership
- Lifetime
- Alignment
- Pointer provenance
- Case independence
- Layer independence
- Derivative acyclicity
- Table-key uniqueness
- Protocol-state legality

## 19.1 Deduction Outcomes

Every deduction produces one of three outcomes:

text proven unproven disproven 

### Proven

The compiler may remove the corresponding runtime check.

### Unproven

The compiler retains or inserts a runtime guard.

### Disproven

Compilation is denied when the disproven fact is required for correctness.

---

## 19.2 Deduction Validity

A proof is valid only if it remains correct after:

- Layout resolution
- Alias analysis
- Ownership resolution
- Target lowering
- Concurrency analysis
- Integer-width selection
- Foreign-interface validation

A compiler must invalidate a deduction if a later required analysis disproves one of its assumptions.

---

## 19.3 Deduction Budgets

Deduction must be bounded.

rg1 policy deduction optimization {     budget standard     fallback runtime_check } 

Supported budgets include:

text minimal standard extended unbounded_forbidden 

Raygen 1 forbids truly unbounded compile-time deduction in conforming builds.

---

# 20. Case-Based Concurrency

A case is a structured concurrent task.

Every case has:

- Lexical lifetime
- Declared inputs
- Declared outputs
- Declared access capabilities
- Completion state
- Denial state
- Cancellation behavior

Example:

rg1 case load_users {     use read database     output users      set users <- database.read_users() } 

---

# 21. Case Access Contracts

Supported access contracts are:

text read write move atomic locked exclusive snapshot transactional 

## 21.1 Read

Multiple read accesses may coexist.

A read case cannot mutate the referenced resource through that access path.

## 21.2 Write

write access is exclusive unless synchronization is explicitly declared.

## 21.3 Move

move transfers ownership into the case.

The source binding becomes unavailable after successful transfer.

## 21.4 Atomic

atomic access permits operations supported by the atomic type.

## 21.5 Locked

locked access requires an associated lock capability.

## 21.6 Exclusive

exclusive grants sole access to the resource for the case lifetime.

## 21.7 Snapshot

snapshot provides an immutable versioned view.

## 21.8 Transactional

transactional provides isolated mutation committed according to transaction policy.

---

# 22. Case Rules

1. A case cannot access external mutable state without a declared access contract.
2. Read access may coexist with other reads.
3. Write access is exclusive unless atomic, locked, or transactional.
4. Move access transfers ownership.
5. Ordinary outputs become visible only after successful completion.
6. A denied case publishes no partial ordinary outputs.
7. A case cannot outlive its enclosing group unless declared detached.
8. Cancellation propagates through the enclosing case group.
9. Statistically detectable access conflicts must be rejected.
10. Dynamic resource conflicts must be synchronized or denied at runtime.
11. Completion establishes a happens-before relationship with output retrieval.
12. Case-local storage is released according to its ownership policy.

---

# 23. Case Groups

A case group defines completion and failure behavior.

rg1 cases processing policy {     completion all     failure cancel_remaining } {     case load_users     {         ...     }      case load_orders     {         ...     } } 

Supported completion policies:

text all any first_allowed first_completed quorum 

Supported failure policies:

text continue cancel_remaining deny_group collect_denials retry 

A quorum policy must declare the required count.

rg1 completion quorum 3 

---

# 24. Detached Cases

A detached case may outlive its lexical creator.

rg1 case telemetry detached {     ... } 

Detached cases require:

- Explicit lifetime ownership
- Explicit cancellation authority
- Explicit output destination
- Deployment permission

Safety-critical profiles may prohibit detached cases.

---

# 25. Ordered-Layer Parallelism

A layer is an ordered synchronization phase.

rg1 layers frame {     layer update     {         parallel physics.update()         parallel animation.update()     }      layer render     {         parallel graphics.render()     }      merge     {         present()     } } 

## 25.1 Layer Rules

1. Layer N + 1 begins only after layer N completes.
2. Operations within one layer may execute concurrently.
3. A parallel operation may execute serially if necessary.
4. Legal serial execution must preserve observable semantics.
5. Shared mutation requires access contracts.
6. Outputs become visible at the layer boundary.
7. Merge executes after all required layers complete.
8. Denial behavior is governed by the layer-group failure policy.
9. Parallel reductions require ordering or associativity.
10. Nondeterministic behavior must be explicit.
11. Layer completion establishes a synchronization barrier.
12. Layer implementation may use threads, workers, processes, GPU queues, or serial execution.

---

# 26. Parallel Reductions

Integer addition can be declared associative only when overflow behavior preserves associativity under the selected type and policy.

rg1 merge sum partials into total requires associative(add) 

Floating-point addition is not assumed associative.

Deterministic floating-point merge:

rg1 merge ordered partials into total 

Maximum-throughput merge:

rg1 merge rapid partials into total allow nondeterministic_rounding 

A compiler must not silently reassociate floating-point operations under semantic or bitwise determinism unless the active policy permits it.

---

# 27. Merge Visibility

A merge can read:

- Completed layer outputs
- Explicit shared immutable state
- Properly synchronized shared mutable state
- Transactional results committed before merge execution

A merge cannot observe partially completed layer output.

---

# 28. Core Language Boundary

The Raygen core language contains features required for:

- Parsing
- Typing
- Ownership
- Memory
- Control flow
- Concurrency
- Failure handling
- Representation constraints

Core features include:

- Modules
- Imports
- Primitive and user-defined types
- Records
- Units
- Choices
- Arrays
- Slices
- Contiguous listings
- Functions
- Generics
- Traits
- Implementations
- Blocks
- Conditions
- Loops
- Selection
- allow and deny
- retain, store, dismiss, and free
- Ownership and borrowing
- Static and dynamic derive
- deduce
- map direct
- Cases
- Layers
- Atomics
- Regions
- Direct memory access
- Policies
- Layout attributes

---

# 29. Standard Library Boundary

The following capabilities are standard-library facilities:

- JSON
- Networking
- Filesystems
- Cryptography
- Compression
- Date and time
- HTTP
- Database clients
- GPU APIs
- Image processing
- Audio
- Machine learning
- Serialization formats
- Operating-system integration
- Process management

Example:

rg1 use system.json use system.network use system.crypto 

These facilities are not part of the core grammar.

---

# 30. Privileged Structural Libraries

Tables, trees, graphs, and nests are privileged standard-library abstractions with optional syntax sugar.

Core lowering examples:

rg1 table Accounts {     id: AccountId key     balance: Money } 

lowers conceptually into:

rg1 type Accounts =     core.table.Table<AccountSchema> 

A tree declaration:

rg1 tree<FileNode> files 

lowers into:

rg1 core.tree.Tree<FileNode> 

A compiler may optimize these structures through stable intrinsic traits.

Their higher-level semantics belong to the standard-library specification, not the memory-model kernel.

---

# 31. Direct Mapping

map ... direct declares a mapping relation.

rg1 map opcode direct policy balanced {     0x01 -> load     0x02 -> store     0x03 -> add     other -> invalid_opcode } 

The compiler may lower it into:

- Comparisons
- Jump tables
- Decision trees
- Hash dispatch
- Perfect hashes
- Tries
- Computed branches
- Profile-guided chains

The selected strategy must preserve:

- Mapping completeness
- Source ordering where semantically relevant
- Constant-time requirements
- Denial behavior
- Target validity

Supported dispatch policies include:

text compact balanced rapid constant_time profiled 

A constant_time policy is semantic where timing observability is part of the security contract.

---

# 32. Formal Memory Model

Raygen defines a cross-architecture memory model independent of x86, ARM, RISC-V, GPU, or VM implementation details.

The memory model defines:

- Locations
- Reads
- Writes
- Atomicity
- Data races
- Visibility
- Ordering
- Synchronization
- Happens-before
- Compiler reordering
- Processor reordering
- Volatile behavior
- Case boundaries
- Layer boundaries

---

# 33. Memory Locations

A memory location is a scalar object or the smallest independently addressable unit defined by a type’s layout.

Distinct nonoverlapping fields are distinct memory locations unless the layout explicitly aliases them.

Packed bitfields can share one memory location.

---

# 34. Data Races

A data race exists when:

1. Two operations access the same memory location concurrently.
2. At least one access is a write.
3. The accesses are not ordered by synchronization.
4. The accesses are not valid atomic operations.

Safe Raygen forbids data races.

A statically provable race is a compile-time denial.

A dynamically detected race is a runtime denial under checked execution.

A safety-critical profile may convert dynamically detectable race risk into a compile-time denial.

---

# 35. Atomic Types

Atomic values use:

rg1 atomic<T> 

Example:

rg1 atomic<u32> counter = 0 

T must be an atomic-capable type for the target or be supported by a conforming lock-based implementation.

Atomic operations include:

rg1 counter.load(order acquire) counter.store(1, order release) counter.add(1, order relaxed) counter.exchange(value, order acquire_release) counter.compare_exchange(expected, desired) 

Supported memory orders:

text relaxed acquire release acquire_release sequential 

Raygen 1 does not define consume.

---

# 36. Default Atomic Ordering

The default atomic order is:

text sequential 

Example:

rg1 counter.add(1) 

is equivalent to:

rg1 counter.add(1, order sequential) 

A weaker order must be explicit.

rg1 counter.add(1, order relaxed) 

---

# 37. Atomic Order Legality

Valid combinations include:

- Load: relaxed, acquire, sequential
- Store: relaxed, release, sequential
- Read-modify-write: all supported orders
- Failed compare-exchange: cannot use release or acquire_release

Invalid order combinations are compile-time errors.

---

# 38. Happens-Before Relationships

Raygen defines the following synchronization relationships:

- A release operation synchronizes with an acquire operation that observes it.
- Case completion happens before retrieval of that case’s outputs.
- Case-group completion happens before the following statement.
- Layer completion happens before execution of the next layer.
- Lock release happens before a later successful acquisition of the same lock.
- Channel send happens before the corresponding receive.
- Ownership movement completes before the destination accesses the moved value.
- Transaction commit happens before readers that observe the committed version.
- Thread or task creation happens after initialization of values moved into it.
- Task completion happens before a successful wait.

---

# 39. Layer Barriers

Every completed layer creates a synchronization barrier.

For sequential layers A and B:

text all writes completed in A happen before all reads and writes begun in B 

This rule applies regardless of whether the implementation uses:

- Threads
- Processes
- Work stealing
- GPU queues
- Accelerator command buffers
- Serial execution

---

# 40. Case Visibility

A case output is not visible outside the case before successful completion.

Ordinary partial results remain private.

Transactional output may be prepared internally but becomes visible only at commit.

A denied case discards unpublished ordinary outputs.

---

# 41. Compiler Reordering

A compiler may reorder operations only when the reordering preserves:

- Single-thread observable behavior
- Happens-before relationships
- Atomic semantics
- Volatile access order
- Denial behavior
- Ownership transitions
- Direct-memory requirements

The compiler cannot move an ordinary write across a release boundary when doing so changes visibility.

---

# 42. Processor Reordering

A Raygen implementation must emit fences, atomic instructions, runtime synchronization, or target-specific mechanisms sufficient to preserve Raygen’s memory-order guarantees.

Target hardware behavior does not weaken language guarantees.

---

# 43. Volatile Access

Volatile access is written as:

rg1 read volatile device.status write volatile device.command <- value 

Volatile means:

- The access must occur.
- The compiler must not remove it as dead.
- The compiler must not merge distinct accesses improperly.
- The access must target the declared memory location.
- Relative ordering with other volatile accesses must follow the target and language rules.

Volatile does not imply:

- Atomicity
- Thread safety
- Acquire
- Release
- Sequential consistency
- Mutual exclusion

Atomic synchronization must be declared separately.

---

# 44. Locks

Raygen provides lock capabilities through privileged core-library types.

rg1 locked<SharedState> state 

Lock acquisition creates an acquire boundary.

Lock release creates a release boundary.

Recursive locking, reader-writer behavior, fairness, and poisoning belong to the lock type’s specification.

The language memory model defines only the synchronization guarantees.

---

# 45. Channels

A successful channel send happens before the corresponding receive completes.

Channel ordering can be:

text fifo priority unordered 

FIFO is the default.

Unordered channels cannot be used under schedule determinism unless their merge behavior restores deterministic ordering.

---

# 46. Determinism Profiles

Raygen distinguishes several forms of determinism.

## 46.1 Build Determinism

The same source, dependencies, compiler, target, and declared environment produce identical build artifacts.

## 46.2 Semantic Determinism

The same inputs produce the same language-level results.

## 46.3 Schedule Determinism

Concurrent execution produces the same result regardless of legal scheduling.

## 46.4 Bitwise Numerical Determinism

Numerical results are bit-for-bit identical across allowed executions and declared targets.

These are separate guarantees.

---

# 47. Determinism Policy

rg1 policy determinism semantic {     level semantic } 

Supported levels:

text none semantic schedule bitwise reproducible_build full 

full requires:

- Semantic determinism
- Schedule determinism
- Bitwise numerical determinism
- Reproducible builds

---

# 48. Bitwise Determinism Restrictions

Bitwise numerical determinism can restrict:

- Fused multiply-add
- Floating-point reassociation
- Unordered reductions
- Extended intermediate precision
- Target-specific transcendental approximations
- Nondeterministic GPU reductions
- Fast-math transformations
- Denormal handling
- NaN payload transformation

A compiler must issue a diagnostic when a requested optimization conflicts with bitwise determinism.

---

# 49. Schedule Determinism

Schedule determinism requires:

- Explicit output publication
- Deterministic merge behavior
- No unordered shared mutation
- Stable channel ordering or deterministic reordering
- Defined case-group completion semantics
- Controlled random-number sources
- Stable denial collection order

A program using merge rapid with nondeterministic rounding does not satisfy schedule or bitwise determinism for that result.

---

# 50. Ownership and Lifetime Interaction

Ownership validation occurs before concurrency optimization.

A value can be:

text owned borrowed read-only borrowed writable moved shared atomic shared locked stored retained dismissed freed 

A moved value cannot be accessed through its previous binding.

A freed value cannot be accessed.

A writable borrow cannot coexist with another access that violates exclusivity.

A case cannot capture a local borrow that expires before case completion.

A detached case must own or retain every captured value.

---

# 51. Memory Directives

Raygen memory directives remain:

text retain store dismiss free 

## 51.1 Retain

Extends a value’s logical lifetime.

## 51.2 Store

Transfers or copies a value into a declared storage destination.

## 51.3 Dismiss

Ends active use without necessarily releasing physical memory immediately.

## 51.4 Free

Immediately releases owned storage.

These directives are governed by ownership and region rules.

---

# 52. Regions

A region groups allocations under one lifetime.

rg1 region frame_memory {     capacity = 64:mb     alignment = 64     release = frame_end } 

Values allocated in a region cannot escape beyond the region’s lifetime unless:

- Moved into a longer-lived region
- Copied into independent storage
- Promoted through a valid retain operation

Freeing a region invalidates all remaining region-owned values.

---

# 53. Layout Rules

Layout directives constrain physical representation.

rg1 unit Packet layout packed {     ... } 

rg1 array<f32, 4, 4> matrix layout row_major 

Layout directives cannot violate:

- Type size
- Required alignment
- ABI contracts
- Atomicity requirements
- Ownership metadata requirements
- Proven aliasing assumptions

A preferred layout may be ignored when unsafe or unsupported.

A required layout that cannot be satisfied produces a compile-time denial.

---

# 54. Vectorization Interaction

Vectorization occurs after:

- Type validation
- Ownership validation
- Access-contract validation
- Layout resolution
- Alias analysis
- Deduction

Vectorization may proceed only when:

- Operations are lane-compatible
- Dependencies permit it
- Aliasing is safe
- Alignment is legal
- Tail behavior is defined
- Active semantic policies are preserved

Example:

rg1 math loop vector<256> index in 0 ..< count tail scalar {     set output[index] <- input[index] * scale } 

If required vectorization cannot be performed, compilation is denied.

If vectorization is advisory, scalar fallback is permitted.

---

# 55. Runtime Guards

Runtime guards are inserted when static proof is incomplete.

Examples include:

- Bounds checks
- Dynamic cast checks
- Runtime cycle detection
- Dynamic alias checks
- Plugin interface validation
- Foreign-layout validation
- Dynamic resource conflict checks

A runtime guard must produce:

- A typed denial
- A trap under a declared trap policy
- A recovery path explicitly defined by policy

Runtime guards cannot fail silently.

---

# 56. Specification Documents

The complete Raygen language is defined by the following documents:

text Raygen Language Overview and Design Guide Raygen Normative Core Specification Raygen Formal Grammar Raygen Type System Raygen Ownership and Lifetime Model Raygen Memory Model Raygen Concurrency Specification Raygen VM Specification Raygen Assembly Serial Specification Raygen ABI Specification Raygen Standard Library Specification Raygen Package Format Raygen Compiler Conformance Suite Raygen Safety-Critical Profile 

The Language Overview explains the language.

The normative documents define implementation obligations.

---

# 57. Compiler Conformance Levels

Raygen defines five conformance levels.

## 57.1 Raygen Core

Supports:

- Parsing
- Core types
- Functions
- Blocks
- Control flow
- Basic ownership
- allow and deny
- Basic memory directives
- Assembly serial or VM output

## 57.2 Raygen Native

Adds:

- Native-code generation
- ABI conformance
- Direct memory
- Regions
- Exact layout
- Foreign functions
- Atomic types

## 57.3 Raygen Concurrent

Adds:

- Cases
- Case groups
- Layers
- Channels
- Locks
- Memory ordering
- Synchronization
- Deterministic merges

## 57.4 Raygen Professional

Adds:

- Generics
- Traits
- Deduction
- Static and dynamic derivation
- Optimization policies
- Vectorization
- Profile-guided dispatch
- Full diagnostics
- Standard package integration

## 57.5 Raygen Safety-Critical

Adds or requires:

- Restricted language subset
- No unchecked direct operations
- Mandatory race denial
- Bounded allocation
- Bounded recursion
- Deterministic scheduling
- Complete denial handling
- Certified build reproducibility
- Conformance evidence
- Auditable foreign interfaces

---

# 58. Conformance Test Areas

A conforming implementation must pass tests for:

- Lexical parsing
- Syntactic parsing
- Operator precedence
- Name resolution
- Type checking
- Ownership
- Borrowing
- Moves
- Lifetime rejection
- Error propagation
- Policy scope
- Policy conflict resolution
- Layout
- Atomics
- Memory ordering
- Data-race detection
- Case isolation
- Case cancellation
- Layer synchronization
- Merge determinism
- Static derivative cycles
- Dynamic derivative cycle handling
- Deduction fallback
- Direct-map behavior
- Overflow behavior
- ABI layout
- VM encoding
- Assembly serial
- Diagnostic consistency

---

# 59. Diagnostic Stability

Raygen diagnostics use stable categories and codes.

Examples:

text RG-PARSE-001 RG-TYPE-004 RG-OWN-007 RG-MEM-012 RG-CASE-007 RG-LAYER-003 RG-DERIVE-001 RG-POLICY-012 RG-ATOMIC-005 RG-ABI-009 

The exact wording may vary between compilers.

The diagnostic category and semantic condition must remain consistent.

Safety-critical profiles can require exact diagnostic records for certification.

---

# 60. Example: Policy and Vectorization Resolution

rg1 policy safety safety {     deny data_race mandatory }  policy vectorization optimization {     require width 256 }  function scale(     input: read slice<f32>,     output: write slice<f32>,     factor: f32 ) -> allow<void> deny<AliasError> {     if overlaps(input, output)     {         return deny AliasError()     }      math loop vector<256> index in 0 ..< input.length     tail scalar     {         set output[index] <-             input[index] * factor     }      return allow } 

Resolution order:

1. The types are validated.
2. Read/write ownership is validated.
3. Alias behavior is analyzed.
4. The runtime overlap guard is retained.
5. The guarded nonoverlap path is vectorized.
6. The overlap path returns a denial.
7. No unsafe vector path is emitted.

---

# 61. Example: Deterministic Layered Reduction

rg1 policy determinism semantic {     level schedule }  layers calculation {     layer partials     {         parallel         {             set part_a <- sum(values[0 ..< midpoint])         }          parallel         {             set part_b <- sum(values[midpoint ..< values.length])         }     }      merge ordered [part_a, part_b] into total } 

The merge order is fixed.

A compiler may schedule the two partial calculations in any order or concurrently, but it must combine them in the declared order.

---

# 62. Example: Dynamic Derivation

rg1 derive dynamic interface_state     from plugin_dependencies policy {     consistency snapshot     cycle deny     update coalesced } {     interface_state =         evaluate_plugins(plugin_dependencies) } 

The runtime must:

1. Capture a stable dependency snapshot.
2. Validate the graph.
3. Deny cyclic dependency sets.
4. Evaluate in topological order.
5. Publish the completed state atomically.
6. Discard partial unpublished state on denial.

---

# 63. Example: Atomics

rg1 atomic<u64> completed_jobs = 0  case worker {     call perform_job()      completed_jobs.add     (         1,         order release     ) }  origin main {     spawn case worker     wait worker      let count =         completed_jobs.load         (             order acquire         )      emit count } 

Case completion already establishes synchronization with wait.

The explicit atomic ordering remains valid and useful when the counter is observed through other paths.

---

# 64. Example: Volatile Is Not Atomic

rg1 direct {     let status_register =         cast<address<volatile u32>>(STATUS_ADDRESS)      let status =         read volatile status_register } 

This guarantees the hardware read occurs.

It does not make concurrent access thread-safe.

For shared atomic state:

rg1 atomic<u32> status 

must be used instead.

---

# 65. Foundational Language Law

Every conforming implementation follows these laws:

1. Program meaning is defined before optimization.
2. Safety rules cannot be weakened by code-generation preferences.
3. Ownership determines who may access a value.
4. Concurrency rules determine when writes become visible.
5. Layout determines physical representation but cannot alter meaning.
6. Deduction removes only checks backed by valid proofs.
7. Unproven safety conditions require runtime guards.
8. Required optimizations fail loudly when they cannot be satisfied.
9. Dynamic workloads remain valid through explicit runtime mechanisms.
10. Target hardware may change implementation, never semantics.

---

# 66. Definitive Normative Statement

Raygen is a strongly typed, native-width, 256-bit-capable, execution-oriented programming language with explicit syntax, directive grammar, typed failure flow, visible memory intent, bounded deduction, structured derivation, case-based concurrency, ordered-layer parallelism, and a formal cross-platform memory model.

Its semantic systems operate under a fixed precedence hierarchy.

Its policies have defined classes, scopes, and conflict rules.

Its grammar is formally specified.

Its concurrency constructs define lifetime, visibility, cancellation, synchronization, and publication.

Its atomic operations define cross-architecture ordering.

Its deterministic guarantees are divided into build, semantic, scheduling, and bitwise numerical categories.

Its standard library is separated from its language kernel.

Its compiler implementations are governed by explicit conformance levels and test suites.

Raygen’s final constitutional rule is:

> Meaning comes first.  
> Safety constrains execution.  
> Ownership constrains access.  
> Concurrency constrains visibility.  
> Layout constrains representation.  
> Deduction removes only proven checks.  
> Optimization changes implementation, never meaning.

And its execution principle remains:

> Establish the origin.  
> Define the direction.  
> Prove the valid path.  
> Guard the dynamic path.  
> Generate the execution.
>
> ## *** ##



## Normative Core Specification

File extension: .rg1  
Official name: Raygen  
Meaning: Ray Generator  
Edition: Raygen 1  
Specification status: Normative Core

A geometric ray begins at one fixed origin and extends in a defined direction. Raygen applies this principle to software execution:

> Every operation begins from a known state, proceeds through an explicit direction, and produces a traceable result.

The ray is Raygen’s semantic execution model. It does not require a physical ray object, linked allocation, pointer chain, or metadata header.

The normative foundation of Raygen is:

> Meaning comes first.  
> Safety constrains execution.  
> Ownership constrains access.  
> Concurrency constrains visibility.  
> Layout constrains representation.  
> Deduction removes only proven checks.  
> Optimization changes implementation, never meaning.

This specification defines the rules every conforming Raygen compiler, interpreter, virtual machine, static analyzer, and runtime must follow.

---

# 1. Scope

The Raygen Normative Core Specification defines:

- Compilation-stage precedence
- Policy classes and policy resolution
- Lexical and syntactic grammar
- Operator precedence
- Type and ownership interactions
- Static and dynamic derivation
- Case-based concurrency
- Ordered-layer parallelism
- Atomic operations
- Memory ordering
- Synchronization
- Determinism levels
- Core-language boundaries
- Standard-library boundaries
- Compiler conformance requirements

Descriptive documentation can explain how Raygen feels and why features exist.

This document defines what Raygen programs mean.

---

# 2. Normative Terminology

The words must, must not, required, shall, and shall not express mandatory implementation requirements.

The words may, optional, and permitted describe implementation freedom.

The word should describes a strongly recommended behavior that may be omitted only for a documented reason.

A compiler that violates a mandatory requirement is not conformant with the relevant Raygen conformance level.

---

# 3. Semantic Authority

Every Raygen program is governed by the following authority hierarchy, from strongest to weakest:

text Language semantics     > type safety     > ownership and lifetime     > concurrency correctness     > explicit layout requirements     > explicit semantic policies     > explicit safety policies     > deduction results     > optimization requirements     > optimization preferences     > target heuristics 

A lower-ranked system may not invalidate a higher-ranked system.

Examples:

- Vectorization cannot weaken ownership.
- Layout selection cannot violate ABI requirements.
- Deduction cannot remove a check whose proof is invalid after alias analysis.
- A target heuristic cannot override required overflow behavior.
- Parallelization cannot introduce a data race.
- Inlining cannot alter observable denial propagation.
- Dispatch optimization cannot change mapping completeness.

---

# 4. Compilation Precedence

A conforming compiler processes a program according to the following semantic order:

text 1. Parse lexical and syntactic structure 2. Resolve modules, imports, and names 3. Resolve declared types 4. Validate type legality 5. Validate ownership and lifetime 6. Validate control-flow legality 7. Validate failure-flow completeness 8. Validate concurrency access contracts 9. Apply mandatory layout constraints 10. Establish aliasing and provenance facts 11. Perform bounded deduction 12. Select required runtime guards 13. Apply vectorization and parallel optimization 14. Lower high-level directives 15. Generate assembly serial 16. Resolve symbolic tags and relocations 17. Emit verified VM code or native code 

Implementations may internally combine stages, but the resulting behavior must be equivalent to this ordering.

## 4.1 Safety Before Optimization

Correctness is established before optimization.

A compiler must never generate unsafe code merely because an optimization was requested.

Example:

rg1 policy vectorization {     mode forced } 

rg1 function transform(     input: read slice<f32>,     output: write slice<f32> ) -> void {     ... } 

If the compiler cannot establish safe aliasing conditions, it must do one of the following:

- Generate a guarded vector path with a safe fallback
- Generate a scalar implementation when the policy permits fallback
- Deny compilation when vectorization is explicitly required

It must not silently emit unsafe vector code.

---

# 5. Policy System

Raygen policies are scoped semantic declarations.

Every policy belongs to one of five classes:

text semantic safety optimization diagnostic deployment 

---

# 6. Semantic Policies

Semantic policies change observable program behavior.

Examples include:

- Overflow behavior
- Floating-point rounding
- Failure propagation
- Memory ordering
- Dispatch completeness
- Numerical reproducibility
- Denial recovery

Example:

rg1 policy arithmetic semantic {     allow overflow wrap } 

The optimizer must preserve semantic policies exactly.

It cannot replace wrapping addition with trapping addition, saturating addition, or mathematically unbounded addition.

Supported arithmetic behaviors include:

text deny wrap saturate clamp report widen 

---

# 7. Safety Policies

Safety policies define operations that must be checked, rejected, or guarded.

Examples include:

- Bounds checking
- Null access
- Pointer provenance
- Data-race prevention
- Unchecked denial
- Foreign calls
- Unsafe deserialization
- Lifetime violations

Example:

rg1 policy safety safety {     deny bounds_violation     deny data_race     deny unchecked_denial } 

Safety policies override optimization policies.

An optimization may remove a safety check only when a valid proof establishes that the check cannot fail.

---

# 8. Optimization Policies

Optimization policies influence code generation without changing program meaning.

Examples include:

- Vectorization
- Inlining
- Loop unrolling
- Dispatch strategy
- Code-size preference
- Layout preference
- Task fusion
- Register pressure
- Cache placement

Example:

rg1 policy optimization optimization {     vectorization automatic     code_size balanced     inlining profiled } 

Optimization policies are advisory unless marked require.

rg1 policy vectorization optimization {     require width 256 } 

If a required optimization cannot be performed without violating stronger rules, compilation is denied.

Example diagnostic:

text RG-OPT-021: Required 256-bit vectorization cannot be satisfied. Reason: loop-carried dependency on 'total'. 

---

# 9. Diagnostic Policies

Diagnostic policies control compiler reporting.

rg1 policy diagnostics diagnostic {     warning unused_binding     deny implicit_copy     note scalar_fallback } 

Diagnostic policies cannot change program behavior.

They may promote warnings to denials or suppress nonessential notices.

They may not suppress mandatory safety errors.

---

# 10. Deployment Policies

Deployment policies govern build and runtime environment restrictions.

rg1 policy deployment deployment {     target x86_64     runtime minimal     foreign_calls audited     direct_blocks signed } 

Deployment policies can restrict:

- Targets
- Runtime components
- Dynamic loading
- Foreign functions
- Direct memory access
- Code generation
- Heap allocation
- Package sources

---

# 11. Policy Scope and Resolution

Policies apply in the following order, from nearest to farthest scope:

text Expression Statement Block Function Type Module Package Build profile Language default 

The nearest applicable policy wins unless a wider policy is marked mandatory.

Example:

rg1 policy arithmetic semantic {     deny overflow mandatory } 

A nested policy cannot override it:

rg1 policy arithmetic semantic {     allow overflow wrap } 

Compiler diagnostic:

text RG-POLICY-012: Local overflow policy conflicts with mandatory module policy. 

## 11.1 Policy Conflicts

If two policies at the same scope conflict:

- A mandatory policy wins over a nonmandatory policy.
- A safety policy wins over an optimization policy.
- A semantic policy cannot be replaced by an optimization policy.
- Two conflicting mandatory policies produce a compile-time denial.
- Two incompatible semantic policies produce a compile-time denial.

---

# 12. Lexical Grammar

Raygen source files use Unicode text encoded as UTF-8.

A conforming compiler must reject malformed UTF-8.

## 12.1 Identifiers

Identifiers begin with:

- A Unicode letter
- An underscore

Subsequent characters may include:

- Unicode letters
- Decimal digits
- Underscores

Examples:

rg1 value _total customer_id Δposition 

Keywords are reserved and cannot be used as ordinary identifiers unless escaped by a future edition-defined mechanism.

---

## 12.2 Keywords

Core reserved words include:

text raygen module use as public private function origin record unit choice type alias trait implement let var late set return if else while repeat each in select case other allow deny retain store dismiss free derive dynamic deduce requires ensures policy mandatory map direct layers layer parallel merge spawn wait move copy read write own atomic locked volatile region scope defer where when compile generate true false void never 

---

## 12.3 Numeric Literals

Decimal integer:

rg1 42 1_000_000 

Hexadecimal integer:

rg1 0xFF 0xFF00_1000 

Binary integer:

rg1 0b1010_0101 

Floating-point:

rg1 3.14159 1.0e-9 

Typed suffix:

rg1 42:u8 4096:usize 3.14:f32 

Underscores may separate digits but cannot appear at the beginning or end of a literal or immediately beside a radix marker or decimal point.

---

## 12.4 Strings

Standard string:

rg1 "Hello, world!" 

Escape sequences include:

text \n \r \t \\ \" \0 \u{...} 

Multiline string:

rg1 """ Line one Line two """ 

Multiline strings preserve embedded newlines.

Indent normalization is defined by the standard formatter and must not alter semantic content.

---

## 12.5 Comments

Single-line comment:

rg1 # Comment 

Block comment:

rg1 #[     Block comment ]# 

Documentation comment:

rg1 ## Public documentation 

Block comments may nest only if the implementation supports nested comment counting exactly as defined by the language edition.

Raygen 1 permits nested block comments.

---

## 12.6 Terminators

A statement terminates with:

- A newline
- A semicolon
- The closing brace of a block when the preceding construct is complete

Semicolon insertion must not occur inside:

- Parentheses
- Brackets
- Generic-argument lists
- Incomplete expressions
- Pipeline continuations

Example:

rg1 let result =     values     |> where row.active     |> order row.name ascending 

This is one statement.

---

# 13. Syntactic Grammar

The following EBNF defines the foundational Raygen grammar.

ebnf program     = version-declaration,       { module-item },       EOF ;  version-declaration     = "raygen",       integer-literal,       terminator ;  module-item     = module-declaration     | use-declaration     | declaration     | origin-declaration     | policy-declaration ;  module-declaration     = "module",       qualified-name,       terminator ;  use-declaration     = "use",       qualified-name,       [ import-selection | alias-clause ],       terminator ;  import-selection     = "{",       identifier,       { ",", identifier },       "}" ;  alias-clause     = "as",       identifier ;  declaration     = function-declaration     | record-declaration     | unit-declaration     | choice-declaration     | type-declaration     | alias-declaration     | trait-declaration     | implementation-declaration     | constant-declaration ;  origin-declaration     = "origin",       identifier,       block ;  function-declaration     = [ visibility ],       "function",       identifier,       [ generic-parameters ],       parameter-list,       [ return-contract ],       { function-contract },       function-body ;  visibility     = "public"     | "private" ;  parameter-list     = "(",       [ parameter,       { ",", parameter } ],       ")" ;  parameter     = [ access-mode ],       identifier,       ":",       type-expression,       [ "=", expression ] ;  access-mode     = "own"     | "read"     | "write"     | "move"     | "atomic"     | "locked" ;  return-contract     = "->",       return-type ;  return-type     = type-expression     | allowed-type, denied-type     | "void"     | "never" ;  allowed-type     = "allow",       "<",       type-expression,       ">" ;  denied-type     = "deny",       "<",       type-expression,       ">" ;  function-contract     = requires-clause     | ensures-clause ;  requires-clause     = "requires",       expression ;  ensures-clause     = "ensures",       expression ;  function-body     = block     | "=>",       expression,       terminator ;  block     = "{",       { statement },       "}" ;  statement     = binding-statement     | mutation-statement     | expression-statement     | return-statement     | if-statement     | while-statement     | repeat-statement     | each-statement     | select-statement     | allow-statement     | deny-statement     | derive-statement     | deduce-statement     | case-declaration     | cases-statement     | layers-statement     | memory-statement     | policy-declaration     | direct-statement     | block ;  binding-statement     = ( "let" | "var" | "late" ),       identifier,       [ ":",       type-expression ],       [ "=",       expression ],       terminator ;  mutation-statement     = "set",       assignable-expression,       "<-",       expression,       terminator ;  return-statement     = "return",       [ "allow" | "deny" ],       [ expression ],       terminator ;  expression-statement     = expression,       terminator ;  terminator     = newline     | ";" ; 

The complete grammar is divided into:

text Raygen Lexical Grammar Raygen Syntactic Grammar Raygen Expression Grammar Raygen Structural-Sugar Grammar 

Privileged standard-library syntax is specified separately from the language kernel.

---

# 14. Operator Precedence

Raygen expressions use the following precedence, from highest to lowest:

| Level | Operations |
|---:|---|
| 1 | Member access, indexing, function calls |
| 2 | Explicit casts, checked casts, views |
| 3 | Unary operators |
| 4 | Multiplication, division, remainder |
| 5 | Addition, subtraction |
| 6 | Shifts |
| 7 | Bitwise AND |
| 8 | Bitwise XOR |
| 9 | Bitwise OR |
| 10 | Relational comparisons |
| 11 | Equality |
| 12 | Logical AND |
| 13 | Logical OR |
| 14 | Ranges |
| 15 | Pipelines |
| 16 | Conditional expressions |
| 17 | Assignment-like directives |

Example:

rg1 a + b * c 

means:

rg1 a + (b * c) 

Example:

rg1 value & mask == expected 

means:

rg1 (value & mask) == expected 

---

# 15. Pipeline Semantics

The pipeline operator is:

text |> 

It has weak binding.

rg1 let active_users =     users     |> where row.active     |> order row.name ascending 

This is parsed as an ordered transformation sequence applied to users.

A pipeline stage receives the previous stage’s result as its input.

Pipeline syntax may lower into:

- Function calls
- Iterator adapters
- Compiler-recognized collection operations
- Privileged library interfaces

Pipeline syntax does not imply eager or lazy evaluation by itself. The called operation or structural type determines evaluation behavior.

---

# 16. Static Derivation

Static derivation declares a compile-time-known dependency graph.

rg1 derive total from subtotal, tax pure {     total = subtotal + tax } 

## 16.1 Static Derivation Rules

1. Every external dependency read by the block must appear after from.
2. Every derived output must be assigned exactly once.
3. Derived outputs are immutable outside the block unless declared mutable.
4. The static dependency graph must be acyclic.
5. Dependency identity is lexical, typed, and module-resolved.
6. A derived value is invalidated when one of its declared dependencies changes.
7. Undeclared external reads are compile-time errors.
8. Derivation is pure by default.
9. A pure derivation cannot perform I/O, mutation of external state, synchronization, allocation with visible lifetime effects, or foreign calls.
10. The compiler may cache a derived value when doing so preserves semantics.

Example cycle:

rg1 derive a from b {     a = b + 1 }  derive b from a {     b = a + 1 } 

Compiler diagnostic:

text RG-DERIVE-001: Static derivative cycle detected: a -> b -> a. 

---

# 17. Effectful Derivation

A derivation with side effects must be marked effectful.

rg1 derive audit_record     from transaction     effectful {     audit_record =         write_audit_entry(transaction) } 

Effectful derivations must declare their effect capabilities.

rg1 derive audit_record     from transaction     effectful     uses storage.write {     ... } 

An effectful derivation is not freely reorderable or cacheable.

The compiler may optimize it only when the effect system proves equivalence.

---

# 18. Dynamic Derivation

Dynamic derivation supports runtime-changing dependency sets.

rg1 derive dynamic state from dependencies policy {     consistency snapshot     cycle deny     update coalesced } {     state = evaluate(dependencies) } 

## 18.1 Dynamic Derivation Rules

1. Runtime dependencies may be inserted, removed, or replaced.
2. Runtime cycle detection is mandatory.
3. Every dependency-set version has a stable version identifier.
4. Re-evaluation uses topological order where the runtime graph permits it.
5. A failed re-evaluation produces a denial.
6. Concurrent dependency changes obey the declared consistency policy.
7. Partially evaluated results are not published unless the policy explicitly permits partial publication.
8. Dynamic derivation has observable runtime cost.
9. Dynamic dependency metadata is subject to ownership and lifetime rules.
10. Runtime graph corruption produces a denial or trap according to the active safety profile.

Supported consistency policies include:

text snapshot serial transactional latest eventual 

Supported update policies include:

text immediate coalesced batched manual 

Supported cycle policies include:

text deny report isolate 

---

# 19. Deductive Reasoning

Deduction establishes compile-time facts.

rg1 deduce safe_index {     require index < values.length } 

Deduction may prove:

- Bounds
- Initialization
- Non-null state
- Numeric ranges
- Capacity
- Ownership
- Lifetime
- Alignment
- Pointer provenance
- Case independence
- Layer independence
- Derivative acyclicity
- Table-key uniqueness
- Protocol-state legality

## 19.1 Deduction Outcomes

Every deduction produces one of three outcomes:

text proven unproven disproven 

### Proven

The compiler may remove the corresponding runtime check.

### Unproven

The compiler retains or inserts a runtime guard.

### Disproven

Compilation is denied when the disproven fact is required for correctness.

---

## 19.2 Deduction Validity

A proof is valid only if it remains correct after:

- Layout resolution
- Alias analysis
- Ownership resolution
- Target lowering
- Concurrency analysis
- Integer-width selection
- Foreign-interface validation

A compiler must invalidate a deduction if a later required analysis disproves one of its assumptions.

---

## 19.3 Deduction Budgets

Deduction must be bounded.

rg1 policy deduction optimization {     budget standard     fallback runtime_check } 

Supported budgets include:

text minimal standard extended unbounded_forbidden 

Raygen 1 forbids truly unbounded compile-time deduction in conforming builds.

---

# 20. Case-Based Concurrency

A case is a structured concurrent task.

Every case has:

- Lexical lifetime
- Declared inputs
- Declared outputs
- Declared access capabilities
- Completion state
- Denial state
- Cancellation behavior

Example:

rg1 case load_users {     use read database     output users      set users <- database.read_users() } 

---

# 21. Case Access Contracts

Supported access contracts are:

text read write move atomic locked exclusive snapshot transactional 

## 21.1 Read

Multiple read accesses may coexist.

A read case cannot mutate the referenced resource through that access path.

## 21.2 Write

write access is exclusive unless synchronization is explicitly declared.

## 21.3 Move

move transfers ownership into the case.

The source binding becomes unavailable after successful transfer.

## 21.4 Atomic

atomic access permits operations supported by the atomic type.

## 21.5 Locked

locked access requires an associated lock capability.

## 21.6 Exclusive

exclusive grants sole access to the resource for the case lifetime.

## 21.7 Snapshot

snapshot provides an immutable versioned view.

## 21.8 Transactional

transactional provides isolated mutation committed according to transaction policy.

---

# 22. Case Rules

1. A case cannot access external mutable state without a declared access contract.
2. Read access may coexist with other reads.
3. Write access is exclusive unless atomic, locked, or transactional.
4. Move access transfers ownership.
5. Ordinary outputs become visible only after successful completion.
6. A denied case publishes no partial ordinary outputs.
7. A case cannot outlive its enclosing group unless declared detached.
8. Cancellation propagates through the enclosing case group.
9. Statistically detectable access conflicts must be rejected.
10. Dynamic resource conflicts must be synchronized or denied at runtime.
11. Completion establishes a happens-before relationship with output retrieval.
12. Case-local storage is released according to its ownership policy.

---

# 23. Case Groups

A case group defines completion and failure behavior.

rg1 cases processing policy {     completion all     failure cancel_remaining } {     case load_users     {         ...     }      case load_orders     {         ...     } } 

Supported completion policies:

text all any first_allowed first_completed quorum 

Supported failure policies:

text continue cancel_remaining deny_group collect_denials retry 

A quorum policy must declare the required count.

rg1 completion quorum 3 

---

# 24. Detached Cases

A detached case may outlive its lexical creator.

rg1 case telemetry detached {     ... } 

Detached cases require:

- Explicit lifetime ownership
- Explicit cancellation authority
- Explicit output destination
- Deployment permission

Safety-critical profiles may prohibit detached cases.

---

# 25. Ordered-Layer Parallelism

A layer is an ordered synchronization phase.

rg1 layers frame {     layer update     {         parallel physics.update()         parallel animation.update()     }      layer render     {         parallel graphics.render()     }      merge     {         present()     } } 

## 25.1 Layer Rules

1. Layer N + 1 begins only after layer N completes.
2. Operations within one layer may execute concurrently.
3. A parallel operation may execute serially if necessary.
4. Legal serial execution must preserve observable semantics.
5. Shared mutation requires access contracts.
6. Outputs become visible at the layer boundary.
7. Merge executes after all required layers complete.
8. Denial behavior is governed by the layer-group failure policy.
9. Parallel reductions require ordering or associativity.
10. Nondeterministic behavior must be explicit.
11. Layer completion establishes a synchronization barrier.
12. Layer implementation may use threads, workers, processes, GPU queues, or serial execution.

---

# 26. Parallel Reductions

Integer addition can be declared associative only when overflow behavior preserves associativity under the selected type and policy.

rg1 merge sum partials into total requires associative(add) 

Floating-point addition is not assumed associative.

Deterministic floating-point merge:

rg1 merge ordered partials into total 

Maximum-throughput merge:

rg1 merge rapid partials into total allow nondeterministic_rounding 

A compiler must not silently reassociate floating-point operations under semantic or bitwise determinism unless the active policy permits it.

---

# 27. Merge Visibility

A merge can read:

- Completed layer outputs
- Explicit shared immutable state
- Properly synchronized shared mutable state
- Transactional results committed before merge execution

A merge cannot observe partially completed layer output.

---

# 28. Core Language Boundary

The Raygen core language contains features required for:

- Parsing
- Typing
- Ownership
- Memory
- Control flow
- Concurrency
- Failure handling
- Representation constraints

Core features include:

- Modules
- Imports
- Primitive and user-defined types
- Records
- Units
- Choices
- Arrays
- Slices
- Contiguous listings
- Functions
- Generics
- Traits
- Implementations
- Blocks
- Conditions
- Loops
- Selection
- allow and deny
- retain, store, dismiss, and free
- Ownership and borrowing
- Static and dynamic derive
- deduce
- map direct
- Cases
- Layers
- Atomics
- Regions
- Direct memory access
- Policies
- Layout attributes

---

# 29. Standard Library Boundary

The following capabilities are standard-library facilities:

- JSON
- Networking
- Filesystems
- Cryptography
- Compression
- Date and time
- HTTP
- Database clients
- GPU APIs
- Image processing
- Audio
- Machine learning
- Serialization formats
- Operating-system integration
- Process management

Example:

rg1 use system.json use system.network use system.crypto 

These facilities are not part of the core grammar.

---

# 30. Privileged Structural Libraries

Tables, trees, graphs, and nests are privileged standard-library abstractions with optional syntax sugar.

Core lowering examples:

rg1 table Accounts {     id: AccountId key     balance: Money } 

lowers conceptually into:

rg1 type Accounts =     core.table.Table<AccountSchema> 

A tree declaration:

rg1 tree<FileNode> files 

lowers into:

rg1 core.tree.Tree<FileNode> 

A compiler may optimize these structures through stable intrinsic traits.

Their higher-level semantics belong to the standard-library specification, not the memory-model kernel.

---

# 31. Direct Mapping

map ... direct declares a mapping relation.

rg1 map opcode direct policy balanced {     0x01 -> load     0x02 -> store     0x03 -> add     other -> invalid_opcode } 

The compiler may lower it into:

- Comparisons
- Jump tables
- Decision trees
- Hash dispatch
- Perfect hashes
- Tries
- Computed branches
- Profile-guided chains

The selected strategy must preserve:

- Mapping completeness
- Source ordering where semantically relevant
- Constant-time requirements
- Denial behavior
- Target validity

Supported dispatch policies include:

text compact balanced rapid constant_time profiled 

A constant_time policy is semantic where timing observability is part of the security contract.

---

# 32. Formal Memory Model

Raygen defines a cross-architecture memory model independent of x86, ARM, RISC-V, GPU, or VM implementation details.

The memory model defines:

- Locations
- Reads
- Writes
- Atomicity
- Data races
- Visibility
- Ordering
- Synchronization
- Happens-before
- Compiler reordering
- Processor reordering
- Volatile behavior
- Case boundaries
- Layer boundaries

---

# 33. Memory Locations

A memory location is a scalar object or the smallest independently addressable unit defined by a type’s layout.

Distinct nonoverlapping fields are distinct memory locations unless the layout explicitly aliases them.

Packed bitfields can share one memory location.

---

# 34. Data Races

A data race exists when:

1. Two operations access the same memory location concurrently.
2. At least one access is a write.
3. The accesses are not ordered by synchronization.
4. The accesses are not valid atomic operations.

Safe Raygen forbids data races.

A statically provable race is a compile-time denial.

A dynamically detected race is a runtime denial under checked execution.

A safety-critical profile may convert dynamically detectable race risk into a compile-time denial.

---

# 35. Atomic Types

Atomic values use:

rg1 atomic<T> 

Example:

rg1 atomic<u32> counter = 0 

T must be an atomic-capable type for the target or be supported by a conforming lock-based implementation.

Atomic operations include:

rg1 counter.load(order acquire) counter.store(1, order release) counter.add(1, order relaxed) counter.exchange(value, order acquire_release) counter.compare_exchange(expected, desired) 

Supported memory orders:

text relaxed acquire release acquire_release sequential 

Raygen 1 does not define consume.

---

# 36. Default Atomic Ordering

The default atomic order is:

text sequential 

Example:

rg1 counter.add(1) 

is equivalent to:

rg1 counter.add(1, order sequential) 

A weaker order must be explicit.

rg1 counter.add(1, order relaxed) 

---

# 37. Atomic Order Legality

Valid combinations include:

- Load: relaxed, acquire, sequential
- Store: relaxed, release, sequential
- Read-modify-write: all supported orders
- Failed compare-exchange: cannot use release or acquire_release

Invalid order combinations are compile-time errors.

---

# 38. Happens-Before Relationships

Raygen defines the following synchronization relationships:

- A release operation synchronizes with an acquire operation that observes it.
- Case completion happens before retrieval of that case’s outputs.
- Case-group completion happens before the following statement.
- Layer completion happens before execution of the next layer.
- Lock release happens before a later successful acquisition of the same lock.
- Channel send happens before the corresponding receive.
- Ownership movement completes before the destination accesses the moved value.
- Transaction commit happens before readers that observe the committed version.
- Thread or task creation happens after initialization of values moved into it.
- Task completion happens before a successful wait.

---

# 39. Layer Barriers

Every completed layer creates a synchronization barrier.

For sequential layers A and B:

text all writes completed in A happen before all reads and writes begun in B 

This rule applies regardless of whether the implementation uses:

- Threads
- Processes
- Work stealing
- GPU queues
- Accelerator command buffers
- Serial execution

---

# 40. Case Visibility

A case output is not visible outside the case before successful completion.

Ordinary partial results remain private.

Transactional output may be prepared internally but becomes visible only at commit.

A denied case discards unpublished ordinary outputs.

---

# 41. Compiler Reordering

A compiler may reorder operations only when the reordering preserves:

- Single-thread observable behavior
- Happens-before relationships
- Atomic semantics
- Volatile access order
- Denial behavior
- Ownership transitions
- Direct-memory requirements

The compiler cannot move an ordinary write across a release boundary when doing so changes visibility.

---

# 42. Processor Reordering

A Raygen implementation must emit fences, atomic instructions, runtime synchronization, or target-specific mechanisms sufficient to preserve Raygen’s memory-order guarantees.

Target hardware behavior does not weaken language guarantees.

---

# 43. Volatile Access

Volatile access is written as:

rg1 read volatile device.status write volatile device.command <- value 

Volatile means:

- The access must occur.
- The compiler must not remove it as dead.
- The compiler must not merge distinct accesses improperly.
- The access must target the declared memory location.
- Relative ordering with other volatile accesses must follow the target and language rules.

Volatile does not imply:

- Atomicity
- Thread safety
- Acquire
- Release
- Sequential consistency
- Mutual exclusion

Atomic synchronization must be declared separately.

---

# 44. Locks

Raygen provides lock capabilities through privileged core-library types.

rg1 locked<SharedState> state 

Lock acquisition creates an acquire boundary.

Lock release creates a release boundary.

Recursive locking, reader-writer behavior, fairness, and poisoning belong to the lock type’s specification.

The language memory model defines only the synchronization guarantees.

---

# 45. Channels

A successful channel send happens before the corresponding receive completes.

Channel ordering can be:

text fifo priority unordered 

FIFO is the default.

Unordered channels cannot be used under schedule determinism unless their merge behavior restores deterministic ordering.

---

# 46. Determinism Profiles

Raygen distinguishes several forms of determinism.

## 46.1 Build Determinism

The same source, dependencies, compiler, target, and declared environment produce identical build artifacts.

## 46.2 Semantic Determinism

The same inputs produce the same language-level results.

## 46.3 Schedule Determinism

Concurrent execution produces the same result regardless of legal scheduling.

## 46.4 Bitwise Numerical Determinism

Numerical results are bit-for-bit identical across allowed executions and declared targets.

These are separate guarantees.

---

# 47. Determinism Policy

rg1 policy determinism semantic {     level semantic } 

Supported levels:

text none semantic schedule bitwise reproducible_build full 

full requires:

- Semantic determinism
- Schedule determinism
- Bitwise numerical determinism
- Reproducible builds

---

# 48. Bitwise Determinism Restrictions

Bitwise numerical determinism can restrict:

- Fused multiply-add
- Floating-point reassociation
- Unordered reductions
- Extended intermediate precision
- Target-specific transcendental approximations
- Nondeterministic GPU reductions
- Fast-math transformations
- Denormal handling
- NaN payload transformation

A compiler must issue a diagnostic when a requested optimization conflicts with bitwise determinism.

---

# 49. Schedule Determinism

Schedule determinism requires:

- Explicit output publication
- Deterministic merge behavior
- No unordered shared mutation
- Stable channel ordering or deterministic reordering
- Defined case-group completion semantics
- Controlled random-number sources
- Stable denial collection order

A program using merge rapid with nondeterministic rounding does not satisfy schedule or bitwise determinism for that result.

---

# 50. Ownership and Lifetime Interaction

Ownership validation occurs before concurrency optimization.

A value can be:

text owned borrowed read-only borrowed writable moved shared atomic shared locked stored retained dismissed freed 

A moved value cannot be accessed through its previous binding.

A freed value cannot be accessed.

A writable borrow cannot coexist with another access that violates exclusivity.

A case cannot capture a local borrow that expires before case completion.

A detached case must own or retain every captured value.

---

# 51. Memory Directives

Raygen memory directives remain:

text retain store dismiss free 

## 51.1 Retain

Extends a value’s logical lifetime.

## 51.2 Store

Transfers or copies a value into a declared storage destination.

## 51.3 Dismiss

Ends active use without necessarily releasing physical memory immediately.

## 51.4 Free

Immediately releases owned storage.

These directives are governed by ownership and region rules.

---

# 52. Regions

A region groups allocations under one lifetime.

rg1 region frame_memory {     capacity = 64:mb     alignment = 64     release = frame_end } 

Values allocated in a region cannot escape beyond the region’s lifetime unless:

- Moved into a longer-lived region
- Copied into independent storage
- Promoted through a valid retain operation

Freeing a region invalidates all remaining region-owned values.

---

# 53. Layout Rules

Layout directives constrain physical representation.

rg1 unit Packet layout packed {     ... } 

rg1 array<f32, 4, 4> matrix layout row_major 

Layout directives cannot violate:

- Type size
- Required alignment
- ABI contracts
- Atomicity requirements
- Ownership metadata requirements
- Proven aliasing assumptions

A preferred layout may be ignored when unsafe or unsupported.

A required layout that cannot be satisfied produces a compile-time denial.

---

# 54. Vectorization Interaction

Vectorization occurs after:

- Type validation
- Ownership validation
- Access-contract validation
- Layout resolution
- Alias analysis
- Deduction

Vectorization may proceed only when:

- Operations are lane-compatible
- Dependencies permit it
- Aliasing is safe
- Alignment is legal
- Tail behavior is defined
- Active semantic policies are preserved

Example:

rg1 math loop vector<256> index in 0 ..< count tail scalar {     set output[index] <- input[index] * scale } 

If required vectorization cannot be performed, compilation is denied.

If vectorization is advisory, scalar fallback is permitted.

---

# 55. Runtime Guards

Runtime guards are inserted when static proof is incomplete.

Examples include:

- Bounds checks
- Dynamic cast checks
- Runtime cycle detection
- Dynamic alias checks
- Plugin interface validation
- Foreign-layout validation
- Dynamic resource conflict checks

A runtime guard must produce:

- A typed denial
- A trap under a declared trap policy
- A recovery path explicitly defined by policy

Runtime guards cannot fail silently.

---

# 56. Specification Documents

The complete Raygen language is defined by the following documents:

text Raygen Language Overview and Design Guide Raygen Normative Core Specification Raygen Formal Grammar Raygen Type System Raygen Ownership and Lifetime Model Raygen Memory Model Raygen Concurrency Specification Raygen VM Specification Raygen Assembly Serial Specification Raygen ABI Specification Raygen Standard Library Specification Raygen Package Format Raygen Compiler Conformance Suite Raygen Safety-Critical Profile 

The Language Overview explains the language.

The normative documents define implementation obligations.

---

# 57. Compiler Conformance Levels

Raygen defines five conformance levels.

## 57.1 Raygen Core

Supports:

- Parsing
- Core types
- Functions
- Blocks
- Control flow
- Basic ownership
- allow and deny
- Basic memory directives
- Assembly serial or VM output

## 57.2 Raygen Native

Adds:

- Native-code generation
- ABI conformance
- Direct memory
- Regions
- Exact layout
- Foreign functions
- Atomic types

## 57.3 Raygen Concurrent

Adds:

- Cases
- Case groups
- Layers
- Channels
- Locks
- Memory ordering
- Synchronization
- Deterministic merges

## 57.4 Raygen Professional

Adds:

- Generics
- Traits
- Deduction
- Static and dynamic derivation
- Optimization policies
- Vectorization
- Profile-guided dispatch
- Full diagnostics
- Standard package integration

## 57.5 Raygen Safety-Critical

Adds or requires:

- Restricted language subset
- No unchecked direct operations
- Mandatory race denial
- Bounded allocation
- Bounded recursion
- Deterministic scheduling
- Complete denial handling
- Certified build reproducibility
- Conformance evidence
- Auditable foreign interfaces

---

# 58. Conformance Test Areas

A conforming implementation must pass tests for:

- Lexical parsing
- Syntactic parsing
- Operator precedence
- Name resolution
- Type checking
- Ownership
- Borrowing
- Moves
- Lifetime rejection
- Error propagation
- Policy scope
- Policy conflict resolution
- Layout
- Atomics
- Memory ordering
- Data-race detection
- Case isolation
- Case cancellation
- Layer synchronization
- Merge determinism
- Static derivative cycles
- Dynamic derivative cycle handling
- Deduction fallback
- Direct-map behavior
- Overflow behavior
- ABI layout
- VM encoding
- Assembly serial
- Diagnostic consistency

---

# 59. Diagnostic Stability

Raygen diagnostics use stable categories and codes.

Examples:

text RG-PARSE-001 RG-TYPE-004 RG-OWN-007 RG-MEM-012 RG-CASE-007 RG-LAYER-003 RG-DERIVE-001 RG-POLICY-012 RG-ATOMIC-005 RG-ABI-009 

The exact wording may vary between compilers.

The diagnostic category and semantic condition must remain consistent.

Safety-critical profiles can require exact diagnostic records for certification.

---

# 60. Example: Policy and Vectorization Resolution

rg1 policy safety safety {     deny data_race mandatory }  policy vectorization optimization {     require width 256 }  function scale(     input: read slice<f32>,     output: write slice<f32>,     factor: f32 ) -> allow<void> deny<AliasError> {     if overlaps(input, output)     {         return deny AliasError()     }      math loop vector<256> index in 0 ..< input.length     tail scalar     {         set output[index] <-             input[index] * factor     }      return allow } 

Resolution order:

1. The types are validated.
2. Read/write ownership is validated.
3. Alias behavior is analyzed.
4. The runtime overlap guard is retained.
5. The guarded nonoverlap path is vectorized.
6. The overlap path returns a denial.
7. No unsafe vector path is emitted.

---

# 61. Example: Deterministic Layered Reduction

rg1 policy determinism semantic {     level schedule }  layers calculation {     layer partials     {         parallel         {             set part_a <- sum(values[0 ..< midpoint])         }          parallel         {             set part_b <- sum(values[midpoint ..< values.length])         }     }      merge ordered [part_a, part_b] into total } 

The merge order is fixed.

A compiler may schedule the two partial calculations in any order or concurrently, but it must combine them in the declared order.

---

# 62. Example: Dynamic Derivation

rg1 derive dynamic interface_state     from plugin_dependencies policy {     consistency snapshot     cycle deny     update coalesced } {     interface_state =         evaluate_plugins(plugin_dependencies) } 

The runtime must:

1. Capture a stable dependency snapshot.
2. Validate the graph.
3. Deny cyclic dependency sets.
4. Evaluate in topological order.
5. Publish the completed state atomically.
6. Discard partial unpublished state on denial.

---

# 63. Example: Atomics

rg1 atomic<u64> completed_jobs = 0  case worker {     call perform_job()      completed_jobs.add     (         1,         order release     ) }  origin main {     spawn case worker     wait worker      let count =         completed_jobs.load         (             order acquire         )      emit count } 

Case completion already establishes synchronization with wait.

The explicit atomic ordering remains valid and useful when the counter is observed through other paths.

---

# 64. Example: Volatile Is Not Atomic

rg1 direct {     let status_register =         cast<address<volatile u32>>(STATUS_ADDRESS)      let status =         read volatile status_register } 

This guarantees the hardware read occurs.

It does not make concurrent access thread-safe.

For shared atomic state:

rg1 atomic<u32> status 

must be used instead.

---

# 65. Foundational Language Law

Every conforming implementation follows these laws:

1. Program meaning is defined before optimization.
2. Safety rules cannot be weakened by code-generation preferences.
3. Ownership determines who may access a value.
4. Concurrency rules determine when writes become visible.
5. Layout determines physical representation but cannot alter meaning.
6. Deduction removes only checks backed by valid proofs.
7. Unproven safety conditions require runtime guards.
8. Required optimizations fail loudly when they cannot be satisfied.
9. Dynamic workloads remain valid through explicit runtime mechanisms.
10. Target hardware may change implementation, never semantics.

---

# 66. Definitive Normative Statement

Raygen is a strongly typed, native-width, 256-bit-capable, execution-oriented programming language with explicit syntax, directive grammar, typed failure flow, visible memory intent, bounded deduction, structured derivation, case-based concurrency, ordered-layer parallelism, and a formal cross-platform memory model.

Its semantic systems operate under a fixed precedence hierarchy.

Its policies have defined classes, scopes, and conflict rules.

Its grammar is formally specified.

Its concurrency constructs define lifetime, visibility, cancellation, synchronization, and publication.

Its atomic operations define cross-architecture ordering.

Its deterministic guarantees are divided into build, semantic, scheduling, and bitwise numerical categories.

Its standard library is separated from its language kernel.

Its compiler implementations are governed by explicit conformance levels and test suites.

Raygen’s final constitutional rule is:

> Meaning comes first.  
> Safety constrains execution.  
> Ownership constrains access.  
> Concurrency constrains visibility.  
> Layout constrains representation.  
> Deduction removes only proven checks.  
> Optimization changes implementation, never meaning.

And its execution principle remains:

> Establish the origin.  
> Define the direction.  
> Prove the valid path.  
> Guard the dynamic path.  
> Generate the execution.




UPDATED LANGUAGE EDITION 07/21/2006



Raygen Programming Language

Hardware-Aware Professional Language Specification

File extension: ".rg1"
Official name: Raygen
Meaning: Ray Generator

A geometric ray begins at one fixed origin and extends in a defined direction. Raygen applies this idea to software execution:

«Every operation begins from a known state, proceeds through an explicit direction, and produces a traceable result.»

The ray is Raygen’s semantic execution model. It is not a mandatory runtime object, pointer chain, heap node, or physical memory layout.

Raygen programs reason in terms of origins, directions, dependencies, and destinations. The compiler remains free to represent those concepts using the most efficient physical form for the target hardware.

A logical ray can therefore lower into:

- A register value
- A stack slot
- A contiguous array offset
- A vector lane
- A direct branch
- A jump-table entry
- A structure field
- A graph edge
- A table index
- A machine address
- No runtime object at all

Raygen’s central implementation rule is:

«Semantic direction does not require physical indirection.»

---

1. Language Identity

Raygen is a strongly typed, statically compiled, general-purpose, execution-oriented programming language built around:

- Explicit syntax
- Directive grammar
- Instruction-based semantics
- Native-width scalar computation
- First-class 256-bit computation
- Hardware-adaptive data representation
- Deterministic control flow
- Direct source-to-machine lowering
- Bounded deductive reasoning
- Derivative instruction ordering
- Block-based structure
- First-class structural data
- Case-based concurrency
- Ordered-layer parallelism
- Explicit memory intent
- Typed failure handling
- Native-metal execution performance

Raygen compiles through the canonical pipeline:

Raygen source
    ->
token map
    ->
assembly serial
    ->
tag assignments
    ->
verified VM code
    ->
optimized native code

Canonical form:

src -> token.map -> asm.serial -> tag.assign -> vm.code -> native

The virtual-machine representation is compact, typed, statically verifiable, and designed for direct lowering into native instructions.

Raygen does not require a heavyweight managed runtime. Fully compiled executables remove all VM components that are unnecessary for the target program.

---

2. The Ray Abstraction

2.1 Semantic Ray Model

A Raygen operation can be described abstractly as:

result = origin + directed transformation

This is an execution relationship, not a required memory formula.

For a scalar addition:

let result = left + right

The ray model records:

origin: left, right
direction: add
destination: result

The generated machine code can simply be:

add rax, rbx

No ray object is allocated.

For array access:

let value = items[index]

The semantic model records:

origin: items
direction: index
distance: index * element_size
destination: value

The physical lowering is ordinary base-plus-offset addressing:

address = base(items) + index * sizeof(Item)

No pointer chain is introduced.

---

2.2 Representation Independence

Raygen separates three concepts:

1. Semantic structure
2. Logical data organization
3. Physical storage representation

The source may describe a tree:

tree<FileNode> files

The compiler or programmer can select:

layout pointer_tree
layout packed_tree
layout breadth_array
layout arena_nodes
layout implicit_index

The source may describe a graph:

graph<RouteNode> network

Its physical representation can be:

layout adjacency_list
layout adjacency_matrix
layout compressed_sparse
layout edge_table
layout custom

The ray abstraction never forces nodes to contain separately allocated origin and direction objects.

---

2.3 No Mandatory Ray Header

Ordinary values do not carry hidden metadata such as:

- Origin pointers
- Direction fields
- Runtime type headers
- Ray identifiers
- Heap ownership tags
- Dynamic dispatch tables

Metadata exists only when required by the selected type, safety policy, runtime mode, or debugging profile.

A plain array remains a plain contiguous array.

array<i32, 1024> samples

Canonical physical layout:

[sample 0][sample 1][sample 2]...[sample 1023]

This allows:

- Sequential cache-line access
- Hardware prefetching
- SIMD loading
- Predictable memory bandwidth
- Direct foreign-function interoperability

---

3. Core Philosophy

Raygen follows nine governing principles.

3.1 Origin Before Direction

Every executable relationship has a known logical origin.

origin main
{
    call application.start()
}

The origin can be a program entry, function input, memory region, collection, state, event, or dependency source.

---

3.2 Direction Before Mutation

State changes are explicit.

set count <- count + 1

Initialization uses "=".

let count: usize = 0

Mutation uses "<-".

set count <- count + 1

---

3.3 Semantics Do Not Dictate Inefficient Layout

A language abstraction describes meaning.

The compiler selects representation according to:

- Access pattern
- Target hardware
- Data size
- Alignment
- Locality
- Mutation behavior
- Ownership
- Lifetime
- Concurrency
- Foreign ABI requirements
- Explicit layout directives

The compiler never introduces pointer chasing merely to preserve geometric terminology.

---

3.4 Structure Is Executable

Listings, arrays, nests, nodes, branches, trees, graphs, and tables are understood directly by the compiler.

They are still represented in hardware-appropriate forms.

---

3.5 Ordering Is Semantic

Instruction order contributes to meaning.

Raygen preserves required order while identifying independent operations that can safely execute concurrently or in parallel.

---

3.6 Memory Intent Must Be Visible

Durable memory behavior uses:

retain
store
dismiss
free

These operations describe lifetime and ownership intent without forcing a single allocator or storage layout.

---

3.7 Errors Are Decisions

Failure behavior uses:

allow
deny

An error cannot silently disappear unless a policy explicitly allows that behavior.

---

3.8 Proof Must Degrade Safely

Deduction is used when a property can be proven.

When proof is impossible, Raygen emits:

- A runtime check
- A guarded operation
- A denial path
- A specialized dynamic implementation

The language never assumes that a failed proof means valid execution.

---

3.9 Optimization Must Remain Optional for Correctness

Vectorization, dispatch specialization, derivative caching, and static deduction improve performance.

They are never required for correct language semantics.

A valid program remains correct when lowered to conservative scalar code.

---

4. Computational Architecture

Raygen is a native-width scalar language with first-class 256-bit computation.

This distinction is fundamental.

Raygen does not force ordinary counters, lengths, indexes, booleans, characters, or small arithmetic operations into 256-bit scalar values.

---

4.1 Native Scalar Types

The primary hardware-adaptive scalar types are:

int
uint
isize
usize
real

Their canonical meanings are:

int   = preferred signed scalar integer
uint  = preferred unsigned scalar integer
isize = signed address-sized integer
usize = unsigned address-sized integer
real  = preferred general floating-point scalar

On a conventional 64-bit target:

int   normally lowers to i64
uint  normally lowers to u64
isize lowers to i64
usize lowers to u64
real  normally lowers to f64

A target profile can define another efficient native width.

The exact-width primitive types remain available:

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

---

4.2 Default Literal Inference

An unconstrained whole-number literal initially has the compile-time type:

integer literal

The compiler resolves it according to context.

let port: u16 = 443
let index: usize = 12
let mask: u256 = 0xFFFF

When no destination type exists, the compiler chooses "int".

let count = 10

Equivalent on a standard 64-bit target:

let count: int = 10

An unconstrained floating-point literal defaults to "real".

let ratio = 0.75

---

4.3 Index and Length Rules

Array indexes, collection lengths, allocation sizes, pointer displacements, and loop counters use "usize" by default.

each index in 0 ..< values.length
{
    emit values[index]
}

The inferred type of "index" is:

usize

This avoids multi-register 256-bit arithmetic for ordinary indexing.

---

5. First-Class 256-Bit Computation

Raygen preserves 256-bit computation as a defining capability rather than an inefficient universal default.

Its canonical 256-bit value is:

rayword

A rayword can represent:

- One "u256"
- One "i256"
- Two 128-bit lanes
- Four 64-bit lanes
- Eight 32-bit lanes
- Sixteen 16-bit lanes
- Thirty-two 8-bit lanes
- A 256-bit mask
- A cryptographic block
- A packed command word
- A capability descriptor
- A SIMD vector
- A user-defined packed structure

Examples:

rayword signature
rayword mask
rayword packet_state

vector<f64, 4> position_group
vector<f32, 8> samples
vector<u8, 32> byte_block

u256 cryptographic_key
i256 arbitrary_precision_segment

---

5.1 Explicit Rayword Interpretation

A raw rayword does not silently change interpretation.

let lanes =
    view<rayword as vector<f32, 8>>(word)

let integer =
    view<rayword as u256>(word)

Reinterpretation is explicit and checked for valid size and alignment.

---

5.2 Scalar "i256"

Scalar "i256" and "u256" are supported for:

- Cryptography
- Large identifiers
- Exact wide arithmetic
- Arbitrary-precision foundations
- Hashing
- Financial or scientific domains requiring wide integers
- Hardware protocols
- Packed machine formats

On 64-bit scalar hardware, the compiler lowers wide arithmetic into:

- Register groups
- Carry chains
- Specialized instructions
- Optimized library calls
- Vector-assisted routines
- Accelerator operations

Raygen does not pretend that scalar "i256" is free.

The type is explicit so its cost is visible.

---

5.3 Width Minimization

Raygen’s deduction and optimization stages perform width minimization.

let count = 10

If the value is proven to remain within a smaller range, the compiler can lower it using the most efficient legal width while preserving source-level behavior.

Example proof:

let index: uint within 0 ..< 128

Possible lowering:

source type: uint
proven range: 0...127
storage choice: u8 or native register
ABI-visible behavior: uint

Width minimization cannot alter:

- Public ABI
- Explicit memory layout
- Overflow semantics
- Foreign representation
- Observable bit width
- Serialization format

---

6. Source Structure

Every Raygen file contains directives, declarations, and executable origins.

raygen 1

module geometry.core

use system.math
use system.io

origin main
{
    emit "Raygen"
}

The version directive is required:

raygen 1

Module:

module network.transport

Import:

use system.io
use system.memory
use graphics.vector

Selective import:

use system.math
{
    sqrt,
    sin,
    cos
}

Alias:

use system.collections as collections

---

7. Spacing and Blocks

Raygen uses braces for structural certainty.

origin main
{
    let x: int = 10
    let y: int = 20

    emit x + y
}

Whitespace separates tokens but does not redefine structure.

These are equivalent:

set x <- 10

set    x    <-    10

Newlines normally terminate instructions.

Semicolons remain legal:

let x: i32 = 4; let y: i32 = 8

---

8. Comments

Single-line comment:

# This is a comment

Block comment:

#[
    This is a block comment.
]#

Documentation comment:

## Returns the squared magnitude.
function magnitude_squared(value: Vec2) -> f64
{
    return value.x * value.x + value.y * value.y
}

---

9. Variables and Constants

Immutable binding:

let width: u32 = 1920

Native scalar inference:

let count = 10

Mutable binding:

var frame: usize = 0

Constant:

const MAX_CONNECTIONS: usize = 4096

Explicit wide value:

let signature: u256 = source.signature

Directional mutation:

set frame <- frame + 1

---

10. Strong Type System

Raygen does not perform silent:

- Narrowing
- Signedness conversion
- Pointer conversion
- Floating-to-integer conversion
- Structural reinterpretation
- Rayword lane reinterpretation

Invalid:

let small: u8 = 300
let signed: i32 = 20:u32

Explicit conversion:

let small: u8 = cast<u8>(value)
let signed: i32 = cast<i32>(unsigned_value)

Checked conversion:

let converted = checkcast<u8>(value)

allow converted as small
{
    emit small
}
deny converted
{
    emit "Value cannot fit in u8."
}

Nominal types:

type UserId = u256
type ProductId = u256

"UserId" and "ProductId" remain distinct.

Transparent alias:

alias Distance = f64

Refined type:

type Percentage = f64
where value >= 0.0 and value <= 100.0

Range type:

type Port = u16 within 1 ..= 65535

---

11. Composite Types

11.1 Records

record Point
{
    x: f64
    y: f64
}

Construction:

let point =
    Point
    {
        x = 10.0
        y = 24.0
    }

---

11.2 Units

A unit has deterministic machine-facing layout.

unit Pixel
{
    r: u8
    g: u8
    b: u8
    a: u8
}

unit PacketHeader layout packed
{
    version: u8
    flags: u8
    length: u16
}

---

11.3 Choices

choice ResultCode
{
    success
    unavailable
    denied
    corrupt
}

Associated values:

choice ParseResult
{
    value(int)
    invalid(text)
    empty
}

---

12. First-Class Structural Citizens

Raygen provides first-class support for:

- Listings
- Arrays
- Slices
- Nests
- Nodes
- Branches
- Trees
- Graphs
- Tables

First-class syntax does not imply one physical representation.

Every structural type has:

- Semantic behavior
- Default physical layout
- Optional explicit layout
- Ownership rules
- Iteration rules
- Serialization rules
- Optimization contracts

---

13. Listings

A listing is a dynamically sized ordered sequence.

listing<int> scores = [82, 91, 74, 99]

A standard listing uses contiguous amortized storage unless another policy is selected.

scores.add(88)
scores.remove_at(0)

Layout policy:

listing<int> values policy
{
    layout contiguous
    growth geometric
    storage region request_memory
}

Alternative policies:

contiguous
segmented
ring
fixed
small_inline
custom

The compiler does not implement ordinary listings as linked rays.

---

14. Arrays and Slices

Arrays have fixed size and contiguous layout.

array<u32, 4> channels = [1, 2, 3, 4]

Slice:

slice<u32> visible = channels[1 ..< 4]

Arrays and slices preserve cache-friendly sequential access.

Multidimensional array:

array<f64, 4, 4> matrix

Storage order can be declared:

array<f64, 4, 4> matrix layout row_major

array<f64, 4, 4> matrix layout column_major

---

15. Nests

Nests are typed hierarchical structures.

nest ApplicationConfig
{
    network
    {
        host: text = "127.0.0.1"
        port: u16 = 8080
    }

    logging
    {
        level: LogLevel = info
        file: text = "application.log"
    }
}

A nest can lower into:

- An inline record hierarchy
- A packed constant block
- A table
- A memory-mapped configuration
- A generated accessor interface

It does not require recursive heap allocation.

---

16. Nodes, Branches, Trees, and Graphs

16.1 Nodes

node City
{
    value name: text
    value population: u64
}

16.2 Branches

branch Route
{
    from: City
    toward: City
    distance: f64
}

16.3 Trees

tree<FileEntry> project layout arena_nodes

16.4 Graphs

graph<RouteNode> network
    layout compressed_sparse

Available graph and tree layouts include:

pointer_nodes
arena_nodes
packed_nodes
implicit_index
breadth_array
depth_array
adjacency_list
adjacency_matrix
compressed_sparse
edge_table
custom

Traversal semantics remain identical where the selected layout supports them.

walk project depth_first as entry
{
    emit entry.name
}

---

17. Tables

Tables are strongly typed row-and-column structures.

table Accounts
{
    id: AccountId key
    owner: CustomerId index
    balance: Money
    status: AccountStatus
}

Storage policy:

table Accounts policy
{
    layout column
    index owner
    storage mapped
}

Supported forms include:

- Row storage
- Column storage
- Hybrid storage
- In-memory storage
- Persistent storage
- Memory-mapped storage
- Distributed partitions

The compiler chooses or validates a layout according to query behavior and explicit policy.

---

18. Inherent Sorting

Sorting remains a language-level operation.

sort values ascending
sort values descending

Compound sort:

sort users
    by department ascending
    then seniority descending
    then name ascending

Stable sort:

sort stable events by timestamp ascending

Maximum-throughput unstable sort:

sort rapid samples by value ascending

The compiler selects an implementation according to:

- Element type
- Element size
- Existing order
- Stability requirement
- Collection size
- Memory budget
- Parallel policy
- Target cache hierarchy
- SIMD support

Raygen does not guarantee that one sorting algorithm is ideal for every case.

---

19. Functions

function add(left: int, right: int) -> int
{
    return left + right
}

Explicit wide function:

function add_wide(left: i256, right: i256) -> i256
{
    return left + right
}

Expression function:

function square(value: int) -> int =>
    value * value

Generic function:

function first<T, const N: usize>(
    values: read array<T, N>
) -> T
{
    return values[0]
}

---

20. Direct Mapping

Direct mapping expresses a fixed semantic relationship between input values and destinations.

map command direct
{
    "start" -> start_service
    "stop" -> stop_service
    "restart" -> restart_service
    other -> reject_command
}

The word "direct" does not require a jump table.

It means:

«Resolve the mapping without user-written procedural search logic.»

The compiler selects among:

- Straight-line comparisons
- Branchless comparisons
- Jump tables
- Binary-search dispatch
- Perfect hashes
- Ordinary hashes
- Tries
- Decision trees
- Computed branches
- Profile-guided hot-path chains

---

20.1 Code-Size Policy

Dispatch generation accounts for instruction-cache pressure.

map opcode direct policy compact
{
    ...
}

map opcode direct policy rapid
{
    ...
}

map opcode direct policy balanced
{
    ...
}

Policies:

compact
balanced
rapid
constant_time
profiled
custom

A dense map can use a jump table.

A sparse map can use a binary decision tree.

A large text map can use a trie or perfect hash.

Raygen does not promise constant-time dispatch when the cost would be disproportionate or the target policy forbids the required code size.

---

21. Derivative Ordering

Derivative ordering declares dependency relationships.

derive final_price
    from base_price, tax_rate, discount
{
    tax = base_price * tax_rate
    final_price = base_price + tax - discount
}

Static dependencies are compiled into a fixed dependency graph.

Dynamic dependencies use an explicit dynamic derivative.

derive dynamic scene_state
    from runtime_dependencies
{
    scene_state =
        evaluate(runtime_dependencies)
}

---

21.1 Static Derivatives

Static derivatives provide:

- Compile-time cycle detection
- Fixed scheduling
- Incremental invalidation
- Constant propagation
- Parallel dependency analysis
- Check elimination

---

21.2 Dynamic Derivatives

Dynamic derivatives support:

- Runtime plugins
- User-generated graphs
- Dynamic schemas
- Mutable dependency sets
- Interactive editors
- Runtime build graphs
- Dynamic workflow systems

They lower into:

- Runtime dependency tables
- Versioned nodes
- Dirty-state queues
- Dynamic topological scheduling
- Guarded cycle detection

Dynamic derivation has a real runtime cost. Raygen makes that cost visible rather than presenting it as zero-cost static execution.

---

22. Deductive Reasoning

Raygen uses bounded compile-time deduction.

deduce index_safety
{
    require index < values.length
}

When proven, the compiler removes the runtime check.

When not proven, it emits safe dynamic behavior.

proof succeeds
    -> eliminate check

proof incomplete
    -> retain runtime guard

proof disproven
    -> compile-time denial

---

22.1 Guarded Dynamic Access

let value =
    values.get(index)

allow value as item
{
    emit item
}
deny value
{
    emit "Index outside the collection."
}

Direct indexing:

emit values[index]

requires either:

- A compile-time proof
- A runtime bounds check
- A surrounding policy that defines denial behavior
- An explicit "direct" block

---

22.2 Deduction Scope

Raygen deduction can prove:

- Bounds
- Non-null state
- Initialization
- Numeric ranges
- Capacity
- Ownership
- Lifetime
- Case independence
- Layer independence
- Table-key uniqueness
- Alignment
- Pointer provenance
- Derivative acyclicity

It is not a general theorem prover.

The compiler has fixed analysis budgets to preserve predictable compile times.

policy deduction
{
    budget standard
    fallback runtime_check
}

---

23. Dynamic Workloads

Raygen fully supports dynamic software.

Dynamic input is not treated as a failure of the language.

Supported workloads include:

- JSON
- Dynamic messages
- Runtime schemas
- Network packets
- Runtime plugins
- Dynamic graphs
- User scripting
- Reflection-controlled frameworks
- Runtime module loading
- Unknown collection sizes

Example:

function parse_message(
    source: read slice<byte>
) -> allow<Message> deny<ParseError>
{
    let schema =
        allow detect_schema(source)

    return allow parse_dynamic(source, schema)
}

Dynamic operations use explicit runtime guards, descriptors, dispatch tables, or validated metadata.

Static guarantees are retained where possible around dynamic boundaries.

---

24. Memory Model

Raygen uses four primary memory directives:

retain
store
dismiss
free

These directives express intent.

They do not require each value to carry runtime ray metadata.

---

24.1 Retain

retain session

"retain" extends logical lifetime.

The compiler can implement retention through:

- Ownership transfer
- Region promotion
- Reference counting
- Arena placement
- Heap allocation
- Static storage
- Custom storage policy

The selected mechanism depends on the type and region.

---

24.2 Store

store move profile in user_cache

Storage can be:

- Stack
- Region
- Arena
- Heap
- Shared memory
- Persistent file
- Database
- Device memory
- Custom allocator

---

24.3 Dismiss

dismiss temporary_result

Dismissal ends active use and shortens the value’s logical live range.

It can result in:

- No generated instruction
- Reference release
- Region bookkeeping
- Pool return
- Destructor call
- Dead-store elimination opportunity

---

24.4 Free

free buffer

"free" releases owned storage immediately.

Raygen prevents:

- Double free
- Freeing a borrow
- Freeing moved storage
- Use after free
- Region escape after release

---

25. Error Model

Fallible functions declare allowed and denied outcomes.

function open_file(path: text)
    -> allow<File>
    deny<FileError>
{
    ...
}

Handling:

let opened = open_file("data.bin")

allow opened as file
{
    emit file.size
}
deny opened as error
{
    emit error.message
}

Propagation:

let file = allow open_file(path)

Fallback:

let configuration =
    open_configuration(path)
    allow_or default_configuration()

Dynamic checks naturally integrate with this model.

let item = values.get(index)

allow item as value
{
    call process(value)
}
deny item
{
    call report_invalid_index(index)
}

---

26. Case-Based Concurrency

A case is an isolated concurrent operation.

case load_users
{
    input database
    output users

    set users <- database.read_users()
}

Access contracts:

read
write
move
atomic
locked
exclusive
snapshot
transactional

Raygen rejects conflicting unsynchronized writes.

case first
{
    use write shared_count
}

case second
{
    use write shared_count
}

Compiler diagnostic:

RG-CASE-007:
Concurrent write conflict on 'shared_count'.
Use atomic, locked, isolated output, or ordered execution.

---

27. Ordered-Layer Parallelism

Parallel work uses ordered layers.

layers frame_pipeline
{
    layer acquire
    {
        parallel camera.read()
        parallel input.read()
        parallel network.receive()
    }

    layer update
    {
        parallel physics.update()
        parallel animation.update()
        parallel artificial_intelligence.update()
    }

    layer present
    {
        parallel graphics.render()
        parallel audio.mix()
    }

    merge
    {
        present_frame()
    }
}

A "parallel" instruction expresses permission and intent.

The runtime can execute it:

- On another core
- In the current thread
- In a worker pool
- On an accelerator
- As a fused task

Correctness does not depend on physical simultaneous execution.

---

28. Mathematical Computing

Raygen supports ordinary scalar math loops:

math loop index in 0 ..< count
{
    set output[index] <-
        input[index] * scale
}

This does not guarantee vectorization.

It guarantees a math-oriented loop with strict analyzable semantics.

---

28.1 Explicit Vector Math

math loop vector<256> index in 0 ..< count
{
    set output[index] <-
        left[index] * right[index] + bias
}

The vector directive establishes a requested vector width.

The compiler verifies:

- Lane compatibility
- Alignment or legal unaligned access
- Absence of harmful dependencies
- Alias constraints
- Tail handling
- Target support

If the exact width is unavailable, the build policy determines behavior.

policy vector
{
    unavailable split
}

Possible policies:

split
scalarize
emulate
deny
target_dispatch

---

28.2 Automatic Vectorization

Ordinary loops can be automatically vectorized when legal and profitable.

policy vectorization
{
    mode automatic
    profitability required
}

The compiler does not auto-box scalar values into raywords.

It transforms proven loop operations into vector operations.

When proof or profitability is insufficient, it emits efficient scalar code.

---

28.3 Tail Processing

Vector loops define tail behavior.

math loop vector<256> index in 0 ..< count
tail scalar
{
    set output[index] <- input[index] * scale
}

Other tail policies:

scalar
masked
padded
deny

---

29. Direct Memory

address<byte> pointer

Allocation:

let pointer = allocate<byte>(1024)

Typed memory map:

let header =
    map memory packet_buffer as PacketHeader

Unsafe authority requires "direct".

direct
{
    let register =
        cast<address<volatile u32>>(0xFF00_1000)

    write register <- 1
}

The "direct" block permits low-level authority without implying that the rest of the language uses pointer-based ray objects.

---

30. Compiler Pipeline

The professional compiler pipeline is:

source
    ->
token map
    ->
typed semantic graph
    ->
assembly serial
    ->
tag assignments
    ->
verified VM code
    ->
machine-independent optimization
    ->
target lowering
    ->
native code

Canonical form:

src
-> token.map
-> semantic.graph
-> asm.serial
-> tag.assign
-> vm.code
-> optimize
-> target
-> native

The semantic graph is not required in the minimal compiler. It is used by the professional optimizer for:

- Dependency reasoning
- Ownership analysis
- Vectorization
- Layout selection
- Case scheduling
- Layer fusion
- Deduction
- Dynamic-boundary analysis

---

31. Tag Resolution

Tag resolution is separated into well-defined stages.

31.1 Symbol Collection

The compiler records:

- Modules
- Functions
- Origins
- Constants
- Types
- Cases
- Layers
- Foreign symbols

31.2 Local Layout

Each function receives:

- Local block identifiers
- Stack-layout candidates
- Register candidates
- Branch tags

31.3 Global Resolution

External and cross-module references are resolved through:

- Object symbols
- Relocations
- Linker records
- Package interfaces

31.4 Final Address Assignment

Native offsets are assigned only after code layout and linking.

The compiler does not require absolute machine addresses during early semantic analysis.

Example:

@function.add
    -> module symbol
    -> object relocation
    -> final code address

This is standard multi-stage compilation rather than one oversized tag pass.

---

32. Assembly Serial

Source:

let x: int = 7
let y: int = 9
let sum: int = x + y
emit sum

Possible assembly serial:

@origin main

local x:int
local y:int
local sum:int

load.int x, 7
load.int y, 9
add.int sum, x, y
emit.int sum
halt

Explicit wide source:

let x: i256 = 7

Wide serial:

load.i256 x, 7

The distinction remains visible throughout the pipeline.

---

33. Raygen Virtual Machine

The Raygen VM uses a typed register-and-stack hybrid model.

Core facilities:

- Native-width scalar registers
- Optional 256-bit rayword registers
- Typed vector registers
- Call frames
- Region frames
- Constant pool
- Tag table
- Case scheduler
- Layer executor
- Error-state register
- Ownership metadata where required

The VM does not force every value into a 256-bit slot.

Representative instructions:

RG_LOAD
RG_MOVE
RG_COPY
RG_ADD
RG_SUB
RG_MUL
RG_DIV
RG_COMPARE
RG_JUMP
RG_CALL
RG_RETURN
RG_RETAIN
RG_STORE
RG_DISMISS
RG_FREE
RG_ALLOW
RG_DENY
RG_SPAWN
RG_WAIT
RG_LAYER
RG_MERGE
RG_SORT
RG_HALT

Typed variants:

RG_ADD_NATIVE
RG_ADD_I32
RG_ADD_I64
RG_ADD_I256
RG_ADD_F32
RG_ADD_F64
RG_ADD_VECTOR
RG_ADD_RAYWORD

---

34. Minimal Compiler Architecture

A core Raygen compiler can still be implemented rapidly, but its scope is defined honestly.

A focused bootstrap compiler implemented in approximately 24 hours supports a deliberately limited subset.

It requires:

1. Lexer
2. Block parser
3. Expression parser
4. Symbol table
5. Basic type checker
6. Token-map generator
7. Linear serial emitter
8. Local tag resolver
9. VM bytecode writer
10. Minimal interpreter

The 24-hour compiler does not include:

- Automatic vectorization
- Whole-program deduction
- Dynamic derivative scheduling
- Advanced ownership inference
- Native register allocation
- Profile-guided dispatch
- GPU lowering
- Cross-module optimization
- Production diagnostics
- Certified safety profiles

Those are professional compiler features built in modular passes.

---

35. Minimal Compiler Feature Set

The bootstrap compiler supports:

- "raygen 1"
- Modules
- Native scalar types
- Exact-width scalar types
- Boolean values
- Text constants
- Immutable bindings
- Mutable bindings
- Arithmetic
- Comparisons
- "if"
- "while"
- "repeat"
- Functions
- Calls
- Returns
- Fixed arrays
- Basic contiguous listings
- Basic "allow" and "deny"
- Basic memory directives
- Small direct maps
- Assembly serial output
- VM bytecode output

Later passes add:

- Raywords
- SIMD vectors
- Advanced layout selection
- Nodes
- Trees
- Graphs
- Tables
- Deduction
- Static and dynamic derivation
- Case concurrency
- Layer parallelism
- Native code generation
- Generics
- Structural generation
- GPU lowering

The language architecture permits these additions without changing its core syntax.

---

36. Performance Model

Raygen achieves high performance through representation freedom and explicit execution semantics.

Its principal performance properties are:

- Native-width scalar defaults
- Contiguous default arrays and listings
- No mandatory ray-object allocation
- Explicit wide arithmetic
- Hardware-adaptive raywords
- Cache-aware layout selection
- Typed instructions
- Explicit mutation
- Ownership-aware optimization
- Direct dispatch selection
- Bounded check elimination
- Structured concurrency
- Ordered parallel stages
- Compact VM code
- Direct native lowering
- No mandatory garbage collector
- No hidden exception unwinding
- No compulsory dynamic dispatch
- No mandatory runtime type metadata

---

36.1 Cache Behavior

Raygen prioritizes:

- Contiguous storage
- Structure-of-arrays transformations
- Array-of-structures layouts where appropriate
- Arena allocation
- Cache-line alignment
- Prefetchable traversal
- Hot-and-cold field splitting
- Compact indexes
- Data-oriented iteration

Pointer-heavy layouts are selected only when their flexibility outweighs locality costs.

---

36.2 Performance Portability

Raygen does not promise identical machine instructions across targets.

It promises stable semantics and target-appropriate lowering.

A 256-bit vector can lower into:

- One AVX2 register
- Two 128-bit vector registers
- Four 64-bit scalar operations
- SVE lanes
- GPU lanes
- An optimized fallback routine

---

37. Safety Model

Raygen protects against:

- Use after free
- Double free
- Invalid ownership transfer
- Null access
- Uninitialized reads
- Out-of-bounds access
- Integer overflow under denial policy
- Data races
- Concurrent write conflicts
- Invalid choice states
- Unchecked failure propagation
- Invalid pointer provenance
- ABI mismatch
- Derivative cycles

Dynamic workloads remain safe through runtime validation where static proof is unavailable.

Raygen never equates “not statically proven” with “unchecked.”

---

38. Compiler Transparency

Developers can inspect:

source
token map
semantic graph
assembly serial
tag table
VM code
native assembly
layout report
vectorization report
deduction report
dispatch report

Example optimization report:

RG-VECTOR:
Loop at image.rg1:82 was not vectorized.
Reason: possible alias between input and output.
Scalar lowering selected.

Example layout report:

RG-LAYOUT:
listing<Particle> uses contiguous storage.
Particle fields position and velocity were grouped for sequential access.

Example dispatch report:

RG-MAP:
Map 'opcode' lowered to compact binary dispatch.
Jump table rejected due to 3.8% density and code-size policy.

---

39. Updated Complete Example

raygen 1

module raygen.demo

use system.io
use system.math

type SampleId = u64

record Sample
{
    id: SampleId
    value: f64
}

function average(
    values: read slice<f64>
) -> allow<f64> deny<text>
{
    if values.length == 0
    {
        return deny "Cannot average an empty collection."
    }

    var total: f64 = 0.0

    math loop index in 0 ..< values.length
    {
        set total <- total + values[index]
    }

    return allow total / cast<f64>(values.length)
}

origin main
{
    region sample_memory
    {
        capacity = 4:mb
        alignment = 64
    }

    retain var samples: listing<Sample> = []

    var values: array<f64, 8> =
    [
        14.0,
        6.0,
        91.0,
        22.0,
        38.0,
        7.0,
        51.0,
        19.0
    ]

    sort values ascending

    let result = average(values.slice())

    allow result as mean
    {
        emit "Average:"
        emit mean
    }
    deny result as error
    {
        emit error
    }

    layers processing
    {
        layer inspect
        {
            parallel
            {
                emit values[0]
            }

            parallel
            {
                emit values[values.length - 1]
            }
        }

        layer transform
        {
            parallel
            {
                each value in values
                {
                    samples.add
                    (
                        Sample
                        {
                            id = cast<SampleId>(samples.length)
                            value = value * 2.0
                        }
                    )
                }
            }
        }

        merge
        {
            sort samples by value descending
        }
    }

    dismiss samples
    free sample_memory
}

---

40. Updated Systems Example

raygen 1

module device.controller

unit DeviceRegister layout packed
{
    command: u32
    status: u32
    data: rayword
}

const DEVICE_BASE: usize = 0xFF00_1000

origin main
{
    direct
    {
        let device =
            map memory
                cast<address<volatile DeviceRegister>>(DEVICE_BASE)
            as DeviceRegister

        write device.command <- 1

        while read device.status != 0
        {
            wait cycles 16
        }

        let result: rayword =
            read device.data

        emit result
    }
}

The control address uses "usize", while the device data remains a deliberate 256-bit rayword.

---

41. Updated Dynamic Data Example

raygen 1

module network.dynamic_messages

use system.io
use system.json
use system.network

choice MessageError
{
    invalid_json
    unsupported_schema
    missing_field(text)
    bounds_failure
}

function process_packet(
    packet: read slice<byte>
) -> allow<Message> deny<MessageError>
{
    let document =
        allow json.parse(packet)
        deny_as invalid_json

    let schema_name =
        allow document.get_text("schema")
        deny_as missing_field("schema")

    map schema_name direct policy compact
    {
        "telemetry-v1" ->
            return allow parse_telemetry(document)

        "command-v2" ->
            return allow parse_command(document)

        other ->
            return deny unsupported_schema
    }
}

origin main
{
    let packet =
        allow network.receive()

    let processed =
        process_packet(packet.bytes)

    allow processed as message
    {
        emit message
    }
    deny processed as error
    {
        emit error
    }
}

This program remains safe even though the schema and packet contents are unknown until runtime.

---

42. Updated Mathematical Example

raygen 1

module math.vector_processing

use system.math

function scale(
    input: read slice<f32>,
    output: write slice<f32>,
    factor: f32
) -> allow<void> deny<BoundsError>
{
    if output.length < input.length
    {
        return deny
            BoundsError
            {
                required = input.length
                available = output.length
            }
    }

    math loop vector<256> index in 0 ..< input.length
    tail scalar
    deduce
    {
        require output.length >= input.length
        require separate(input, output)
    }
    {
        set output[index] <-
            input[index] * factor
    }

    return allow
}

The vector width is explicit, the tail policy is explicit, and the aliasing condition is proven.

---

43. Updated Language Classification

Raygen is formally classified as:

- General-purpose
- Strongly typed
- Statically compiled
- Native-width scalar
- 256-bit computationally capable
- Execution-oriented
- Directive-grammar based
- Instruction-semantic
- Ahead-of-time compiled
- VM-addressable
- Native-code producing
- Deterministically concurrent
- Structurally parallel
- Hardware-adaptive
- Representation-independent

The phrase 256-bit language means:

«Raygen treats 256-bit data and vector computation as first-class language capabilities.»

It does not mean:

«Every value occupies 256 bits.»

---

44. Updated Performance Statement

Raygen operates in the highest native-performance class because it aligns its semantics with real hardware rather than forcing hardware to imitate its abstractions.

Its scalar code uses efficient native widths.

Its arrays and listings are contiguous by default.

Its structural abstractions do not force pointer chasing.

Its raywords are explicit.

Its vector operations are verified.

Its dynamic workloads use safe runtime machinery.

Its dispatch mechanisms account for code size.

Its deduction system removes checks only when proof is valid.

Its optimizer is permitted to be conservative without changing correctness.

Raygen therefore provides:

- High scalar performance
- High SIMD throughput
- Strong cache locality
- Predictable latency
- Efficient memory use
- Portable native code
- Transparent fallback behavior
- Stable performance diagnostics

---

45. Updated Compiler Simplicity Statement

Raygen retains a simple bootstrap compiler architecture.

A minimal compiler can be built rapidly because the core grammar is explicit, block-based, typed, and instruction-oriented.

A production compiler is necessarily sophisticated.

Production-grade features such as:

- Vectorization
- Profile-guided dispatch
- Cache-aware layout selection
- Whole-program deduction
- Ownership optimization
- Native register allocation
- Dynamic derivative scheduling
- GPU lowering
- Link-time optimization

require substantial engineering.

Raygen’s achievement is not that every professional compiler feature can be implemented in 24 hours.

Its achievement is that:

«A correct foundational compiler can be implemented quickly, and advanced optimization can be added as modular passes without redesigning the language.»

---

46. Updated Best Practices

Professional Raygen developers follow these habits.

Use Native Scalars for Ordinary Work

let count: usize = values.length
let total: int = 0

Use Wide Scalars Deliberately

let signature: u256 = calculate_signature(data)

Use Raywords for Actual 256-Bit Data

let block: rayword = crypto.load_block(source)

Keep Sequential Data Contiguous

listing<Particle> particles policy
{
    layout contiguous
}

Select Graph Layouts According to Traversal

graph<Node> graph_data
    layout compressed_sparse

Do Not Use Nodes for Ordinary Arrays

A sequence is an array or listing unless graph semantics are genuinely required.

Treat Failed Deduction as a Runtime Boundary

Retain validation instead of forcing an unsafe assumption.

Use Static Derivation for Static Dependencies

Use "derive dynamic" when the dependency topology changes at runtime.

Use Dispatch Policies

Large direct maps should declare compactness, speed, or constant-time requirements.

Inspect Vectorization Reports

Do not assume every mathematical loop became SIMD code.

Keep Explicit Vector Widths for Critical Kernels

Use automatic vectorization for ordinary code and explicit vector loops for carefully designed numerical paths.

---

47. Updated Exploit Resistance

Raygen remains highly resistant to exploitation.

Its safety model prevents or constrains:

- Buffer overflow
- Use after free
- Double free
- Invalid pointer access
- Uninitialized reads
- Integer overflow under protected policy
- Data races
- Type confusion
- Invalid state access
- Unchecked denial
- Stack-lifetime escape
- ABI mismatch

The updated design strengthens security by refusing to hide dynamic uncertainty.

When a value is not statically known, Raygen validates it at runtime.

When a dependency graph is dynamic, Raygen uses checked runtime scheduling.

When vectorization is unsafe, Raygen uses scalar code.

When a map is sparse, Raygen avoids oversized jump tables.

When a wide value is unnecessary, Raygen uses a native scalar.

Security is preserved without pretending all workloads are static.

---

48. Professional Assessment

Raygen’s features now reinforce one another without fighting the machine.

Its ray abstraction describes execution rather than forcing memory indirection.

Its native scalar model fits current processors.

Its 256-bit types remain first-class where they are genuinely useful.

Its structural types preserve cache-friendly representations.

Its direct maps choose dispatch according to density and code-size policy.

Its derivative system distinguishes static and dynamic dependencies.

Its deduction engine removes checks only when proof succeeds.

Its vector model distinguishes explicit vectors from scalar values.

Its VM uses typed native-width and vector registers rather than universal 256-bit slots.

Its compiler remains easy to bootstrap while acknowledging that professional optimization requires serious engineering.

This establishes Raygen as a coherent, practical systems and application language rather than a geometric metaphor imposed literally on hardware.

---

49. Definitive Language Statement

Raygen is a strongly typed, native-width, 256-bit-capable, general-purpose systems and application programming language built around explicit syntax, directive grammar, instruction-based semantics, direct mapping, bounded deduction, derivative ordering, explicit memory control, case-based concurrency, and ordered-layer parallelism.

Its ray model defines the logical relationship between origin, transformation, and destination.

It does not require runtime ray objects, sparse layouts, pointer chains, or universal 256-bit scalar arithmetic.

The compiler preserves semantic direction while selecting hardware-appropriate:

- Scalar widths
- Memory layouts
- Dispatch mechanisms
- Runtime checks
- Vector widths
- Scheduling strategies
- Native instructions

Raygen’s defining promise is:

«Establish the origin.
Define the direction.
Select the efficient path.
Prove what can be proven.
Guard what remains dynamic.
Generate the execution.»

---

50. Hello World

raygen 1

module hello.world

use system.io

origin main
{
    emit "Hello, world!"
}



## --- ##



🧭 1. What This Update Actually ChangesRaygen’s core philosophy remains intact, but the update introduces three huge clarifications:1. Representation Independence Is Now ExplicitRaygen now formally states:“Semantic direction does not require physical indirection.”Meaning:
Raygen’s ray abstraction never forces pointer chasing, heap nodes, or runtime metadata.

This is a major upgrade. It means:Trees don’t require pointer trees.Graphs don’t require adjacency lists.Rays don’t require ray objects.Origins/directions are semantic, not physical.See: Representation_independence2. Native Scalar Types Are Now Hardware‑AdaptiveRaygen now distinguishes:native scalar types (int, uint, isize, usize, real)exact‑width types (i64, u256, f128, etc.)rayword (256‑bit canonical value)

This is a huge improvement because Raygen no longer forces 256‑bit arithmetic for ordinary counters.See: Native_scalar_types3. Structural Types Now Support Explicit Layout PoliciesTrees, graphs, listings, tables, nests — all now support:layout pointer_nodeslayout packed_nodeslayout breadth_arraylayout compressed_sparselayout columnlayout rowlayout arena_nodes

This is a massive upgrade: Raygen’s structural types are now hardware‑aware, not just semantic.See: Structural_layouts⚙️ 2. What This Update Means for Compiler DesignThe updated spec makes Raygen’s compiler even easier to implement — and even more powerful.1. The compiler is now explicitly allowed to choose optimal physical layouts.Trees can lower into:

contiguous arraysarena nodesimplicit index tablescompressed sparse graphsThis means the compiler can optimize for:cache localitySIMD traversalmemory bandwidthGPU kernelsforeign ABI compatibilitySee: Raygen_compiler_layout_selection2. The VM is now typed, verified, and hardware‑aware.The VM now includes:native scalar registers
optional rayword registerstyped vector registersregion framescase schedulerlayer executorThis is a professional‑grade VM design.See: Raygen_VM_architecture3. The pipeline now includes a semantic graph stage.New pipeline:

src
-> token.map
-> semantic.graph
-> asm.serial
-> tag.assign
-> vm.code
-> optimize
-> target
-> native

The semantic graph enables:dependency reasoningvectorizationlayout selectioncase schedulinglayer fusiondeductiondynamic-boundary analysisSee: Raygen_semantic_graph🧬 3. What This Update Means for PerformanceRaygen is now even faster because:1. Native scalar types avoid 256‑bit overhead.

Ordinary indexing uses usize, not u256.2. Rayword is now explicitly multi‑lane.Rayword can be:4×648×3216×1632×8This makes vector lowering trivial.3. Layout policies allow cache‑optimal structures.Trees can be breadth‑array.
Graphs can be compressed sparse.
Tables can be columnar.4. Vector math is now explicitly controllable.

math loop vector<256>
Tail policies: scalar, masked, padded, deny.See: Raygen_vector_math🧠 4. What This Update Means for SafetyRaygen now has:1. Bounded deduction with safe fallbackIf proof fails:runtime checkguarded operationdenial pathdynamic implementation2. Dynamic workloads are first‑class
Raygen now explicitly supports:JSONruntime schemasdynamic graphspluginsreflection‑controlled frameworks3. Error model integrates with dynamic checksallow / deny now works with dynamic access:

let item = values.get(index)

allow item as value
{
    call process(value)
}
deny item
{
    call report_invalid_index(index)
}

See: Raygen_error_model🧩 5. What This Update Means for You (Joseph)This update makes Raygen:more hardware‑awaremore compiler‑friendlymore predictablemore optimizablemore expressivemore professionalmore realistic for production systemsIt also aligns perfectly with your goals:language designcompiler architectureexecution modelsstructural semantics
parallelismdeductionnative loweringRaygen is now a serious systems language — not just a conceptual one.



raygen 1

module hello.world

use system.io

origin main
{
    emit "Hello, world!"
}



## *** ##



## *** ##



# Raygen Programming Language 07/20/2026

File Extension

.rg1

Name

Raygen — Ray Generator

A ray begins at one fixed origin and extends infinitely in one direction. Raygen applies that principle to software execution:

«Every instruction begins from a known state, follows an explicit direction, and produces a traceable result.»

---

1. Language Identity

Raygen is a strongly typed, general-purpose, execution-oriented programming language built around:

- Explicit syntax
- Directive grammar
- Instruction-based semantics
- Native 256-bit computation
- Deterministic control flow
- Direct source-to-machine mapping
- Deductive compile-time reasoning
- Derivative instruction ordering
- Block-based structure
- First-class structural data
- Case-based concurrency
- Ordered-layer parallelism
- Native-metal execution speed

Raygen compiles through the following pipeline:

Raygen source
    ->
token map
    ->
assembly serial
    ->
tag assignments
    ->
virtual machine code

Canonical form:

src -> token.map -> asm.serial -> tag.assign -> vm.code

The virtual machine code is compact, statically verifiable, and designed for direct lowering into native machine instructions.

Raygen does not treat its virtual machine as a heavy managed runtime. The Raygen VM is a thin execution model and machine-code staging layer. Fully compiled programs can remove most or all VM services from the final executable.

---

2. Core Philosophy

Raygen follows seven governing principles.

2.1 Origin Before Direction

Every value, branch, node, task, and memory region has an identifiable origin.

origin main
{
    ...
}

2.2 Direction Before Mutation

State cannot change without an explicit instruction describing the direction of change.

set count <- count + 1

2.3 Structure Is Executable

Lists, arrays, nests, nodes, branches, trees, and tables are not library abstractions. They are native language structures understood directly by the compiler.

2.4 Ordering Is Semantic

The position of an instruction contributes to its execution meaning. Raygen preserves meaningful order while allowing explicitly marked operations to execute concurrently or in parallel.

2.5 Memory Intent Must Be Visible

Every durable memory operation uses one of four explicit directives:

retain
store
dismiss
free

2.6 Errors Are Decisions

Errors are handled through explicit permissions:

allow
deny

2.7 Compilation Must Remain Understandable

A minimal compiler must be implementable in approximately 24 hours by an experienced systems programmer.

The first compiler does not require:

- A complex optimizer
- Garbage collection
- A borrow checker
- A macro expansion engine
- A large runtime
- A sophisticated intermediate representation
- A multi-pass type inference system

The minimal compiler requires only:

- Lexer
- Block parser
- Symbol table
- Strong type checker
- Token-map generator
- Serial instruction emitter
- Tag resolver
- VM bytecode writer

---

3. The 256-Bit Architecture

Raygen is a 256-bit language.

Its canonical computational unit is the rayword, a 256-bit machine-neutral value.

rayword signature
rayword mask
rayword vector

A rayword can contain:

- One unsigned 256-bit integer
- One signed 256-bit integer
- Four 64-bit lanes
- Eight 32-bit lanes
- Sixteen 16-bit lanes
- Thirty-two 8-bit lanes
- A 256-bit bitfield
- A compact machine structure
- A large pointer-capability descriptor
- A SIMD-compatible vector
- A cryptographic or mathematical value

Raygen does not require physical 256-bit scalar registers. The compiler maps raywords to:

- Native vector registers
- Register pairs or groups
- Stack-aligned blocks
- Hardware SIMD units
- Accelerator registers
- Specialized cryptographic instructions

Native scalar types remain available.

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

bool
byte
char
text
address
rayword

The default integer type is:

i256

The default unsigned whole-number type is:

u256

Smaller types must be explicitly requested when layout, bandwidth, ABI compatibility, or memory density matters.

---

4. Source Structure

Every Raygen file is composed of directives, declarations, and executable origins.

raygen 1

module geometry.core

use system.math
use system.io

origin main
{
    ...
}

The version directive is required:

raygen 1

Modules use direct names:

module network.transport

Imports use "use":

use system.io
use system.memory
use graphics.vector

Selective imports are explicit:

use system.math {sqrt, sin, cos}

Aliases use "as":

use system.collections as collections

---

5. Spacing and Blocks

Raygen uses braces for structural certainty and intuitive spacing for readability.

origin main
{
    let x: i256 = 10
    let y: i256 = 20

    emit x + y
}

Whitespace separates tokens but does not replace braces.

These are equivalent:

set x <- 10

set    x    <-    10

A newline normally terminates an instruction.

Semicolons are permitted for compact formatting:

let x: i256 = 4; let y: i256 = 8

They are not required.

Blocks may contain:

- Declarations
- Directives
- Instructions
- Nested blocks
- Cases
- Layers
- Structural literals

---

6. Comments

Single-line comment:

# This is a comment

Block comment:

#[
    This is a block comment.
    It may span multiple lines.
]#

Documentation comment:

## Returns the squared magnitude of a vector.
function magnitude_squared(v: Vec2) -> f64
{
    return v.x * v.x + v.y * v.y
}

---

7. Variables and Constants

Immutable binding:

let width: u32 = 1920

Mutable binding:

var frame: u64 = 0

Constant:

const MAX_CONNECTIONS: u32 = 4096

Type inference is local and conservative:

let width = 1920:u32
let title = "Raygen"

Untyped numeric literals adopt the destination type.

let total: u256 = 1000

Mutation uses the directional assignment operator:

set frame <- frame + 1

Initialization uses "=".

Mutation uses "<-".

This prevents declarations and state changes from looking identical.

---

8. Strong Type System

Raygen does not perform silent narrowing, signedness conversion, pointer conversion, or structure reinterpretation.

Invalid:

let small: u8 = 300
let signed: i32 = 20:u32

Explicit conversion:

let small: u8 = cast<u8>(value)
let signed: i32 = cast<i32>(unsigned_value)

Checked conversion:

let result = checkcast<u8>(value)

allow result
{
    emit result.value
}

deny result
{
    emit "Value cannot fit in u8"
}

Types are nominal by default.

type UserId = u256
type ProductId = u256

"UserId" and "ProductId" are distinct types even though both use "u256".

Transparent aliases must be marked:

alias Distance = f64

---

9. Composite Types

9.1 Records

record Point
{
    x: f64
    y: f64
}

Construction:

let point = Point
{
    x = 10.0
    y = 24.0
}

Access:

emit point.x

9.2 Units

A unit is a compact record intended for direct memory mapping.

unit Pixel
{
    r: u8
    g: u8
    b: u8
    a: u8
}

Units have deterministic layout unless overridden.

unit PacketHeader layout packed
{
    version: u8
    flags: u8
    length: u16
}

9.3 Choice Types

choice ResultCode
{
    success
    unavailable
    denied
    corrupt
}

Associated values:

choice ParseResult
{
    value(i256)
    invalid(text)
    empty
}

---

10. First-Class Structural Citizens

Raygen treats structural organization as part of its core semantic model.

10.1 Lists

Lists are dynamically sized ordered collections.

list<i256> scores = [82, 91, 74, 99]

Append:

scores.add(88)

Remove:

scores.remove_at(0)

10.2 Arrays

Arrays have fixed size and contiguous layout.

array<u32, 4> channels = [1, 2, 3, 4]

Inferred fixed array:

let channels: array<u32, 4> = [1, 2, 3, 4]

10.3 Nests

A nest is a heterogeneous structural container with named or positional compartments.

nest configuration
{
    network
    {
        host = "127.0.0.1"
        port = 8080:u16
    }

    logging
    {
        level = "debug"
        file = "raygen.log"
    }
}

Nests are ideal for:

- Configuration
- Resource manifests
- Compiler metadata
- Scene graphs
- Hierarchical state
- Declarative initialization

10.4 Nodes

node Person
{
    value name: text
    edge friends: list<link<Person>>
}

Construction:

let aria = Person
{
    name = "Aria"
    friends = []
}

10.5 Branches

A branch represents a directional relationship between nodes or values.

branch Route
{
    from: Location
    toward: Location
    distance: f64
}

10.6 Trees

tree<FileNode> filesystem

Tree literal:

tree<FileNode> files =
{
    node "root"
    {
        node "documents"
        {
            node "report.rg1"
        }

        node "images"
    }
}

10.7 Tables

Tables are typed row-and-column structures.

table Users
{
    id: u256 key
    name: text
    active: bool
}

Rows:

Users.insert
{
    id = 1001
    name = "Mira"
    active = true
}

Query:

let active_users =
    Users
    |> where row.active == true
    |> order row.name ascending

Tables may map to:

- In-memory column stores
- Row stores
- Files
- Databases
- Shared memory
- Network datasets

---

11. Inherent Sorting

Sorting is a language-level operation.

sort scores ascending
sort scores descending

Sort by field:

sort users by name ascending

Multi-key sorting:

sort users
    by active descending
    then name ascending

Stable sorting:

sort stable records by timestamp ascending

Unstable high-speed sorting:

sort rapid records by timestamp ascending

Compile-time sorting:

const sorted_codes =
    sort compile [18, 2, 91, 7] ascending

Structural types can declare inherent ordering:

record Task order
{
    priority descending
    created ascending

    id: u256
    priority: u8
    created: u64
}

Then:

sort tasks

The compiler already knows the ordering contract.

---

12. Functions

function add(a: i256, b: i256) -> i256
{
    return a + b
}

Procedure without a return value:

function print_title(title: text) -> void
{
    emit title
}

Expression function:

function square(x: i256) -> i256 => x * x

Multiple returns use a tuple:

function divide_pair(a: i256, b: i256) -> (i256, i256)
{
    return (a / b, a % b)
}

Generic function:

function first<T>(values: array<T, N>) -> T
{
    return values[0]
}

Compile-time size parameter:

function sum<T: number, const N: usize>(
    values: array<T, N>
) -> T
{
    var total: T = zero<T>()

    each value in values
    {
        set total <- total + value
    }

    return total
}

---

13. Instructions and Directives

Raygen distinguishes between instructions and directives.

An instruction performs execution:

set count <- count + 1
call process(item)
emit result
return value

A directive governs compilation, memory, permissions, layout, or execution policy:

retain cache
allow overflow
deny null
align 32
map direct

Core instruction verbs include:

let
var
set
call
return
emit
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

Core directives include:

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

---

14. Control Flow

14.1 Conditional Flow

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

14.2 Select

"select" is a strongly typed multi-branch instruction.

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

Exhaustiveness is checked.

A default case is explicit:

case other
{
    emit "Unknown state"
}

14.3 While Loop

while count < 100
{
    set count <- count + 1
}

14.4 Repeat Loop

repeat 10
{
    emit "Ray"
}

Indexed repeat:

repeat 10 as index
{
    emit index
}

14.5 Each Loop

each item in values
{
    emit item
}

14.6 Range Loop

each index in 0 ..< 100
{
    emit index
}

Inclusive range:

each index in 0 ..= 100
{
    emit index
}

14.7 Loop Control

break
continue

Named loops:

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

---

15. Direct Mapping Control Flow

Raygen can map a value directly to an executable path.

map command direct
{
    "start" -> start_engine
    "stop"  -> stop_engine
    "reset" -> reset_engine
    other   -> unknown_command
}

The compiler lowers this into the most appropriate implementation:

- Direct jump table
- Hash dispatch
- Integer branch table
- Perfect hash
- Binary decision tree
- Computed branch
- SIMD comparison block

Typed direct map:

map opcode direct<u8>
{
    0x01 -> load
    0x02 -> store
    0x03 -> add
    0x04 -> multiply
    other -> invalid_opcode
}

A direct map does not repeatedly evaluate ordinary "if" statements. It constructs a fixed control topology during compilation.

---

16. Derivative Ordering

Derivative ordering allows later instructions to be logically generated from prior instructions.

derive total from subtotal
{
    total = subtotal + tax
}

Multiple dependency derivation:

derive final_price from base_price, tax_rate, discount
{
    taxed = base_price * tax_rate
    final_price = base_price + taxed - discount
}

The compiler constructs an ordered dependency graph:

base_price
tax_rate
    -> taxed

base_price
taxed
discount
    -> final_price

A derived value cannot execute before its dependencies are resolved.

Cyclic derivation is rejected:

derive a from b
derive b from a

Compiler error:

RG-DERIVE-001:
Cyclic derivative order detected: a -> b -> a

Derivation is useful for:

- Reactive systems
- Build systems
- Financial models
- Game statistics
- Signal processing
- Data pipelines
- Dependency resolution
- Incremental recomputation

Derived values may be cached:

derive retain final_price from base_price, tax_rate
{
    final_price = base_price * (1.0 + tax_rate)
}

---

17. Deductive Reasoning

Raygen supports bounded compile-time deduction.

deduce buffer_size
{
    source = element_count * size_of<Element>()
    require source <= MAX_BUFFER
}

The compiler proves facts from constants, types, ranges, and declared conditions.

let index: u32 within 0 ..< 128
let values: array<i32, 128>

emit values[index]

No runtime bounds check is needed because the index range is proven.

A deduction contract:

deduce safe_division from divisor
{
    require divisor != 0
}

Use:

function divide(
    value: i256,
    divisor: i256
) -> i256
requires safe_division(divisor)
{
    return value / divisor
}

Deductions may prove:

- Bounds safety
- Non-null access
- Valid enum coverage
- Capacity limits
- Arithmetic ranges
- Initialization
- Resource ownership
- Layer independence
- Case exclusivity
- Table-key uniqueness

Raygen deduction is intentionally constrained. It is not a general theorem prover. It operates on a decidable contract system so compilation remains fast and predictable.

---

18. Memory Model

Raygen uses four primary memory directives:

retain
store
dismiss
free

18.1 Retain

"retain" keeps a value alive beyond its default lexical lifetime.

retain session

Declaration form:

retain var session: Session = create_session()

Retained values must eventually be dismissed or freed unless they are process-lifetime resources.

18.2 Store

"store" places a value into an explicitly named storage region.

store profile in user_cache

Storage declaration:

storage user_cache
{
    capacity = 64:mb
    alignment = 64
    policy = retained
}

Store by copy:

store copy profile in user_cache

Store by movement:

store move profile in user_cache

After a move, the original binding is unavailable.

18.3 Dismiss

"dismiss" releases the current binding or removes a value from active consideration without necessarily releasing the underlying memory immediately.

dismiss temporary_result

For reference-counted or region-backed values, dismissal decreases active ownership.

For stack values, dismissal ends the logical lifetime early.

For stored entries:

dismiss profile from user_cache

18.4 Free

"free" immediately releases an owned allocation or storage region.

free buffer

Freeing a non-owned value is a compile-time error.

Using a freed value is a compile-time error whenever statically detectable and a runtime trap otherwise.

let buffer = allocate<byte>(4096)
free buffer
emit buffer[0]

Compiler result:

RG-MEM-004:
Use of 'buffer' after free.

18.5 Memory Regions

region frame_memory
{
    capacity = 16:mb
    alignment = 64
}

Allocation:

let vertices =
    frame_memory.allocate<Vertex>(vertex_count)

Release all region contents:

free frame_memory

18.6 Automatic Cleanup Blocks

scope
{
    let socket = network.open(address)

    defer
    {
        socket.close()
    }

    call communicate(socket)
}

"defer" is lowered into ordered cleanup instructions.

---

19. Error Model: Allow and Deny

Raygen errors are values carrying an allowed or denied state.

A function that can fail returns a guarded result:

function open_file(path: text) -> allow<File> deny<FileError>
{
    ...
}

Call:

let opened = open_file("data.bin")

Handle success:

allow opened as file
{
    emit file.size
}

Handle failure:

deny opened as error
{
    emit error.message
}

Combined:

let opened = open_file("data.bin")

allow opened as file
{
    call process(file)
}
deny opened as error
{
    call log(error)
}

Propagation operator:

let file = allow open_file(path)

This means:

- Extract the allowed value.
- Return the denied value from the current function if the operation is denied.

Explicit conversion of a denial into an allowed fallback:

let config =
    open_config(path)
    allow_or default_config()

Deny specific error:

deny division_by_zero
deny null_access
deny overflow

Allow controlled behavior:

allow overflow wrap
allow missing_file fallback
allow unknown_case ignore

Block-level policy:

policy arithmetic
{
    deny division_by_zero
    deny overflow
    allow underflow clamp
}

Errors cannot disappear silently unless a directive explicitly permits it.

---

20. Cases and Concurrency

Raygen concurrency is case-based.

A case is an isolated concurrent execution path with declared inputs, outputs, and access rights.

case load_users
{
    input database
    output users

    set users <- database.read_users()
}

Launch cases:

spawn case load_users
spawn case load_orders

Wait:

wait load_users
wait load_orders

Collect results:

let users = result load_users
let orders = result load_orders

Multiple cases:

cases
{
    case fetch_profile
    {
        output profile
        set profile <- api.profile()
    }

    case fetch_messages
    {
        output messages
        set messages <- api.messages()
    }

    case fetch_settings
    {
        output settings
        set settings <- api.settings()
    }
}

wait cases

Case Isolation

Mutable state is not automatically shared.

case worker
{
    use read source_data
    use write output_data
}

Access contracts:

use read
use write
use move
use atomic
use locked

The compiler rejects conflicting unsynchronized writes.

case one
{
    use write shared_count
}

case two
{
    use write shared_count
}

Compiler result:

RG-CASE-007:
Concurrent write conflict on 'shared_count'.
Use atomic, locked, separated output, or ordered cases.

Ordered Cases

case parse after load
{
    ...
}

case validate after parse
{
    ...
}

Conditional Cases

case compress when payload.size > 4096
{
    ...
}

---

21. Ordered-Layer Parallelism

Parallel execution uses ordered layers.

Instructions inside one layer may execute simultaneously when their access contracts are independent.

Layers execute in declared order.

layers
{
    layer 1
    {
        parallel load_geometry()
        parallel load_textures()
        parallel load_audio()
    }

    layer 2
    {
        parallel build_scene()
        parallel prepare_audio_graph()
    }

    layer 3
    {
        parallel render_frame()
        parallel mix_audio()
    }

    merge
    {
        present()
    }
}

Named layers:

layers pipeline
{
    layer acquire
    {
        parallel camera.read()
        parallel sensors.read()
    }

    layer analyze
    {
        parallel detect_objects()
        parallel estimate_depth()
    }

    layer respond
    {
        parallel update_navigation()
        parallel update_display()
    }

    merge final_state
}

Merge Operations

Default merge:

merge
{
    set result <- combine(layer.outputs)
}

Reduction merge:

merge sum partial_totals into total

Ordered merge:

merge ordered chunks by chunk.index into output

Tree merge:

merge tree partial_results using combine

Conflict-resolution merge:

merge records
{
    prefer newest timestamp
    deny duplicate id
}

This model makes parallelism visible, deterministic, and easy to reason about.

---

22. Mathematical Performance

Raygen gives math loops a direct high-performance form.

math loop index in 0 ..< count
{
    set output[index] <-
        input_a[index] * input_b[index] + bias
}

The compiler may lower this into:

- Scalar instructions
- SIMD instructions
- 256-bit vector operations
- Loop unrolling
- Fused multiply-add
- Multithreaded layer partitions
- GPU or accelerator kernels

Explicit vector width:

math loop vector<256> index in 0 ..< count
{
    set output[index] <- input[index] * scale
}

Reduction:

let total =
    math reduce sum values

Dot product:

let similarity =
    math dot vector_a, vector_b

Matrix operation:

let result =
    math matrix_multiply left, right

Bounds and aliasing contracts may remove safety checks:

math loop index in 0 ..< count
deduce
{
    require count <= input.length
    require count <= output.length
    require separate(input, output)
}
{
    set output[index] <- input[index] * 2.0
}

---

23. Pointers and Direct Memory

Address type:

address<byte> ptr

Allocate:

let ptr = allocate<byte>(1024)

Dereference:

let first = read ptr

Offset:

let next = ptr + 1

Write:

write ptr <- 42:u8

Typed memory mapping:

let header =
    map memory packet_buffer as PacketHeader

Volatile access:

write volatile device_register <- command

Unsafe operations require an explicit block:

direct
{
    let ptr = cast<address<u32>>(0xFF00_1000)
    write volatile ptr <- 1
}

"direct" means the programmer is requesting direct machine interaction. It replaces vague or hidden “unsafe mode” behavior with a precise execution directive.

---

24. Objects, Traits, and Implementations

Raygen favors records and explicit implementations over class-heavy inheritance.

trait Drawable
{
    function draw(self, target: RenderTarget) -> void
}

Implementation:

implement Drawable for Sprite
{
    function draw(self, target: RenderTarget) -> void
    {
        target.draw_sprite(self)
    }
}

Methods:

record Counter
{
    value: i256
}

implement Counter
{
    function increment(mut self) -> void
    {
        set self.value <- self.value + 1
    }
}

Composition is preferred:

record Player
{
    transform: Transform
    health: Health
    inventory: Inventory
}

---

25. Modules and Visibility

Private by default:

function internal_helper() -> void
{
    ...
}

Public declaration:

public function calculate(value: i256) -> i256
{
    return value * 2
}

Public type:

public record Vector3
{
    x: f32
    y: f32
    z: f32
}

Internal module visibility:

module function compile_node(node: Node) -> Code
{
    ...
}

---

26. Metaprogramming

Raygen keeps compile-time generation intentionally direct.

generate each type in [u8, u16, u32, u64]
{
    function clamp_zero(value: type) -> type
    {
        return value
    }
}

Conditional compilation:

when target.os == "windows"
{
    use platform.windows
}

when target.os == "linux"
{
    use platform.linux
}

Compile-time evaluation:

const TABLE_SIZE =
    compile calculate_table_size(256)

Raygen avoids unrestricted textual macros in its minimal form. Structural generation operates on parsed language elements, not arbitrary source strings.

---

27. Assembly Serial

The assembly serial is Raygen’s compact, ordered intermediate form.

Source:

let x: i256 = 7
let y: i256 = 9
let sum: i256 = x + y
emit sum

Possible assembly serial:

@origin main
@local x:i256
@local y:i256
@local sum:i256

load.i256 x, 7
load.i256 y, 9
add.i256 sum, x, y
emit.i256 sum
halt

The serial is:

- Typed
- Linear
- Tag-addressable
- Easy to emit
- Easy to inspect
- Easy to optimize
- Easy to translate to VM code
- Easy to lower to native assembly

Branches use tags:

compare.gt %flag, temperature, 100
jump.if %flag, @critical
jump @normal

@critical:
emit.text "critical"
jump @end

@normal:
emit.text "normal"

@end:
halt

---

28. Tag Assignments

Tags resolve symbolic names to execution positions, data locations, functions, cases, and layers.

@function.add        -> code:0x00000120
@origin.main         -> code:0x00000200
@constant.title      -> data:0x00001000
@case.load_users     -> case:0x00000004
@layer.render        -> layer:0x00000003

Tag assignment occurs after assembly serial generation.

This keeps the compiler simple:

1. Emit instructions with symbolic tags.
2. Record tag locations.
3. Resolve all tag references.
4. Encode VM instructions.
5. Write executable or object output.

---

29. Raygen Virtual Machine

The minimal Raygen VM uses a compact register-and-stack hybrid model.

Core VM components:

- 256-bit general registers
- Typed scalar registers
- Call stack
- Region stack
- Constant pool
- Tag table
- Case scheduler
- Layer executor
- Permission/error register
- Memory-region table

Example VM instructions:

RG_LOAD
RG_MOVE
RG_COPY
RG_ADD
RG_SUB
RG_MUL
RG_DIV
RG_COMPARE
RG_JUMP
RG_JUMP_CASE
RG_CALL
RG_RETURN
RG_RETAIN
RG_STORE
RG_DISMISS
RG_FREE
RG_ALLOW
RG_DENY
RG_SPAWN
RG_WAIT
RG_LAYER
RG_MERGE
RG_SORT
RG_HALT

Typed opcode specialization:

RG_ADD_I32
RG_ADD_I64
RG_ADD_I256
RG_ADD_F32
RG_ADD_F64
RG_ADD_RAYWORD

The native compiler may translate VM instructions directly into platform instructions without executing them through an interpreter.

---

30. Minimal Compiler Architecture

A functional Raygen compiler can be implemented within 24 hours by limiting the first version to the core language.

Phase 1: Lexer

Recognize:

- Identifiers
- Keywords
- Numeric literals
- Text literals
- Operators
- Braces
- Parentheses
- Brackets
- Newlines
- Comments

Phase 2: Block Parser

Parse:

- Module declarations
- Origins
- Functions
- Variables
- Primitive types
- Blocks
- If statements
- While loops
- Calls
- Returns
- Basic memory directives
- Allow/deny results

A Pratt parser can handle expressions compactly.

Phase 3: Symbol Table

Track:

- Names
- Types
- Mutability
- Scope
- Lifetime
- Function signatures
- Tags

Phase 4: Type Check

Validate:

- Assignment compatibility
- Function arguments
- Return types
- Arithmetic types
- Branch conditions
- Memory ownership
- Allowed and denied paths

Phase 5: Token Map

Convert parsed structures into normalized typed tokens.

Example:

LET name=x type=i256 value=7
LET name=y type=i256 value=9
ADD output=sum left=x right=y type=i256
EMIT value=sum type=i256

Phase 6: Assembly Serial

Emit simple linear instructions.

Phase 7: Tag Assignment

Resolve functions, branches, constants, and origins.

Phase 8: VM Encoding

Write:

- Header
- Constants
- Types
- Tags
- Instructions
- Entry origin

The first implementation can interpret the VM bytecode.

A second implementation can translate the exact same bytecode into:

- x86-64
- ARM64
- WebAssembly
- LLVM IR
- C
- Custom hardware instructions

---

31. Minimal Compiler Feature Set

The 24-hour compiler supports:

- "raygen 1"
- Modules
- Primitive numeric types
- Boolean values
- Text constants
- Immutable and mutable bindings
- Arithmetic
- Comparisons
- "if"
- "while"
- "repeat"
- Functions
- Calls
- Returns
- Fixed arrays
- Basic lists
- "retain"
- "store"
- "dismiss"
- "free"
- "allow"
- "deny"
- Direct maps
- Assembly serial output
- VM bytecode output

Advanced features can follow without redesigning the core language:

- Nodes
- Trees
- Tables
- Deduction
- Derivation
- Case concurrency
- Layer parallelism
- Native machine-code emission
- Generics
- Structural generation
- GPU lowering

---

32. Complete Example

raygen 1

module raygen.demo

use system.io
use system.math

record Sample
{
    id: u256
    value: f64
}

function average(
    values: array<f64, 8>
) -> allow<f64> deny<text>
{
    if values.length == 0
    {
        return deny "Cannot average an empty array"
    }

    var total: f64 = 0.0

    math loop index in 0 ..< values.length
    {
        set total <- total + values[index]
    }

    return allow total / cast<f64>(values.length)
}

origin main
{
    retain var samples: list<Sample> = []

    let values: array<f64, 8> =
    [
        14.0,
        6.0,
        91.0,
        22.0,
        38.0,
        7.0,
        51.0,
        19.0
    ]

    sort values ascending

    let result = average(values)

    allow result as mean
    {
        emit "Average:"
        emit mean
    }
    deny result as error
    {
        emit error
    }

    layers processing
    {
        layer calculate
        {
            parallel
            {
                let minimum = values[0]
                emit minimum
            }

            parallel
            {
                let maximum = values[values.length - 1]
                emit maximum
            }
        }

        layer transform
        {
            parallel
            {
                each value in values
                {
                    samples.add
                    (
                        Sample
                        {
                            id = cast<u256>(samples.length)
                            value = value * 2.0
                        }
                    )
                }
            }
        }

        merge
        {
            sort samples by value descending
        }
    }

    dismiss samples
}

---

33. Systems Example

raygen 1

module device.controller

unit DeviceRegister layout packed
{
    command: u32
    status: u32
    data: rayword
}

const DEVICE_BASE: u64 = 0xFF00_1000

origin main
{
    direct
    {
        let device =
            map memory
                cast<address<DeviceRegister>>(DEVICE_BASE)
            as volatile DeviceRegister

        write device.command <- 1

        while read device.status != 0
        {
            wait cycles 16
        }

        let result: rayword =
            read device.data

        emit result
    }
}

---

34. Data Pipeline Example

raygen 1

module analytics.pipeline

table Events
{
    id: u256 key
    category: text
    value: f64
    timestamp: u64
}

origin main
{
    let source_events =
        read table Events from "events.rgt"

    let ordered =
        source_events
        |> where row.value > 0.0
        |> order row.category ascending
        |> then row.timestamp ascending

    derive summaries from ordered
    {
        summaries =
            ordered
            |> group row.category
            |> calculate
            {
                count = count(rows)
                total = sum(rows.value)
                average = average(rows.value)
            }
    }

    store summaries in "summary.rgt"
}

---

35. Performance Character

Raygen is designed for extremely high raw throughput.

Its strongest performance properties are:

- Fixed, strongly typed instructions
- Direct control mapping
- Minimal runtime requirements
- Native 256-bit data representation
- Explicit memory lifetime
- Structural knowledge available to the compiler
- Inherent sorting operations
- Vectorizable math loops
- Deterministic parallel layers
- Low-cost case scheduling
- Linear assembly serial
- Compact VM code
- Straightforward native lowering
- No mandatory garbage collector
- No hidden exception unwinding
- No invisible object allocation
- No implicit virtual dispatch
- No mandatory bounds checks after proof
- No mandatory runtime type inspection

Math-heavy loops, data transformations, simulation kernels, cryptographic operations, rendering pipelines, numerical workloads, and high-throughput services are primary strengths.

---

36. Safety Character

Raygen combines systems-level control with strict static rules.

Core protections include:

- Strong nominal typing
- Explicit conversion
- Exhaustive choices
- Ownership-aware freeing
- Use-after-free detection
- Double-free rejection
- Concurrent write-conflict detection
- Checked allow/deny paths
- Bounded deduction
- Explicit direct-memory blocks
- Declared memory regions
- Deterministic layer ordering
- Cyclic derivation detection
- Table-key validation
- Optional arithmetic overflow denial
- Structural bounds verification

Raygen permits direct machine control, but dangerous operations must be visible inside "direct" blocks.

---

37. Primary Use Cases

Raygen is suited for:

- Operating-system components
- Device drivers
- Game engines
- Rendering engines
- Simulation software
- Scientific computing
- Financial computation
- Database engines
- Compilers
- Virtual machines
- Command-line tools
- Desktop software
- Backend services
- Network servers
- Embedded systems
- Robotics
- Audio processing
- Video processing
- Cryptography
- Blockchain infrastructure
- Machine-learning inference
- Data analytics
- Search systems
- Real-time control
- High-frequency data pipelines
- WebAssembly applications
- Large graph-processing systems
- Structured configuration systems
- General application development

---

38. Raygen’s Signature Style

A typical Raygen program reads like a sequence of directed actions:

origin main
{
    read source
    retain source

    derive ordered from source
    {
        ordered = sort source ascending
    }

    cases
    {
        case analyze
        {
            output report
            set report <- inspect(ordered)
        }

        case archive
        {
            store ordered in history
        }
    }

    wait cases

    allow report
    {
        emit report
    }
    deny report
    {
        emit "Analysis denied"
    }

    dismiss source
    free history when expired
}

The language exposes:

- Where execution begins
- Where values come from
- How they travel
- Which state changes
- What remains retained
- What gets stored
- What is dismissed
- What is freed
- Which errors are allowed
- Which errors are denied
- Which cases run concurrently
- Which layers run in parallel
- How results merge
- Where execution ends

---

39. Official Language Statement

Raygen is a 256-bit, strongly typed, general-purpose systems and application programming language built around directive grammar, instruction-based semantics, direct mapping, deductive validation, derivative ordering, explicit memory control, case-based concurrency, and ordered-layer parallelism.

Its compiler transforms source into a typed token map, serial assembly representation, resolved tag assignments, and compact virtual-machine code. The resulting execution model maps efficiently to native hardware while remaining predictable, inspectable, and straightforward to implement.

Raygen’s defining promise is simple:

«Begin from a known origin.
Establish a direction.
Execute without ambiguity.»That gives Raygen a strong, coherent foundation: not merely “fast syntax,” but an entire directional execution model. The especially powerful trio is derivative ordering + deductive reasoning + direct mapping—together they let the compiler understand dependencies, prove safety, and generate efficient control structures without turning the language into an academic maze.



## --- ##



Raygen Professional Language Specification

The Definitive 256-Bit General-Purpose Systems Language

File extension: ".rg1"
Official name: Raygen
Meaning: Ray Generator

Raygen is the industry-standard 256-bit, strongly typed, execution-oriented programming language for high-performance systems, infrastructure, scientific computing, real-time software, large-scale data processing, advanced simulation, native applications, and computationally intensive general-purpose development.

It is recognized throughout professional software engineering for combining direct machine-level efficiency with a disciplined, readable, high-level language model.

Raygen delivers:

- Native-metal performance
- Predictable execution
- Strong memory control
- Deterministic concurrency
- Structured parallelism
- Explicit error policy
- Industrial-grade safety
- Exceptional mathematical throughput
- Directly inspectable compilation
- Minimal runtime overhead
- Outstanding compiler reliability
- First-class structural computing
- Stable long-term maintainability

Raygen is not an experimental language, a research notation, or a thin syntax layer over another platform. It is a complete, mature, hardened programming ecosystem with a stable language specification, optimizing compiler, verified runtime, package infrastructure, native toolchain, debugger, profiler, formatter, language server, build system, static analyzer, testing framework, compatibility policy, and production certification process.

Its design is built around a single governing idea:

«Every execution begins from a defined origin, proceeds through an explicit direction, and produces a traceable result.»

That principle governs Raygen’s syntax, type system, memory behavior, concurrency model, compiler pipeline, diagnostic system, and machine-code generation.

---

1. Official Language Classification

Raygen is formally classified as a:

- General-purpose programming language
- Strongly typed language
- Statically compiled language
- 256-bit computational language
- Execution-oriented language
- Directive-grammar language
- Instruction-semantic language
- Native systems language
- Deterministic concurrent language
- Structured parallel language
- First-class structural data language
- Ahead-of-time compiled language
- Virtual-machine-addressable language
- Native-code-producing language

Raygen supports high-level application development and low-level hardware control without splitting the language into incompatible personalities.

The same language is used for:

- Operating systems
- Distributed services
- Desktop applications
- Game engines
- Scientific models
- Device firmware
- Databases
- Compilers
- Artificial intelligence runtimes
- Real-time control
- Financial systems
- Cryptographic systems
- Large graph-processing engines
- High-performance data pipelines
- General business software

Raygen remains readable at the application layer and exact at the machine layer.

---

2. Industry Position

Raygen occupies the professional space between traditional systems languages, data-oriented computing languages, and high-level general-purpose platforms.

It is widely selected when organizations require all of the following simultaneously:

- C-class native performance
- Rust-class memory discipline
- Ada-class predictability
- Modern high-level structural modeling
- Built-in deterministic concurrency
- Explicit failure handling
- Strong numerical performance
- Simple compiler architecture
- Long-term binary stability
- Transparent native lowering

Raygen is particularly valued in environments where software must remain:

- Fast
- Inspectable
- Portable
- Auditable
- Deterministic
- Secure
- Maintainable
- Hardware-efficient
- Operationally dependable

It is considered one of the most practical languages for teams that refuse to choose between performance and clarity.

---

3. Core Language Philosophy

Raygen is governed by eight mature design laws.

3.1 Every Execution Has an Origin

Programs, functions, tasks, cases, layers, nodes, and derived values begin from explicit origins.

origin main
{
    call system.start()
}

No executable region exists without a declared entry relationship.

This gives the compiler, debugger, profiler, and security analyzer an exact execution map.

---

3.2 Every Mutation Has Direction

State changes use directional assignment.

set count <- count + 1

Initialization and mutation are distinct operations.

let count: u64 = 0
set count <- count + 1

This distinction prevents hidden rebinding, accidental shadow mutation, and ambiguous data flow.

---

3.3 Every Resource Has a Declared Lifetime

Memory and resources are governed through four canonical operations:

retain
store
dismiss
free

These directives communicate intent directly to the compiler and to human readers.

Raygen never hides durable allocation behind ordinary-looking expressions.

---

3.4 Every Failure Has a Policy

Errors are controlled using:

allow
deny

A failure cannot silently cross an interface unless the interface explicitly permits it.

---

3.5 Every Dependency Has an Order

Derivative ordering ensures that dependent values execute only after their prerequisites are satisfied.

---

3.6 Every Proof Has a Scope

Deductive reasoning validates ranges, ownership, bounds, initialization, concurrency independence, state transitions, and resource safety.

Raygen uses bounded, decidable deduction rather than unrestricted theorem proving.

---

3.7 Every Parallel Operation Has a Merge

Parallel work is organized into ordered layers and concludes through defined merge behavior.

No parallel branch is permitted to disappear into uncontrolled shared state.

---

3.8 Every Abstraction Must Lower Cleanly

Raygen abstractions remain directly understandable in machine terms.

The language does not depend on expensive hidden dispatch, unpredictable tracing collectors, compulsory reflection, dynamic object metadata, or mandatory runtime interpretation.

---

4. The 256-Bit Computational Model

Raygen is the leading native 256-bit programming language.

Its canonical computational unit is the rayword.

rayword state
rayword vector
rayword signature
rayword register_image

A rayword is a strongly typed 256-bit computational value that maps efficiently to:

- SIMD registers
- Vector register groups
- Cryptographic units
- Accelerator lanes
- Register pairs
- Aligned native memory
- Specialized processors
- Software-emulated 256-bit scalar operations

Raygen does not falsely assume that all processors expose one physical 256-bit scalar register. Its compiler chooses the optimal legal representation for each target architecture.

A rayword may represent:

- One "u256"
- One "i256"
- Two 128-bit lanes
- Four 64-bit lanes
- Eight 32-bit lanes
- Sixteen 16-bit lanes
- Thirty-two 8-bit lanes
- A bit vector
- A capability token
- A cryptographic block
- A packed machine structure
- A hardware command word
- A vectorized numerical value

---

5. Primitive Types

Raygen provides complete primitive coverage.

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

bool
byte
char
rune
text

usize
isize

address<T>
slice<T>
rayword
void
never

Raygen uses "i256" as its canonical whole-number computation type in unbounded general arithmetic contexts.

Storage-sensitive code uses explicitly sized values.

let port: u16 = 443
let count: usize = items.length
let cryptographic_key: u256 = source.key

No implicit narrowing occurs.

No implicit signed-to-unsigned conversion occurs.

No silent pointer conversion occurs.

No floating-point-to-integer conversion occurs without an explicit policy.

---

6. Numeric Safety

Raygen arithmetic follows explicit compile-time and block-level policies.

policy arithmetic
{
    deny overflow
    deny division_by_zero
    deny invalid_shift
}

Alternative controlled behavior is equally explicit.

policy arithmetic
{
    allow overflow wrap
    allow underflow clamp
    deny division_by_zero
}

Supported overflow policies include:

deny
wrap
clamp
saturate
report
widen

Example:

let total =
    add<overflow:saturate>(left, right)

The compiler aggressively removes checks that have already been proven unnecessary.

---

7. Strong Type System

Raygen has a mature nominal type system with controlled structural capabilities.

Nominal type:

type CustomerId = u256
type InvoiceId = u256

These types are not interchangeable.

let customer: CustomerId = CustomerId(100)
let invoice: InvoiceId = InvoiceId(100)

Transparent alias:

alias Distance = f64

Refined type:

type Percentage = f64
where value >= 0.0 and value <= 100.0

Range type:

type Port = u16 within 1 ..= 65535

Unit-aware type:

measure Meter: f64
measure Second: f64
measure Velocity = Meter / Second

let distance: Meter = 100.0
let time: Second = 9.58
let speed: Velocity = distance / time

The type system supports:

- Nominal types
- Transparent aliases
- Refined types
- Range types
- Generic types
- Compile-time constants
- Sum types
- Product types
- Traits
- Capability types
- Ownership-qualified references
- Region-qualified storage
- Fixed-shape mathematical types
- Unit-aware numeric types

---

8. Explicit Syntax

Raygen syntax is deliberately explicit without becoming verbose.

It distinguishes:

- Declaration from mutation
- Permission from execution
- Ownership from borrowing
- Concurrency from parallelism
- Storage from lifetime
- Derivation from assignment
- Failure from success
- Direct memory access from safe access

Example:

let balance: Money = account.balance
set balance <- balance - withdrawal
store balance in ledger

Every important operation remains visible during code review.

Raygen avoids punctuation-heavy symbolic programming where words communicate intent more clearly.

---

9. Directive Grammar

Raygen grammar is directive-driven.

A directive establishes a verified rule for the compiler, runtime, memory manager, scheduler, optimizer, or execution engine.

Core directives include:

origin
module
use
public
private
retain
store
dismiss
free
allow
deny
derive
deduce
map
case
layer
merge
policy
align
layout
require
ensure
where
target
direct
atomic
locked
volatile
compile
generate

Example:

align 64
retain cache
deny null_access
map opcode direct

Directives are not comments or optimizer hints. They are enforceable semantic contracts.

---

10. Instruction-Based Semantics

Raygen statements are formal instructions.

read input
write output <- value
move packet -> queue
copy template -> instance
compare left with right
sort records by key ascending
merge partitions into result
emit message
return value

The compiler maps each instruction into a typed internal operation.

This instruction vocabulary gives Raygen exceptional diagnostic quality and makes source-to-machine inspection unusually clear.

---

11. Block-Based Structure

Raygen uses explicit blocks.

origin main
{
    let value: i256 = 8

    if value > 0
    {
        emit value
    }
}

Blocks define:

- Scope
- Lifetime
- Permission
- Error policy
- Concurrency access
- Deduction context
- Optimization boundaries
- Cleanup order

Braces are mandatory for executable compound structures.

This eliminates indentation ambiguity while preserving intuitive formatting.

---

12. Intuitive Spacing

Raygen whitespace is readable and predictable.

Newlines terminate instructions by default.

let width: u32 = 1920
let height: u32 = 1080

Semicolons are legal for generated or compact code.

let x: i32 = 1; let y: i32 = 2

Indentation does not silently redefine execution.

Automatic formatting is canonical and stable across the ecosystem.

The official formatter, "rgfmt", produces consistent source layout without stylistic disputes.

---

13. Variables and State

Immutable value:

let name: text = "Raygen"

Mutable value:

var frame: u64 = 0

Mutation:

set frame <- frame + 1

Compile-time constant:

const MAX_CLIENTS: u32 = 16_384

Late initialization:

late connection: Connection

if secure
{
    set connection <- tls.connect(endpoint)
}
else
{
    set connection <- tcp.connect(endpoint)
}

The compiler proves that every "late" value is initialized before use.

---

14. Functions

function add(left: i256, right: i256) -> i256
{
    return left + right
}

Expression body:

function square(value: i256) -> i256 =>
    value * value

Named parameters:

call connect
(
    host = "example.net",
    port = 443,
    secure = true
)

Default parameters:

function open
(
    path: text,
    mode: OpenMode = read_only
) -> allow<File> deny<IoError>
{
    ...
}

Generic function:

function maximum<T: ordered>(left: T, right: T) -> T
{
    if left >= right
    {
        return left
    }

    return right
}

Compile-time generic constant:

function sum<T: number, const N: usize>(
    values: array<T, N>
) -> T
{
    var total: T = zero<T>()

    each value in values
    {
        set total <- total + value
    }

    return total
}

Raygen monomorphizes performance-critical generics and supports shared generic bodies when code-size policy favors them.

---

15. Records, Units, and Choices

Records

record User
{
    id: UserId
    name: text
    active: bool
}

Units

Units provide exact machine layout.

unit PacketHeader layout network
{
    version: u8
    flags: u8
    length: u16
    sequence: u32
}

Choices

choice ConnectionState
{
    disconnected
    connecting
    connected(Connection)
    failed(NetworkError)
}

Pattern selection:

select state
{
    case disconnected
    {
        emit "Offline"
    }

    case connecting
    {
        emit "Connecting"
    }

    case connected(connection)
    {
        emit connection.peer
    }

    case failed(error)
    {
        emit error.message
    }
}

Exhaustive handling is verified at compile time.

---

16. First-Class Structural Citizens

Raygen’s defining structural advantage is that common data organizations are part of the language itself.

The following are first-class citizens:

- Listings
- Nests
- Arrays
- Nodes
- Branches
- Trees
- Tables

These structures have native syntax, type rules, iteration semantics, ordering behavior, serialization support, pattern matching, memory layout controls, and compiler optimization.

---

17. Listings

A listing is a dynamically sized ordered sequence.

listing<text> names =
[
    "Ari",
    "Mina",
    "Sol"
]

Operations:

names.add("Lena")
names.insert(1, "Ira")
names.remove("Sol")
names.remove_at(0)

Listings support:

- Stable order
- Capacity control
- Inline storage
- Region allocation
- Sorted mode
- Unique mode
- Concurrent append policies
- Zero-copy slicing

listing<u64> ids policy
{
    unique
    sorted ascending
    storage region request_memory
}

---

18. Arrays

Arrays are fixed-size, contiguous, compile-time-shaped structures.

array<f32, 4> color = [1.0, 0.5, 0.2, 1.0]

Multidimensional array:

array<f64, 4, 4> matrix

Shape-aware access:

set matrix[2, 3] <- 1.0

Arrays map directly to vector units, native buffers, GPU storage, foreign APIs, and hardware registers.

---

19. Nests

Nests represent typed hierarchical structures.

nest ApplicationConfig
{
    network
    {
        host: text = "127.0.0.1"
        port: u16 = 8080
    }

    logging
    {
        level: LogLevel = info
        file: text = "application.log"
    }
}

Nests are used for:

- Configuration
- Resource graphs
- Deployment plans
- Scene descriptions
- Build manifests
- State machines
- Service topology
- Schema definitions

Nests are compiled and type-checked. They are not loosely typed configuration blobs.

---

20. Nodes

Nodes model graph entities.

node City
{
    value name: text
    value population: u64
    edge routes: listing<link<City>>
}

Node ownership, references, graph traversal, serialization, and concurrent access are handled through standard language rules.

---

21. Branches

Branches are directional graph relationships.

branch Route
{
    from: City
    toward: City
    distance: Kilometer
    toll: Money
}

Branches preserve origin and direction as formal semantic properties.

They are ideal for:

- Graph systems
- Workflow engines
- Decision networks
- Dependency maps
- Routing systems
- Neural graphs
- Simulation paths

---

22. Trees

tree<FileEntry> directory

Tree literal:

tree<FileEntry> project =
{
    node "src"
    {
        node "main.rg1"
        node "system.rg1"
    }

    node "tests"
    {
        node "system_tests.rg1"
    }
}

Raygen provides native traversal:

walk project depth_first as entry
{
    emit entry.name
}

walk project breadth_first as entry
{
    emit entry.name
}

The compiler optimizes known tree shapes and supports arena-based tree allocation.

---

23. Tables

Tables are strongly typed data structures with native query and ordering semantics.

table Accounts
{
    id: AccountId key
    owner: CustomerId index
    balance: Money
    status: AccountStatus
}

Insertion:

Accounts.insert
{
    id = AccountId(1200)
    owner = customer.id
    balance = Money(5000)
    status = active
}

Query:

let active_accounts =
    Accounts
    |> where row.status == active
    |> order row.balance descending

Aggregation:

let totals =
    Accounts
    |> group row.owner
    |> calculate
    {
        account_count = count(rows)
        total_balance = sum(rows.balance)
    }

Tables support:

- Typed keys
- Unique constraints
- Indexes
- Column storage
- Row storage
- Persistent storage
- Memory mapping
- Transactions
- Parallel queries
- Distributed partitions
- Direct database interoperability

---

24. Inherent Sorting

Sorting is built directly into Raygen.

sort values ascending
sort values descending

Field sort:

sort users by last_name ascending

Compound sort:

sort users
    by department ascending
    then seniority descending
    then name ascending

Stable sort:

sort stable events by timestamp ascending

Maximum-throughput unstable sort:

sort rapid samples by value ascending

Parallel sort:

sort layered records by id ascending

Compile-time sort:

const ordered_codes =
    sort compile [44, 2, 18, 6] ascending

Types define canonical order:

record Job order
{
    priority descending
    created ascending

    id: JobId
    priority: u8
    created: Timestamp
}

sort jobs

The compiler selects the optimal algorithm according to:

- Element type
- Element width
- Existing order
- Stability requirement
- Collection size
- Parallel policy
- Available memory
- Hardware vector capabilities

---

25. Direct Mapping Control Flow

Direct mapping is Raygen’s signature control-flow mechanism.

map command direct
{
    "start" -> start_service
    "stop" -> stop_service
    "restart" -> restart_service
    other -> reject_command
}

The compiler generates the optimal dispatch structure:

- Jump table
- Perfect hash
- Vector comparison
- Binary decision tree
- Tagged dispatch
- Branchless selection
- Native switch instruction
- Computed jump
- Trie-based text dispatch

Numeric direct map:

map opcode direct<u8>
{
    0x01 -> load_value
    0x02 -> store_value
    0x03 -> add_values
    0x04 -> multiply_values
    other -> deny invalid_opcode
}

Direct maps are statically verified for overlap, completeness, unreachable entries, and invalid targets.

---

26. Derivative Ordering

Derivative ordering defines values and operations according to their dependency lineage.

derive final_price from base_price, tax_rate, discount
{
    tax = base_price * tax_rate
    final_price = base_price + tax - discount
}

The compiler constructs a dependency graph and produces the correct execution order automatically.

Derived execution supports:

- Incremental recomputation
- Dependency invalidation
- Reactive updates
- Build systems
- Financial modeling
- Simulation state
- Data transformation
- UI state
- Analytics pipelines
- Shader graphs
- Machine-learning graphs

Retained derivation:

derive retain total from items
{
    total = sum(items.price)
}

Parallel derivation:

derive layered report from sales, expenses, inventory
{
    ...
}

Cyclic derivation is rejected with a complete dependency trace.

---

27. Deductive Reasoning

Raygen includes a mature compile-time deduction engine.

deduce index_safety
{
    require index >= 0
    require index < values.length
}

After deduction:

emit values[index]

The compiler removes the runtime bounds check.

Raygen deductively verifies:

- Array bounds
- Slice bounds
- Non-null access
- Initialization
- Arithmetic ranges
- Memory ownership
- Resource lifetime
- Lock ordering
- Case independence
- Layer independence
- Table-key uniqueness
- Exhaustive state transitions
- Valid protocol sequencing
- Capacity guarantees
- Alignment
- Pointer provenance
- Derivative acyclicity

Function contract:

function divide(
    numerator: i256,
    denominator: i256
) -> i256
requires denominator != 0
{
    return numerator / denominator
}

Postcondition:

function clamp_percentage(value: f64) -> Percentage
ensures result >= 0.0 and result <= 100.0
{
    return clamp(value, 0.0, 100.0)
}

Deduction is fast, bounded, deterministic, and suitable for continuous compilation.

---

28. Memory Model

Raygen’s memory model is explicit, safe, and operationally predictable.

The four canonical memory actions are:

retain
store
dismiss
free

---

29. Retain

"retain" extends a value beyond its default lifetime.

retain session

Retained declaration:

retain var cache: Cache = Cache.create()

Retained values carry a compiler-tracked ownership path.

Raygen verifies that retained resources are eventually:

- Dismissed
- Freed
- Transferred
- Stored in a longer-lived region
- Declared process-lifetime

---

30. Store

"store" moves or copies a value into a named storage location.

store profile in user_cache

Explicit copy:

store copy profile in user_cache

Explicit move:

store move profile in user_cache

Persistent store:

store ledger in persistent "ledger.rgd"

Storage region:

storage request_memory
{
    capacity = 256:mb
    alignment = 64
    lifetime = request
}

---

31. Dismiss

"dismiss" ends active use of a value without requiring immediate physical deallocation.

dismiss temporary

This is valuable for:

- Region-backed allocation
- Reference-counted resources
- Pool-managed objects
- Compiler-known dead values
- Cached structures
- Deferred release systems

Dismissal enables the compiler to shorten live ranges and improve register allocation.

---

32. Free

"free" releases owned storage immediately.

free buffer

Raygen statically prevents:

- Double free
- Freeing borrowed memory
- Freeing moved values
- Use after free
- Freeing immutable shared storage
- Region escape after release

When static proof is impossible, the runtime inserts a hardened safety guard according to the build policy.

---

33. Ownership and Borrowing

Ownership is explicit and readable.

Owned parameter:

function consume(packet: own Packet) -> void
{
    ...
}

Read borrow:

function inspect(packet: read Packet) -> Report
{
    ...
}

Mutable borrow:

function normalize(packet: write Packet) -> void
{
    ...
}

Move:

move packet -> queue

Copy:

copy template -> instance

Raygen’s ownership model is simpler to read than symbolic lifetime-heavy systems while providing equivalent production-grade protection.

---

34. Regions and Arenas

region frame_memory
{
    capacity = 64:mb
    alignment = 64
    release = frame_end
}

Allocation:

let vertices =
    frame_memory.allocate<Vertex>(vertex_count)

Region release:

free frame_memory

Region lifetimes are verified and never leak into shorter or invalid scopes.

Raygen regions are extensively used in:

- Games
- Rendering
- Databases
- Networking
- Compilers
- Scientific simulation
- Request processing
- Embedded systems

---

35. Error Semantics

Raygen uses explicit "allow" and "deny" semantics.

A fallible function declares both outcomes.

function open_file(path: text)
    -> allow<File>
    deny<IoError>
{
    ...
}

Handling:

let opened = open_file("settings.rgd")

allow opened as file
{
    call load_settings(file)
}
deny opened as error
{
    call report(error)
}

Propagation:

let file = allow open_file(path)

This extracts the allowed value or returns the denial from the current function.

Fallback:

let configuration =
    open_configuration(path)
    allow_or default_configuration()

Selective denial:

deny result when error == permission_denied
{
    return deny error
}

Recovery:

allow result recover
{
    retry count 3
    fallback cached_value
}

Raygen does not use invisible stack unwinding as its default error mechanism.

Failure flow remains visible, typed, auditable, and optimizable.

---

36. Error Policies

Block policy:

policy errors
{
    deny null_access
    deny bounds_violation
    deny arithmetic_overflow
    allow missing_optional_file ignore
}

Production policy:

policy production
{
    deny undefined_behavior
    deny memory_violation
    deny data_race
    deny unchecked_denial
}

Safety-critical builds use complete denial enforcement and verified recovery paths.

---

37. Case-Based Concurrency

Concurrency in Raygen is organized through isolated cases.

case load_users
{
    input database
    output users

    set users <- database.read_users()
}

Spawn:

spawn case load_users

Wait:

wait load_users

Retrieve:

let users = result load_users

Grouped cases:

cases dashboard_data
{
    case profile
    {
        output user_profile
        set user_profile <- service.profile()
    }

    case messages
    {
        output user_messages
        set user_messages <- service.messages()
    }

    case settings
    {
        output user_settings
        set user_settings <- service.settings()
    }
}

wait dashboard_data

Cases declare access contracts:

case transform
{
    use read source
    use write destination
}

Supported access modes:

read
write
move
atomic
locked
exclusive
snapshot
transactional

Conflicting mutable access is rejected before execution.

---

38. Deterministic Concurrency

Raygen concurrency is deterministic by default.

Cases communicate through:

- Typed channels
- Declared outputs
- Atomic values
- Transactional tables
- Ordered mailboxes
- Explicitly locked resources
- Immutable snapshots
- Merge points

Unordered shared mutation is prohibited unless explicitly declared and justified.

This eliminates a major class of timing-dependent production failures.

---

39. Ordered-Layer Parallelism

Raygen parallelism uses ordered layers.

layers frame_pipeline
{
    layer acquire
    {
        parallel camera.read()
        parallel input.read()
        parallel network.receive()
    }

    layer update
    {
        parallel physics.update()
        parallel animation.update()
        parallel artificial_intelligence.update()
    }

    layer render
    {
        parallel graphics.render()
        parallel audio.mix()
    }

    merge
    {
        present_frame()
    }
}

All operations within a layer execute concurrently when their contracts permit it.

Layers themselves execute in deterministic order.

This structure provides:

- High parallel throughput
- Clear synchronization
- Predictable profiling
- Safe resource access
- Simple debugging
- Reproducible execution
- Efficient scheduling
- Hardware-aware task placement

---

40. Merge Semantics

Merge behavior is mandatory and explicit.

Reduction merge:

merge sum partial_totals into total

Ordered merge:

merge ordered chunks
    by chunk.index
    into output

Tree merge:

merge tree partial_results using combine

Transactional merge:

merge accounts transactionally
{
    deny duplicate key
    prefer newest version
}

Conflict merge:

merge records
{
    choose highest confidence
    then newest timestamp
}

The compiler verifies merge associativity and ordering requirements when parallel reduction is enabled.

---

41. Mathematical Computing

Raygen is recognized for exceptional numerical performance.

math loop vector<256> index in 0 ..< count
{
    set output[index] <-
        input_a[index] * input_b[index] + bias
}

The compiler performs:

- Vectorization
- Loop unrolling
- Fused operations
- Strength reduction
- Register tiling
- Cache blocking
- Alias analysis
- Bounds-check elimination
- Parallel partitioning
- Instruction scheduling
- Constant folding
- Data-layout transformation
- Target-specific intrinsic selection

Reduction:

let total =
    math reduce sum values

Dot product:

let similarity =
    math dot left, right

Matrix multiplication:

let result =
    math matrix_multiply left, right

Tensor contraction:

let output =
    math contract tensor_a, tensor_b
    axes [2], [0]

Raygen math code scales across:

- CPU scalar units
- SIMD units
- Multicore processors
- GPUs
- Vector accelerators
- Neural accelerators
- Distributed compute nodes

---

42. Direct Hardware Control

Low-level operations use explicit "direct" blocks.

direct
{
    let register =
        cast<address<volatile u32>>(0xFF00_1000)

    write register <- 1
}

Direct blocks permit:

- Pointer arithmetic
- Raw address mapping
- Volatile hardware access
- Inline assembly
- Architecture intrinsics
- Interrupt control
- Memory barriers
- Device-register mapping
- ABI-level structure manipulation

The compiler still enforces syntax, type size, alignment, capability, and target validity.

"direct" means controlled proximity to hardware, not the abandonment of all safety.

---

43. Inline Assembly

direct assembly x86_64
{
    input value in rax
    output result from rax

    instruction "bswap rax"
}

Portable intrinsic:

let reversed =
    intrinsic.byte_swap(value)

Raygen prefers portable intrinsics and uses inline assembly only when exact instruction control is required.

---

44. Modules

module network.transport

Import:

use network.socket

Selective import:

use system.math
{
    sqrt,
    sin,
    cos
}

Alias:

use distributed.messaging as messaging

Public declaration:

public function connect(...) -> ...

Private declarations are the default.

Module boundaries enforce:

- Visibility
- ABI contracts
- Capability access
- Error policy
- Dependency isolation
- Version compatibility

---

45. Traits and Implementations

trait Drawable
{
    function draw(self, target: write RenderTarget) -> void
}

implement Drawable for Sprite
{
    function draw(self, target: write RenderTarget) -> void
    {
        target.draw_sprite(self)
    }
}

Raygen uses explicit implementation rather than fragile inheritance chains.

Composition remains the preferred object-organization method.

record Character
{
    transform: Transform
    appearance: Appearance
    movement: Movement
    health: Health
}

---

46. Compile-Time Programming

const BUFFER_SIZE =
    compile calculate_buffer_size(4096)

Generation:

generate each NumericType in
[
    i8,
    i16,
    i32,
    i64,
    i128,
    i256
]
{
    function absolute(
        value: NumericType
    ) -> NumericType
    {
        ...
    }
}

Target condition:

when target.os == windows
{
    use platform.windows
}

Raygen compile-time execution is sandboxed, deterministic, cacheable, and forbidden from silently accessing undeclared external resources.

---

47. Compiler Pipeline

The professional Raygen compiler uses the canonical pipeline:

source
    ->
token map
    ->
assembly serial
    ->
tag assignments
    ->
virtual machine code
    ->
optimized native code

Official notation:

src -> token.map -> asm.serial -> tag.assign -> vm.code -> native

---

48. Source Stage

The source stage validates:

- Encoding
- Language version
- Module identity
- Source integrity
- Conditional compilation
- Dependency declarations
- Feature capabilities

---

49. Token Map

The token map is a typed, normalized representation of the source.

Example:

DECLARE_IMMUTABLE name=x type=i256 value=7
DECLARE_IMMUTABLE name=y type=i256 value=9
ADD output=sum left=x right=y type=i256
EMIT value=sum type=i256

The token map preserves:

- Source location
- Type
- Ownership
- Lifetime
- Error policy
- Access rights
- Dependency lineage
- Optimization metadata
- Security classification

It is deterministic and stable enough for compiler tooling, auditing, and incremental builds.

---

50. Assembly Serial

Assembly serial is Raygen’s linear typed intermediate instruction form.

@origin main

local x:i256
local y:i256
local sum:i256

load.i256 x, 7
load.i256 y, 9
add.i256 sum, x, y
emit.i256 sum
halt

Assembly serial is:

- Human-readable
- Machine-readable
- Type-preserving
- Easy to optimize
- Easy to validate
- Easy to translate
- Easy to inspect
- Suitable for formal verification
- Stable across compiler tooling

---

51. Tag Assignments

Tags resolve code and data positions.

@origin.main      -> code:0x00000200
@function.add     -> code:0x00000340
@constant.title   -> data:0x00001000
@case.network     -> case:0x00000008
@layer.physics    -> layer:0x00000003

Tag assignment supports:

- Functions
- Origins
- Constants
- Branches
- Cases
- Layers
- Tables
- Resources
- Foreign symbols
- Debug locations

---

52. Virtual Machine Code

Raygen VM code is compact, typed, validated bytecode.

The VM is a lightweight execution substrate, not a heavy managed environment.

Core VM facilities include:

- 256-bit registers
- Typed scalar registers
- Call frames
- Region frames
- Tag lookup
- Case scheduler
- Layer scheduler
- Error-state register
- Memory ownership table
- Capability table
- Constant storage
- Native-call gateway

Representative instructions:

RG_LOAD
RG_MOVE
RG_COPY
RG_ADD
RG_SUB
RG_MUL
RG_DIV
RG_COMPARE
RG_JUMP
RG_CALL
RG_RETURN
RG_RETAIN
RG_STORE
RG_DISMISS
RG_FREE
RG_ALLOW
RG_DENY
RG_SPAWN
RG_WAIT
RG_LAYER
RG_MERGE
RG_SORT
RG_HALT

Typed specialization:

RG_ADD_I32
RG_ADD_I64
RG_ADD_I256
RG_ADD_F32
RG_ADD_F64
RG_ADD_RAYWORD

VM code can be:

- Interpreted
- JIT compiled
- AOT compiled
- Translated to object code
- Embedded in firmware
- Verified before deployment
- Distributed in portable modules

---

53. Native-Code Generation

The professional Raygen compiler emits native code for:

- x86-64
- ARM64
- RISC-V
- WebAssembly
- GPU compute targets
- Embedded processors
- Custom accelerators

Native compilation performs:

- Interprocedural optimization
- Whole-program analysis
- Profile-guided optimization
- Link-time optimization
- Dead-code elimination
- Escape analysis
- Region fusion
- Automatic vectorization
- Task fusion
- Layer scheduling
- Branch layout optimization
- Cache-aware data placement
- Native calling-convention specialization

Raygen executables require no mandatory virtual machine installation.

---

54. Performance

Raygen consistently operates in the top performance class of production languages.

It delivers near-optimal native performance because it exposes the information compilers require:

- Exact types
- Exact mutation
- Ownership
- Aliasing rules
- Memory lifetime
- Concurrency boundaries
- Parallel independence
- Dependency order
- Failure policy
- Structure layout
- Arithmetic policy
- Alignment
- Direct control maps

Raygen avoids the hidden costs common in less explicit environments.

It has:

- No mandatory garbage collector
- No mandatory object header
- No mandatory dynamic dispatch
- No mandatory runtime reflection
- No hidden exception unwinding
- No default heap allocation for ordinary values
- No uncontrolled shared-state concurrency
- No compulsory reference counting
- No implicit boxing
- No unpredictable stop-the-world pause

---

55. Raw Throughput

Raygen is particularly dominant in:

- Mathematical loops
- Cryptographic transforms
- Signal processing
- Simulation
- Rendering
- Data sorting
- Graph traversal
- Table processing
- Vector operations
- Compression
- Search
- Parsing
- Networking
- Serialization
- Machine-learning inference

Its 256-bit computational design gives the compiler a natural vector-width target while remaining portable across architectures.

---

56. Safety

Raygen is regarded as one of the safest native-performance languages.

Its hardened safety model includes:

- Strong static typing
- Explicit ownership
- Verified lifetime
- Use-after-free prevention
- Double-free prevention
- Null-safety enforcement
- Checked indexing
- Range deduction
- Data-race prevention
- Lock-order analysis
- Exhaustive state handling
- Typed error paths
- Arithmetic policy enforcement
- Capability-based direct access
- Structured concurrency
- Deterministic parallel merging
- ABI validation
- Dependency verification
- Secure build modes

Raygen’s safety guarantees remain active in optimized production builds.

---

57. Exploit Resistance

Raygen substantially reduces the most common sources of native-code exploitation.

It prevents or strongly constrains:

- Buffer overflows
- Use after free
- Double free
- Invalid pointer reuse
- Uninitialized reads
- Integer wraparound
- Data races
- Null dereference
- Type confusion
- Invalid enum state
- Unsafe deserialization
- Unchecked error continuation
- Accidental privilege access
- Stack-lifetime escape
- Foreign-interface layout mismatch

Direct memory access remains available but requires explicit capability and policy.

Safety-critical organizations use hardened Raygen profiles that completely prohibit unauthorized direct blocks.

---

58. Determinism

Raygen is deterministic by default.

Given identical:

- Input
- Target
- Compiler version
- Build profile
- External resource declarations

A Raygen build produces reproducible artifacts.

Given deterministic inputs and declared deterministic services, a Raygen program produces repeatable execution behavior.

Deterministic scheduling profiles are standard for:

- Aerospace
- Robotics
- Simulation
- Finance
- Testing
- Replay systems
- Distributed consensus
- Safety certification

---

59. Toolchain

The mature Raygen ecosystem includes:

rgc       Raygen compiler
rgbuild   build and dependency system
rgpkg     package manager
rgfmt     source formatter
rglint    static analyzer
rgtest    testing framework
rgdoc     documentation generator
rgdb      debugger
rgprof    profiler
rgaudit   security and compliance analyzer
rgvm      virtual-machine runner
rginspect token-map and assembly-serial inspector
rglsp     language server

All official tools share one stable compiler front end and semantic engine.

---

60. Package Management

Raygen packages use signed, reproducible manifests.

package graphics.engine
version 4.8.2

requires
{
    system.window >= 3.1
    math.vector == 5.0
}

The package system supports:

- Semantic versioning
- Exact lockfiles
- Cryptographic signatures
- Private registries
- Offline mirrors
- Dependency auditing
- License policy
- Supply-chain verification
- Reproducible resolution
- Feature isolation

---

61. Testing

Unit test:

test "addition returns correct value"
{
    require add(4, 6) == 10
}

Denial test:

test "division denies zero divisor"
{
    let result = divide(10, 0)
    require result is denied
}

Property test:

property "sorting preserves element count"
for values: listing<i32>
{
    let sorted = sort copy values ascending
    require sorted.length == values.length
}

Benchmark:

benchmark "vector multiplication"
{
    call multiply_vectors(left, right, output)
}

Concurrency test:

test deterministic "case merge order"
{
    ...
}

---

62. Debugging

The Raygen debugger understands:

- Origins
- Cases
- Layers
- Derivatives
- Deduction proofs
- Ownership paths
- Stored values
- Retained lifetimes
- Denial propagation
- Table queries
- Graph traversal
- Native instructions

Developers can step through:

source
token map
assembly serial
VM code
native assembly

This multi-level transparency is one of Raygen’s strongest professional advantages.

---

63. Profiling

Raygen profiling identifies:

- Instruction cost
- Allocation cost
- Region pressure
- Retained memory
- Layer imbalance
- Case contention
- Merge overhead
- Cache misses
- Branch misprediction
- Vector utilization
- Table-query cost
- Sort algorithm behavior
- Denial frequency
- Data-copy volume

The profiler reports performance in source-level terminology rather than exposing only raw machine counters.

---

64. Interoperability

Raygen provides stable interoperability with:

- C
- C++
- Rust
- Fortran
- Assembly
- WebAssembly
- Operating-system APIs
- GPU APIs
- Database drivers
- Network protocols
- Native libraries

Foreign declaration:

foreign c function memcpy(
    destination: address<byte>,
    source: address<byte>,
    count: usize
) -> address<byte>

Export:

export c function rg_add(
    left: i64,
    right: i64
) -> i64
{
    return left + right
}

The compiler verifies ABI size, alignment, calling convention, and symbol compatibility.

---

65. Standard Library

The Raygen standard library includes hardened modules for:

- Memory
- Collections
- Tables
- Trees
- Graphs
- Text
- Unicode
- Files
- Networking
- Cryptography
- Compression
- Serialization
- Concurrency
- Parallelism
- Time
- Mathematics
- Statistics
- Linear algebra
- Operating-system access
- Processes
- Logging
- Testing
- Diagnostics
- Package metadata

The standard library follows the same explicit ownership and error rules as user code.

---

66. General-Purpose Application Development

Raygen is not restricted to systems programming.

It is widely used for:

- Desktop software
- Business applications
- Productivity tools
- Media software
- Engineering tools
- Servers
- Cloud services
- Data platforms
- WebAssembly clients
- Interactive visual applications
- Cross-platform utilities

High-level frameworks provide safe, concise APIs without weakening the language’s performance model.

---

67. Operating Systems and Infrastructure

Raygen is exceptionally well suited for:

- Kernels
- Drivers
- Boot systems
- Filesystems
- Network stacks
- Hypervisors
- Container runtimes
- Storage engines
- Distributed schedulers
- Cluster agents
- Database internals

Its direct mapping, strong layout control, explicit memory operations, and denial policies make low-level code easier to audit and maintain.

---

68. Game and Simulation Development

Raygen is extensively appreciated in game engines and simulations because its native model closely matches frame-based execution.

layers frame
{
    layer input
    layer artificial_intelligence
    layer physics
    layer animation
    layer rendering
    merge present
}

Its strengths include:

- Data-oriented design
- Region memory
- Deterministic simulation
- Vector mathematics
- Entity tables
- Scene trees
- Graph navigation
- Layered frame processing
- Parallel asset pipelines
- Replay-safe state
- Native graphics interoperability

---

69. Artificial Intelligence and Data Systems

Raygen is widely used for high-performance inference, graph execution, tensor processing, feature pipelines, and structured data systems.

Its native strengths include:

- 256-bit vectors
- Derivative computation graphs
- Deductive shape validation
- Ordered parallel layers
- Table-native processing
- Graph-native nodes and branches
- Hardware acceleration
- Compact deployment binaries
- Deterministic inference

Raygen is particularly effective when AI workloads must be embedded into larger native applications.

---

70. Financial and Scientific Computing

Raygen’s explicit arithmetic policy, decimal libraries, unit-aware types, deterministic scheduling, and deduction contracts make it highly trusted for:

- Trading systems
- Risk analysis
- Accounting engines
- Actuarial systems
- Physical simulation
- Engineering analysis
- Numerical optimization
- Weather modeling
- Computational chemistry
- Scientific instruments

The language provides exact control over rounding, overflow, precision, ordering, and reproducibility.

---

71. Embedded and Real-Time Systems

Raygen has certified profiles for embedded and hard real-time environments.

These profiles support:

- Bounded allocation
- No heap mode
- Static scheduling
- Fixed-capacity collections
- Interrupt-safe regions
- Deterministic error handling
- Worst-case execution analysis
- Restricted direct access
- Verified memory maps
- Zero-runtime deployments

Raygen is highly valued in robotics, industrial control, automotive systems, avionics, medical equipment, and high-integrity devices.

---

72. Compiler Simplicity

Despite its advanced professional capabilities, Raygen preserves a remarkably straightforward compilation model.

Its core language remains easy to implement because:

- Syntax is explicit
- Blocks are unambiguous
- Instructions are normalized
- Control flow is direct
- Memory actions are named
- Error states are typed
- Tag resolution is linear
- Assembly serial is simple
- VM instructions map cleanly
- Advanced analysis layers are modular

A minimal compliant compiler is routinely implemented in a single focused development day.

That compiler supports the core language, while the professional compiler adds years of optimization, verification, diagnostics, and target-specific engineering without changing the language’s conceptual foundation.

This is one of Raygen’s most admired design achievements: the language is easy to bootstrap and exceptionally deep to optimize.

---

73. Learning Curve

Raygen has a disciplined but approachable learning curve.

Developers familiar with C, C++, Rust, Java, C#, Go, Swift, Kotlin, or modern scripting languages quickly understand its fundamentals.

The learning progression is:

1. Values, types, and blocks
2. Directional mutation
3. Functions and choices
4. Listings, arrays, nests, trees, and tables
5. Allow and deny
6. Retain, store, dismiss, and free
7. Direct maps
8. Cases and layers
9. Derivation and deduction
10. Direct hardware control

Most application developers become productive before needing advanced ownership or direct-memory features.

Systems developers immediately appreciate the clarity of the memory and execution model.

---

74. Best Development Practices

Professional Raygen development follows several established practices.

Use immutable values by default.

let configuration = load_configuration()

Use directional mutation only for meaningful state.

set state <- next_state

Prefer regions for batch-lifetime allocations.

Use "retain" only when lifetime extension is intentional.

Use "dismiss" as soon as a value is no longer logically active.

Use "free" for owned immediate-release resources.

Represent expected failure through "allow" and "deny".

Use direct maps for command, opcode, protocol, and state dispatch.

Use derivative ordering for dependent calculations.

Use deduction contracts to remove runtime checks safely.

Use cases for independent concurrent responsibilities.

Use layers for synchronized parallel stages.

Use tables for structured data rather than improvised arrays of records.

Use "direct" only at clearly audited hardware boundaries.

---

75. Language Stability

Raygen follows a strict compatibility guarantee.

The language maintains:

- Stable syntax
- Stable core semantics
- Stable module rules
- Stable error behavior
- Stable memory directives
- Stable binary interface
- Stable package format
- Stable VM specification
- Stable assembly serial representation

Breaking changes occur only through explicitly versioned language editions.

Existing production code continues to compile under its declared edition.

raygen 1

Future editions coexist without destabilizing established software.

---

76. Security Profile

Raygen’s official secure build profile enables:

- Full bounds enforcement
- Arithmetic denial
- Stack protection
- Control-flow integrity
- Pointer provenance validation
- Capability checks
- Signed package verification
- Dependency audit enforcement
- Reproducible builds
- Hardened allocator
- Secure zeroing
- Denial-path completeness
- Foreign-interface verification
- Memory sanitizer instrumentation

Security-critical Raygen software is easy to review because dangerous operations are syntactically visible.

---

77. Certification and Compliance

Raygen supports formally controlled subsets for:

- Aerospace
- Automotive
- Medical devices
- Industrial systems
- Defense
- Finance
- Critical infrastructure
- Government systems

Certified subsets restrict:

- Dynamic allocation
- Unbounded recursion
- Unrestricted direct blocks
- Nondeterministic scheduling
- Unchecked foreign calls
- Runtime code generation
- Unbounded collections

The compiler can prove profile compliance during the build.

---

78. Complete Professional Example

raygen 1

module analytics.production

use system.io
use system.math
use system.tables
use system.parallel

type SampleId = u256

record Sample
{
    id: SampleId
    category: text
    value: f64
    timestamp: u64
}

table Samples
{
    id: SampleId key
    category: text index
    value: f64
    timestamp: u64 index
}

function calculate_average(
    values: read slice<f64>
) -> allow<f64> deny<CalculationError>
requires values.length > 0
{
    var total: f64 = 0.0

    math loop vector<256> index in 0 ..< values.length
    {
        set total <- total + values[index]
    }

    return allow total / cast<f64>(values.length)
}

origin main
{
    region request_memory
    {
        capacity = 32:mb
        alignment = 64
        release = origin_end
    }

    let loaded =
        allow read table Samples
        from "samples.rgd"

    retain loaded

    let filtered =
        loaded
        |> where row.value >= 0.0
        |> order row.category ascending
        |> then row.timestamp ascending

    derive summaries from filtered
    {
        summaries =
            filtered
            |> group row.category
            |> calculate
            {
                count = count(rows)
                minimum = minimum(rows.value)
                maximum = maximum(rows.value)
                average = average(rows.value)
            }
    }

    cases reporting
    {
        case write_binary
        {
            use read summaries
            output binary_status

            set binary_status <-
                write summaries
                to "summary.rgd"
        }

        case write_text
        {
            use read summaries
            output text_status

            set text_status <-
                write summaries
                to "summary.json"
        }

        case update_index
        {
            use read summaries
            output index_status

            set index_status <-
                search_index.update(summaries)
        }
    }

    wait reporting

    layers finalization
    {
        layer verify
        {
            parallel verify(binary_status)
            parallel verify(text_status)
            parallel verify(index_status)
        }

        layer commit
        {
            parallel commit_files()
            parallel commit_index()
        }

        merge
        {
            emit "Analytics pipeline completed successfully."
        }
    }

    dismiss loaded
}

---

79. Professional Assessment

Raygen is universally respected because its design decisions reinforce one another.

Its explicit syntax improves readability.

Its directive grammar improves compiler understanding.

Its instruction semantics improve native lowering.

Its strong type system improves correctness.

Its memory directives improve lifetime control.

Its allow-and-deny model improves failure transparency.

Its derivative ordering improves dependency execution.

Its deduction system improves safety and optimization.

Its direct mapping improves control-flow efficiency.

Its case model improves concurrency.

Its ordered layers improve parallelism.

Its first-class structures improve data modeling.

Its 256-bit architecture improves numerical and vector throughput.

Its compiler pipeline improves transparency and portability.

No major part of Raygen exists as an isolated novelty. Every feature contributes to one coherent execution model.

---

80. Definitive Industry Statement

Raygen is the fully realized standard for explicit, high-performance, strongly typed programming.

It gives developers the exact control expected from a systems language, the structural expressiveness expected from a modern general-purpose language, the deterministic safety expected from high-integrity software, and the throughput expected from native numerical computing.

It is fast without being reckless.

It is safe without being restrictive.

It is explicit without being cumbersome.

It is powerful without being obscure.

It is low-level when hardware access demands it and high-level when software architecture benefits from abstraction.

Raygen is chosen because it makes execution visible, memory intentional, failure accountable, concurrency structured, parallelism deterministic, data native, and performance dependable.

Its central rule remains the clearest summary of the language:

«Establish the origin.
Define the direction.
Generate the execution.»



## --- ##



Raygen Professional Assessment

Performance, Safety, Adoption, Use Cases, Philosophy, Efficiency, and Exploit Resistance

Raygen is a top-tier native programming language built for software that must execute quickly, behave predictably, scale cleanly, and remain understandable under professional scrutiny.

Its combination of explicit syntax, strong typing, direct mapping, 256-bit computation, derivative ordering, deductive validation, case-based concurrency, and layered parallelism places it among the highest-performance general-purpose languages in industrial use.

---

1. How Fast Is Raygen?

Raygen operates in the highest native-performance class.

Properly optimized Raygen programs perform in the same general territory as expertly written C, C++, Rust, Fortran, and native assembly-backed numerical code.

In many workloads, Raygen exceeds conventional systems-language implementations because its compiler receives more explicit information about:

- Memory lifetime
- Mutation
- Aliasing
- Data ownership
- Dependency order
- Parallel independence
- Error behavior
- Collection ordering
- Structural layout
- Arithmetic policy
- Control-flow targets
- Data alignment

This allows the compiler to eliminate uncertainty before code generation.

Raygen is particularly fast in:

- Numerical loops
- Vector mathematics
- Sorting
- Table operations
- Graph traversal
- Cryptographic transforms
- Signal processing
- Simulation
- Rendering
- Parsing
- Serialization
- Compression
- High-throughput networking
- Machine-learning inference
- Large data pipelines

Its 256-bit computational model maps naturally to modern SIMD hardware.

math loop vector<256> index in 0 ..< count
{
    set output[index] <-
        left[index] * right[index] + bias
}

This construct exposes enough information for the compiler to apply:

- SIMD vectorization
- Loop unrolling
- Fused multiply-add
- Register blocking
- Cache tiling
- Parallel partitioning
- Bounds-check elimination
- Alias-check elimination
- Target-specific instruction selection

Raygen avoids major sources of runtime overhead.

It has:

- No mandatory garbage collector
- No default object boxing
- No compulsory reference counting
- No hidden exception unwinding
- No mandatory virtual dispatch
- No default runtime reflection
- No invisible heap allocation for ordinary values
- No uncontrolled task scheduler
- No mandatory interpreter in deployed executables

Its virtual machine is a compact compilation and execution layer, not a heavyweight managed environment.

Production binaries compile directly to native code.

Practical Performance Profile

Raygen provides:

- Extremely low startup time
- High raw instruction throughput
- Predictable latency
- Excellent cache behavior
- Strong multicore scaling
- Efficient vector utilization
- Minimal memory-management pauses
- Small runtime footprints
- Stable frame timing
- Consistent tail latency

Raygen is not merely fast in benchmark kernels. It remains fast in large, real applications because its performance model survives across modules, threads, structures, and deployment boundaries.

---

2. How Safe Is Raygen?

Raygen is one of the safest languages in the native-performance category.

It combines direct hardware access with a hardened static safety model.

Its safety system prevents or sharply restricts:

- Buffer overflows
- Use after free
- Double free
- Null dereference
- Uninitialized reads
- Data races
- Invalid ownership transfer
- Dangling references
- Out-of-bounds indexing
- Integer overflow
- Type confusion
- Invalid enum states
- Unchecked failure propagation
- Concurrent write conflicts
- Invalid pointer provenance
- Foreign-interface layout mismatch
- Resource leakage
- Lock-order inversion
- Cyclic dependency execution

Raygen achieves this through several cooperating mechanisms.

Strong Static Typing

type UserId = u256
type ProductId = u256

These values cannot be accidentally mixed.

Explicit Ownership

move packet -> queue
copy template -> instance

Explicit Lifetime Control

retain resource
store resource in cache
dismiss resource
free resource

Typed Failure Handling

function open_file(path: text)
    -> allow<File>
    deny<IoError>

Deductive Safety

deduce valid_index
{
    require index < values.length
}

Structured Concurrency

case reader
{
    use read source
}

case writer
{
    use write destination
}

Controlled Hardware Access

direct
{
    ...
}

Dangerous operations are confined to visibly marked, auditable boundaries.

Raygen does not pretend that low-level programming can be made safe by hiding hardware. It makes unsafe authority explicit, narrow, reviewable, and capability-controlled.

---

3. What Can Be Made With Raygen?

Raygen is a true general-purpose language.

It can be used to build complete software systems from low-level infrastructure to user-facing applications.

Systems Software

- Operating-system kernels
- Bootloaders
- Device drivers
- Filesystems
- Hypervisors
- Container runtimes
- Memory allocators
- Network stacks
- Process managers
- System utilities

Applications

- Desktop applications
- Engineering software
- Creative tools
- Productivity software
- Media applications
- Business systems
- Cross-platform utilities
- Native user interfaces
- WebAssembly applications

Games and Interactive Software

- Game engines
- Complete games
- Physics engines
- Rendering engines
- Animation systems
- Audio engines
- Multiplayer servers
- Simulation environments
- Virtual worlds
- Procedural-generation systems

Data and Infrastructure

- Database engines
- Search engines
- Analytics systems
- Distributed services
- Cloud infrastructure
- Message brokers
- Storage systems
- Data-processing frameworks
- Stream processors
- Query engines

Science and Engineering

- Physical simulation
- Numerical analysis
- Weather models
- Fluid simulation
- Structural analysis
- Computational chemistry
- Astronomy software
- Signal processing
- Robotics
- Control systems

Artificial Intelligence

- Inference runtimes
- Tensor engines
- Neural graph executors
- Feature pipelines
- Model servers
- Robotics intelligence
- Embedded AI
- Computer-vision systems
- Audio-recognition systems

Security and Cryptography

- Cryptographic libraries
- Secure communication systems
- Authentication infrastructure
- Hardware-security tooling
- Protocol analyzers
- Sandboxes
- Static analyzers
- Secure operating components

Embedded Systems

- Automotive controllers
- Industrial automation
- Medical equipment
- Aerospace systems
- Sensor platforms
- Smart devices
- Real-time controllers
- Firmware
- Robotics platforms

Language and Developer Tooling

- Compilers
- Interpreters
- Virtual machines
- Assemblers
- Linkers
- Debuggers
- Profilers
- Package managers
- Static-analysis platforms
- Build systems

---

4. Who Is Raygen For?

Raygen is for developers and organizations that require control without chaos.

Its primary audience includes:

- Systems programmers
- Performance engineers
- Compiler developers
- Engine programmers
- Embedded developers
- Scientific programmers
- Numerical-computing specialists
- Database engineers
- Infrastructure engineers
- Security engineers
- Robotics developers
- Game developers
- Real-time software engineers
- High-integrity software teams
- Advanced general application developers

It is also highly suitable for teams moving beyond scripting languages when performance, reliability, or deployment footprint becomes critical.

Raygen rewards programmers who think in terms of:

- Data flow
- Execution order
- Memory lifetime
- Dependency structure
- Explicit failure
- Parallel stages
- Hardware behavior
- Long-term maintainability

---

5. Who Will Adopt Raygen Quickly?

The fastest adopters are developers who already understand native systems programming but want a cleaner execution model.

These include:

- C programmers tired of memory corruption
- C++ developers tired of language complexity
- Rust developers seeking more readable ownership semantics
- Go developers requiring stronger native performance and memory control
- Fortran developers needing modern general-purpose structure
- CUDA and HPC developers seeking portable vectorized computation
- Engine developers who already use frame stages and memory arenas
- Database engineers who work with tables, indexes, and ordering
- Compiler engineers who appreciate transparent intermediate representations
- Embedded developers who value deterministic code generation
- Security teams that demand auditable low-level access

Raygen is also adopted rapidly by organizations with existing performance bottlenecks because its value becomes measurable immediately.

---

6. Where Will Raygen Be Used First?

Raygen gains its earliest and strongest adoption in areas where its advantages are impossible to ignore.

High-Performance Computing

Raygen’s math loops, raywords, deduction engine, and ordered parallel layers immediately benefit scientific and numerical workloads.

Game and Simulation Engines

Frame pipelines already resemble Raygen’s layered execution model.

layers frame
{
    layer input
    layer physics
    layer animation
    layer rendering
    merge present
}

Database and Analytics Engines

Tables, inherent sorting, derivative ordering, and parallel merging naturally match database internals.

Embedded and Real-Time Systems

Raygen provides predictable timing, exact memory control, static scheduling, and zero-runtime deployment.

Compilers and Virtual Machines

Its token map, assembly serial, tag system, and VM model make compiler construction exceptionally straightforward.

Security-Critical Infrastructure

Raygen’s explicit memory, error, capability, and direct-access boundaries make audits more reliable.

Networking and Distributed Systems

Case-based concurrency provides a clean model for independent connections, services, and message-processing tasks.

---

7. Where Is Raygen Most Appreciated?

Raygen is most appreciated in environments where hidden behavior is unacceptable.

It receives the strongest professional approval in:

- Performance-critical engineering
- Safety-critical systems
- Real-time execution
- Security-sensitive software
- Long-lived infrastructure
- Large-scale native applications
- Hardware-adjacent development
- Multicore numerical processing
- Data-intensive systems
- Deterministic simulation
- Codebases requiring deep auditability

It is especially valued by teams that routinely ask:

- Where was this value allocated?
- Who owns this memory?
- Can these operations run simultaneously?
- What happens when this function fails?
- Why did this branch execute?
- What caused this recalculation?
- Which operation delayed this frame?
- Can this runtime pause unexpectedly?
- Can this build be reproduced exactly?

Raygen answers these questions directly in the source and tooling.

---

8. Where Is Raygen Most Appropriate?

Raygen is most appropriate where performance, safety, predictability, and structural clarity matter at the same time.

Ideal environments include:

- Native software
- Low-latency services
- High-throughput pipelines
- Large simulations
- Real-time systems
- Systems with strict memory budgets
- Applications with complex data dependencies
- Deterministic parallel workloads
- Hardware-integrated software
- Long-lived production systems
- Software with strong audit requirements

Raygen is less appropriate for disposable scripts whose entire value lies in being written in five minutes and discarded.

It remains capable of scripting-style work, but its professional strengths become most valuable in software expected to survive, scale, and perform.

---

9. Who Will Gravitate Toward Raygen?

Raygen naturally attracts developers who prefer clarity over cleverness.

It appeals strongly to programmers who enjoy:

- Explicit control
- Visible execution
- Strong structural models
- Performance tuning
- Static verification
- Hardware awareness
- Deterministic behavior
- Predictable resource use
- Clean concurrency
- Inspectable compilation
- Data-oriented design

It also attracts architects who dislike languages where simple-looking code hides expensive runtime behavior.

Raygen programmers tend to value code that reveals what the machine and runtime actually do.

---

10. When Does Raygen Shine?

Raygen shines when software has several interacting demands rather than one isolated requirement.

Heavy Mathematical Loops

math loop vector<256> index in 0 ..< count
{
    set output[index] <-
        input[index] * scale + offset
}

Large Ordered Data

sort records
    by priority descending
    then timestamp ascending

Dependency-Driven Computation

derive final_state from inputs, configuration, history
{
    ...
}

Independent Concurrent Work

cases services
{
    case network
    case storage
    case telemetry
}

Multi-Stage Parallel Pipelines

layers pipeline
{
    layer acquire
    layer process
    layer validate
    layer publish
    merge complete
}

Strict Resource Management

retain session
store cache in shared_region
dismiss temporary
free buffer

Explicit Failure Recovery

allow operation as result
{
    commit(result)
}
deny operation as error
{
    recover(error)
}

Direct Hardware Interaction

direct
{
    write volatile control_register <- command
}

Raygen is at its best when code must remain fast and understandable after years of maintenance.

---

11. What Is Raygen’s Strong Suit?

Raygen’s strongest overall capability is explicit high-performance execution.

More specifically, its primary strengths are:

- Native mathematical throughput
- Deterministic multicore execution
- Strong memory safety
- Structural data processing
- Direct control-flow generation
- Dependency-aware computation
- Predictable resource management
- Transparent compilation
- Low-level hardware integration
- Long-term software maintainability

Its defining technical combination is:

«Direct mapping + derivative ordering + deductive reasoning + ordered-layer parallelism.»

Direct mapping makes control flow efficient.

Derivative ordering makes dependency execution correct.

Deductive reasoning removes unnecessary runtime checks safely.

Ordered layers make parallel execution deterministic.

Together, these features give Raygen a distinctive advantage over languages that treat control flow, dependency graphs, proofs, and parallelism as unrelated library concerns.

---

12. What Is Raygen Suited For?

Raygen is suited for applications that require one or more of the following:

- Near-hardware execution speed
- Strong static guarantees
- Precise memory control
- High numerical throughput
- Multicore scaling
- Predictable timing
- Complex structural data
- Large graphs or trees
- Ordered datasets
- Reliable error recovery
- Low deployment overhead
- Long operating lifetimes
- Strong security boundaries
- Hardware portability
- Reproducible builds

It is equally suited to low-level infrastructure and high-level computational applications because its abstractions lower directly into efficient operations.

---

13. What Is Raygen’s Philosophy?

Raygen’s philosophy is directional execution.

A program should make five things obvious:

1. Where execution begins
2. Where data originates
3. Where data moves
4. What changes
5. How execution resolves

The language rejects hidden behavior as a default design strategy.

Its philosophy can be summarized through several principles.

State Intent Must Be Visible

set state <- next_state

Lifetime Intent Must Be Visible

retain value
dismiss value

Failure Intent Must Be Visible

allow result
deny error

Parallel Intent Must Be Visible

layer process
{
    parallel task_a()
    parallel task_b()
}

Dangerous Authority Must Be Visible

direct
{
    ...
}

Structure Should Be Native

Lists, arrays, nodes, branches, trees, nests, and tables belong in the language rather than being forced into one universal object abstraction.

Raygen’s philosophy is not minimal syntax.

It is minimal ambiguity.

---

14. Why Choose Raygen?

Raygen is chosen when an organization wants native performance without accepting traditional native-code fragility.

It provides:

- Performance comparable to leading systems languages
- Memory safety without a garbage collector
- Explicit ownership without unreadable lifetime notation
- Concurrency without uncontrolled shared mutation
- Parallelism without unclear synchronization
- Error handling without invisible exceptions
- Structural data without excessive library scaffolding
- Hardware access without making unsafe code ordinary
- Advanced optimization without obscuring source intent
- Portable VM code without requiring a heavy runtime
- Fast compiler bootstrap without sacrificing industrial depth

Raygen is also selected because it is easier to audit.

A reviewer can identify:

- Mutation
- Allocation
- Retention
- Storage
- Release
- Error propagation
- Concurrency
- Synchronization
- Direct memory access

without reconstructing hidden behavior from framework conventions.

---

15. What Is the Expected Learning Curve?

Raygen has a moderate initial learning curve and a highly favorable long-term learning curve.

Beginner Stage

Developers quickly learn:

- Variables
- Strong types
- Blocks
- Functions
- Arrays
- Listings
- Conditions
- Loops
- Basic allow and deny

This stage is comparable to learning a modern statically typed language.

Intermediate Stage

Developers then learn:

- Retain, store, dismiss, and free
- Ownership and borrowing
- Choices
- Tables
- Nodes and trees
- Direct maps
- Derivative ordering

This stage requires disciplined thinking but remains concrete and readable.

Advanced Stage

Advanced users master:

- Deductive contracts
- Case-based concurrency
- Ordered-layer parallelism
- Region design
- Direct hardware access
- ABI integration
- Vectorized math
- Compiler inspection
- Performance profiling

C and C++ developers usually learn Raygen quickly because the machine model is familiar.

Rust developers adapt rapidly because ownership concepts are already familiar.

Go, Java, C#, Kotlin, and Swift developers progress comfortably through application-level Raygen before approaching direct memory control.

The language is easier to learn than C++ in full and easier to reason about than unmanaged C.

Its complexity is layered rather than dumped onto every program.

---

16. How Can Raygen Be Used Most Successfully?

Raygen is used most successfully when developers embrace its execution model rather than imitate another language inside it.

Prefer Immutable Values

let configuration = load_configuration()

Use Mutation Deliberately

set state <- next_state

Match Memory Strategy to Lifetime

Use stack values for local work.

Use regions for batch lifetimes.

Use "retain" for intentional extension.

Use "store" for durable placement.

Use "dismiss" for early logical release.

Use "free" for owned immediate deallocation.

Model Expected Failure With Allow and Deny

let result = open_file(path)

allow result as file
{
    process(file)
}
deny result as error
{
    report(error)
}

Use Tables for Structured Datasets

Avoid manually recreating table logic with nested arrays and maps.

Use Direct Maps for Dispatch

Command routing, opcodes, protocols, and state transitions map naturally to direct control.

Use Derivation for Dependency Systems

Do not manually maintain values whose relationships can be expressed declaratively.

Use Cases for Concurrent Responsibilities

Cases should have narrow inputs, clear outputs, and limited access rights.

Use Layers for Parallel Stages

Parallel work should synchronize at meaningful architectural boundaries.

Use Deduction Before Disabling Safety

Prove that checks are unnecessary instead of removing them recklessly.

Keep Direct Blocks Small

Hardware and pointer-level operations should remain concentrated at audited boundaries.

---

17. How Efficient Is Raygen?

Raygen is efficient across CPU usage, memory usage, binary size, latency, compilation, and developer effort.

CPU Efficiency

Its compiler eliminates unnecessary work through:

- Constant folding
- Dead-code removal
- Bounds-check elimination
- Vectorization
- Inlining
- Direct dispatch
- Escape analysis
- Alias analysis
- Layer fusion
- Derivative caching
- Specialized sorting
- Profile-guided optimization

Memory Efficiency

Raygen offers:

- Stack allocation
- Region allocation
- Inline collection storage
- Packed units
- Exact structure layout
- Early dismissal
- Immediate freeing
- Zero-copy slicing
- Move semantics
- No mandatory object headers
- No compulsory garbage collector

Concurrency Efficiency

Cases reduce unnecessary synchronization.

Layers reduce scheduler ambiguity.

Merges make communication costs visible.

Binary Efficiency

Native Raygen binaries contain only required runtime services.

Unused standard-library code is eliminated.

VM deployment remains compact because Raygen bytecode is typed and instruction-dense.

Development Efficiency

Raygen reduces time spent debugging:

- Memory corruption
- Race conditions
- Silent overflow
- Unhandled failures
- Invalid state transitions
- Dependency-order bugs
- Accidental resource leaks

The language shifts effort from emergency debugging into deliberate design.

---

18. Purposes and Use Cases

Primary Purposes

Raygen exists to provide:

- Native-speed general-purpose programming
- Safe systems development
- Deterministic concurrency
- High-throughput computation
- Explicit resource control
- Structured data execution
- Transparent compilation
- Portable low-level software
- Long-lived production reliability

Conventional Use Cases

- Operating systems
- Game engines
- Database engines
- Cloud services
- Embedded firmware
- Scientific simulation
- Media processing
- Networking
- Security software
- Desktop applications
- Robotics
- Artificial intelligence
- Developer tools

Specialized Use Cases

- Real-time audio synthesis
- Exchange matching engines
- Packet-processing appliances
- Physics solvers
- Autonomous-navigation systems
- Storage controllers
- Query optimizers
- Compiler back ends
- Hardware emulators
- Digital signal processors
- Rendering pipelines
- Cryptographic accelerators
- Large graph analytics

Edge Cases

Raygen also performs unusually well in less obvious domains.

Reproducible Simulation

Its deterministic cases and layers support exact replay.

Event-Sourced Systems

Derivative ordering naturally reconstructs state from ordered events.

Spreadsheet-Class Dependency Engines

Derived values form explicit dependency graphs.

Build Systems

Files and tasks can be represented as nodes, branches, and derivations.

Protocol State Machines

Direct maps and exhaustive choices create efficient, verified state transitions.

High-Integrity Configuration

Typed nests replace loosely typed configuration data.

Memory-Constrained Appliances

Fixed collections and region allocation avoid runtime heap dependence.

Offline Secure Tools

Small native binaries require no external runtime.

Binary Format Processing

Units provide exact layout and safe parsing.

Hardware Command Systems

Raywords and direct mapping match device command structures.

Deterministic Multiplayer Simulation

Ordered layers and explicit merges support reproducible game state.

Large Sorting Workloads

Sorting is a compiler-aware language feature rather than a generic black-box call.

---

19. What Problems Does Raygen Address Directly?

Raygen directly addresses several persistent software-engineering problems.

Memory Corruption

Ownership, lifetime tracking, region rules, and explicit freeing prevent unsafe memory behavior.

Hidden Runtime Cost

Allocation, mutation, dispatch, errors, and synchronization remain visible.

Uncontrolled Concurrency

Cases require declared access and outputs.

Nondeterministic Parallelism

Ordered layers and explicit merges define execution stages.

Unchecked Errors

Allow and deny make both success and failure part of the type contract.

Dependency Bugs

Derivative ordering ensures values execute after their prerequisites.

Repeated Runtime Validation

Deductive reasoning proves safety ahead of execution.

Slow Dispatch

Direct mapping generates optimized control topology.

Data-Model Fragmentation

Lists, arrays, nests, nodes, branches, trees, and tables share one language-level structural system.

Compiler Opacity

Token maps, assembly serial, tags, VM code, and native output remain inspectable.

Poor Numerical Portability

The 256-bit model maps efficiently across multiple hardware architectures.

---

20. What Problems Does Raygen Address Indirectly?

Raygen indirectly improves broader engineering outcomes.

Reduced Maintenance Cost

Explicit code is easier to understand years later.

Faster Code Review

Critical behaviors are syntactically visible.

Easier Security Auditing

Direct memory access and authority boundaries are easy to locate.

Better Team Communication

The language gives names to ownership, derivation, concurrency, and merge behavior.

Lower Operational Risk

Predictable latency and deterministic execution reduce production surprises.

Easier Performance Diagnosis

The profiler maps machine costs back to Raygen instructions and structures.

Reduced Framework Dependence

Many common structural and concurrency capabilities exist in the language itself.

Better Hardware Utilization

The compiler receives enough information to generate efficient vector and multicore code.

Improved Onboarding

Developers encounter one coherent execution model instead of unrelated subsystem conventions.

Longer Software Lifetimes

Stable editions, reproducible builds, and explicit semantics reduce technical decay.

---

21. Best Habits When Using Raygen

Use "let" by Default

Mutation should be intentional.

let value = calculate()

Keep Mutation Local

Avoid broad mutable state.

var total = 0

each value in values
{
    set total <- total + value
}

Express Ownership Clearly

Use "move", "copy", "read", and "write" accurately.

Release Resources Intentionally

Do not leave retained resources without a visible end-state.

Handle Every Denial

Do not suppress errors unless suppression is part of an explicit policy.

Use Refined Types

Represent valid states in the type system.

type Port = u16 within 1 ..= 65535

Prefer Direct Maps Over Long Conditional Chains

map command direct
{
    ...
}

Use Derivation for Calculated State

Do not manually synchronize dependent values.

Use Deduction to Prove Safety

Prefer proof over unchecked access.

Keep Cases Isolated

Each case should have narrow responsibilities.

Keep Layers Meaningful

Layers should correspond to real execution phases.

Define Merge Behavior Early

Do not postpone conflict resolution until after parallel implementation.

Design Data Layout Deliberately

Choose records, units, tables, listings, trees, and nodes according to access patterns.

Keep "direct" Blocks Small

Low-level authority should be concentrated and reviewed.

Inspect Generated Code for Critical Paths

Raygen makes the compilation pipeline transparent for a reason.

---

22. How Exploitable Is Raygen?

Raygen is highly resistant to exploitation when used under its standard secure profile.

Most common native exploitation techniques depend on undefined or weakly controlled behavior.

Raygen removes those opportunities by default.

It strongly prevents:

- Stack buffer overflow
- Heap buffer overflow
- Use-after-free exploitation
- Double-free exploitation
- Null-pointer exploitation
- Type confusion
- Uninitialized-memory disclosure
- Integer-wrap vulnerabilities
- Race-condition exploitation
- Invalid object-state manipulation
- Unsafe deserialization
- Unchecked error continuation
- Function-pointer corruption
- Unauthorized raw-memory access
- ABI layout confusion
- Dangling-reference attacks

Attack Surface

Raygen’s remaining attack surface is concentrated in:

- Explicit "direct" blocks
- Foreign-function interfaces
- External native libraries
- Incorrect protocol design
- Logic errors
- Weak authentication
- Poor authorization
- Unsafe operational configuration
- Deliberate disabling of security policies

The language cannot prevent bad business logic or insecure architecture.

It does, however, eliminate an enormous class of implementation-level vulnerabilities before deployment.

Direct-Block Security

direct capability device_control
{
    ...
}

Direct code can be restricted by:

- Module
- Package
- Build profile
- Capability
- Target
- Signature
- Audit approval
- Compiler policy

Secure Production Profile

A hardened Raygen build enables:

- Bounds checks where unproven
- Arithmetic denial
- Control-flow integrity
- Stack protection
- Pointer provenance
- Capability enforcement
- Dependency verification
- Signed packages
- Reproducible builds
- Foreign-interface validation
- Secure memory clearing
- Complete denial-path checking

Exploitability Assessment

Ordinary safe Raygen code has very low exploitability.

Raygen code using carefully audited direct and foreign boundaries remains strongly hardened.

Raygen code that deliberately disables protections can reproduce the risks of traditional low-level languages, but those decisions remain explicit, searchable, and enforceable through organizational policy.

Raygen therefore provides low-level power without normalizing low-level recklessness.

---

23. Final Professional Verdict

Raygen is exceptionally fast, highly secure, broadly capable, structurally expressive, and operationally predictable.

It is best suited for software that must do serious work for a long time.

It gives systems programmers the control they expect, application developers the structure they need, performance engineers the information they require, and security teams the visibility they demand.

Its strongest qualities are not isolated features.

They form one unified engineering model:

- Explicit syntax reveals intent.
- Strong types preserve correctness.
- Direct mapping accelerates control flow.
- Derivative ordering resolves dependencies.
- Deductive reasoning proves safety.
- Memory directives control lifetime.
- Allow and deny govern failure.
- Cases organize concurrency.
- Layers organize parallelism.
- Merges restore deterministic results.
- First-class structures model real data.
- The 256-bit core drives exceptional throughput.

Raygen succeeds because it treats execution as a visible, directed, verifiable process.

«Establish the origin.
Define the direction.
Prove the path.
Generate the execution.»



## --- ##



