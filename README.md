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



