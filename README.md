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



