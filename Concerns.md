## 1. Authority and Document Integrity

### Overall finding

The 07/22/2026 Normative Core contains a strong semantic foundation, but the supplied document is not yet suitable for compiler conformance because multiple editions, duplicated normative text, explanatory commentary, and promotional claims appear in one source.

### Critical issues

#### TC-AUTH-001 — The normative specification is duplicated

The complete 66-section Normative Core appears twice.

Impact: Section anchors, quotations, conformance references, and future amendments become ambiguous. A test suite cannot reliably reference “Section 35” without identifying which copy is authoritative.

Required correction: Retain one copy only. Assign the document a stable identifier, such as:

text RG-NCORE-1.0 Revision: 2026-07-22 

---

#### TC-AUTH-002 — Multiple incompatible editions are combined

The document includes material dated:

- 07/22/2026
- 07/21/2026
- 07/20/2026
- 07/21/2006

The 07/21/2006 date is presumed to be a typographical error for 07/21/2026.

Earlier editions conflict with the Normative Core on matters including:

- Default integer width
- Whether structural types are core-language features
- Whether tables and trees are privileged library abstractions
- Whether every program is deterministic by default
- Whether a minimal compiler requires ownership analysis
- Whether 256-bit computation is the default or merely first-class
- Whether advanced directives are mandatory for core conformance

Impact: An implementer cannot determine which language behavior is binding.

Required correction: Move earlier editions into a separate historical archive. Add an explicit supersession statement:

> The Raygen 1 Normative Core dated July 22, 2026 supersedes all earlier language descriptions where they conflict.

---

#### TC-AUTH-003 — Normative and promotional claims are mixed

Statements such as the following are not normative language rules:

- “industry-standard”
- “universally respected”
- “widely selected”
- “leading native 256-bit programming language”
- “dominant”
- “one of the safest languages”
- “a minimal compliant compiler is routinely implemented in a single focused development day”

Impact: These claims cannot be tested by a compiler conformance suite and weaken the precision of the specification.

Required correction: Move market positioning, performance expectations, adoption claims, and professional assessments into a separate Language Overview or project prospectus.

---

#### TC-AUTH-004 — The authority hierarchy is internally questionable

Section 3 orders the following elements:

text explicit layout requirements > explicit semantic policies > explicit safety policies 

This permits a required layout to outrank an explicitly selected semantic or safety policy.

That conflicts with later statements such as:

- Layout cannot violate atomicity requirements.
- Safety overrides optimization.
- Meaning and safety are established before representation.
- Layout cannot invalidate ownership or aliasing assumptions.

Recommended correction:

text Language semantics > type safety > ownership and lifetime > concurrency and memory-model correctness > semantic policies > safety policies > ABI and mandatory representation contracts > other explicit layout requirements > deduction results > optimization requirements > optimization preferences > target heuristics 

Alternatively, treat ABI and layout rules as constraints rather than placing them in one linear precedence chain.

---

#### TC-AUTH-005 — Compilation precedence conflicts with deduction validity

Section 4 places:

text 9. Apply mandatory layout constraints 10. Establish aliasing and provenance facts 11. Perform bounded deduction ... 14. Lower high-level directives 

Section 19.2 states that deduction remains valid only after:

- Target lowering
- Concurrency analysis
- Integer-width selection
- Foreign-interface validation

Some of those analyses occur after deduction or are not represented as explicit stages.

Impact: A compiler cannot determine when a proof becomes final or when it must be invalidated.

Recommended correction: Separate deduction into two phases:

text Preliminary deduction Target and representation analysis Proof revalidation Runtime-guard selection Optimization 

No check may be removed until proof revalidation completes.

---

### Proposed authority rule

The specification should adopt the following rule:

> Only text contained in the identified Normative Core document is binding. Examples are normative only when explicitly marked normative. Explanatory notes, historical editions, implementation guidance, and promotional material have no semantic authority.

### Status

Not yet conformable.

The language design can proceed, but compiler conformance should not begin until the authoritative text is deduplicated, versioned, and separated from historical and descriptive material.

## 2. Grammar Completeness and Syntax Consistency

### Overall finding

The foundational grammar establishes the intended shape of Raygen, but it is not yet sufficient to build a conforming parser. Several referenced productions are undefined, some examples contradict the grammar, and newline handling lacks the formal rules needed for deterministic parsing.

### Critical issues

#### TC-GRAM-001 — The published EBNF is incomplete

The grammar references productions that are not defined in the Normative Core, including:

ebnf qualified-name generic-parameters type-expression expression assignable-expression record-declaration unit-declaration choice-declaration type-declaration alias-declaration trait-declaration implementation-declaration constant-declaration if-statement while-statement repeat-statement each-statement select-statement allow-statement deny-statement derive-statement deduce-statement case-declaration cases-statement layers-statement memory-statement policy-declaration direct-statement newline integer-literal EOF 

Impact: Independent implementations can produce incompatible parsers while all claiming conformance.

Required correction: Publish a complete grammar or classify the current grammar explicitly as informative and non-conformance-bearing.

A conforming grammar package should contain:

text Lexical token grammar Tokenization and trivia rules Statement grammar Declaration grammar Expression grammar Type grammar Policy grammar Concurrency grammar Memory-directive grammar Structural-sugar grammar Error-recovery guidance 

---

#### TC-GRAM-002 — Terminator rules contradict the EBNF

Section 12.6 states that a statement may terminate with:

- A newline
- A semicolon
- A closing brace when the preceding construct is complete

The EBNF defines:

ebnf terminator = newline | ";" ; 

A closing brace is not included.

Impact: The following may be parsed differently by different compilers:

rg1 if ready {     call run() } return allow 

One implementation may treat } as completing the if statement. Another may require a newline token after the brace.

Recommended correction:

Do not define } as a terminator token. Instead, define compound statements as self-delimiting syntactic forms:

ebnf statement     = simple-statement, terminator     | compound-statement ;  compound-statement     = block     | if-statement     | while-statement     | repeat-statement     | each-statement     | select-statement     | cases-statement     | layers-statement     | direct-statement ; 

This avoids synthetic semicolon insertion after braces.

---

#### TC-GRAM-003 — The version declaration conflicts with file examples

The grammar requires:

ebnf version-declaration     = "raygen", integer-literal, terminator ; 

Examples consistently use:

rg1 raygen 1 

However, integer-literal also includes typed suffixes and potentially arbitrary-sized integer forms.

This would appear to permit:

rg1 raygen 1:u8 raygen 0x01 raygen 1_0 

Impact: Edition identification becomes unnecessarily ambiguous.

Required correction: Define a separate edition token:

ebnf version-declaration     = "raygen", edition-number, terminator ;  edition-number     = decimal-digit, { decimal-digit } ; 

Raygen 1 should require the exact normalized value 1.

---

#### TC-GRAM-004 — Return-type grammar is ambiguous

The current production is:

ebnf return-type     = type-expression     | allowed-type, denied-type     | "void"     | "never" ; 

This permits only the exact ordering:

rg1 -> allow<T> deny<E> 

but the specification does not state whether the following are legal:

rg1 -> deny<E> allow<T> -> allow<T> -> deny<E> 

It also does not define how allow<void> is represented by a bare:

rg1 return allow 

Required correction: Define one canonical fallible-result syntax.

Suggested form:

ebnf return-type     = type-expression     | fallible-return-type     | "void"     | "never" ;  fallible-return-type     = "allow", "<", type-expression, ">",       "deny", "<", type-expression, ">" ; 

Then define:

text return allow 

as valid only when the allowed type is void.

---

#### TC-GRAM-005 — Parameter access syntax conflicts with examples

The grammar defines:

ebnf parameter     = [ access-mode ], identifier, ":", type-expression ; 

Therefore, the canonical syntax is:

rg1 read input: slice<f32> write output: slice<f32> 

Most examples use:

rg1 input: read slice<f32> output: write slice<f32> 

The examples and grammar encode access qualification in different positions.

Impact: This is a direct parser incompatibility.

Recommended correction: Adopt the example syntax because it reads naturally as a qualified type:

ebnf parameter     = identifier, ":", [ access-mode ], type-expression,       [ "=", expression ] ; 

Canonical examples:

rg1 input: read slice<f32> output: write slice<f32> packet: own Packet state: locked SharedState 

The specification must also clarify whether access modes are part of the type, part of the parameter contract, or both.

---

#### TC-GRAM-006 — Mutation syntax conflicts with derivation syntax

Ordinary mutation uses:

rg1 set total <- subtotal + tax 

Derivation examples use:

rg1 derive total from subtotal, tax pure {     total = subtotal + tax } 

The derivation rules call this an assignment, but the language otherwise reserves = for initialization.

Impact: It is unclear whether derivation bodies introduce bindings, assign outputs, or use a special equation syntax.

Recommended correction: Define derivation output equations as a distinct grammar production:

ebnf derivation-equation     = identifier, "=", expression, terminator ; 

State explicitly:

> Within a derivation body, = defines a derived output equation and is not ordinary initialization or mutation.

Alternatively, use directional assignment consistently:

rg1 set total <- subtotal + tax 

The first option better communicates declarative dependency definition but must be formally distinguished.

---

#### TC-GRAM-007 — Function calls are inconsistently written

The specification uses both:

rg1 perform_job() 

and:

rg1 call perform_job() 

The keyword list contains call, but the statement grammar has no call-statement.

Impact: It is unclear whether call is mandatory, optional sugar, or a separate instruction with different semantics.

Required correction: Choose one of these models:

Model A — Calls are expressions

rg1 perform_job() let value = calculate() 

call is removed from the core keywords.

Model B — Discarded-result calls require call

rg1 call perform_job() let value = calculate() 

Then define:

ebnf call-statement     = "call", call-expression, terminator ; 

Model B is more consistent with Raygen’s explicit instruction philosophy.

---

#### TC-GRAM-008 — emit, sort, math, order, and other used forms are absent

Normative examples rely on syntax that is neither declared nor assigned to the standard-library sugar boundary:

rg1 emit value sort values ascending math loop vector<256> ... order row.name ascending completion quorum 3 wait worker spawn case worker read volatile device.status write volatile device.command <- value 

Impact: These examples cannot be used as parser conformance tests.

Required correction: Every syntax form must be assigned to exactly one category:

text Core grammar Privileged standard-library syntax Ordinary function-call sugar Future-edition syntax Informative pseudocode 

No normative example should depend on unclassified syntax.

---

#### TC-GRAM-009 — Pipeline stages are under-specified syntactically

The pipeline example is:

rg1 users |> where row.active |> order row.name ascending 

Neither where nor order is defined as an expression. The phrase “a stage receives the previous result” does not define how the implicit input binds.

Open questions include:

- Is row a reserved implicit parameter?
- Is where row.active a closure?
- May arbitrary expressions follow |>?
- Does x |> f(a) lower to f(x, a) or f(a, x)?
- Are pipeline stages overload-resolved?
- Can stages consume ownership?
- How are denials propagated between stages?

Recommended correction: Define a minimal universal lowering:

text x |> f       lowers to f(x)  x |> f(a, b)       lowers to f(x, a, b) 

Structural forms such as where and order should then be specified as privileged callable operators or explicit structural sugar.

---

#### TC-GRAM-010 — Newline continuation rules are not decidable as written

Semicolon insertion must not occur inside “incomplete expressions” or “pipeline continuations,” but incompleteness is not lexically defined.

Examples requiring formal treatment include:

rg1 let value =     calculate()  let value = calculate (     left,     right )  let result = source     |> transform     |> validate  set output[index] <-     input[index] * factor 

Impact: Parser behavior may depend on implementation-specific lookahead and error recovery.

Recommended correction: Define newline significance through token context.

A newline should not terminate a statement when the preceding significant token is one of:

text = <- |> , . :: ( [ { + - * / % & | ^ && || < <= > >= == != 

or when delimiters are unclosed.

A newline should also not terminate when the next token is a defined continuation token, such as:

text else deny allow then where 

The exact list must be normative.

---

#### TC-GRAM-011 — Generic-angle parsing is unresolved

The specification states that semicolon insertion does not occur inside generic-argument lists, but it does not define how < and > are distinguished from relational operators.

Examples:

rg1 vector<f32, 8> allow<File> a < b value >> shift 

Impact: Tokenization and parsing of nested generics can diverge, especially around >>.

Required correction: State that angle brackets are parsed contextually after a generic-capable name in type grammar. Also define whether:

rg1 Outer<Inner<T>> 

produces two > tokens or one shift token.

Recommended rule:

> The lexer always emits individual < and > tokens. Shift operators are formed by the parser from adjacent tokens in expression context.

---

#### TC-GRAM-012 — Unicode identifier normalization is unspecified

Identifiers may contain Unicode letters, but the specification does not define normalization or confusable handling.

The visually equivalent identifiers below may have different byte sequences:

rg1 café café 

Impact: Name resolution, linking, package identity, security review, and reproducible builds may differ between implementations.

Required correction: Require Unicode normalization before identifier comparison, preferably NFC, and define whether normalization affects source spans and diagnostics.

The language should also require a diagnostic for mixed-script or confusable identifiers under a secure diagnostic profile.

---

#### TC-GRAM-013 — Nested block-comment wording is self-contradictory

The specification states:

> Block comments may nest only if the implementation supports nested comment counting exactly as defined by the language edition.

It then states:

> Raygen 1 permits nested block comments.

“Permits” can be interpreted as optional implementation support, while Raygen 1 conformance should require consistent lexical behavior.

Required correction:

> Raygen 1 block comments nest. Every conforming lexer must count nested #[ and ]# delimiters and must reject an unterminated block comment.

---

#### TC-GRAM-014 — Keyword inventory is incomplete and unstable

Normative examples use undeclared words that may need reserved status:

text pure effectful uses output input detached completion failure quorum retry order ascending rapid tail scalar masked padded order sequential acquire release acquire_release relaxed 

Conversely, some reserved words do not yet have grammar productions.

Impact: Implementers cannot know whether these words are identifiers, contextual keywords, or globally reserved keywords.

Recommended correction: Divide keywords into:

text Hard keywords Contextual keywords Predeclared identifiers Standard-library names 

For example, memory-order names can be contextual enum members rather than globally reserved words.

---

### Proposed parser-conformance rule

A Raygen implementation should not claim parser conformance until every normative example can be classified as one of:

1. Valid Raygen 1 source
2. Intentionally invalid source with a required diagnostic category
3. Informative pseudocode explicitly marked as non-source text

### Status

Parser implementation is blocked for full Raygen 1.

A bootstrap parser can be built for a restricted subset, but the complete language requires a closed lexical grammar, complete production set, canonical access-mode syntax, formal newline rules, and classification of all privileged syntax.



## 3. Type System and Failure-Flow Consistency

### Overall finding

Raygen’s type model has coherent goals—strong nominal typing, explicit access, typed denial, and no silent conversion—but several foundational relationships remain undefined. The most significant blockers concern the representation of fallible results, the distinction between aliases and nominal wrappers, access qualifiers, void, conversion rules, and generic type legality.

### Critical issues

#### TC-TYPE-001 — allow and deny lack a formal type constructor

The specification uses:

rg1 function open_file(path: text)     -> allow<File> deny<FileError> 

but does not establish whether this return type is:

- One sum type with two variants
- Two parallel return channels
- A function effect attached to an allowed type
- A specialized control-flow contract
- Syntax sugar for a standard type such as Result<File, FileError>

This distinction affects:

- ABI representation
- Generic use
- Storage in variables
- Nesting
- Pattern matching
- Trait implementation
- Ownership transfer
- Denial propagation
- Foreign interfaces

Required correction: Define a single formal type constructor.

Recommended model:

text outcome<A, D> 

with two variants:

text allowed(A) denied(D) 

The source syntax:

rg1 allow<A> deny<D> 

can be defined as canonical sugar for:

rg1 outcome<A, D> 

This makes the following variable declaration well-defined:

rg1 let result: outcome<File, FileError> =     open_file(path) 

---

#### TC-TYPE-002 — Success and failure syntax are overloaded as types, values, statements, and policies

The words allow and deny are used for several independent purposes:

rg1 -> allow<File> deny<FileError> return allow file return deny error allow result as file { ... } deny result as error { ... } allow overflow wrap deny data_race let file = allow open_file(path) 

These represent at least five semantic categories:

1. Type syntax
2. Outcome construction
3. Outcome matching
4. Outcome propagation
5. Policy declaration

Impact: Grammar and semantic analysis become context-sensitive, and diagnostics may be unclear.

Required correction: Formally define each role.

Suggested terminology:

text allow<T> deny<E>       fallible return type return allow value     allowed outcome construction return deny error      denied outcome construction allow result as value  allowed-branch matching deny result as error   denied-branch matching allow expression       denial propagation policy ... allow ...   policy rule 

The specification should state that the meanings are syntactically distinct despite sharing keywords.

---

#### TC-TYPE-003 — Denial propagation is not type-safe as currently described

The form:

rg1 let file = allow open_file(path) 

means that a denial is returned from the current function. However, no compatibility rule is defined between the callee’s denial type and the caller’s denial type.

Example:

rg1 function load(path: text)     -> allow<Document>     deny<LoadError> {     let file =         allow open_file(path) } 

If open_file denies with IoError, the compiler needs one of the following:

- Exact denial-type equality
- Implicit variant injection
- An explicit conversion
- A declared from implementation
- A denial-mapping clause

Required correction: Adopt a precise propagation rule.

Recommended rule:

> allow expression may propagate a denial only when the expression’s denial type is identical to the enclosing function’s denial type or can be converted through one unambiguous, explicitly declared denial conversion.

Example:

rg1 implement convert<IoError, LoadError> {     ... } 

or explicit syntax:

rg1 let file =     allow open_file(path)     deny_as LoadError.io 

No broad implicit conversion should occur.

---

#### TC-TYPE-004 — Bare return allow is under-specified

Examples use:

rg1 return allow 

for functions returning:

rg1 -> allow<void> deny<AliasError> 

The specification does not state whether void has a value, whether allow without an expression constructs that value, or whether this is special return syntax.

Recommended correction: Define a unit type distinct from void, or precisely define void.

Preferred model:

text unit is a zero-size inhabited type with one value: unit. void means no value position and is not a storable type. 

Then fallible procedures return:

rg1 -> allow<unit> deny<AliasError> 

and use:

rg1 return allow unit 

A concise special form may still be allowed:

rg1 return allow 

but it must lower to return allow unit.

Using an inhabited unit type avoids special cases in generics and outcome representation.

---

#### TC-TYPE-005 — void and never are not semantically defined

The grammar includes both:

text void never 

but does not define:

- Whether variables can have these types
- Whether void is zero-sized or uninhabited
- Whether never coerces to other types
- Whether a denied-only function returns never on its success path
- Whether diverging expressions satisfy any expected type
- ABI treatment

Required correction:

text void:     permitted only as a non-value function return marker;     not permitted for variables, fields, parameters, or generic arguments.  never:     uninhabited;     produced by expressions that do not complete normally;     may coerce to any expected value type;     has no runtime representation. 

Alternatively, replace void with a regular unit type throughout the language.

---

#### TC-TYPE-006 — Nominal type construction is inconsistent

The specification declares:

rg1 type UserId = u256 

and states that UserId is nominally distinct from other types based on u256.

Elsewhere, examples use direct values or casts without defining construction syntax.

Potential forms include:

rg1 let id: UserId = 42 let id = UserId(42) let id = cast<UserId>(42) 

These have materially different safety properties.

Required correction: Define nominal type introduction and extraction.

Recommended rules:

rg1 type UserId = u256 

creates an opaque nominal wrapper.

Construction:

rg1 let id = UserId(42:u256) 

Extraction requires an explicit operation:

rg1 let raw = id.value 

or a declared conversion.

A numeric literal may initialize a nominal numeric type only when the literal is compile-time representable and the initialization is explicitly typed:

rg1 let id: UserId = 42 

Conversions from runtime values remain explicit.

---

#### TC-TYPE-007 — Transparent aliases and nominal types need layout rules

The specification distinguishes:

rg1 type UserId = u256 alias Distance = f64 

but does not define whether a nominal type:

- Has the same layout as its underlying type
- Has the same ABI
- Can use the same atomic operations
- Inherits arithmetic operators
- Implements the same traits
- Can be directly serialized as the underlying value

Required correction: Separate identity, representation, and behavior.

Suggested rule:

> A nominal type has distinct type identity. It has the underlying representation only when declared representation transparent or when the ABI specification explicitly guarantees transparent newtype layout. Operators and traits are not inherited unless declared by the language or implemented explicitly.

Example:

rg1 type UserId = u256 representation transparent 

Without such a declaration, layout should remain implementation-controlled.

---

#### TC-TYPE-008 — Refined and range types lack runtime-validation semantics

Examples include:

rg1 type Percentage = f64 where value >= 0.0 and value <= 100.0  type Port = u16 within 1 ..= 65535 

The specification does not define:

- How values are constructed
- Whether constraints are checked at compile time, runtime, or both
- What denial type is produced
- Whether constraints are preserved after mutation
- Whether arithmetic returns the refined type or the base type
- How foreign values enter the type
- Whether NaN can inhabit Percentage

Required correction: Define refined types as invariant-bearing nominal types.

Recommended construction:

rg1 let port =     checkconstruct<Port>(raw_port) 

with result:

rg1 allow<Port> deny<RefinementError> 

Compile-time constants can be accepted directly when proven valid.

Mutation of a refined value must preserve the invariant or produce a checked outcome.

For floating-point refinements, the predicate must explicitly address NaN.

---

#### TC-TYPE-009 — Access modes are not integrated into the type system

The specification uses:

text own read write move atomic locked 

as parameter access modes, while concurrency contracts also use:

text read write move atomic locked exclusive snapshot transactional 

It is unclear whether these are:

- Reference types
- Borrow modes
- Capabilities
- Parameter-passing conventions
- Effects
- Concurrency resource declarations

For example:

rg1 input: read slice<f32> 

could mean a read-only borrow of the slice descriptor, a read-only borrow of the elements, or ownership of an immutable slice.

Required correction: Define access qualification structurally.

At minimum:

text own T:     ownership of T is transferred into the callee.  read T:     shared nonmutating borrow of T for the call lifetime.  write T:     exclusive mutable borrow of T for the call lifetime.  move T:     redundant with own T unless it has distinct transactional semantics.  atomic T:     access to an atomic-capable shared object.  locked T:     access requiring a specific lock capability. 

The relationship between the qualification of a container and its elements must also be explicit.

---

#### TC-TYPE-010 — own and move appear semantically redundant

The parameter grammar supports both:

text own move 

The ownership section says a value may be “owned” or “moved,” and case contracts include move.

No formal distinction is provided.

Possible interpretations include:

- own T describes the callee’s resulting ownership
- move T describes the caller-side transfer operation
- Both are synonyms

Recommended correction:

Use own as a type/access qualification:

rg1 packet: own Packet 

Use move only as an expression or transfer operation:

rg1 call consume(move packet) move packet -> queue 

Remove move from the parameter access-mode grammar unless a separate meaning is documented.

---

#### TC-TYPE-011 — atomic<T> and atomic access mode conflict conceptually

Atomic storage is declared as:

rg1 atomic<u32> counter 

The parameter grammar also allows:

rg1 atomic counter: u32 

or, under the examples’ syntax:

rg1 counter: atomic u32 

This may imply either an atomic type or atomic permission over a non-atomic value.

Required correction: Atomicity should primarily be a storage/type property:

rg1 counter: read atomic<u32> 

or:

rg1 counter: atomic<u32> 

A concurrency access contract may then declare permission to use atomic operations on that object. It must not transform ordinary u32 storage into atomic storage.

---

#### TC-TYPE-012 — Type inference rules are insufficient

The Normative Core identifies typed numeric literals but does not define:

- Default integer type
- Default floating type
- Integer-literal range
- Inference across arithmetic expressions
- Mixed-width arithmetic
- Generic literal inference
- Unsuffixed negative literals
- Target dependence
- Overflow during constant evaluation

Historical material says native unconstrained integers default to int, but that rule is not clearly normative in the core.

Required correction: Add literal inference rules.

Suggested baseline:

text Unsuffixed integer literals have an abstract compile-time integer type. They are converted to the expected integer type when representable. Without an expected type, they default to int.  Unsuffixed floating literals have an abstract compile-time real type. They are converted to the expected floating type when representable. Without an expected type, they default to real. 

Unary negation should apply after literal parsing, so -128:i8 requires a defined representability rule.

---

#### TC-TYPE-013 — Arithmetic operator typing is undefined

The specification prohibits silent signedness conversion and narrowing, but does not define whether these expressions are legal:

rg1 1:u8 + 2:u16 1:i32 + 2:u32 index: usize + offset: u32 value: f32 + 1.0 

Impact: Implementations may adopt incompatible promotion systems.

Recommended correction: Raygen should avoid C-style implicit promotions.

Suggested rule:

> Binary arithmetic operands must have the same concrete numeric type after contextual literal resolution. Nonliteral cross-type arithmetic requires an explicit conversion.

Therefore:

rg1 let total =     cast<u16>(small) + large 

This is predictable and consistent with strong typing.

---

#### TC-TYPE-014 — Overflow behavior is incomplete for constant evaluation

Arithmetic policy supports:

text deny wrap saturate clamp report widen 

but constant expressions are not addressed.

Questions include:

- Does 255:u8 + 1:u8 fail compilation under deny?
- Does report produce a value plus a report object?
- How does widen choose the destination type?
- What distinguishes clamp from saturate?
- Does wrapping apply to signed values using two’s complement?
- Are compile-time and runtime results guaranteed identical?

Required correction: Every arithmetic behavior needs a formal operation table.

At minimum:

text deny:     overflow produces a compile-time denial for constant evaluation     or a runtime denial for dynamic evaluation.  wrap:     arithmetic is performed modulo 2^N.  saturate:     result is limited to the numeric type’s minimum or maximum.  clamp:     invalid without an explicit user-provided range.  report:     returns an outcome or report-bearing result; cannot be an ordinary scalar operator result without additional syntax.  widen:     requires a statically defined wider result type; forbidden when no such type exists. 

clamp and report cannot remain bare operator policies without defining result typing.

---

#### TC-TYPE-015 — Floating-point semantics are not sufficiently specified

Semantic policies mention rounding and reproducibility, but the core does not define:

- IEEE 754 conformance
- Supported rounding modes
- NaN comparison behavior
- Signed zero
- Signaling NaNs
- Conversion overflow
- Denormal handling
- Whether f16 and f128 may be software-emulated
- Intermediate precision
- Literal rounding

Impact: Cross-target semantic and bitwise determinism cannot be evaluated.

Required correction: Adopt an explicit floating-point baseline, normally IEEE 754 where supported, and specify permitted emulation.

The bitwise-deterministic profile must define canonical behavior for NaNs, denormals, fused operations, and transcendental functions.

---

#### TC-TYPE-016 — Generic declarations are referenced but not defined

Examples use:

rg1 function first<T, const N: usize>(...) function maximum<T: ordered>(...) 

but the grammar and type system do not define:

- Type parameters
- Constant parameters
- Trait bounds
- Where clauses
- Monomorphization
- Variance
- Higher-ranked lifetimes
- Specialization
- Generic recursion
- Generic outcome types
- Generic layout constraints

Required correction: Raygen Core should either exclude generics or publish a minimal formal generic model.

A practical Raygen 1 baseline could support:

text Type parameters Constant integer parameters Trait-bound conjunction No specialization No higher-kinded types No implicit variance Monomorphized semantics Optional implementation-level code sharing 

Semantic behavior must not depend on whether code is physically monomorphized.

---

#### TC-TYPE-017 — Trait coherence and method resolution are undefined

Traits and implementations are listed as core declarations, but no rules define:

- Orphan/coherence restrictions
- Overlapping implementations
- Inherent versus trait method priority
- Generic implementation overlap
- Associated types or constants
- Dynamic dispatch
- Object safety
- Import-dependent method resolution

Impact: Separate compilation and library compatibility are blocked.

Recommended correction: Adopt a coherence rule such as:

> An implementation is legal only in the package defining the trait or the package defining the implementing type. For any concrete type and trait pair, at most one applicable implementation may exist in a conforming program.

Raygen 1 should prohibit specialization until a separate specification defines it.

---

#### TC-TYPE-018 — Type legality and target support are conflated

The compilation stages distinguish declared-type legality from target lowering, but atomic types and native-width types depend on the target.

For example:

rg1 atomic<u256> f128 vector<f64, 4> 

may not have direct hardware support.

The specification permits lock-based atomic implementations but does not generalize this distinction to other types.

Required correction: Define three states:

text Language-valid Target-supported natively Target-supported through conforming emulation 

A type should not become semantically illegal merely because a target lacks a direct instruction, unless a deployment policy forbids emulation.

---

#### TC-TYPE-019 — Array and slice mutability are not formally separated

Examples use:

rg1 input: read slice<f32> output: write slice<f32> 

but no rule defines whether:

- A slice owns its elements
- A slice can outlive its source
- A read slice<T> permits copying elements
- A write slice<T> permits resizing
- Slice metadata can be mutated
- Overlapping writable slices are legal
- A slice can refer to region, stack, foreign, or atomic storage

Required correction: Define a slice as a non-owning view containing provenance, length, and access qualification.

Suggested semantics:

text read slice<T>:     shared borrow of a contiguous sequence of T.  write slice<T>:     exclusive mutable borrow of a contiguous sequence of T.  own slice<T>:     ownership of a slice descriptor and its backing storage only when     produced by an owning slice type; ordinary slices remain non-owning. 

A separate owning buffer type may be clearer than own slice<T>.

---

#### TC-TYPE-020 — text, strings, bytes, and Unicode representation are undefined

The lexical grammar defines UTF-8 source strings, but the runtime type text is not specified.

Open questions include:

- Is text UTF-8?
- Is it immutable?
- Is indexing by byte, scalar value, or grapheme?
- Does slicing require code-point boundaries?
- What is the type of a string literal?
- Does char represent a byte, Unicode scalar, or grapheme?
- Is byte an alias for u8 or a nominal type?
- Are malformed runtime strings representable?

Required correction: Establish a minimal text model.

Recommended baseline:

text byte:     exact 8-bit unsigned storage unit, distinct or aliased to u8 by specification.  char:     Unicode scalar value.  text:     immutable, valid UTF-8 sequence.  string literal:     compile-time text value. 

Byte indexing should require an explicit byte view. Text indexing should not silently imply constant-time character access.

---

### Proposed type-system foundation

A coherent Raygen 1 type hierarchy could begin with:

text Value types     Primitive numeric types     bool     char     unit     records     units     choices     arrays     nominal wrappers  Reference and view types     read borrow     write borrow     slices     addresses     locked views  Special semantic types     atomic<T>     outcome<A, D>     never     region and capability handles 

Access qualifications, ownership state, and concurrency permissions should not be collapsed into one undifferentiated type syntax.

### Status

Type checking is implementable only for a restricted subset.

Full Raygen 1 type-system conformance requires a formal outcome type, denial-conversion rules, unit and never semantics, nominal construction rules, access qualification, numeric coercion rules, refined-type validation, generic coherence, and a defined runtime text model.

## 4. Ownership, Lifetime, Regions, and Memory Directives

### Overall finding

Raygen clearly intends to provide statically enforced ownership with explicit lifetime operations. However, the current specification mixes semantic ownership states, storage actions, optimizer hints, and physical deallocation mechanisms without defining their exact transitions.

The most serious risks concern retain, dismiss, explicit free, region escape, borrowing, partial moves, destruction, and the boundary between safe code and runtime-guarded ownership.

### Critical issues

#### TC-OWN-001 — The ownership state model is not formalized

Section 50 lists states including:

text owned borrowed read-only borrowed writable moved shared atomic shared locked stored retained dismissed freed 

These are not all the same kind of concept.

For example:

- owned describes authority.
- borrowed writable describes temporary access.
- stored describes placement.
- retained describes lifetime extension.
- moved describes a completed transition.
- freed describes terminal storage invalidation.
- shared atomic describes synchronization.

Impact: A compiler cannot represent these reliably as one linear state machine.

Required correction: Separate at least four independent dimensions:

text Ownership:     owned     shared-owned     borrowed-read     borrowed-write     unavailable-after-move  Storage:     stack     region     heap     static     foreign     device     persistent  Lifetime status:     live     retained     dismissed     destroyed  Synchronization:     local     atomic     locked     transactional 

A value may simultaneously be, for example:

text owned + region-stored + live + local 

These properties should not be treated as mutually exclusive states.

---

#### TC-OWN-002 — retain does not define a unique ownership transition

The specification says:

> Retain extends a value’s logical lifetime.

It also permits implementations using:

- Ownership transfer
- Region promotion
- Reference counting
- Arena placement
- Heap allocation
- Static storage

These mechanisms have different observable consequences.

Questions include:

- Does retain value create another owner?
- Does it move the value?
- Does the original binding remain usable?
- Can retaining copy a noncopyable resource?
- Can it fail due to allocation?
- Can it change the address of the value?
- Does it preserve pointer identity?
- What releases the retained lifetime?
- Is retention visible through ABI or foreign pointers?

Required correction: retain must describe one semantic action independent of implementation.

Recommended definition:

> retain x creates or confirms an owning lifetime obligation for x that extends beyond the current default lexical release point. It does not create an additional owner unless an explicit shared-ownership type permits that operation. It does not change object identity. If extension requires fallible promotion or allocation, the operation must return a typed outcome.

For explicit destination-based retention:

rg1 let retained =     retain move value into session_region 

This makes ownership transfer and possible failure visible.

A bare:

rg1 retain value 

should be legal only when the compiler can extend lifetime without a fallible runtime operation.

---

#### TC-OWN-003 — dismiss is semantically ambiguous

The specification says dismissal:

- Ends active use
- May decrease active ownership
- May trigger reference release
- May invoke a destructor
- May perform no instruction
- Does not necessarily release physical memory

This makes dismiss potentially equivalent to any of:

- End borrow
- Drop binding
- Run destructor
- Decrement reference count
- Mark value dead for optimization
- Remove an entry from storage

These operations are not interchangeable.

Impact: Resource lifetime and side effects become unpredictable.

Recommended correction: Define dismiss narrowly:

> dismiss x ends the current binding’s right to access x. After dismissal, the binding is statically unavailable. Dismissal performs the ordinary destruction obligation associated with that binding, but does not force immediate reclamation of region storage.

Then specify separate operations for storage removal:

rg1 remove key from cache 

and borrow termination, which should normally occur lexically rather than through dismiss.

---

#### TC-OWN-004 — free conflicts with automatic destruction

Raygen supports ownership tracking and also permits:

rg1 free buffer 

The specification does not explain how this interacts with:

- End-of-scope cleanup
- Destructors
- defer
- Retained values
- Partial moves
- Shared ownership
- Region ownership
- Unwinding or denial propagation
- Early returns
- Case cancellation

Impact: Double destruction is possible unless every ownership path is precisely invalidated.

Required correction: Define free as an ownership-consuming operation:

text free(x):     requires exclusive ownership of manually releasable storage;     performs destruction and immediate storage reclamation;     consumes x;     leaves the binding permanently unavailable. 

Automatic cleanup must recognize the consumed state and perform no second destruction.

Types should declare whether explicit free is legal. Stack values and ordinary inline record fields should not be individually freeable.

---

#### TC-OWN-005 — The language does not distinguish destruction from deallocation

A value may own resources even when its storage is not independently deallocatable.

Examples include:

- A file handle stored on the stack
- A string inside a region
- A record containing heap allocations
- A lock guard
- A foreign resource
- A field inside another object

free cannot correctly describe all cleanup.

Required correction: Define two separate concepts:

text drop:     execute the type’s destruction behavior and end the value’s lifetime.  deallocate:     release a storage allocation after contained values are destroyed. 

Raygen may continue using dismiss and free as surface terms, but their lowering must preserve this distinction:

text dismiss owned resource     -> drop now, storage reclamation according to allocator or region  free owned allocation     -> drop contents, then deallocate allocation immediately 

---

#### TC-OWN-006 — Destructor semantics are absent

The specification refers indirectly to cleanup and resource release but does not define:

- Whether user types can define destructors
- Destruction order for record fields
- Destruction order for local variables
- Behavior after partial initialization
- Whether destruction may deny
- Whether destruction may block
- Whether destruction may access other values
- How panics or traps during destruction behave
- Whether destructors run during case cancellation

Required correction: Add deterministic destruction rules.

Suggested baseline:

text Local bindings are destroyed in reverse order of successful initialization.  Record fields are destroyed in reverse declaration order.  Moved or explicitly freed fields are skipped.  Destruction must not produce an ordinary denial.  Fallible cleanup must be performed explicitly before destruction.  Destructors run on normal block exit, return, denial propagation, and structured cancellation unless execution terminates by an immediate trap. 

A separate trait such as Drop may define cleanup, but its restrictions must be normative.

---

#### TC-OWN-007 — Partial moves are undefined

Records and composite values can contain independently owned fields, but the specification only describes moving an entire value.

Example:

rg1 move request.body -> queue emit request.id 

It is unclear whether moving one field:

- Invalidates the entire record
- Leaves the remaining fields usable
- Creates a partially moved state
- Requires compiler-generated drop flags
- Is forbidden for types with destructors

Required correction: Choose one model.

A practical Raygen 1 model is:

> Field moves are permitted when the field is independently movable and the containing value is not subsequently used as a whole. Remaining unmoved fields may be accessed individually. Destruction skips moved fields.

For a simpler bootstrap subset, partial moves may be prohibited.

---

#### TC-OWN-008 — Borrow creation syntax is missing

The specification defines read and writable borrowing conceptually, but it does not show how a caller creates a borrow.

Possible calls include:

rg1 inspect(packet) normalize(packet) consume(packet) 

Yet the parameter access modes differ semantically.

Open questions include:

- Is borrowing inferred from the parameter?
- Must the call use read packet or write packet?
- Can overload resolution differ only by access mode?
- How is a reborrow expressed?
- Is an implicit mutable borrow allowed?
- When exactly does the borrow end?

Recommended correction: Define call-site behavior explicitly.

Suggested model:

rg1 call inspect(read packet) call normalize(write packet) call consume(move packet) 

For ergonomic use, implicit read may be allowed when unambiguous, but write and move should remain explicit.

Borrow lifetime should normally be the smallest enclosing expression or statement unless the borrow is stored in a declared reference binding.

---

#### TC-OWN-009 — Borrowed-reference types are not defined

Parameters can borrow values, but the type grammar does not define a first-class borrowed reference that can be:

- Returned
- Stored in a record
- Assigned to a variable
- Passed through a generic function
- Placed in a collection
- Captured by a case

Impact: Lifetime checking beyond direct function calls is impossible to specify.

Required correction: Either:

1. Restrict Raygen 1 borrows to non-storable parameter and local-view capabilities, or
2. Define explicit reference types and lifetime relationships.

A minimal syntax could be:

rg1 readref<T> writeref<T> 

with inferred lexical lifetimes.

Named lifetime parameters should be avoided in the first edition unless required. Return borrowing can use source-relative rules:

rg1 function first(values: read slice<T>)     -> readref<T>     returns_from values 

---

#### TC-OWN-010 — Borrow exclusivity is not formally scoped

The specification states that a writable borrow cannot coexist with violating access, but does not define when a borrow begins and ends.

Example:

rg1 let item = values[index] call mutate(write values) emit item 

The legality depends on whether item is:

- A copied value
- A read borrow
- A view into the array
- A compiler-selected representation

Required correction: Value extraction and reference extraction must have distinct syntax or type-directed semantics.

For example:

rg1 let item = copy values[index] let item_ref = readref values[index] 

Borrow liveness should be based on semantic use, not merely lexical scope, provided this does not alter observable destruction behavior.

---

#### TC-OWN-011 — Aliasing rules are not stated as language invariants

The specification relies heavily on alias analysis for deduction and vectorization, but it does not define what aliases safe Raygen programs may create.

Questions include:

- Can two read borrows alias?
- Can a read and write borrow overlap partially?
- Can two writable slices refer to disjoint parts of one array?
- Can raw addresses alias safe references inside direct?
- Can foreign code retain a pointer beyond the call?
- Can packed fields create overlapping writable references?

Required correction: Define baseline aliasing guarantees:

text Multiple read borrows may alias.  A live writable borrow must have exclusive access to the borrowed memory range.  Disjoint writable borrows may coexist when disjointness is proven or dynamically guarded.  Raw-address access that may overlap a safe borrow requires explicit direct authority and invalidates affected alias proofs unless constrained by a foreign or direct-memory contract. 

The compiler’s optimizer may rely on these guarantees only after provenance validation.

---

#### TC-OWN-012 — Region ownership and region handles are conflated

Examples declare:

rg1 region frame_memory {     capacity = 64:mb     alignment = 64     release = frame_end } 

and later:

rg1 free frame_memory 

It is unclear whether frame_memory is:

- A lexical region declaration
- A runtime allocator object
- A capability
- An owned allocation
- A storage namespace
- A lifetime parameter

Required correction: Separate the lexical region identity from its runtime allocator handle.

Example:

rg1 region frame_lifetime {     let arena =         region_allocator(             capacity = 64:mb,             alignment = 64         ) } 

Alternatively, define that a region declaration creates a unique owned region capability whose destruction releases its allocations.

The latter is simpler but must be explicit.

---

#### TC-OWN-013 — Region escape rules are incomplete

The specification permits a region value to escape when:

- Moved into a longer-lived region
- Copied into independent storage
- Promoted through valid retention

However, it does not define:

- How region lifetimes are ordered
- Whether region-to-region moves preserve identity
- Whether promotion can fail
- How interior references are updated
- Whether self-referential structures can be promoted
- Whether foreign pointers into a region block release
- Whether a region can contain references to shorter-lived regions

Required correction: Define a lifetime partial order.

At minimum:

text A stored reference may only point to a value whose lifetime is at least as long as the storage containing the reference. 

Promotion should be a typed operation that may deny:

rg1 let promoted =     retain move value into application_region 

Values containing interior references should be promotable only when the compiler proves all internal references remain valid or a type-specific relocation implementation exists.

---

#### TC-OWN-014 — Freeing a region while borrows exist is not addressed

Section 52 says freeing a region invalidates its values, but safe Raygen must prevent such invalidation while references remain usable.

Example:

rg1 let view = readref frame_memory.buffer free frame_memory emit view 

Required correction: A region cannot be freed while any live safe borrow, retained interior reference, active case access, foreign loan, or lock guard refers to its contents.

A statically known conflict must be denied at compile time.

Dynamic lifetime relationships require a checked region capability, but ordinary safe code should avoid runtime dangling-reference checks.

---

#### TC-OWN-015 — Explicit capacity does not define allocation failure

Regions may declare a fixed capacity:

rg1 capacity = 64:mb 

The specification does not define what happens when capacity is exceeded.

Possible outcomes include:

- Typed denial
- Trap
- Heap fallback
- Region growth
- Compilation failure
- Undefined behavior

Required correction: Region allocation policy must declare behavior:

rg1 region frame_memory policy {     capacity = 64:mb     overflow deny<RegionExhausted> } 

A fallback allocator must be explicit because it changes latency and deployment behavior.

---

#### TC-OWN-016 — store lacks a formal transfer model

The specification says store can transfer or copy a value into a destination, but examples include:

rg1 store profile in user_cache store move profile in user_cache store copy profile in user_cache 

The unsuffixed form is ambiguous.

Questions include:

- Is store x a move by default?
- Does it use the type’s copy behavior?
- Can it retain shared ownership?
- What type is the destination?
- Does replacement destroy the previous value?
- Can storing fail?
- Does persistent storage serialize or preserve object identity?

Required correction: Require an explicit transfer mode unless the type has exactly one legal mode.

Recommended syntax:

rg1 store move profile in user_cache store copy profile in user_cache store share profile in user_cache 

Storage operations that can fail must return an outcome.

Persistent storage should be a library operation, not assumed to preserve in-memory ownership semantics.

---

#### TC-OWN-017 — Storage destinations are not typed

A destination such as:

rg1 user_cache history persistent "ledger.rgd" 

has no normative type model.

A conforming compiler needs to know:

- Accepted value type
- Ownership mode
- Lifetime
- Capacity
- Synchronization
- Replacement behavior
- Failure type
- Serialization format
- Whether references may be stored

Required correction: Storage destinations must be typed capabilities.

For example:

rg1 let user_cache:     storecap<UserId, Profile, SessionLifetime> 

High-level declarations may provide sugar, but the lowered type and effects must be defined.

---

#### TC-OWN-018 — Shared ownership is mentioned but not specified

The implementation discussion permits reference counting, and the ownership-state list includes shared atomic and shared locked states. Yet the core language does not define a shared-owner type.

Impact: retain may be interpreted as implicit shared ownership, creating hidden runtime cost and cycles.

Required correction: Shared ownership must use an explicit standard type or core capability, such as:

rg1 shared<T> weak<T> 

Cycle behavior, atomicity, destruction, and upgrade failure belong to that type’s specification.

Ordinary retain must not silently convert unique ownership into reference counting.

---

#### TC-OWN-019 — Ownership across denial paths is undefined

Consider:

rg1 store move profile in cache 

when storage can fail.

Does a denied operation:

- Return ownership to the caller?
- Consume and destroy the value?
- Leave ownership indeterminate?
- Partially store it?

Similarly:

rg1 let result = transfer(move packet) 

needs a defined ownership result on denial.

Required correction: Fallible consuming operations must specify recovery ownership.

A robust pattern is:

text outcome<Success, TransferFailure<T>> 

where the denial carries the unconsumed value when transfer did not occur.

Example:

rg1 choice StoreError<T> {     capacity_exceeded(T)     unavailable(T) } 

No operation may leave ownership ambiguous after denial.

---

#### TC-OWN-020 — Case capture and lexical lifetime need formal capture rules

The specification states:

- A case cannot capture a local borrow that expires before completion.
- Detached cases must own or retain captured values.

However, it does not define:

- Whether capture is explicit
- Whether bindings are copied, moved, or borrowed
- Whether case declarations capture at declaration or spawn
- Whether repeated spawns reuse captures
- Whether a case may capture a writable borrow
- Whether output variables are initialized before spawn

Required correction: Case capture should be explicit through its access contract:

rg1 case worker {     use read configuration     use move packet     output result } 

Capture should occur at spawn time.

For each declared resource, the compiler should determine:

text read          shared borrow through case completion write         exclusive borrow through case completion move          ownership transfer into the case atomic        shared atomic capability locked        shared lock capability snapshot      immutable owned or versioned view transactional transactional capability 

Undeclared capture should be a compile-time denial.

---

#### TC-OWN-021 — Cancellation cleanup is insufficiently defined

Cases may be cancelled, and case-local storage is released according to ownership policy, but cancellation timing and cleanup obligations are unclear.

Questions include:

- Is cancellation cooperative or immediate?
- Are destructors guaranteed to run?
- Can a case be cancelled while holding a lock?
- What happens to moved inputs?
- Are transactional changes rolled back?
- What happens to partially initialized outputs?
- Can cleanup block or deny?

Required correction: Define cancellation as structured control flow, not arbitrary thread termination.

Recommended baseline:

> Cancellation requests are observed at defined cancellation points. A cancelled case executes deterministic cleanup for initialized local values, releases guards, discards unpublished outputs, and rolls back uncommitted transactional state. Values moved into the case are destroyed unless explicitly returned through a cancellation outcome.

Safety-critical profiles may require bounded cancellation latency.

---

#### TC-OWN-022 — late initialization interacts with destruction and borrowing

The language supports:

rg1 late connection: Connection 

but does not define:

- The initialization state across branches
- Whether late fields are permitted in records
- Whether a partially initialized value can be moved
- Whether destruction skips uninitialized values
- Whether a borrow can be created before definite initialization
- Whether initialization may happen more than once

Required correction: A late binding must have a compiler-tracked initialization bit in semantic analysis, though no runtime bit is required when statically proven.

Rules should include:

text A late binding begins uninitialized.  It may be initialized exactly once unless also declared var.  It cannot be read, borrowed, moved, dismissed, or freed before initialization.  Cleanup applies only if initialization completed.  All control-flow paths reaching a use must prove initialization. 

---

#### TC-OWN-023 — Copyability is not defined

Examples use copy, but the type system does not define which types may be copied.

Questions include:

- Are integers implicitly copyable?
- Are records copyable when all fields are copyable?
- Are file handles noncopyable?
- Can users implement copy behavior?
- Is copying potentially fallible?
- Can a copied value preserve identity?
- Does assignment copy or move?

Required correction: Define at least two traits or intrinsic properties:

text Copy:     duplication is infallible, implicit where allowed, and has no custom side effects.  Clone:     explicit duplication that may allocate or deny. 

Assignment and parameter passing should follow clear rules:

text Copy values may be copied.  Non-Copy owned values are moved when passed by ownership.  Borrowing never copies ownership. 

Raygen should not allow arbitrary user-defined implicit copy side effects.

---

#### TC-OWN-024 — Assignment ownership semantics are missing

Initialization uses:

rg1 let b = a 

but it is unclear whether this copies, moves, or borrows a.

Mutation uses:

rg1 set target <- source 

but replacement behavior is also undefined.

Required correction: Define assignment by source category:

text For Copy values:     assignment copies.  For owned non-Copy values:     assignment moves unless `copy` or `clone` is explicit.  For borrowed values:     assignment reborrows within valid lifetime constraints. 

Replacing an initialized owned destination must destroy its previous value before or after transfer according to a defined exception-safe sequence.

---

#### TC-OWN-025 — Self-assignment and overlapping assignment are unspecified

Operations such as:

rg1 set value <- value set slice[1 ..< 5] <- slice[0 ..< 4] 

may involve self-move or overlapping memory.

Required correction: Self-move of a non-Copy value should either be rejected or defined as a no-op before destruction.

Bulk overlapping assignment must specify whether it behaves like memmove, requires disjointness, or produces a denial.

No optimizer may assume non-overlap without proof.

---

#### TC-OWN-026 — Direct and foreign access can invalidate ownership proofs

The core states that pointer provenance and foreign-interface validation occur before proof removal, but it does not define how unsafe operations affect existing ownership facts.

Example:

rg1 let view = readref buffer[0] direct {     foreign_mutate(buffer.address) } emit view 

Required correction: Direct or foreign operations must declare their memory effects.

For example:

rg1 foreign c function mutate(     data: address<byte>,     length: usize ) effects {     write memory(data, length)     retain none } 

Absent an audited contract, the compiler must conservatively invalidate aliasing, initialization, and provenance facts for reachable memory.

---

#### TC-OWN-027 — Runtime detection of use-after-free conflicts with safe-code guarantees

The specification says use after free is a compile-time error when detectable and a runtime trap otherwise.

That implies safe code may contain dynamically dangling references.

This is weaker than the stated ownership model and complicates optimization.

Recommended correction: In ordinary safe Raygen, well-typed references must never outlive their referents. Use-after-free should be impossible without:

- direct
- Foreign code
- A dynamically checked unsafe handle type
- Runtime corruption

Runtime guards are appropriate at those boundaries, not as the normal fallback for unresolved safe-reference lifetimes.

“Unproven lifetime” should generally be a compile-time denial, unlike an unproven bounds condition that can safely use a runtime check.

---

#### TC-OWN-028 — Memory directives need effect classification

The derivation rules prohibit visible lifetime effects in pure derivations, but the specification does not formally classify:

text retain store dismiss free allocate region creation borrowing destruction 

Required correction: Add memory and ownership effects to the effect system:

text memory.allocate memory.deallocate lifetime.extend lifetime.end storage.read storage.write ownership.move ownership.share foreign.retain 

Pure derivations may perform only non-observable temporary allocation when the allocation cannot escape and allocation failure is semantically impossible or abstracted away.

---

### Recommended ownership transition model

A minimal compiler-visible transition model could use:

text Owned live value:     read borrow        -> owned + temporary shared borrow     write borrow       -> owned + temporary exclusive borrow     move               -> source unavailable, destination owned     dismiss            -> destroyed, binding unavailable     free               -> destroyed and deallocated, binding unavailable     store move         -> destination owned, source unavailable     retain move        -> longer-lived destination owned, source unavailable  Borrowed read value:     may be read or reborrowed read     may not be mutated, moved, dismissed, or freed  Borrowed write value:     may be read or mutated     may be reborrowed under exclusivity rules     may not be moved, retained, or freed through the borrow  Moved, dismissed, or freed binding:     no further access 

Storage, synchronization, and region lifetime remain orthogonal properties.

### Status

Ownership checking is not yet conformable.

A safe bootstrap subset can prohibit retained aliases, shared ownership, partial moves, stored references, returned borrows, and dynamic lifetime checks. Full Raygen 1 requires formal ownership transitions, destruction rules, typed storage capabilities, region lifetime ordering, cancellation cleanup, copy/move semantics, and explicit foreign-memory effects.



## 5. Concurrency, Cases, Layers, Cancellation, and Determinism

### Overall finding

Raygen’s concurrency model is one of its strongest architectural ideas: lexical cases, declared access contracts, publication on completion, ordered layers, and explicit merges form a coherent structured-concurrency foundation.

The current specification does not yet define enough operational detail for interoperable runtimes. The largest gaps concern case identity, spawn semantics, completion policies, cancellation, retries, output ownership, conflict detection, detached execution, layer failure, and the exact relationship between schedule determinism and legal serial execution.

### Critical issues

#### TC-CONC-001 — A case declaration and a case execution instance are not distinguished

Examples use:

rg1 case worker {     ... }  spawn case worker wait worker 

The name worker may refer to:

- A lexical case declaration
- One active execution instance
- A reusable task template
- A statically unique task
- The latest spawned instance

This becomes ambiguous when a case is spawned more than once:

rg1 spawn case worker spawn case worker wait worker 

Required correction: Distinguish declarations from handles.

Recommended model:

rg1 case worker {     ... }  let first = spawn worker() let second = spawn worker()  wait first wait second 

Each spawn creates a unique case instance and returns a typed handle:

text case_handle<AllowedOutput, DenialType> 

A case declaration may be reusable unless explicitly declared single-use.

---

#### TC-CONC-002 — Spawn timing and capture timing are undefined

The specification does not state whether a case begins:

- Immediately at spawn
- When a scheduler selects it
- At case-group activation
- At the next layer boundary
- Lazily when waited upon

It also does not clearly state whether captured inputs are evaluated at declaration or spawn.

Required correction:

> Case arguments and access captures are evaluated atomically at spawn. Ownership transfers and borrow acquisition occur before the new case becomes runnable. The case may begin execution at any time after successful spawn, subject to group and layer constraints.

Spawn itself may deny if dynamic resource acquisition or runtime conflict checking fails.

---

#### TC-CONC-003 — Case outputs lack a formal type and initialization model

Cases declare:

rg1 output users set users <- database.read_users() 

The specification does not define:

- The declared type of users
- Whether outputs must be listed with types
- Whether outputs are single-assignment
- Whether an output may be mutable internally
- Whether every successful path must initialize every output
- Whether outputs can borrow case-local data
- Whether outputs are copied or moved at publication

Required correction: Require typed output declarations:

rg1 case load_users {     output users: listing<User>      set users <- database.read_users() } 

Recommended rules:

text Every ordinary output must be definitely initialized on every allowed completion path.  Outputs are case-local until publication.  Publication moves the output value into the case result.  An output cannot contain references to case-local storage unless that storage is moved with the output into a valid longer-lived owner. 

Multiple outputs should lower to a record or tuple result.

---

#### TC-CONC-004 — Case denial typing is missing

A case has a denial state, but no syntax declares its denial type.

Open questions include:

- Can a case deny with any error produced inside it?
- Is denial type inferred?
- Can different paths deny with different types?
- Does a group aggregate heterogeneous denials?
- How is cancellation represented?

Required correction: Give cases an explicit result contract:

rg1 case load_users     -> allow<listing<User>>     deny<DatabaseError> {     ... } 

For named outputs:

rg1 case load_users     output users: listing<User>     deny<DatabaseError> {     ... } 

Cancellation should be distinct from ordinary denial unless explicitly mapped into the denial type.

---

#### TC-CONC-005 — Ordinary denial and cancellation are conflated

The specification states that cancellation propagates through a group, while denied cases discard unpublished outputs. It does not define whether cancellation:

- Is a denial
- Produces a separate terminal state
- Can be caught
- Can be retried
- Counts toward quorum
- Is included in collected denials
- Can occur after successful completion

Recommended correction: Define terminal case states:

text allowed denied(D) cancelled(CancellationReason) trapped 

A group policy may map cancellation into a typed denial, but cancellation should remain operationally distinct.

A completed case cannot later become cancelled.

---

#### TC-CONC-006 — Completion policy any is ambiguous

Supported completion policies include:

text all any first_allowed first_completed quorum 

any is not distinguishable from either first_allowed or first_completed without more rules.

Possible meanings include:

- Return when any case completes, regardless of result
- Return when any case allows
- Require any one case eventually to allow
- Wait until one result becomes externally observable

Required correction: Remove any or assign it one precise meaning.

Recommended set:

text all:     wait for every noncancelled case.  first_completed:     complete when the first case reaches any terminal state.  first_allowed:     complete when the first case allows;     deny only when no remaining case can allow.  quorum N:     complete when N cases allow;     deny when reaching N allowed results becomes impossible. 

---

#### TC-CONC-007 — Tie-breaking for simultaneous completion is undefined

Under:

text first_completed first_allowed quorum 

multiple cases may become terminal without an observable ordering.

Without a tie-break rule, schedule determinism cannot be guaranteed.

Required correction: Every deterministic profile must use a stable tie-break key, such as:

1. Declared case order
2. Explicit case priority
3. Monotonic spawn sequence

Suggested rule:

> When multiple qualifying completions are observed at the same logical synchronization point, the earliest declaration order wins unless the group declares another deterministic ordering.

Under nondeterministic profiles, implementation-selected winners may be permitted.

---

#### TC-CONC-008 — Quorum semantics are incomplete

A quorum policy must declare a count, but the specification does not define:

- Whether quorum counts allowed cases or all completed cases
- Whether denied cases reduce the required count
- Whether cancelled cases count
- What result is returned
- Whether remaining cases are cancelled
- Whether order of selected results is deterministic

Recommended correction:

text completion quorum N:     succeeds after N allowed case results are available;     returns those results in declared case order;     denies when fewer than N cases can still allow. 

The group must separately declare whether remaining cases continue or are cancelled after quorum is reached.

---

#### TC-CONC-009 — Failure policies can conflict with completion policies

Supported failure policies include:

text continue cancel_remaining deny_group collect_denials retry 

These are not mutually exclusive categories.

For example:

text completion first_allowed failure deny_group 

raises a question: does the first denial immediately deny the group, or may other cases still allow?

Similarly:

text completion quorum 3 failure continue 

needs rules for when success becomes impossible.

Required correction: Split group policy into orthogonal dimensions:

text completion condition denial handling post-completion handling retry policy cancellation policy result ordering 

Example:

rg1 cases processing policy {     completion first_allowed     denial collect     after_completion cancel_remaining     result_order declaration } 

---

#### TC-CONC-010 — retry is not a complete failure policy

Retry requires at least:

- Maximum attempts
- Backoff
- Retryable denial classification
- Input ownership rules
- Idempotency assumptions
- Cancellation interaction
- Randomness and timing behavior
- Output reset behavior

Required correction: Define retry as a separate structured policy:

rg1 retry {     attempts 3     when transient_error     backoff fixed 10:ms } 

A case can be retried only when its consumed inputs can be recreated, retained, copied, or returned on denial.

Under schedule determinism, retry timing cannot affect observable ordering unless explicitly permitted.

---

#### TC-CONC-011 — Case-group result typing is absent

A group may contain heterogeneous cases:

rg1 case load_users case load_orders case load_settings 

The type of:

rg1 wait processing 

or any group result is not defined.

Required correction: Define result shapes by completion policy.

For example:

text all:     record containing each case result.  first_allowed:     tagged choice identifying the winning case and its output.  first_completed:     tagged choice containing allowed, denied, or cancelled terminal result.  quorum:     ordered collection of tagged allowed results.  collect_denials:     ordered collection of tagged denials. 

The compiler should generate a concrete structural result type.

---

#### TC-CONC-012 — Access contracts lack resource-range semantics

The current conflict rule refers to a resource as a whole:

rg1 use write shared_count 

For arrays, tables, files, graphs, and address ranges, this may be unnecessarily coarse.

Example:

rg1 case left {     use write values[0 ..< midpoint] }  case right {     use write values[midpoint ..< values.length] } 

Required correction: Access contracts should apply to typed resource regions.

The compiler may prove disjointness statically or generate a runtime conflict guard for dynamic ranges.

Resource identity must include:

text base object subobject or range access mode provenance lifetime version, when applicable 

---

#### TC-CONC-013 — Dynamic resource-conflict denial is under-specified

Section 22 allows dynamic conflicts to be synchronized or denied at runtime. It does not define:

- When acquisition occurs
- Which case is denied
- Whether acquisition is fair
- Whether a case waits first
- Whether denial is typed
- Whether partial work may have occurred
- How deadlocks are prevented

Recommended correction: Dynamic conflict handling must occur before the case performs observable work unless a transaction provides rollback.

A resource-acquisition policy should be explicit:

text wait deny ordered_lock transactional_retry 

For deterministic profiles, acquisition order should use a stable total resource order.

---

#### TC-CONC-014 — locked does not identify a specific lock

A contract such as:

rg1 use locked state 

does not establish which lock protects state.

Different cases could use different locks and falsely appear synchronized.

Required correction: Associate the resource with a lock capability:

rg1 use locked state by state_lock 

or define:

rg1 locked<SharedState, StateLock> state 

The compiler must verify that all locked access to the same protected location uses the same synchronization domain.

---

#### TC-CONC-015 — Lock acquisition order and deadlock behavior are unspecified

Raygen promises concurrency correctness, but multiple locks can still deadlock without an ordering rule.

Required correction: Define at least one of:

- Static global lock ordering
- Runtime deadlock detection
- Scoped multi-lock acquisition
- Explicit permission for potentially blocking lock order

Recommended safe form:

rg1 lock ordered [account_a.lock, account_b.lock] {     ... } 

The runtime acquires locks in a deterministic canonical order.

Safety-critical profiles should reject unbounded or unverified lock acquisition cycles.

---

#### TC-CONC-016 — Snapshot semantics are insufficient

snapshot is described as an immutable versioned view, but the specification does not define:

- Snapshot creation point
- Storage lifetime
- Whether creation may deny
- Whether the view is deep or shallow
- How referenced mutable objects behave
- Version identity
- Whether snapshots are consistent across multiple resources

Required correction: Define snapshot as a capability produced from a versioned resource.

A multi-resource consistent snapshot requires an explicit snapshot domain or transaction. Ordinary per-resource snapshots must not imply global consistency.

---

#### TC-CONC-017 — Transactional access lacks a transaction model

transactional access is described as isolated mutation committed according to policy, but no policy defines:

- Isolation level
- Conflict detection
- Commit ordering
- Rollback
- Nested transactions
- Retry
- Side effects
- Publication
- Denial types
- Interaction with external I/O

Required correction: Keep transaction syntax out of the core until a separate privileged specification defines it, or define a minimal model.

A viable baseline is optimistic serializable transactions over declared transactional resources:

text Reads observe one version snapshot.  Writes remain private.  Commit validates versions.  Successful commit publishes atomically.  Conflict produces a typed denial.  External irreversible effects are forbidden before commit. 

---

#### TC-CONC-018 — Output publication is not atomic for multiple outputs

The specification says outputs become visible after successful completion, but a case may have several outputs.

It must be clear whether publication happens:

- Individually
- In declaration order
- Atomically as one result
- Through separate channels

Required correction: Ordinary case outputs must publish atomically as one result aggregate. No observer may see only a subset.

Large outputs may be physically transferred incrementally, but visibility must remain atomic unless streaming is explicitly declared.

---

#### TC-CONC-019 — Output retrieval and ownership transfer are unclear

“Completion happens before retrieval” defines ordering but not ownership.

Questions include:

- May outputs be retrieved more than once?
- Does retrieval move or borrow?
- Can multiple consumers read an output?
- Where is the output stored before retrieval?
- What happens when a handle is dropped without retrieval?

Recommended correction:

text Unique output:     first retrieval moves the output; later retrieval is invalid.  Shared read result:     must use an explicit shared or snapshot result type.  Dropping an unretrieved handle:     destroys or releases the stored output according to its ownership rules. 

A convenience wait may return the result directly.

---

#### TC-CONC-020 — Dropping a case handle is undefined

Structured concurrency requires a rule for:

rg1 let task = spawn worker() # task is never waited 

Possible behaviors include:

- Implicit wait at scope exit
- Cancellation
- Detachment
- Compile-time denial
- Runtime leak

Required correction: For non-detached cases:

> A case handle must be consumed by wait, group completion, explicit cancellation followed by wait, or structured scope exit. Scope exit implicitly cancels unfinished child cases and waits for cleanup.

Silently detaching is forbidden.

---

#### TC-CONC-021 — Detached cases undermine lexical structured concurrency

Detached cases may outlive their creators, but requirements such as “explicit output destination” and “explicit cancellation authority” remain abstract.

Required correction: Detached spawn should require a runtime supervisor capability:

rg1 let handle =     spawn detached telemetry()     under application_supervisor 

The supervisor owns:

- Lifetime
- Cancellation
- Failure reporting
- Output routing
- Shutdown behavior

A detached case cannot capture lexical borrows or unsupervised resources.

---

#### TC-CONC-022 — Detached denial handling is missing

A detached case may deny after its creator has exited. The specification does not say where that denial goes.

Required correction: Every detached case must declare one of:

text supervisor reporting typed channel destination explicit ignore policy permitted by deployment profile process termination policy 

Unchecked detached denial must be a compile-time error under the default safety policy.

---

#### TC-CONC-023 — Layer syntax does not declare access contracts

Examples contain:

rg1 parallel physics.update() parallel animation.update() 

but layer rules require shared mutation to use access contracts.

It is unclear whether access is inferred from function signatures or must be declared at each parallel operation.

Recommended correction: Access should be derived from function parameter and effect contracts, with optional explicit bindings:

rg1 parallel physics.update(     write physics_state,     read frame_input ) 

Hidden global effects must be rejected unless declared by the called function’s effect signature.

---

#### TC-CONC-024 — “Parallel may execute serially” needs a stronger equivalence rule

The specification permits serial execution when necessary. This is correct only if:

- Parallel operations have no dependence on wall-clock overlap
- Cancellation behavior remains equivalent
- Failure selection remains equivalent
- Channel interactions remain legal
- Timing-sensitive operations are excluded
- Deadlock behavior does not change

Required correction: Define the semantic reference execution for a layer.

Recommended rule:

> A conforming implementation may execute parallel operations in any legal schedule, including serially, provided all observable results are allowed by the active determinism and timing policies.

For schedule determinism, every legal schedule must produce the same semantic result.

Timing, external I/O ordering, and progress guarantees require separate effects or policies.

---

#### TC-CONC-025 — Layer failure policy is referenced but not defined

Layer Rule 8 says denial behavior is governed by the layer-group failure policy, but no syntax or supported policy set is given.

Required correction: Layers need explicit policy dimensions similar to case groups:

rg1 layers frame policy {     failure cancel_layer     publication atomic     merge on_success } 

Possible rules include:

text cancel_layer complete_layer deny_group collect_denials rollback_transactional 

The specification must define whether later layers execute after a denial.

---

#### TC-CONC-026 — Layer output publication and case output publication may conflict

Case outputs publish at case completion, while layer outputs publish at the layer boundary.

A case running inside a layer may complete before the layer does.

Required correction: Define nested visibility:

> Outputs produced by work inside a layer may become visible to other operations in the same layer only through explicitly synchronized communication. Ordinary layer outputs remain unpublished outside the layer until the layer boundary, even if their producing case has completed.

Case completion still establishes ordering for direct handle retrieval where retrieval is permitted inside the layer.

---

#### TC-CONC-027 — Merge is both a barrier body and a reduction directive

Examples use:

rg1 merge {     present() } 

and:

rg1 merge ordered partials into total 

These are semantically different:

- Final sequential phase
- Reduction operation
- Conflict-resolution operation
- Output publication point

Required correction: Define separate grammar categories or one formal generalized merge model.

Suggested split:

text merge block:     final post-layer block.  reduce:     combines values with ordering and algebraic contracts. 

Keeping both under merge is possible but requires unambiguous productions and semantics.

---

#### TC-CONC-028 — Associativity requirements are insufficient for parallel reduction

The specification correctly notes that floating-point addition is not assumed associative, but parallel reduction also depends on:

- Identity
- Closure
- Purity
- Determinism
- Side effects
- Commutativity, when order is not fixed
- Overflow policy

Required correction: Define reduction contracts:

text ordered reduction:     requires a deterministic sequence and a pure binary operation.  tree reduction:     requires associativity and identity.  unordered reduction:     requires associativity, commutativity, identity, and permitted nondeterminism. 

A user assertion such as requires associative(add) must either be compiler-proven, trusted only in direct, or backed by a declared law with conformance responsibility.

---

#### TC-CONC-029 — Stable denial collection order is not fully defined

Schedule determinism requires stable denial collection order, but no ordering rule is given.

Required correction: Collected case denials should be ordered by a stable key such as declaration order, not completion time.

Each collected denial should include:

text case identity attempt number denial value logical completion sequence, when relevant 

Wall-clock timestamps must not determine semantic order.

---

#### TC-CONC-030 — Channel semantics are too limited for determinism claims

The memory model states that send happens before the corresponding receive and lists:

text fifo priority unordered 

It does not define:

- Multiple sender ordering
- Multiple receiver selection
- Buffering
- Rendezvous channels
- Closure
- Cancellation
- Backpressure
- Failed send ownership
- Priority tie-breaking

Required correction: A separate channel specification is required.

For FIFO channels, define whether FIFO is:

- Per sender
- Global across all sends
- Based on synchronization order
- Based on scheduler observation

Schedule determinism generally requires a stable global arbitration rule or explicit deterministic reordering.

---

#### TC-CONC-031 — Randomness and time are mentioned but not capability-scoped

Schedule determinism requires controlled random-number sources, but the core does not classify randomness, clocks, or external timing as effects.

Required correction: Define nondeterministic capabilities:

text random.read clock.monotonic clock.wall external.receive scheduler.observe 

A schedule-deterministic computation may use them only through deterministic inputs, recorded streams, or explicitly nondeterministic boundaries.

---

#### TC-CONC-032 — Semantic determinism is overbroad without environmental boundaries

“The same inputs produce the same language-level results” depends on what counts as input.

Files, network responses, clocks, environment variables, hardware registers, and foreign calls must be included or excluded explicitly.

Required correction:

> Semantic determinism applies relative to the complete declared input environment, including external capability results. Undeclared external observations are prohibited under deterministic profiles.

The build and execution manifests should identify these dependencies.

---

#### TC-CONC-033 — full determinism combines incompatible target freedoms

The full profile requires bitwise numerical determinism and reproducible builds, while Raygen allows multiple declared targets.

Bitwise identity across x86-64, ARM64, GPUs, and software emulation may require a canonical numerical implementation rather than target-native behavior.

Required correction: Define whether bitwise means:

- Across executions on one target profile
- Across a declared set of targets
- Across all conforming targets

Recommended syntax:

rg1 policy determinism semantic {     level bitwise     targets [x86_64-v3, arm64-v8.5] } 

Without a target set, bitwise determinism should apply only to the selected build target.

---

#### TC-CONC-034 — Build determinism needs a declared environment model

The same source and dependencies are insufficient for reproducible artifacts if builds observe:

- Paths
- Timestamps
- Locale
- Environment variables
- Random seeds
- Toolchain subversions
- Linker ordering
- Filesystem enumeration
- Debug identifiers

Required correction: The reproducible-build specification must define a canonical build environment and permitted metadata normalization.

This belongs primarily to the package and compiler-conformance documents but is necessary for the reproducible_build semantic policy.

---

#### TC-CONC-035 — Fairness and progress guarantees are absent

The specification defines safety and ordering but not progress.

Questions include:

- Can a runnable case starve indefinitely?
- Are locks fair?
- Can a wait block forever when work is runnable?
- Does cancellation eventually take effect?
- Do channels guarantee eventual delivery?
- Are retries bounded?

Required correction: Distinguish semantic safety from optional progress profiles.

Suggested guarantees:

text No fairness:     implementation may schedule any runnable work.  Weak fairness:     continuously runnable work eventually executes.  Bounded response:     safety-critical profile with declared scheduler bounds. 

The default should not imply real-time or starvation-freedom without enforceable requirements.

---

#### TC-CONC-036 — Blocking operations inside cases and layers are not classified

A case may call I/O or wait on another case. Without restrictions, layer execution can deadlock or block an entire worker pool.

Required correction: Add effects such as:

text may_block may_wait may_suspend external_io 

Layer schedulers must account for blocking operations, and safety-critical profiles may prohibit unbounded blocking inside parallel layers.

---

#### TC-CONC-037 — Nested waits can create dependency cycles

Examples allow cases, groups, layers, spawn, and wait, but no rule prohibits:

text case A waits for B case B waits for A 

or a case waiting for its enclosing group.

Required correction: The compiler must reject statically known wait cycles.

Dynamic wait graphs require runtime cycle detection or an explicit unsafe/direct profile. Waiting on the current case or an enclosing completion scope must always be denied.

---

#### TC-CONC-038 — Case-group cancellation authority is unclear

The specification says cancellation propagates through an enclosing case group, but it does not define who may initiate cancellation.

Required correction: Cancellation must require ownership or an explicit cancellation capability associated with the case handle or group.

Read-only observers should not automatically have cancellation authority.

---

#### TC-CONC-039 — Traps and denials have different cleanup consequences but are not separated

Runtime guards may produce typed denials or traps. A trap inside a case or layer raises unresolved questions:

- Does the group cancel remaining cases?
- Are destructors run?
- Are outputs discarded?
- Is the process terminated?
- Can a supervisor recover?

Required correction: Define trap policy separately from denial policy.

A trap should normally be nonrecoverable within ordinary Raygen control flow. Runtime profiles may convert selected traps into process-level supervisor events, but not ordinary typed denials unless explicitly specified.

---

#### TC-CONC-040 — External side effects break output rollback guarantees

A denied case publishes no partial ordinary outputs, but it may already have:

- Written a file
- Sent a network message
- Updated a device
- Logged an event
- Called foreign code

Discarding ordinary outputs does not roll back those effects.

Required correction: Clarify that output atomicity applies only to ordinary case result publication.

Cases with external effects must declare them. Transactional publication guarantees require transactional effect providers or compensating actions.

The specification must not imply general rollback of irreversible effects.

---

### Recommended concurrency reference model

A minimal operational model could be:

text 1. A case declaration defines reusable code and a typed result contract. 2. Spawn evaluates arguments, acquires declared access rights, and returns a unique handle. 3. The spawned case owns its local state and moved inputs. 4. Ordinary outputs remain private until allowed completion. 5. Completion publishes one atomic result aggregate. 6. Denial publishes only the typed denial. 7. Cancellation performs structured cleanup and publishes a cancelled state. 8. Wait joins the case and establishes happens-before ordering. 9. Non-detached children are cancelled and joined before their scope exits. 10. Case groups aggregate typed terminal results according to orthogonal policies. 11. Layers impose barriers; external layer outputs publish only at boundaries. 12. Merge operates only on fully published inputs. 

### Status

The conceptual model is strong but runtime conformance remains blocked.

A restricted implementation can support single-spawn lexical cases, typed outputs, completion all, failure cancel_remaining, no retries, no detachment, no transactions, and deterministic declaration-order result collection. Full Raygen 1 requires a formal case-handle model, terminal states, orthogonal group policies, structured cancellation, resource-range contracts, layer failure semantics, channel arbitration, and environmental determinism rules.



## 6. Atomics and the Formal Memory Model

### Overall finding

Raygen defines the right high-level ingredients for a portable memory model: atomic operations, data-race prohibition, happens-before, volatile separation, case completion, layer barriers, locks, channels, and target-independent guarantees.

The specification is not yet precise enough for compiler or runtime conformance. The main gaps concern the definition of executions, non-atomic visibility, atomic modification order, compare-exchange behavior, release sequences, synchronization scope, volatile ordering, mixed atomic and non-atomic access, lock-based atomic emulation, and the consequences of data races.

### Critical issues

#### TC-MEM-001 — The memory model lacks a formal execution vocabulary

The specification defines locations, reads, writes, and happens-before informally, but it does not define the relations needed to evaluate a concurrent execution.

At minimum, a formal model needs:

text sequenced-before reads-from modification-order synchronizes-with happens-before dependency or address relation, if used conflicting access visible side effect 

Impact: Two compilers may emit different reorderings or atomic implementations while both claiming to preserve the same informal rules.

Required correction: Add a formal execution model or adopt a referenced established model with Raygen-specific extensions.

A practical baseline can be derived from the C++/Rust-style atomics model, but Raygen must define its own observable semantics rather than relying on implementation folklore.

---

#### TC-MEM-002 — The consequences of a data race are inconsistent

The specification says:

- Safe Raygen forbids data races.
- A statically provable race is a compile-time denial.
- A dynamically detected race is a runtime denial under checked execution.
- Runtime race risk may become a compile-time denial in safety-critical profiles.

This leaves unresolved what happens when a race occurs but is not dynamically detected.

Possible interpretations include:

- Undefined behavior
- A required trap
- Arbitrary but memory-safe behavior
- A language denial
- An implementation defect

Required correction: Choose one semantic rule.

Recommended rule:

> A well-formed safe Raygen program must be data-race-free. A conforming implementation may use static and dynamic enforcement, but execution of safe code must not rely on undefined behavior caused by an undetected race. If the implementation cannot enforce a dynamically possible race safely, compilation must be denied or the access must be synchronized.

This is stronger than C/C++ undefined behavior and matches Raygen’s stated safety goals.

Direct or foreign code may opt into a weaker unsafe execution domain, but that boundary must be explicit.

---

#### TC-MEM-003 — Ordinary non-atomic read visibility is not defined

Happens-before is listed, but the specification does not state which write an ordinary read is permitted to observe.

Example:

rg1 var data: u32 = 0 atomic<bool> ready = false  case writer {     set data <- 42     ready.store(true, order release) }  case reader {     if ready.load(order acquire)     {         emit data     } } 

The intended result is presumably that data == 42 is visible after the acquire observes the release.

Required correction: Define visible side effects:

> An ordinary read must observe a write that is not happens-before-overwritten by another write and that is ordered consistently with the execution’s happens-before relation.

The model should explicitly validate the release/acquire publication pattern.

---

#### TC-MEM-004 — Atomic modification order is missing

Each atomic location requires a total order of modifications.

Without it, the behavior of:

rg1 counter.store(1, order relaxed) counter.store(2, order relaxed) 

observed from other cases is undefined beyond atomicity.

Required correction:

> All modifications of one atomic object participate in a single total modification order consistent with happens-before. Every atomic load observes a value permitted by that modification order and its memory-order constraints.

This order is per atomic object, not a global total order except where sequential consistency requires one.

---

#### TC-MEM-005 — Sequential consistency is under-specified

The default order is sequential, but the specification does not define what sequential consistency adds beyond acquire and release.

Required correction: Define a single global order over sequentially consistent atomic operations and fences, consistent with:

- Per-thread sequenced-before
- Atomic modification order
- Happens-before
- The value observed by sequential loads

Without this, default atomic behavior is not portable.

---

#### TC-MEM-006 — The memory-order names should align with established terminology

The order:

text sequential 

appears to mean sequential consistency, commonly named sequential_consistent or seq_cst.

Recommended correction: Retain the readable Raygen spelling if desired, but define it explicitly as sequentially consistent ordering.

For diagnostics and ABI metadata, a stable internal name such as:

text seq_cst 

may reduce ambiguity.

---

#### TC-MEM-007 — Acquire and release synchronization is too broadly stated

The specification says:

> A release operation synchronizes with an acquire operation that observes it.

This is incomplete for read-modify-write operations and release sequences.

Example:

rg1 flag.store(1, order release) flag.add(1, order relaxed) let value = flag.load(order acquire) 

Whether the acquire synchronizes with the original release depends on release-sequence rules.

Required correction: Either define release sequences or explicitly exclude them.

A conventional rule is:

> A release operation heads a release sequence consisting of following read-modify-write operations in the atomic object’s modification order. An acquire operation reading a value from that sequence synchronizes with the head release.

If Raygen wants a simpler model, it can require an acquire to observe the release operation directly, but that would be less compatible with established lock-free algorithms.

---

#### TC-MEM-008 — Compare-exchange semantics are incomplete

The specification gives:

rg1 counter.compare_exchange(expected, desired) 

and only states that failed compare-exchange cannot use release or acquire_release.

Missing details include:

- Strong versus weak compare-exchange
- Success order
- Failure order
- Return type
- Whether expected is updated on failure
- Spurious failure
- Equality comparison semantics
- Ownership behavior for desired
- Default orders
- Legality table

Required correction: Define at least:

rg1 compare_exchange_strong(     expected,     desired,     success_order,     failure_order )  compare_exchange_weak(     expected,     desired,     success_order,     failure_order ) 

Recommended return type:

text outcome<old_value, observed_value> 

or a record containing success and observed value.

The failure order must be no stronger than the success order and cannot include release semantics.

---

#### TC-MEM-009 — Read-modify-write operation results are unspecified

For:

rg1 counter.add(1, order relaxed) 

does the operation return:

- The previous value
- The new value
- void
- A typed outcome

Required correction: Specify each atomic operation’s result.

A conventional and useful rule is:

text fetch_add returns the previous value. add_fetch returns the new value. 

Using the name add alone is ambiguous.

Raygen should either rename operations or define one stable convention.

---

#### TC-MEM-010 — Atomic initialization is not separated from atomic store

Example:

rg1 atomic<u32> counter = 0 

Initialization before publication is not the same as a sequentially consistent store.

Required correction:

> Initialization of a newly created atomic object establishes its initial value before the object becomes accessible to concurrent execution. It is not an atomic operation in the modification order unless the object is already shared.

This matters for performance and formal reasoning.

---

#### TC-MEM-011 — Mixed atomic and non-atomic access is not addressed

Example:

rg1 atomic<u32> counter set counter <- 1 

or a raw pointer view of atomic storage could bypass atomic semantics.

Required correction:

> A memory location declared atomic may be accessed only through atomic operations, except during exclusive initialization before publication or through explicitly defined direct-memory operations that preserve atomic representation requirements.

Likewise, ordinary storage must not be accessed atomically through a cast unless alignment, size, lifetime, and atomic construction requirements are satisfied.

---

#### TC-MEM-012 — Atomic-capable types are undefined

The specification says T must be atomic-capable or supported by a lock-based implementation.

It does not define whether the following are atomic-capable:

text bool u128 u256 addresses nominal wrappers records choices floating-point values rayword 

Required correction: Define an AtomicValue eligibility rule.

A baseline could permit:

- Integers of supported widths
- Boolean
- Addresses
- Transparent nominal wrappers over eligible types

Larger records should require a separate synchronized container rather than pretending to be primitive atomics.

---

#### TC-MEM-013 — Lock-based atomic emulation can violate lock-free expectations

The specification permits a conforming lock-based implementation for unsupported atomic types. This changes:

- Progress guarantees
- Signal safety
- Interrupt safety
- Reentrancy
- Scheduler interaction
- Deadlock potential
- Real-time behavior

Required correction: Separate semantic atomicity from progress classification.

Atomic implementations should report:

text lock_free wait_free blocking_emulated unsupported 

Deployment policies must be able to require:

rg1 require atomic<u64> lock_free 

Safety-critical and interrupt contexts may prohibit blocking emulation.

---

#### TC-MEM-014 — Atomic alignment and layout requirements are incomplete

An atomic object generally requires target-specific alignment and may have a representation distinct from ordinary T.

Section 53 says layout cannot violate atomicity, but no exact rule is given.

Required correction:

> atomic<T> has its own size, alignment, and ABI classification. It is not required to have the same representation as T. Packed layouts may not reduce its alignment below the target-required atomic alignment.

Embedding an atomic inside a packed unit should be rejected unless a target profile explicitly supports it.

---

#### TC-MEM-015 — Atomic object lifetime rules are missing

An atomic must not be destroyed while another case may access it.

Questions include:

- Can atomics be moved after publication?
- Can they be copied?
- Can a region containing an atomic be freed while waiters exist?
- Can a lock-based atomic move without invalidating its internal lock identity?

Required correction:

text Atomic objects are non-Copy by default.  A published atomic object cannot be moved unless the compiler proves exclusive access.  Its storage must outlive every possible atomic access.  Lock-based atomic identity is tied to the object’s stable address unless its implementation specifies otherwise. 

---

#### TC-MEM-016 — Fences are referenced indirectly but not exposed

Section 42 permits fences, but Raygen does not define a fence operation.

Some lock-free algorithms require a standalone acquire, release, or sequentially consistent fence.

Required correction: Either:

- Add atomic fence operations, or
- Explicitly state that Raygen 1 does not expose standalone fences and cannot directly express algorithms requiring them.

Suggested syntax:

rg1 atomic fence(order acquire) atomic fence(order release) atomic fence(order sequential) 

relaxed fences should be invalid.

---

#### TC-MEM-017 — Compiler reordering constraints need observable-event definitions

The compiler must preserve:

- Denial behavior
- Ownership transitions
- Direct-memory requirements
- Volatile order

However, “single-thread observable behavior” is not defined.

Questions include:

- Are allocation addresses observable?
- Is destructor timing observable?
- Are diagnostic events observable?
- Is logging an effect?
- Can two independent denial-producing checks be reordered?
- May a trap be observed before an earlier ordinary write?

Required correction: Define observable events, including:

text volatile access atomic operation external I/O foreign call with declared effects denial publication trap destructor with visible effects case and layer publication synchronization operation 

Optimizations may reorder only when the resulting trace remains equivalent under the active semantic policy.

---

#### TC-MEM-018 — Denial ordering constrains speculative execution but is not formalized

The specification says inlining and reordering cannot alter observable denial propagation.

Example:

rg1 let a = checked_access(first) let b = checked_access(second) 

If both deny, source order may determine which denial becomes observable.

Required correction: Define whether Raygen has source-ordered denial priority.

A deterministic baseline is:

> In a single sequential execution path, the earliest operation in sequenced-before order whose denial is propagated determines the observed denial unless a construct explicitly aggregates or reorders denials.

Speculative checks may execute early internally, but they may not publish a later denial before an earlier required operation is resolved.

---

#### TC-MEM-019 — Memory locations for overlapping fields need more detail

The specification says distinct nonoverlapping fields are distinct locations and packed bitfields can share a location.

It does not address:

- Unions or variant payload overlap
- Zero-sized fields
- Padding bytes
- Misaligned fields
- Array elements
- Slice metadata
- Atomic subobjects
- Memory-mapped register blocks

Required correction: Extend the location model:

text Each scalar object and nonoverlapping scalar subobject is a memory location.  Each array element is a distinct location unless its representation packs elements.  Overlapping variant storage shares locations.  Padding is not a value location but may be part of object representation.  Concurrent access to different logical fields that share packed storage conflicts at the shared location. 

---

#### TC-MEM-020 — Bytewise access to object representation is undefined

Systems languages need rules for reading and writing object bytes.

Questions include:

- Can any object be viewed as bytes?
- Are padding bytes initialized?
- Can writing bytes create an invalid choice state?
- Does copying object representation preserve provenance?
- Can atomics be copied bytewise?

Required correction: Define safe and direct object-representation access separately.

Safe byte views should permit reading the initialized representation of explicitly byte-compatible types.

Constructing typed values from bytes should require validation unless the type is declared plain-data and every bit pattern is valid.

Atomic and lock-containing objects must not be bytewise copied.

---

#### TC-MEM-021 — Pointer provenance is referenced but not specified

Pointer provenance is central to alias analysis and safety, but the core only lists it as a checked property.

A formal model needs to define:

- Address creation
- Allocation identity
- Pointer arithmetic bounds
- One-past addresses
- Integer-to-address casts
- Address-to-integer casts
- Subobject provenance
- Lifetime invalidation
- Reuse after deallocation
- Foreign pointers
- Device addresses

Required correction: Publish a separate provenance model and reference it normatively.

A minimum rule is:

> An address carries authority derived from a live allocation or declared external memory region. Numeric equality does not by itself establish valid provenance.

Casting an integer to an address should require direct and should not create safe dereference authority without a mapping capability.

---

#### TC-MEM-022 — Allocation reuse and stale addresses are not addressed

After free, a later allocation may reuse the same numeric address. A stale address must not regain validity merely because the number matches.

Required correction: Provenance must include allocation generation or equivalent abstract identity.

Safe references to a freed allocation remain invalid after address reuse.

---

#### TC-MEM-023 — Case and layer synchronization scope needs precision

The specification states:

- Case completion happens before output retrieval.
- Case-group completion happens before the following statement.
- Layer completion happens before the next layer.

Open questions include:

- Does completion order all writes performed by the case, including external shared writes?
- Does it order only declared outputs?
- Does a group barrier include detached descendants?
- Does a layer barrier cover device or GPU memory?
- What about noncoherent accelerators?

Required correction:

> Completion barriers order all memory effects within the synchronization domain declared by the operation. Ordinary CPU memory is always included. Device, process-shared, distributed, and accelerator memory require an explicit synchronization domain and implementation mechanism.

Case completion should not make unsafely shared writes legal; it only orders writes that were already permitted by access contracts.

---

#### TC-MEM-024 — GPU and process execution need explicit memory domains

Layer implementation may use GPU queues or processes, but CPU happens-before alone does not define:

- Device cache flushes
- Host-device visibility
- Process-shared memory
- Serialization
- Distributed message delivery
- Failure recovery

Required correction: Introduce memory and execution domains:

text thread process device distributed persistent 

Cross-domain publication requires a conforming transfer or synchronization operation. A layer barrier must perform the necessary domain-specific actions before the next layer may observe results.

---

#### TC-MEM-025 — Lock synchronization does not define protected-data legality

Lock release/acquire creates ordering, but the model does not define how the compiler knows which ordinary data is protected.

Required correction: A lock capability must be associated with a protection set or guarded object.

Access outside the required lock scope must be rejected.

Example:

rg1 locked<SharedState> state 

should mean that ordinary access to state is legal only while holding the corresponding guard.

---

#### TC-MEM-026 — Lock poisoning is delegated but denial behavior still matters

The lock type specification may define poisoning, but the language must still say whether acquisition:

- Returns an outcome
- Traps
- Recovers automatically
- Produces a guard in a denied state

Required correction: Privileged lock APIs must expose poisoning and acquisition failure through typed outcomes unless a policy explicitly converts them to traps.

Memory-model synchronization applies only after successful acquisition.

---

#### TC-MEM-027 — Volatile ordering is not portable as stated

The specification says relative ordering among volatile accesses follows “the target and language rules,” but the language rules are not provided.

This leaves compilers free to differ materially.

Required correction: Define a language-level minimum:

text Volatile accesses are observable side effects.  Distinct volatile accesses are sequenced according to source sequenced-before within one execution thread unless an explicit policy permits reordering.  A volatile access is not a compiler or hardware memory fence for ordinary or atomic memory. 

Target profiles may impose additional device-specific ordering requirements.

---

#### TC-MEM-028 — Volatile width, tearing, and alignment are unspecified

A volatile read must occur, but it may still tear if the width is not naturally supported.

Required correction: Define whether volatile access is:

- Exactly one target access
- Potentially several accesses preserving total width
- Required to be naturally aligned
- Allowed to tear

For device registers, this is critical.

Suggested rule:

> A volatile access uses the declared scalar width and must not be split unless the target mapping explicitly permits split access. Unsupported or misaligned volatile widths are compile-time denials under native profiles.

---

#### TC-MEM-029 — Volatile compound objects are unsafe without field rules

Example:

rg1 address<volatile DeviceRegister> 

may suggest that every field access is volatile, but object construction, copying, and whole-record reads become ambiguous.

Required correction: Define volatile qualification propagation.

Prefer field-level volatile reads and writes over whole-record volatile copying. A volatile aggregate should not be implicitly copied through ordinary record operations.

---

#### TC-MEM-030 — Memory-mapped I/O requires stronger semantics than ordinary volatile

Device accesses may require:

- Access width
- Alignment
- Endianness
- Ordering
- Read/write side effects
- Non-repeatability
- Reserved bits
- Barriers
- Register-specific access permissions

Required correction: Model device mappings as typed capabilities, not merely volatile addresses.

For example:

rg1 map device STATUS_ADDRESS     as device_register<u32>     policy     {         access read_only         endian little         width exact     } 

Volatile remains the low-level execution property, while the device type carries protocol constraints.

---

#### TC-MEM-031 — Channel happens-before needs failed-operation rules

A successful send happens before the corresponding receive. The specification does not define ordering for:

- Failed send
- Cancelled send
- Closed channel
- Timeout
- Dropped message
- Retried delivery

Required correction: Only a successfully committed send that transfers or copies a message into the channel may synchronize with a successful receive of that same message.

Failed or cancelled sends do not transfer ownership unless the channel contract explicitly states otherwise.

---

#### TC-MEM-032 — Wait synchronization should cover terminal-state observation

The specification says task completion happens before a successful wait.

It should also define what happens for denied and cancelled cases.

Required correction:

> Any successful join or wait that observes a terminal case state acquires all memory effects sequenced before that terminal transition, subject to the case’s declared access rights and publication rules.

This applies to allowed, denied, and cancelled completion after cleanup.

---

#### TC-MEM-033 — Spawn synchronization wording is reversed

The specification states:

> Thread or task creation happens after initialization of values moved into it.

This is directionally understandable but not expressed as a synchronization relation.

Recommended correction:

> Evaluation, initialization, and ownership transfer of spawned inputs happen before the spawned case begins accessing them.

This creates a release/acquire-style publication boundary at spawn.

---

#### TC-MEM-034 — Ownership movement is not itself always a synchronization primitive

The specification says:

> Ownership movement completes before the destination accesses the moved value.

Within one thread, this is ordinary sequencing. Across threads, a move must occur through spawn, channel, synchronized queue, or another publication mechanism.

Required correction: Clarify:

> Move establishes exclusive authority but does not by itself publish memory across concurrent execution domains. Cross-case movement must occur through a construct that also establishes the required synchronization.

Otherwise a raw shared-memory handoff could be mistaken for safe publication.

---

#### TC-MEM-035 — Transaction commit ordering depends on reader protocol

The specification says commit happens before readers that observe the committed version.

This is correct but incomplete. It must define how observation is established.

Required correction: Transactional resources must expose versioned acquire semantics or equivalent synchronization. A reader that does not use the transaction protocol does not gain the commit guarantee.

---

#### TC-MEM-036 — relaxed atomics and determinism need clarification

Relaxed operations preserve atomicity and modification order but allow substantial scheduling-dependent observations.

A schedule-deterministic profile may still use relaxed atomics for counters whose exact intermediate values are not semantic, but the specification does not state this.

Required correction:

> Relaxed atomics are permitted under schedule determinism only when the compiler or programmer proves that all language-level outputs remain invariant across legal observations.

Otherwise stronger ordering or deterministic aggregation is required.

---

#### TC-MEM-037 — Out-of-thin-air values are not prohibited

A formal memory model should state that atomic and ordinary reads cannot invent values unsupported by initialization or writes.

Required correction: Explicitly prohibit out-of-thin-air values.

A read must derive from a valid write, initialization, or defined device/foreign observation.

---

#### TC-MEM-038 — Speculative execution must not expose invalid accesses

Even when the compiler proves a branch result, hardware speculation may touch invalid memory unless properly guarded.

Example:

rg1 if index < values.length {     emit values[index] } 

Language semantics require no out-of-bounds access.

Required correction: Target lowering must preserve architectural safety and any declared side-channel policy. Speculative hardware behavior that can create security-observable effects may require mitigation under hardened profiles.

This is especially relevant to constant-time and safety-critical deployments.

---

#### TC-MEM-039 — Constant-time policies need a memory-access definition

Section 31 recognizes constant-time dispatch as semantic when timing is observable, but no definition covers:

- Branches
- Cache access
- Memory addresses
- Atomic contention
- Allocation
- Speculation
- Vector lanes
- Denial timing

Required correction: Move constant-time behavior into a security profile that defines the permitted observation model.

A compiler cannot certify constant time from dispatch strategy alone.

---

#### TC-MEM-040 — The model does not distinguish safety from liveness for atomics

Atomic operations may be blocking-emulated, and compare-exchange loops may not terminate.

Required correction: Define progress as a separate property:

text blocking lock-free wait-free bounded 

The memory model guarantees safety and ordering. Deployment or API contracts define progress.

---

### Recommended memory-model baseline

Raygen 1 should normatively establish:

text 1. Every execution consists of typed events and relations. 2. Each thread or case has sequenced-before order. 3. Every atomic object has one modification order. 4. Reads observe valid writes according to happens-before and modification order. 5. Synchronizes-with edges create happens-before edges. 6. Sequentially consistent atomics participate in one global SC order. 7. Safe Raygen executions are data-race-free. 8. Safe code has no undefined behavior merely because a race escaped detection. 9. Volatile access is observable but not synchronizing. 10. Cross-domain memory requires explicit transfer and synchronization. 11. Ownership transfer grants authority; synchronization publishes memory. 12. Direct and foreign operations must declare effects or invalidate relevant proofs. 

### Status

The memory model is directionally sound but not yet implementable as a formal conformance target.

A restricted runtime can support sequentially consistent atomics, acquire/release loads and stores, typed mutexes, structured case joins, CPU-only memory, and no standalone fences or lock-free guarantees. Full Raygen 1 requires a formal execution model, atomic modification order, compare-exchange rules, release sequences, non-atomic visibility, precise volatile semantics, provenance rules, memory domains, and an explicit safe-code race guarantee.


## 7. Policies, Deduction, Optimization, and Compilation Precedence

### Overall finding

Raygen’s “meaning before optimization” principle is coherent and valuable. The policy classes, bounded deduction, guarded fallback, and required-optimization denial model provide a strong foundation for predictable compilation.

The current rules are not yet sufficient to determine one unique compiler decision. The largest gaps concern policy precedence, semantic versus safety classification, mandatory policy inheritance, proof lifecycle, guard placement, optimization feasibility, target-dependent policies, and the observable consequences of optimization reports.

### Critical issues

#### TC-POL-001 — The global authority hierarchy conflicts with policy-class precedence

Section 3 places:

text explicit semantic policies > explicit safety policies 

Section 11 says:

text A safety policy wins over an optimization policy. A semantic policy cannot be replaced by an optimization policy. 

It does not define what happens when a semantic policy conflicts with a safety policy.

Example:

rg1 policy arithmetic semantic {     allow overflow wrap }  policy safety safety {     deny arithmetic_overflow } 

Both appear applicable, but they require incompatible behavior.

Required correction: Define compatibility before precedence.

Recommended rule:

> Semantic policies define the result of an operation. Safety policies define whether that operation is permitted in the active context. A safety policy may prohibit a semantic behavior but may not silently replace it with another behavior.

Under this rule, the example is a compile-time policy conflict rather than a precedence winner.

---

#### TC-POL-002 — Policy names and policy classes are syntactically confusing

Examples include:

rg1 policy safety safety {     ... }  policy optimization optimization {     ... } 

The first identifier is apparently the policy name, while the second is its class. Repeating the same word obscures the model.

Recommended correction: Use explicit class syntax:

rg1 policy safety_profile : safety {     deny data_race }  policy release_optimization : optimization {     vectorization automatic } 

This also avoids ambiguity when a policy is named semantic, safety, or deployment.

---

#### TC-POL-003 — “Mandatory” placement is inconsistent

Examples use:

rg1 deny overflow mandatory 

and describe wider policies as “marked mandatory,” but no grammar defines whether mandatory applies to:

- One policy rule
- The entire policy block
- A policy declaration
- One value
- All nested scopes

Required correction: Define both block-level and rule-level forms, or choose one.

Suggested syntax:

rg1 mandatory policy arithmetic_contract : semantic {     overflow deny } 

For individual rules:

rg1 policy deployment : deployment {     mandatory foreign_calls audited } 

The effective mandatory boundary must be visible in the policy-resolution record.

---

#### TC-POL-004 — Mandatory policies do not define their enforcement scope

A module-level mandatory policy may prevent nested override, but can it be overridden by:

- Package policy
- Build profile
- Deployment profile
- Language edition
- Toolchain command-line options

The hierarchy says wider mandatory policies defeat nearer policies, but not which wider sources may mandate behavior.

Required correction: Define authority domains separately from lexical proximity.

A practical order is:

text Language edition invariants Certified or deployment profile mandates Package mandates Module mandates Lexical nonmandatory policies Language defaults 

A command-line option must not weaken a source or certified-profile mandate unless the build is explicitly classified as nonconforming.

---

#### TC-POL-005 — Same-scope policy conflict rules are incomplete

The specification covers mandatory and class conflicts but not:

- Repeated identical declarations
- Partially overlapping policy keys
- List-valued policies
- Independent rules inside one named policy
- Imported policies
- Policy aliases
- Conditional policies

Required correction: Resolve policies by canonical key.

Example keys:

text arithmetic.overflow arithmetic.rounding safety.bounds optimization.vectorization.width deployment.target 

For each key, the compiler computes one effective value. Nonconflicting keys compose.

Repeated identical values are permitted. Incompatible effective values produce a stable policy diagnostic.

---

#### TC-POL-006 — The nearest-scope rule is insufficient for type-scoped policies

The scope order includes:

text Expression Statement Block Function Type Module ... 

A type policy and a function policy may both apply to an operation involving that type. “Nearest” is not meaningful across lexical and nominal scopes.

Example:

rg1 type SecureCounter = u32 # type-level overflow deny  function update(...) # function-level overflow wrap 

Required correction: Separate policy dimensions:

text Lexical policy Declaration-attached policy Type-attached invariant Call-site policy Build/deployment mandate 

Type invariants should constrain all uses of the type and should not behave like ordinary lexical defaults.

---

#### TC-POL-007 — Imported module policies are not defined

A function imported from another module may have been compiled under semantic policies different from the caller’s policies.

Questions include:

- Does the callee preserve its defining policy?
- Can the caller specialize it under local policies?
- Are policies part of the function type or ABI?
- Can cross-module inlining change policy context?
- What happens with generic instantiation?

Required correction:

> Semantic policies that affect a declaration’s behavior are part of that declaration’s semantic interface. A caller’s local policy does not alter already-defined callee semantics unless the callee explicitly declares a policy parameter or is instantiated under a specified generic policy contract.

Inlining must preserve the callee’s policy environment.

---

#### TC-POL-008 — Policy effects on overload resolution are unspecified

A policy may make one operation legal and another illegal. It is unclear whether policy resolution occurs before or after overload selection.

Example:

rg1 add(left, right) 

could resolve to wrapping, checked, saturating, or user-defined addition.

Required correction: Type and overload resolution should identify semantic candidates without using optimization preferences. The effective semantic policy then selects or validates the required operation behavior.

Policy changes must not cause unrelated name-resolution instability.

---

#### TC-POL-009 — Several “semantic policies” are actually protocol requirements

The semantic-policy examples include:

- Memory ordering
- Dispatch completeness
- Denial recovery

These are not all ordinary selectable behavior at every expression.

For example, dispatch completeness is often a static well-formedness rule, while memory ordering belongs to a specific atomic operation.

Recommended correction: Distinguish:

text Semantic mode Static legality requirement Operation parameter Recovery policy 

Do not force every configurable concept into the same policy mechanism.

---

#### TC-POL-010 — Arithmetic behaviors are not all interchangeable policy values

The list:

text deny wrap saturate clamp report widen 

contains behaviors with different result types.

- wrap returns the original numeric type.
- saturate returns the original numeric type.
- deny requires failure flow or trapping.
- report likely returns extra information.
- widen may change the result type.
- clamp requires explicit bounds.

Required correction: A policy may choose only among behaviors with a statically defined expression type, or the syntax must make type changes explicit.

For ordinary operators, a safe subset is:

text deny wrap saturate 

Use explicit operations for type-changing behavior:

rg1 let result = add_report(left, right) let wide = add_widen<u64>(left, right) let bounded = add_clamp(left, right, range) 

---

#### TC-POL-011 — Required optimization semantics are too broad

The specification says required optimizations cause compilation denial when they cannot be performed safely.

This is reasonable for machine-verifiable properties such as minimum vector width, but ambiguous for:

- Inlining
- Code-size preference
- Cache placement
- Register pressure
- Profile-guided dispatch
- Task fusion

These properties may be unstable across compiler versions or impossible to guarantee after linking.

Required correction: Classify optimization requirements:

text Verifiable lowering requirements     vector width     instruction-set availability     no scalar fallback     no heap allocation     constant-time certified lowering  Best-effort preferences     inlining     register pressure     cache placement     branch layout     unrolling 

Only verifiable requirements should support require.

---

#### TC-POL-012 — Optimization feasibility point is undefined

A compiler might prove that a loop is vectorizable before register allocation, then later scalarize because of:

- Target instruction constraints
- Link-time transformation
- Code-size limits
- Backend failure
- ABI preservation
- Alignment changes

Required correction: A required optimization is satisfied only by the final emitted artifact, not merely by an intermediate plan.

The compiler must verify required lowering properties after final code generation and linking or emit a denial.

---

#### TC-POL-013 — Required vector width does not define the measured property

rg1 require width 256 

could mean:

- 256-bit source vectors
- At least one 256-bit instruction
- Every iteration uses 256-bit lanes
- Effective aggregate width of 256 bits
- No splitting into narrower vectors
- A 256-bit logical vector even when scalarized

Required correction: Define exact requirement forms, such as:

rg1 require vector logical_width 256 require vector native_width at_least 256 require vector no_scalar_fallback require vector no_split 

Logical width and physical instruction width must not be conflated.

---

#### TC-POL-014 — Target dispatch conflicts with required optimization verification

A multi-target binary may contain:

- AVX2 path
- SSE path
- Scalar path

A require width 256 policy must state whether:

- Every target path must satisfy it
- At least one runtime-selected path may satisfy it
- Deployment guarantees the required CPU capability
- Unsupported machines are denied at startup

Required correction: Bind requirements to a target set:

rg1 policy vectorization : optimization {     require native_width 256     targets x86_64+avx2 } 

A dispatching build needs explicit fallback semantics.

---

#### TC-POL-015 — Advisory optimization fallback is not consistently defined

Section 4.1 allows scalar fallback “when the policy permits fallback,” but the optimization policy examples do not define a universal fallback field.

Required correction: Every advisory or required optimization directive should have an explicit failure mode:

text fallback conservative fallback alternate deny_compile deny_runtime_target 

A default may be defined, but it must be normative.

---

#### TC-DED-001 — deduce syntax does not identify its consequence

Example:

rg1 deduce safe_index {     require index < values.length } 

It is unclear whether this:

- Merely asks the compiler to prove a fact
- Establishes an assumption
- Guards the following operation
- Defines a reusable proof object
- Produces a Boolean compile-time value
- Inserts a runtime check automatically

Required correction: Separate proof request from assumption and guarded use.

Suggested forms:

rg1 deduce safe_index {     prove index < values.length } 

and:

rg1 requires safe_index 

or:

rg1 guard index < values.length 

A failed proof request should not silently change program behavior unless fallback semantics are declared.

---

#### TC-DED-002 — require is overloaded across contracts and deduction

The specification uses require in:

- Deduction blocks
- Function preconditions
- Vectorization constraints
- Reduction-law assertions
- Optimization requirements

These have different enforcement semantics.

Required correction: Use distinct semantic categories, even if some share surface syntax:

text requires expression       caller obligation or checked precondition prove expression          deduction goal assume expression         unsafe or externally justified assumption require optimization ...  lowering obligation law associative(...)      algebraic contract 

The compiler must never interpret an unproven proof goal as an assumption.

---

#### TC-DED-003 — “Unproven means runtime guard” is not universally valid

Some properties can be dynamically checked safely:

- Bounds
- Numeric ranges
- Dynamic cast validity

Other properties cannot generally be repaired with a local guard:

- Lifetime validity
- Absence of all data races
- Trait coherence
- ABI compatibility after foreign mutation
- Lock freedom
- Purity
- Associativity
- Ownership uniqueness after arbitrary foreign escape

Required correction: Classify deduction goals by fallback capability:

text Guardable     bounds     null     dynamic type tag     finite range     runtime graph cycle  Synchronizable     dynamic resource conflict     dynamic alias separation  Non-guardable     static lifetime soundness     trait coherence     language-level purity     exact ABI identity     general race freedom 

Unproven non-guardable conditions must cause compilation denial or require an explicit direct boundary.

---

#### TC-DED-004 — Proof outcomes need reason and dependency data

The three outcomes:

text proven unproven disproven 

are useful but insufficient for incremental compilation and proof invalidation.

Required correction: Each proof result should record:

text goal outcome assumptions source facts target facts layout facts alias facts policy environment dependency versions budget exhaustion status 

An unproven result due to budget exhaustion should be distinguishable from one due to missing information.

---

#### TC-DED-005 — Proof revalidation is stated but not operationalized

Section 19.2 requires invalidation after later analyses, but Section 4 performs deduction before target lowering and does not include a revalidation stage.

Required correction: Revise the compilation precedence to include:

text Initial semantic proof Representation and target analysis Proof revalidation Runtime-guard commitment Optimization Final required-property verification 

No proof-dependent check removal becomes final before revalidation.

---

#### TC-DED-006 — Integer-width selection can invalidate source arithmetic semantics

The specification says proof validity depends on integer-width selection, while historical material permits width minimization.

If a source value has uint semantics but is stored in u8, intermediate operations must still preserve uint overflow behavior and observable width.

Required correction: Distinguish:

text semantic type width storage width register width 

Width minimization may change representation only. Arithmetic must still behave as the semantic type unless a proven range guarantees equivalence.

---

#### TC-DED-007 — Alias guards need path-sensitive proof semantics

Example:

rg1 if overlaps(input, output) {     return deny AliasError() }  # vectorized loop 

The compiler needs to know that the non-denied continuation implies nonoverlap.

Required correction: Control-flow facts must be path-sensitive.

A guard or conditional denial should refine facts only along the surviving path. Invalidation must occur after mutation or foreign effects that could alter the relevant objects.

---

#### TC-DED-008 — Runtime guard failure typing is inconsistent

Section 55 allows guards to produce:

- Typed denial
- Trap
- Explicit recovery path

The guarding operation may appear inside a function whose denial type does not include the guard’s failure.

Required correction: Guard selection must be type-checked against the enclosing failure context.

A compiler may insert a typed denial only when:

- The current function can return that denial type
- A declared conversion exists
- A local recovery clause handles it

Otherwise, compilation must be denied or the active policy must explicitly require a trap.

---

#### TC-DED-009 — Implicitly inserted denials change public function behavior

Suppose a function is declared:

rg1 function get(values: read slice<i32>, index: usize) -> i32 {     return values[index] } 

If bounds cannot be proven, inserting a typed denial would change the return type.

Required correction: Every potentially checked operation must have a statically defined failure behavior.

Options include:

1. Indexing has a built-in trap policy unless enclosed in a fallible context.
2. The function must declare a denial type.
3. Safe direct indexing is illegal without proof.
4. A module policy defines trap versus denial and that choice is reflected in the function contract.

The compiler cannot invent a hidden denial channel.

---

#### TC-DED-010 — Deduction budget unbounded_forbidden is semantically odd

Supported budgets include:

text minimal standard extended unbounded_forbidden 

unbounded_forbidden describes a prohibition, not a budget.

Recommended correction: Use:

text minimal standard extended implementation_maximum 

and state separately:

> Truly unbounded deduction is forbidden in conforming builds.

For reproducibility, each named budget should have implementation-independent minimum obligations or be recorded in build metadata.

---

#### TC-DED-011 — Deduction budgets threaten cross-compiler semantic portability

If one compiler proves a condition and another does not, they may differ in:

- Runtime checks
- Performance
- Denial timing
- Whether a required optimization succeeds
- Whether compilation is denied

Meaning remains safe, but build acceptance may differ.

Required correction: Distinguish semantic conformance from optimization conformance.

For required proofs or safety-critical builds, the proof obligation must use a standardized decidable fragment with deterministic expected outcomes.

Advisory deduction may remain implementation-dependent.

---

#### TC-DED-012 — User-supplied algebraic facts need trust classification

Examples use:

rg1 requires associative(add) 

A compiler generally cannot prove arbitrary user-defined associativity.

Required correction: Algebraic properties must be one of:

text Compiler-proven intrinsic law Standard-library certified law User-provided trusted law requiring direct authority Untrusted assertion checked only for diagnostics, not optimization 

A false user assertion must not permit unsafe memory behavior. At most, it may permit semantically risky numerical reassociation under an explicitly unsafe semantic policy.

---

#### TC-DED-013 — Proof caching and module interfaces are unspecified

A compiler may cache deductions, but no rule defines whether proof artifacts are:

- Compiler-private
- Serialized in package interfaces
- Valid across compiler versions
- Target-specific
- Cryptographically verified
- Rechecked after linking

Required correction: Cached proofs must be treated as invalid unless their full dependency environment matches.

Safety-critical conformance should require proof evidence or revalidation, not blind trust in cached outcomes.

---

#### TC-OPT-001 — Optimization cannot be defined before complete effect analysis

The compilation sequence validates ownership and concurrency before optimization, but the document does not include a complete effect-analysis stage before vectorization and parallel optimization.

Effects such as I/O, allocation, volatile access, denial, blocking, and foreign mutation constrain reordering.

Required correction: Add effect inference and validation before deduction finalization and optimization:

text Resolve declared effects Infer local effects Validate capability use Establish effect summaries 

Optimization must consume these summaries.

---

#### TC-OPT-002 — Parallel optimization must not invent concurrency-visible failures

Automatic parallelization can change:

- Which failure occurs first
- Whether multiple denials occur
- External-effect order
- Resource pressure
- Cancellation behavior

Required correction: Parallelization is legal only when the compiler proves observational equivalence under the active determinism and failure policies.

Pure independent computations are the simplest legal candidates.

Effectful operations require an explicitly parallel semantic construct.

---

#### TC-OPT-003 — Vectorization guard placement can alter observable denial order

A generated alias or alignment guard may execute before source operations that would otherwise deny first.

Required correction: Generated guards must be placed at a point where their failure has the same observable priority as the source-defined safety condition.

When multiple generated guards could fail, their diagnostic or denial ordering must follow a normative rule under deterministic profiles.

---

#### TC-OPT-004 — Scalar fallback may still violate required performance semantics

The specification treats optimization as nonsemantic except where marked required. However, deployment contexts may have timing deadlines.

A scalar fallback can preserve ordinary meaning while violating a real-time contract.

Required correction: Timing and resource bounds should be deployment or safety-profile contracts, not ordinary optimization preferences.

Example:

rg1 policy execution_budget : deployment {     function transform     worst_case 50:us } 

Such requirements need static evidence or target qualification.

---

#### TC-OPT-005 — Profile-guided optimization threatens reproducible builds

Optimization policies permit profiled inlining and dispatch. Reproducible builds require stable profile input.

Required correction: Profile data must be:

- Declared as a build input
- Versioned
- Canonically serialized
- Included in reproducibility metadata
- Rejected if incompatible

Implicit local profiling must be prohibited under reproducible-build profiles.

---

#### TC-OPT-006 — Optimization diagnostics may become de facto semantics

Policies can request notices such as scalar fallback. Safety-critical builds may depend on exact diagnostic records.

If diagnostic presence changes build acceptance, optimization reports become part of the certification interface.

Required correction: Separate:

text Semantic diagnostics Required-lowering diagnostics Advisory optimization reports 

Only stable required-lowering and conformance diagnostics should be certification-bearing.

---

#### TC-OPT-007 — Layout preference and layout requirement are not syntactically separated

Section 53 distinguishes preferred and required layouts, but examples use:

rg1 layout packed layout row_major 

without indicating whether these are mandatory.

Required correction: Use explicit forms:

rg1 layout require packed layout prefer row_major 

A default must be defined if neither word appears. Machine-facing unit layouts may default to required, while collection layouts may default to preferred, but this distinction must be normative.

---

#### TC-OPT-008 — Required layout can conflict with target legality after separate compilation

A type may be compiled independently with a required layout and later used in an ABI context with additional target restrictions.

Required correction: Required layout validation must occur both at type definition and final ABI instantiation.

Generic or target-dependent layouts may remain unresolved until monomorphization or target selection.

---

#### TC-OPT-009 — Lowering high-level directives after optimization is questionable

Section 4 places:

text 13. Apply vectorization and parallel optimization 14. Lower high-level directives 

Vectorization and parallelization normally require high-level constructs to be lowered into analyzable semantic operations first.

Required correction: Distinguish semantic lowering from machine lowering:

text Lower surface sugar to semantic IR Validate and analyze semantic IR Perform deduction and optimization Lower semantic IR to assembly serial and target operations 

Pipeline sugar, structural syntax, cases, and layers must have semantic forms before optimization.

---

#### TC-OPT-010 — Assembly serial generation occurs before symbolic resolution but after all optimization

Historical material describes assembly serial as an inspectable typed intermediate form, yet the normative sequence appears to generate it only near the end.

If optimization occurs before assembly serial, the serial may not represent the canonical semantic program.

Required correction: Define whether there are two serial forms:

text semantic assembly serial lowered assembly serial 

or clarify that assembly serial is a post-optimization low-level IR.

Tooling promises should match the chosen role.

---

#### TC-OPT-011 — VM code and native code relationship is ambiguous

The sequence says:

text Emit verified VM code or native code 

while other material describes VM code followed by optimized native lowering.

Required correction: Define supported compiler modes:

text VM target:     source -> verified VM artifact  Native target:     source -> semantic IR -> native object  VM-mediated native target:     source -> verified VM artifact -> native translation 

Conformance must not require a compiler to emit VM code internally unless that is a constitutional language requirement.

---

#### TC-OPT-012 — Final verification stage is missing

Required policies, ABI constraints, proof-dependent guard removal, relocation, and target lowering all need final validation.

Required correction: Add a final stage:

text Verify emitted artifact against:     semantic policies     safety policies     required layouts     required optimizations     ABI rules     memory-model rules     deployment restrictions 

No artifact should be emitted as conforming until this succeeds.

---

#### TC-POL-013 — Deployment policy target syntax is too coarse

Example:

rg1 target x86_64 

does not identify:

- ISA extensions
- ABI
- Operating system
- Endianness
- Object format
- CPU baseline
- Atomic capabilities
- Floating-point mode

Required correction: Deployment targets need structured profiles:

rg1 target {     architecture x86_64     abi sysv     os linux     features [sse2, avx2]     endian little } 

A named target triple may be used as sugar.

---

#### TC-POL-014 — Deployment restrictions need transitive enforcement

A module may forbid foreign calls or heap allocation, but imported libraries may perform them internally.

Required correction: Deployment and effect restrictions must apply transitively through call graphs unless a declared trusted boundary is permitted.

Binary-only dependencies need signed capability summaries or must be rejected under strict profiles.

---

#### TC-POL-015 — “Foreign calls audited” is not machine-checkable as written

Audit status needs evidence.

Required correction: Deployment policies should reference a verifiable artifact:

rg1 foreign_calls {     require audit_manifest "ffi-audit.rga"     require signature trusted_release_key } 

The compiler or build tool verifies coverage of all reachable foreign symbols.

---

#### TC-POL-016 — Diagnostic suppression can hide optimization fallback relevant to required builds

Diagnostic policies may suppress nonessential notices, but a scalar fallback notice may indicate violation of a performance certification expectation even when vectorization was advisory.

Required correction: Each diagnostic category must declare whether it is:

text mandatory policy-promotable suppressible certification-bearing informative 

Mandatory conformance and safety records cannot be suppressed.

---

#### TC-POL-017 — Policy reflection is not defined

Compiler reports may need to show effective policies, but the language does not define whether programs can inspect them at compile time or runtime.

Recommended correction: Raygen 1 should prohibit ordinary runtime policy reflection unless explicitly standardized.

Build tools should emit an effective-policy manifest for audit and reproducibility.

---

#### TC-POL-018 — Conditional policies can make semantic interfaces target-dependent

Historical syntax supports target-conditional compilation. A semantic policy selected conditionally can make one function behave differently on different targets.

That may be intentional, but it conflicts with broad semantic determinism claims.

Required correction: Target-dependent semantic policies must be part of the declared target interface and build identity.

Portable APIs should avoid target-dependent observable semantics unless explicitly annotated.

---

### Recommended compilation precedence

A more internally consistent semantic pipeline is:

text 1. Decode and validate UTF-8 2. Lex and parse the complete source grammar 3. Lower surface sugar into canonical semantic forms 4. Resolve packages, modules, imports, names, and editions 5. Resolve policy declarations and build an effective policy environment 6. Resolve declarations, types, generics, and traits 7. Validate type legality and refined-type invariants 8. Infer and validate effects and capabilities 9. Validate ownership, borrowing, lifetime, and destruction 10. Validate control flow, initialization, and failure completeness 11. Validate case, layer, channel, and synchronization contracts 12. Establish preliminary layout, ABI, provenance, and alias facts 13. Perform bounded preliminary deduction 14. Select target profile and finalize target-dependent representation 15. Revalidate proofs against final layout, width, alias, ABI, and target facts 16. Classify unproven obligations as guarded, synchronized, or denied 17. Commit runtime guards and denial paths 18. Perform semantics-preserving optimization 19. Lower to assembly serial, VM IR, or target IR 20. Resolve symbols, tags, relocations, and final layout 21. Generate VM or native artifacts 22. Verify final required optimization, ABI, safety, and deployment properties 23. Emit artifact, diagnostics, policy manifest, and conformance evidence 

Implementations may combine stages internally, but all dependencies must be respected.

### Recommended proof rule

> Deduction may justify removing or specializing an operation only after every assumption that affects the proof has reached its final required form. An invalidated proof restores the original guard or causes compilation denial; it never leaves an unchecked operation.

### Status

The policy framework is architecturally promising but not yet deterministic enough for compiler conformance.

A restricted implementation can support canonical policy keys, lexical nonmandatory policies, mandatory build-profile constraints, bounded bounds/range deduction, scalar fallback, and final verification of required vector width. Full Raygen 1 requires formal cross-domain policy resolution, guardability classification, proof evidence and revalidation, target-scoped optimization requirements, effect analysis, and a corrected compilation-stage model.



## 8. Derivation, Reactive Evaluation, and Effect Semantics

### Overall finding

Raygen’s derivation model has a clear conceptual split between static and dynamic dependency graphs. The design appropriately requires declared dependencies, cycle detection, controlled publication, and explicit effect capabilities.

The current specification does not yet define enough operational semantics for consistent compiler and runtime behavior. The primary gaps concern dependency identity, invalidation granularity, evaluation timing, caching, multi-output derivations, failure retention, dynamic graph updates, versioning, concurrency, and effect ordering.

### Critical issues

#### TC-DER-001 — Derivation is not classified as eager, lazy, or demand-driven

The specification says a derived value is invalidated when a dependency changes and that the compiler may cache it. It does not define when reevaluation occurs.

Possible models include:

- Immediately when a dependency changes
- At the next read
- At an explicit update point
- At a layer boundary
- According to implementation heuristics

Impact: Observable timing, denial propagation, resource use, and external effects can differ across conforming implementations.

Required correction: Static derivation must declare or inherit an evaluation policy.

Suggested policies:

text eager lazy manual batched 

A default must be normative. For pure static derivations, lazy is a practical default because caching and invalidation remain implementation-flexible without changing values.

Effectful derivations should not use implicit lazy execution unless their triggering semantics are explicit.

---

#### TC-DER-002 — “Dependency changes” is not formally defined

A dependency may be:

- Rebound
- Mutated internally
- Mutated through an alias
- Updated transactionally
- Replaced with an equal value
- Modified and restored
- Changed by foreign code
- Updated in another case

The specification does not define which events invalidate dependents.

Required correction: Distinguish dependency identity from dependency version.

Recommended rule:

> Every derivation dependency is associated with a semantic version source. A dependency invalidates its dependents when a committed write changes that version, regardless of whether the resulting value compares equal, unless the dependency explicitly uses value-equality invalidation.

This avoids requiring arbitrary equality checks.

Optional policy:

rg1 invalidation value_equality 

should require a pure, deterministic equality relation.

---

#### TC-DER-003 — Dependency granularity is unspecified

Example:

rg1 derive display_name from user {     display_name = user.first_name + user.last_name } 

Does changing user.age invalidate display_name?

Similarly, for arrays and tables, it is unclear whether dependency tracking is:

- Whole-object
- Field-based
- Element-based
- Range-based
- Query-based

Required correction: Define dependency selectors.

Examples:

rg1 derive display_name     from user.first_name, user.last_name {     ... } 

or allow the compiler to infer finer dependencies while preserving the declared dependency as a conservative upper bound.

A derivation must never depend on less than what is declared.

---

#### TC-DER-004 — External-read detection is not enough without transitive effect summaries

A pure derivation may call:

rg1 calculate_total(order) 

The called function might read global state, time, random data, or foreign memory.

Checking only lexical reads inside the derivation block is insufficient.

Required correction: Every called function must have an effect and dependency summary.

A pure derivation may call only functions proven pure relative to the declared dependencies.

Transitive undeclared reads must produce a compile-time denial.

---

#### TC-DER-005 — Derived outputs are not formally declared

Examples use:

rg1 derive total from subtotal, tax {     total = subtotal + tax } 

It is unclear whether total:

- Must already be declared
- Is declared by the derivation
- Has an inferred type
- May have multiple equations
- May be returned as a value
- Is visible before first evaluation

Required correction: Define one canonical declaration form.

Recommended syntax:

rg1 derive total: Money     from subtotal, tax {     total = subtotal + tax } 

The derivation declares the binding and its type.

A separate form may update an existing derived slot, but ordinary mutable variables should not silently become derived values.

---

#### TC-DER-006 — Multiple derived outputs are under-specified

The rules refer to “every derived output,” suggesting a block may define several outputs:

rg1 derive summary from values {     minimum = min(values)     maximum = max(values) } 

It is unclear whether those outputs:

- Publish atomically
- Have one shared dependency set
- May depend on one another
- Can be partially cached
- Can fail independently

Required correction: Multiple outputs should form one result aggregate and publish atomically.

Suggested syntax:

rg1 derive summary:     record     {         minimum: f64         maximum: f64     }     from values {     minimum = min(values)     maximum = max(values) } 

Alternatively, require one output per derivation in Raygen 1.

---

#### TC-DER-007 — Internal dependency order inside one derivation is unclear

Example:

rg1 derive final_price from base_price, tax_rate, discount {     tax = base_price * tax_rate     final_price = base_price + tax - discount } 

tax appears to be an intermediate derived value, but the rules only discuss external dependencies and derived outputs.

Required correction: Define local equations and output equations separately.

Local equations may form an internal acyclic graph. The compiler should derive their order from dependencies rather than source order unless source order is explicitly semantic.

---

#### TC-DER-008 — “Assigned exactly once” conflicts with control flow

Can a derivation contain:

rg1 if condition {     total = a } else {     total = b } 

This assigns the output once per execution path but twice syntactically.

Required correction: Replace “assigned exactly once” with definite single assignment:

> Every output must have exactly one reaching definition on every successful evaluation path and no path may define it more than once.

SSA-style phi merging may be used internally.

---

#### TC-DER-009 — Static cycle detection scope is unclear

Cycles may cross:

- Derivation blocks
- Functions
- Modules
- Packages
- Generic instantiations
- Conditional compilation
- Imported precompiled interfaces

Required correction: Static derivation dependencies must be represented in module interfaces.

The compiler or linker must detect cycles across the complete reachable static derivation graph after generic instantiation and conditional selection.

A cycle hidden behind unavailable binary metadata should block strict conformance.

---

#### TC-DER-010 — Self-dependency through ordinary functions may evade detection

Example:

rg1 derive a from b {     a = calculate_a(b) } 

where calculate_a reads a indirectly.

Required correction: Dependency summaries must include transitive derived-value reads.

Any call path that reads the output being evaluated creates a dependency edge and must participate in cycle detection.

---

#### TC-DER-011 — Caching semantics are too permissive

The compiler may cache pure derivations when semantics are preserved, but cache behavior can affect:

- Allocation failure
- Performance
- Destruction timing
- Address identity
- Debug traces
- Resource pressure

Required correction: Pure derivations must not expose allocation identity or destruction timing as semantic results.

Caching is legal only when reevaluation and reuse are observationally equivalent under the active policies.

Values whose identity is observable should require an explicit cache policy.

---

#### TC-DER-012 — Cached derived-value lifetime is undefined

A derived value may own heap, region, foreign, or shared resources. The specification does not say:

- Where the cache is stored
- When the old value is destroyed
- Whether readers may retain old versions
- Whether invalidation immediately destroys the old value
- Whether the cache can outlive the declaring scope

Required correction: The derivation owner must own its cached result.

On successful replacement:

1. The new result becomes publishable.
2. Publication occurs atomically.
3. The old version remains alive until existing snapshots or borrows end.
4. The old version is then destroyed.

A non-versioned implementation may require exclusive publication when no readers are active.

---

#### TC-DER-013 — Failed reevaluation behavior is undefined for existing values

Dynamic derivation says a failed reevaluation produces a denial and does not publish partial results. It does not say what happens to the previously published valid value.

Possible behaviors include:

- Keep the previous value
- Invalidate the value entirely
- Publish an error state
- Make reads block
- Return stale data with a denial marker

Required correction: Declare a failure-retention policy.

Suggested options:

text retain_last_valid invalidate publish_denial 

A default such as retain_last_valid is useful but semantically observable and must be explicit.

---

#### TC-DER-014 — Denial observation is not specified

When reevaluation fails, who receives the denial?

Potential observers include:

- The writer that changed the dependency
- The next reader
- A derivation supervisor
- The enclosing case group
- A diagnostic channel

Required correction: Every dynamic or effectful derivation needs a denial destination.

Example:

rg1 derive dynamic state     from dependencies     deny<DerivationError>     report to state_supervisor 

Unchecked background derivation failures must not disappear.

---

#### TC-DER-015 — Pure derivation allocation rule is ambiguous

The specification prohibits “allocation with visible lifetime effects,” which appears to allow temporary allocation.

It does not define:

- Whether allocation failure is observable
- Whether allocator contention is an effect
- Whether memory usage limits are semantic
- Whether cached allocation is allowed
- Whether a no-heap deployment permits temporary heap allocation

Required correction: Distinguish abstract allocation from deployment-visible allocation.

A pure derivation may allocate temporary storage only when:

- The allocation cannot escape
- Failure is impossible under the semantic model or represented as a denial
- Deployment policy permits the allocator
- Allocation does not create externally observable identity

---

#### TC-DER-016 — Effect capabilities are not formally declared on functions

Effectful derivations use:

rg1 uses storage.write 

but called functions do not have a normative effect signature syntax.

Required correction: Functions need declared or inferred effect contracts:

rg1 function write_audit_entry(...)     -> allow<AuditRecord> deny<StorageError>     effects     {         storage.write     } 

An effectful derivation’s declared capability set must cover all transitive effects.

---

#### TC-DER-017 — Effect ordering among multiple derivations is undefined

Two effectful derivations may respond to the same change:

text transaction -> write audit transaction -> send notification 

The specification says effectful derivations are not freely reorderable, but does not define their order.

Required correction: Effectful derivations must declare one of:

- Explicit dependency order
- Layer or case membership
- Stable declaration order
- Permission for nondeterministic order

Relying on implementation scheduling is incompatible with schedule determinism.

---

#### TC-DER-018 — Effectful derivation retriggering can duplicate effects

If dependencies change repeatedly during evaluation, an effectful derivation may:

- Run once per version
- Coalesce updates
- Cancel and restart
- Run concurrently for multiple versions
- Commit stale effects

Required correction: Define trigger semantics.

For example:

rg1 update coalesced inflight complete 

or:

rg1 update latest inflight cancel 

Effectful work must not be silently repeated or discarded.

---

#### TC-DER-019 — Effectful derivation caching is conceptually unsafe

The specification says effectful derivations may be optimized when equivalence is proven. “Caching” an effectful derivation could skip required effects.

Required correction: Separate value reuse from effect suppression.

An implementation may reuse computed values, but it may omit an effect only when the effect contract itself declares idempotence, memoizability, or equivalent replay semantics.

The default is that every required trigger executes its effects exactly once.

---

#### TC-DER-020 — Dynamic dependency mutation API is absent

Dynamic derivation permits dependencies to be inserted, removed, or replaced, but no operation defines how.

Questions include:

- Who owns the graph?
- Is mutation transactional?
- Can dependencies change while evaluation is active?
- Can readers observe intermediate graph states?
- What denial types exist?
- Is identity based on object, key, or handle?

Required correction: Define a graph-management capability.

Conceptually:

rg1 let update =     dependencies.begin_update()  update.add(node_a) update.remove(node_b)  allow update.commit() deny as error {     ... } 

Direct mutation without versioned commit should be prohibited.

---

#### TC-DER-021 — Stable dependency-set version identifiers are under-specified

The specification requires stable version identifiers but does not define stability across:

- Process restarts
- Serialization
- Distributed replicas
- Rollback
- Equal graph states
- Failed updates

Required correction: Define the minimum guarantee:

> A version identifier uniquely identifies one committed dependency-set state within the lifetime of its dynamic derivation instance and is monotonically ordered relative to commits.

Global persistence should require a stronger library-level version type.

---

#### TC-DER-022 — Topological evaluation order needs deterministic tie-breaking

Multiple valid topological orders may exist.

Under schedule determinism, evaluation order can affect:

- Denial selection
- Resource usage
- Debug traces
- Effect ordering
- Floating-point merges

Required correction: Pure independent nodes may run in any order only when results are schedule-deterministic.

Otherwise, use a stable topological order based on declared node identity or insertion sequence.

Effectful nodes require explicit ordering.

---

#### TC-DER-023 — Dynamic cycle detection moment is unclear

A cycle may be detected:

- During update construction
- At update commit
- At reevaluation
- During traversal
- Only when a cyclic node is demanded

Required correction: For cycle deny, cycle detection should occur before the new dependency-set version is committed or published.

The previous valid graph remains active after denial.

For incremental graph systems, the implementation may use incremental cycle detection but must produce equivalent behavior.

---

#### TC-DER-024 — cycle report and cycle isolate lack semantics

Supported cycle policies include:

text deny report isolate 

Only deny has an obvious meaning.

Questions include:

- Does report still accept the graph?
- How is a cyclic value evaluated?
- What exactly is isolated: nodes, edges, or a component?
- Are downstream nodes valid?
- How are outputs typed?

Required correction: Either define them precisely or remove them from Raygen 1.

A possible definition:

text report:     reject publication and deliver a typed cycle report without converting it     to the derivation’s ordinary denial path.  isolate:     remove cyclic strongly connected components from evaluation,     mark their outputs unavailable, and evaluate only unaffected components. 

isolate requires an output type capable of representing unavailable nodes.

---

#### TC-DER-025 — Consistency policies are not defined operationally

Supported dynamic consistency policies include:

text snapshot serial transactional latest eventual 

These terms overlap and are not mutually exclusive.

Required correction: Define consistency through independent properties:

text dependency version observed publication atomicity writer serialization reader staleness evaluation restart behavior 

For example:

text snapshot:     one evaluation uses one committed dependency-set version.  serial:     evaluations execute one at a time in commit order.  latest:     stale in-flight evaluations may be discarded before publication.  eventual:     intermediate versions may be skipped, but a quiescent latest version     must eventually be evaluated.  transactional:     dependency update and result publication participate in one transaction domain. 

These properties may need composition rather than a single enum.

---

#### TC-DER-026 — latest can starve publication

Under constant dependency changes, continually discarding stale evaluations may prevent any result from publishing.

Required correction: Progress behavior must be declared.

Options include:

text latest best_effort latest eventually_publish latest bounded_restart N 

Safety-critical profiles should avoid unbounded restart behavior.

---

#### TC-DER-027 — eventual lacks a quiescence and fairness definition

“Eventually” is meaningless without assumptions.

Required correction:

> Under eventual consistency, if dependency updates stop and required resources remain available, the derivation must eventually attempt evaluation of the latest committed dependency version under the active scheduler progress guarantee.

This does not guarantee success if evaluation denies.

---

#### TC-DER-028 — Update coalescing lacks version-selection rules

For versions 10, 11, and 12 arriving during evaluation, coalesced could evaluate:

- Only 12
- 10 and then 12
- A merged synthetic version
- Any one pending version

Required correction: Define:

> Coalesced updates may skip intermediate committed versions but must evaluate the latest committed version available when the next evaluation begins.

Effects associated with skipped versions must not be silently lost unless the effect policy permits coalescing.

---

#### TC-DER-029 — Batched updates need batch boundaries

The batched policy does not define whether batches are formed by:

- Count
- Time
- Layer boundary
- Explicit commit
- Scheduler choice

Required correction: Require a batch trigger:

rg1 update batched {     count 100 } 

or:

rg1 update batched at layer_end 

Time-based batching introduces clock nondeterminism unless the clock is a declared input.

---

#### TC-DER-030 — Manual update does not define stale-read behavior

Under manual, dependencies may change without reevaluation.

A read must know whether it receives:

- Last valid value
- A stale marker
- A denial
- Forced reevaluation

Required correction: Derived values should expose freshness metadata or have a declared stale-read policy.

Example:

text read stale_allowed read require_current 

---

#### TC-DER-031 — Publication atomicity needs a reader model

The specification requires completed state to publish atomically, but does not define whether readers:

- Borrow the current version
- Copy the value
- Receive a snapshot handle
- Block during publication
- Can retain old versions

Required correction: Dynamic derived values should be versioned cells.

A read returns either:

text snapshot<T, Version> 

or a copied value, according to type and policy.

Publication atomically replaces the current version pointer or equivalent representation.

---

#### TC-DER-032 — Dynamic metadata ownership is acknowledged but not specified

Dependency nodes, edges, version records, dirty queues, and snapshots all have lifetimes.

Required correction: The dynamic derivation instance must own its metadata graph. Snapshot handles retain the referenced version. Removing a dependency releases graph ownership only after no active evaluation or snapshot references it.

Detached evaluations must be supervised like detached cases.

---

#### TC-DER-033 — Concurrent dependency updates need commit arbitration

Two cases may update the dependency graph simultaneously.

Missing rules include:

- Conflict resolution
- Commit order
- Lost updates
- Merge behavior
- Retry
- Deterministic ordering

Required correction: Graph updates must use transactional or compare-version commits.

Example:

text commit update based_on version 14 

A stale-base conflict produces a typed denial or deterministic retry.

Under schedule determinism, successful commit order must use a stable arbitration rule or explicit serialization.

---

#### TC-DER-034 — Runtime graph corruption is too vaguely handled

The specification allows denial or trap according to safety profile but does not define corruption.

Possible cases include:

- Invalid edge target
- Duplicate node identity
- Broken ownership link
- Version mismatch
- Impossible topological state
- Memory corruption

Required correction: Separate:

text Validated user graph error:     typed denial.  Runtime metadata invariant violation:     implementation trap or process-level fault.  Direct or foreign corruption:     trap unless an audited recovery mechanism exists. 

Internal runtime corruption should not normally appear as an ordinary user denial.

---

#### TC-DER-035 — Derivation inside cases and layers needs nesting rules

A derivation may be declared or triggered within concurrent execution, but the specification does not define:

- Whether its cache is case-local
- Whether it may publish outside the layer
- Whether updates wait for layer boundaries
- Whether effectful derivations inherit case cancellation
- Whether dynamic evaluation can outlive the case

Required correction: Lexically declared derivations inherit the lifetime, cancellation, and visibility boundary of their owner unless explicitly retained or supervised.

A derivation inside a layer cannot publish ordinary external output before the layer boundary.

---

#### TC-DER-036 — Recursive derivation through dynamic and static graphs is not covered

A static derivation may depend on a dynamic derived value, which may depend on another static result.

Required correction: Define one combined dependency graph for cycle and invalidation analysis wherever identities are statically known.

Dynamic edges require runtime detection but must include static nodes in the evaluated graph.

A cycle crossing static and dynamic boundaries must still be denied.

---

#### TC-DER-037 — Derived values and ordinary mutation can conflict

Section 16 says derived outputs are immutable outside the block unless declared mutable. Allowing mutable derived outputs creates competing authorities:

- Derivation reevaluation
- Ordinary set
- Transactional update
- External case writes

Required correction: Do not allow ordinary mutation of a derived output in Raygen 1.

Use a separate dependency input for overrides:

rg1 derive effective_value     from base_value, manual_override {     ... } 

This preserves a single source of truth.

---

#### TC-DER-038 — Derivation invalidation must interact with transactions

If several dependencies change transactionally, reevaluating after each individual write would observe intermediate states.

Required correction: Transactional writes should invalidate derivations only at commit, producing one dependency version transition for the committed set.

Snapshot evaluation must observe either the precommit or postcommit state, never a partial transaction.

---

#### TC-DER-039 — Denial collection across a dependency graph is undefined

If several independent nodes deny, the runtime may observe multiple failures.

Possible policies include:

- First denial
- Stable collection
- Fail-fast cancellation
- Continue unaffected branches
- Aggregate all reachable failures

Required correction: Add graph failure policy:

text first collect isolate cancel_dependents continue_independent 

Stable collection should use deterministic node order, not completion order.

---

#### TC-DER-040 — Downstream state after upstream denial is unclear

If node A denies, what happens to B and C that depend on A?

Required correction: Dependent nodes become blocked for that evaluation version and must not publish new values based on missing input.

Their prior values may remain available only under an explicit stale-retention policy.

The denial result should preserve the causal chain.

---

#### TC-DER-041 — Derivation denial types need composition

Different nodes may deny with different error types.

Required correction: A derivation graph must have either:

- One common denial type
- Explicit conversion into a graph denial type
- A generated tagged denial union

Implicit loss of error identity should be prohibited.

---

#### TC-DER-042 — Pure derivations must define determinism expectations

Purity alone does not guarantee determinism if operations depend on:

- Unordered iteration
- Floating-point reduction
- Hash randomization
- Target-specific transcendental functions

Required correction: A pure derivation means no externally visible effects, not necessarily deterministic output.

The active determinism policy still constrains evaluation and algorithms.

---

#### TC-DER-043 — Cache keys must include semantic policy context

A derived expression evaluated under one rounding or overflow policy cannot safely reuse a cached result under another.

Required correction: The cache identity must include every semantic policy that can affect the result, as well as target-dependent numerical mode where relevant.

Lexical policy changes should invalidate incompatible cached values.

---

#### TC-DER-044 — Caching across compiler or runtime versions is unspecified

Persistent caches may survive process or compiler updates.

Required correction: Persistent derivation caches must include:

text language edition type schema version dependency versions semantic policy fingerprint compiler/runtime compatibility identifier target numerical profile, when relevant 

Otherwise they must be treated as invalid.

---

#### TC-DER-045 — Evaluation budgets are absent

Dynamic graphs may be arbitrarily large or expensive. The specification requires bounded compile-time deduction but not bounded runtime derivation.

Required correction: Deployment and safety profiles should support limits for:

text maximum nodes maximum edges maximum evaluation depth maximum retries maximum queued versions maximum memory maximum execution budget 

Exceeding a declared limit must produce a typed denial or configured trap.

---

### Recommended static derivation reference model

text 1. A static derivation declares one typed output and a closed set of dependencies. 2. Its body is lowered to an acyclic equation graph. 3. All transitive reads and effects are checked against declarations. 4. Pure derivations may be evaluated eagerly or lazily only according to policy. 5. Dependency commits invalidate the cached output version. 6. Reevaluation uses one consistent dependency version set. 7. Successful evaluation atomically publishes a new output version. 8. Failed evaluation follows the declared stale-value and denial policy. 9. Ordinary external mutation of the derived output is forbidden. 10. Caching must preserve identity, lifetime, policy, and determinism semantics. 

### Recommended dynamic derivation reference model

text 1. A dynamic derivation owns a versioned dependency graph. 2. Graph updates are committed atomically. 3. Cycle validation occurs before a new graph version is published. 4. Evaluation captures one committed graph and dependency snapshot. 5. Pure independent nodes may execute concurrently. 6. Effectful nodes obey explicit ordering and capability contracts. 7. Outputs publish atomically only after successful evaluation. 8. Stale in-flight evaluations are handled by declared update policy. 9. Previous values and denials follow explicit retention and reporting policies. 10. Metadata and snapshots remain valid through ownership-tracked version lifetimes. 

### Status

The derivation model is conceptually valuable but not yet operationally conformable.

A restricted implementation can support pure, single-output static derivations with whole-value dependencies, lazy reevaluation, declaration-order diagnostics, no mutable outputs, and compile-time cycle detection. Full Raygen 1 requires versioned dependency identity, defined triggering and invalidation, atomic publication, stale-value policy, typed denial routing, formal effect summaries, transactional graph updates, deterministic topological scheduling, and bounded runtime graph management.



## 9. Layout, ABI, Direct Memory, VM, and Native Lowering

### Overall finding

Raygen correctly separates semantic meaning from physical representation and allows explicit layout constraints where machine interoperability requires them. The distinction between native-width scalars, exact-width types, rayword, assembly serial, VM code, and native lowering is conceptually sound.

The current specification does not yet provide enough detail for interoperable binaries, verified VM modules, or safe direct-memory access. The primary gaps concern layout categories, padding, endianness, packed access, rayword interpretation, ABI classification, foreign ownership, direct-memory authority, VM verification, symbolic tags, and the relationship between VM and native artifacts.

### Critical issues

#### TC-LAYOUT-001 — Required and preferred layouts are not syntactically distinguishable

The specification states that preferred layouts may be ignored and required layouts must either be satisfied or denied. Examples use only:

rg1 unit Packet layout packed array<f32, 4, 4> matrix layout row_major 

It is unclear whether these are required or preferred.

Required correction: Define explicit syntax:

rg1 unit Packet layout require packed array<f32, 4, 4> matrix layout prefer row_major 

A declaration without require or prefer must have a normative default.

---

#### TC-LAYOUT-002 — Layout categories are mixed together

The document uses layout to describe:

- Object representation
- Field packing
- Matrix traversal order
- Collection implementation
- Graph representation
- Table storage
- Cache placement
- ABI form

These are not one kind of constraint.

Required correction: Separate at least:

text representation layout collection layout traversal order storage layout ABI layout placement preference 

For example, row_major is an index-to-offset rule, while packed is a field-placement constraint.

---

#### TC-LAYOUT-003 — Default record and unit layouts are not defined

The distinction between record and unit suggests:

- record may use implementation-selected layout
- unit has deterministic machine-facing layout

However, the exact guarantees are absent.

Required correction: Define:

text record:     semantic field aggregate;     field order and padding are not ABI-stable unless layout is declared.  unit:     representation-bearing aggregate;     field order follows declaration order;     size, alignment, padding, and endianness follow the selected layout and ABI profile. 

A record should not be used in foreign interfaces without an explicit representation contract.

---

#### TC-LAYOUT-004 — Padding bytes are unspecified

ABI and bitwise reproducibility depend on whether padding bytes are:

- Uninitialized
- Zeroed
- Preserved
- Ignored during equality
- Serialized
- Included in hashing

Required correction:

> Padding bytes are not part of a value’s semantic state. Safe equality, hashing, and serialization must ignore them unless a representation type explicitly includes them. Reading uninitialized padding as data is prohibited.

Safety-critical or reproducible profiles may require zero-initialized padding for exported representations.

---

#### TC-LAYOUT-005 — Packed layout does not define misaligned access behavior

A packed unit may place u16, u32, or larger fields at unaligned offsets.

The specification does not define whether field access:

- Uses legal unaligned instructions
- Is lowered into byte operations
- Requires copying into aligned temporary storage
- Is rejected on targets without support
- Produces a direct-memory requirement

Required correction:

> Access to a misaligned packed field must lower to a target-legal sequence preserving the field’s value semantics. Creating an ordinary aligned reference to such a field is prohibited.

Atomic fields must not be packed below their required alignment.

---

#### TC-LAYOUT-006 — Packed fields can invalidate borrowing assumptions

A writable reference to one packed field may overlap the machine location containing another packed field or bitfield.

Required correction: Borrow and race analysis must use physical memory-location overlap after final layout.

Logical field distinctness is insufficient when representation aliases storage.

---

#### TC-LAYOUT-007 — Bitfield semantics are missing

The memory model mentions packed bitfields, but no syntax or semantics define:

- Bit width
- Signedness
- Bit order
- Storage unit
- Endianness
- Atomic access
- Volatile access
- Overflow

Required correction: Either remove bitfields from Raygen 1 core or publish a precise bitfield representation specification.

---

#### TC-LAYOUT-008 — Endianness is absent from layout rules

Exact machine and wire layouts require defined byte order.

Examples of packed headers and device registers do not declare endianness.

Required correction: Support explicit endian annotations:

rg1 unit PacketHeader layout require network {     version: u8     length: u16 endian big } 

Target-native layout and protocol layout must be distinct concepts.

---

#### TC-LAYOUT-009 — Scalar representation is not fully specified

Exact-width integers imply bit widths but not necessarily:

- Two’s-complement signed representation
- Boolean representation
- Floating-point encoding
- Invalid bit patterns
- Address representation

Required correction: Define object representations for all ABI-visible primitive types.

A practical baseline is:

text Unsigned integers: binary modulo 2^N. Signed integers: two’s complement. bool: semantic false/true; ABI representation profile-defined unless explicitly fixed. f32/f64: IEEE 754 binary32/binary64. 

Other floating formats require target or emulation profiles.

---

#### TC-LAYOUT-010 — Native-width scalar ABI stability is unclear

Types such as:

text int uint isize usize real 

may vary by target.

Public interfaces using them cannot have one cross-target binary layout.

Required correction:

> Native-width types are ABI-stable only within one declared target profile. Cross-target serialized formats and stable foreign interfaces must use exact-width types.

The ABI specification should encode the resolved width in interface metadata.

---

#### TC-LAYOUT-011 — Width minimization risks violating address identity and aliasing

Historical material permits narrow storage for values with wider semantic types. This can affect:

- References to values
- Atomic operations
- Foreign pointers
- Debugger expectations
- Object size
- Arrays
- ABI layout

Required correction: Width minimization may apply only to non-address-exposed internal temporaries or explicitly compressed storage.

Once a value’s storage is observable through layout, address, ABI, atomic access, or reflection, its declared representation must be preserved.

---

#### TC-LAYOUT-012 — rayword lacks one canonical representation contract

rayword may represent scalar integers, vectors, masks, blocks, or packed structures. It is unclear whether it is:

- An opaque 256-bit bit container
- A tagged union
- A SIMD type
- A nominal wrapper
- A family of representations

Required correction: Define rayword as an untagged exact 256-bit value with fixed alignment and explicit interpretation operations.

Suggested baseline:

text rayword:     exactly 256 value bits;     no implicit lane type;     no hidden runtime tag;     alignment is target profile-defined with a normative minimum;     all reinterpretation is explicit. 

Operations requiring lane interpretation must use typed views.

---

#### TC-LAYOUT-013 — view<rayword as T> needs aliasing and lifetime rules

A view may be:

- A value reinterpretation
- A borrowed reference
- A zero-copy alias
- A bitcast

These have different semantics.

Required correction: Separate:

rg1 bitcast<vector<f32, 8>>(word) viewref<vector<f32, 8>>(read word) 

A value bitcast copies semantic bits and creates a new value.

A reference view aliases the same storage and must satisfy size, alignment, validity, ownership, and lifetime constraints.

---

#### TC-LAYOUT-014 — Not every bit pattern is valid for every view type

Viewing arbitrary bits as:

- bool
- A choice type
- A refined type
- A pointer
- A record with invariants

may create invalid values.

Required correction: Define:

text bit-valid types:     every bit pattern is valid.  validated representation types:     construction from raw bits requires checking. 

Unchecked reinterpretation of non-bit-valid types requires direct.

---

#### TC-LAYOUT-015 — vector<T, N> layout and lane order are undefined

The specification relies on vector types but does not define:

- Lane numbering
- Memory order
- Alignment
- Padding
- ABI passing
- Mask representation
- Whether vectors are first-class storable values
- Whether N * sizeof(T) must equal physical size

Required correction: Define logical lane order independently of target register representation.

Memory serialization should use lane 0 first in increasing address order unless another explicit layout applies.

---

#### TC-LAYOUT-016 — Matrix layout needs an index formula

row_major and column_major are named but not formally defined.

Required correction: Provide exact offset equations for multidimensional arrays, including:

- Dimension ordering
- Strides
- Padding
- Bounds
- Slicing behavior

---

#### TC-ABI-001 — The ABI specification boundary is acknowledged but essential rules are missing here

The core claims ABI conformance and exact layout behavior, but does not define what semantic information the ABI document must preserve.

Required correction: The core should require the ABI specification to cover:

text primitive size and alignment aggregate layout calling convention parameter and result passing outcome representation choice discriminants name mangling generic instantiation identity exception or denial ABI ownership transfer foreign unwinding atomic ABI vector ABI rayword ABI thread-local storage symbol visibility versioning 

Without these, Native conformance cannot be tested.

---

#### TC-ABI-002 — Fallible return ABI is undefined

A function returning:

rg1 allow<T> deny<E> 

could lower through:

- Tagged return register
- Hidden result pointer
- Separate status register
- VM error register
- Platform exception mechanism

Required correction: The ABI must define one semantic representation per target profile.

Optimization may alter internal calls, but exported and separately compiled boundaries require stable calling rules.

---

#### TC-ABI-003 — Choice discriminant stability is unspecified

Choice types need rules for:

- Discriminant size
- Variant numbering
- Payload alignment
- Niche optimization
- Invalid states
- ABI stability

Required correction: Internal choices may use optimized representations. Exported or layout-constrained choices require explicit discriminant and representation rules.

Niche optimization must not cross ABI boundaries unless guaranteed by the ABI profile.

---

#### TC-ABI-004 — Nominal transparent wrappers need ABI declarations

A nominal type UserId = u256 may or may not have the same ABI as u256.

Required correction: ABI transparency must be explicit through a representation attribute. Type identity alone must not imply foreign-layout transparency.

---

#### TC-ABI-005 — Generic ABI identity is undefined

Monomorphized generic functions and types require stable symbol identities across modules.

Required correction: The package and ABI specifications must define canonical type encoding and instantiation identity.

Compiler-private mangling is insufficient for cross-compiler conformance unless object interoperability is explicitly out of scope.

---

#### TC-ABI-006 — Cross-compiler object compatibility is not declared

Conformance could mean:

- Source compatibility only
- VM artifact compatibility
- Native object compatibility
- Full linker interoperability

Required correction: Each conformance level must state its interoperability obligation.

For example:

text Raygen Core:     source and VM semantic compatibility.  Raygen Native:     ABI-compatible exported symbols within one target ABI edition.  Professional:     package-interface and generic metadata compatibility. 

---

#### TC-ABI-007 — Foreign ownership transfer is unspecified

Foreign calls may receive or return pointers and owned resources, but the core does not define:

- Who frees returned memory
- Whether foreign code retains arguments
- Whether callbacks may outlive the call
- Whether foreign code mutates through const pointers
- Whether denial maps to error codes
- Whether Raygen destructors cross the boundary

Required correction: Foreign declarations need ownership and effect annotations:

rg1 foreign c function parse(     input: read address<byte>,     length: usize ) -> foreign_owned address<Result> effects {     retain none     write none } 

Every foreign-owned value needs a declared release function or transfer rule.

---

#### TC-ABI-008 — Foreign unwinding behavior is missing

C++, Rust, operating-system exceptions, and callbacks may unwind or long-jump across Raygen frames.

Required correction:

> Foreign unwinding across a Raygen boundary is prohibited unless the target ABI profile explicitly defines it. Unspecified foreign unwinding causes a trap or process termination at the boundary.

Raygen cleanup guarantees cannot be preserved otherwise.

---

#### TC-ABI-009 — Callback lifetime and threading are not defined

Foreign APIs may store callbacks and invoke them later or from foreign threads.

Required correction: Callback declarations must specify:

text lifetime threading domain reentrancy captured ownership denial handling unregistration 

Lexical borrows cannot be captured by callbacks that may outlive the call.

---

#### TC-DIRECT-001 — direct authority is too broad

The specification says direct blocks permit low-level authority but does not enumerate what safety guarantees remain active.

Required correction: Define that direct may relax only specified checks.

Suggested retained guarantees:

text syntax and type-size correctness declared target validity block structure initialized local bindings ordinary control-flow legality explicit effect accounting 

Potentially relaxed guarantees must be explicit:

text raw address provenance unchecked pointer arithmetic manual lifetime claims unaligned access foreign representation validation 

A direct block must not silently disable unrelated arithmetic, concurrency, or denial policies.

---

#### TC-DIRECT-002 — Direct blocks need capability scoping

Any source code may apparently write:

rg1 direct {     ... } 

Deployment policy can restrict direct blocks, but no capability acquisition model exists.

Required correction: Direct authority should require an imported or granted capability:

rg1 direct using device_control {     ... } 

The package/build profile controls which modules receive that capability.

---

#### TC-DIRECT-003 — Integer-to-address casts do not establish a mapped region

Example:

rg1 cast<address<volatile u32>>(STATUS_ADDRESS) 

A numeric address alone does not prove the memory exists or is accessible.

Required correction: Require a mapping operation:

rg1 let device =     map address STATUS_ADDRESS     length 4     permissions read_write     as volatile u32 

Bare integer-to-address casting may remain available in direct, but dereference requires declared mapping authority under safe native profiles.

---

#### TC-DIRECT-004 — Pointer arithmetic bounds are missing

The specification shows:

rg1 let next = ptr + 1 

without defining whether the increment is bytes or elements and whether one-past pointers are legal.

Required correction:

> Arithmetic on address<T> is scaled by sizeof(T). Resulting addresses may range within the originating allocation and one-past its end. Dereferencing one-past is invalid.

Byte arithmetic requires address<byte> or an explicit byte view.

---

#### TC-DIRECT-005 — Null-address semantics are undefined

The safety policies mention null access, but the address type does not state whether it is nullable.

Required correction: Prefer distinct types:

text address<T>          non-null valid-address capability optional<address<T>> nullable address raw_address<T>      possibly null direct address 

At minimum, safe dereference must require proof or checking of non-null state.

---

#### TC-DIRECT-006 — Direct memory mapping and object lifetime are not connected

Mapping bytes as PacketHeader or DeviceRegister creates a typed object view. The specification does not define whether the object’s lifetime has begun or whether all bit patterns are valid.

Required correction: Distinguish:

text memory view:     typed interpretation of existing storage without constructing ordinary ownership.  object construction:     begins the lifetime of a Raygen value in storage. 

Device-register views should not be treated as ordinary constructed records with destructors or copy semantics.

---

#### TC-DIRECT-007 — Direct writes can violate choice and refined-type invariants

Writing raw bytes into a typed object may create invalid values.

Required correction: Raw writes to invariant-bearing storage invalidate its safe typed view until validation reconstructs a valid value.

The compiler must not continue using previous type proofs after arbitrary direct mutation.

---

#### TC-DIRECT-008 — Endianness conversion cannot be inferred from target alone

Memory-mapped files and network packets may use nonnative byte order.

Required correction: Mapping syntax must declare representation endianness or use endian-aware field types.

---

#### TC-DIRECT-009 — Device memory and ordinary memory must remain distinct

Device accesses may not support:

- Caching
- Speculation
- Ordinary loads and stores
- Atomic operations
- Copying
- References

Required correction: Define device address spaces or memory-domain-qualified addresses:

text address<cpu, T> address<device, T> address<shared, T> 

Operations legal in one domain may be illegal in another.

---

#### TC-VM-001 — VM code format is not normatively defined

The core requires VM conformance at several levels but provides no:

- File header
- Versioning
- Section table
- Instruction encoding
- Operand format
- Constant pool
- Type table
- Validation rules
- Endianness
- Alignment
- Relocation format

Required correction: The VM Specification must define a stable binary grammar and validator algorithm.

Until then, “verified VM code” is only an architectural goal.

---

#### TC-VM-002 — VM verification obligations are missing

A verified VM should prove at least:

text valid instruction boundaries valid opcodes and operands type-correct register use initialized reads valid control-flow targets stack discipline function signature agreement ownership state transitions region lifetime denial-flow legality case and layer metadata validity atomic-order legality capability restrictions 

Required correction: The VM verifier must reject any module that violates these invariants before execution or native translation.

---

#### TC-VM-003 — VM register model remains ambiguous

Historical material refers to:

- Native scalar registers
- Optional rayword registers
- Typed vector registers
- A register-and-stack hybrid

The normative core does not state whether these are physical VM register classes or conceptual instruction types.

Required correction: Define the abstract machine independently of host hardware.

For each register class, specify:

- Width
- Type state
- Initialization
- Move/copy behavior
- Spill semantics
- Calling convention

---

#### TC-VM-004 — Ownership metadata in the VM is optional without semantic limits

Historical text says ownership metadata exists “where required.” A verifier must know whether ownership is:

- Fully erased after verification
- Tracked dynamically
- Encoded in bytecode
- Reconstructed from control flow

Required correction: Define semantic obligations, not one implementation.

A verified VM may erase ownership metadata only after proving that all execution paths preserve ownership. An interpreter without such proof must enforce equivalent runtime ownership checks.

---

#### TC-VM-005 — VM instructions for high-level structures are unclear

Representative instructions include:

text RG_SORT RG_LAYER RG_MERGE 

but the core also allows high-level directives to lower before VM emission.

Required correction: Define which VM constructs are mandatory:

- Primitive low-level instructions only
- Optional high-level verified intrinsics
- Standard-library calls
- Runtime service opcodes

Different VM implementations must agree on module semantics even if optional opcodes are lowered away.

---

#### TC-VM-006 — VM denial handling is under-specified

Historical material mentions an error-state register, but the Normative Core uses typed outcomes.

Required correction: The VM must represent denial types and control flow explicitly enough to verify:

- Construction
- Propagation
- Matching
- Cleanup
- ABI crossing

A single untyped global error register would be insufficient.

---

#### TC-VM-007 — VM concurrency scheduling semantics are missing

A VM with case and layer opcodes needs defined behavior for:

- Spawn
- Join
- Cancellation
- Publication
- Atomic ordering
- Resource contracts
- Deterministic scheduling
- Host-thread mapping

Required correction: VM scheduling may vary physically, but its observable behavior must follow the concurrency reference model and memory model.

The bytecode must encode enough metadata for verification.

---

#### TC-VM-008 — VM portability and native-target dependence conflict

If VM code contains native-width types, exact layout, target atomics, or direct addresses, one artifact may not be portable across targets.

Required correction: Classify VM modules:

text portable VM module target-profiled VM module native-bound VM module 

Portable modules cannot contain unresolved target-dependent layout or direct hardware mappings.

---

#### TC-ASM-001 — Assembly serial syntax is only illustrative

The core refers to assembly serial generation as a required compilation stage, but no normative grammar or semantics are provided.

Required correction: Decide whether assembly serial is:

- A mandatory interchange format
- A compiler-internal IR
- An optional diagnostic format
- A source-level assembly language

If mandatory, publish its grammar, type system, control flow, ownership representation, and versioning.

---

#### TC-ASM-002 — Requiring assembly serial may overconstrain compiler architecture

A conforming compiler could validly lower from semantic IR directly to native code without a textual serial form.

Recommended correction: Treat assembly serial as a required semantic artifact only for conformance levels that explicitly claim it, not as a universal internal pipeline mandate.

Equivalent internal representations should be permitted.

---

#### TC-TAG-001 — Symbolic tags and relocations are conflated

Tags may refer to:

- Local blocks
- Functions
- Data
- Cases
- Layers
- Foreign symbols
- Final machine addresses

These require different resolution phases and relocation types.

Required correction: Define separate namespaces and relocation classes:

text code label function symbol data symbol type symbol case metadata symbol layer metadata symbol external symbol 

Local labels should be resolved before object emission; external symbols remain relocatable.

---

#### TC-TAG-002 — Tag numeric examples imply unstable final addresses

Examples map tags directly to code addresses. In real object and dynamic-linking environments, final addresses may not be known until load time.

Required correction: Tags should resolve to symbolic object references and relocations, not necessarily absolute addresses.

The language must not make final address values semantically observable except through explicit direct operations.

---

#### TC-TAG-003 — Reproducible tags need canonical ordering

Tag and symbol numbering can vary with hash-map iteration, parallel compilation, or link order.

Required correction: Reproducible builds require canonical symbol ordering and deterministic relocation emission.

---

#### TC-NATIVE-001 — Native code need not pass through VM code

The specification alternates between:

text source -> VM -> native 

and:

text source -> VM or native 

Required correction: Define VM code as one permitted target or interchange layer, not necessarily a mandatory native-compilation stage unless the language constitution explicitly requires it.

Semantic equivalence is the conformance requirement.

---

#### TC-NATIVE-002 — Final native verification scope is unclear

“Emit verified VM code or native code” implies native code itself may be verified, but no verification method is stated.

Required correction: Distinguish:

text verified source-to-IR transformation verified VM artifact validated native artifact formally verified native artifact 

A normal compiler may validate final properties without proving arbitrary machine code correct.

---

#### TC-NATIVE-003 — Target heuristics may change layout of private types

This is generally legal, but cross-module optimization and incremental compilation require stable interface hashes.

Required correction: Public interface metadata must include every representation fact relied upon by separately compiled code. Private layout may change without ABI impact.

---

#### TC-NATIVE-004 — Native fallback libraries affect conformance

Wide integer arithmetic, f128, atomics, vectors, and transcendental operations may lower to runtime helper libraries.

Required correction: Such helpers are part of the conforming implementation and must preserve:

- Arithmetic policies
- Denial behavior
- determinism
- constant-time requirements where declared
- ABI
- memory ordering

Deployment profiles must be able to prohibit helper runtimes.

---

#### TC-NATIVE-005 — Direct blocks can bypass VM portability

A VM module containing direct machine addresses or architecture-specific assembly cannot be safely portable.

Required correction: Direct code must carry target requirements and capability metadata. A verifier must reject loading it on incompatible targets.

---

#### TC-NATIVE-006 — Link-time optimization may invalidate earlier proofs and reports

LTO can alter:

- Inlining
- Layout
- Alias assumptions
- Symbol visibility
- Vectorization
- Required optimization satisfaction

Required correction: Proofs and required-property reports must be revalidated after LTO and final layout.

---

#### TC-CONFORM-001 — Conformance levels depend on unspecified artifacts

The five conformance levels reference:

- Assembly serial
- VM output
- ABI
- native code
- atomics
- standard package integration
- reproducible builds

Several of these documents are only listed, not defined.

Required correction: A conformance level cannot become certifiable until every referenced normative document has a stable edition and test suite.

---

#### TC-CONFORM-002 — Optional implementation strategies need feature reporting

A compiler may emulate vectors, atomics, and wide values, while another may deny them under deployment policy.

Required correction: Every implementation should publish a machine-readable capability manifest containing:

text supported language edition conformance levels targets native and emulated scalar widths vector widths atomic guarantees VM versions ABI profiles direct-memory capabilities determinism support restricted-profile support 

---

#### TC-CONFORM-003 — Source acceptance and runtime support must be separated

A compiler may parse a feature but not support it on one target.

Required correction: Conformance testing should distinguish:

text front-end conformance semantic-analysis conformance target-lowering conformance runtime conformance artifact interoperability 

---

### Recommended representation model

Raygen should formalize four layers:

text Semantic type     meaning, operators, ownership, invariants  Logical representation     field and lane organization, index formulas, discriminants  Physical target representation     size, alignment, padding, register or memory form  ABI representation     externally stable calling and storage contract 

A change at a lower layer may not alter a higher-layer guarantee.

### Recommended direct-memory model

text 1. Direct authority is capability-scoped. 2. Raw numeric addresses do not automatically grant dereference authority. 3. Address provenance identifies a live allocation or declared external mapping. 4. Direct operations declare read, write, volatile, atomic, and lifetime effects. 5. Raw writes invalidate affected safe type and alias proofs. 6. Device, foreign, and ordinary memory occupy distinct domains. 7. Typed mapping does not imply ordinary object ownership. 8. Endianness, access width, alignment, and permissions are explicit. 

### Recommended artifact model

text Source module     target-independent where possible  Semantic interface     exported types, effects, policies, ownership, and layout obligations  Portable VM module     no unresolved target-specific direct operations  Targeted VM module     bound to a declared target profile  Native object     bound to one ABI and target profile  Executable     final verified policy, relocation, deployment, and capability manifest 

### Status

Representation independence is well founded, but binary and low-level conformance remain blocked.

A restricted implementation can support exact-width primitives, declaration-order units, explicit alignment, native-endian CPU memory, no packed atomics, opaque 256-bit rayword, a target-specific VM format, and audited C-compatible functions. Full Raygen 1 requires normative layout categories, padding and endianness rules, rayword and vector representation, ABI contracts, foreign ownership, direct-memory capabilities, VM validation, symbolic relocation rules, and artifact-level conformance definitions.



## 10. Core Language, Standard Library, and Privileged Syntax Boundaries

### Overall finding

Raygen explicitly separates the language kernel from standard-library facilities, which is the correct architectural direction. The current specification nevertheless assigns several facilities to inconsistent categories.

The principal ambiguity is the concept of a “privileged structural library”: tables, trees, graphs, nests, locks, channels, sorting, pipelines, mathematical loops, and memory mappings sometimes appear as core syntax, sometimes as compiler intrinsics, and sometimes as ordinary library abstractions.

A conforming implementation needs one stable rule for each feature:

1. Is its syntax recognized by the parser?
2. Are its semantics defined by the core specification?
3. Can a user-defined type implement the same behavior?
4. May the compiler replace it with an intrinsic?
5. Is the feature required at each conformance level?

### Critical issues

#### TC-BOUND-001 — The core-language inventory conflicts with the privileged-library boundary

Section 28 lists as core features:

text Arrays Slices Contiguous listings Static and dynamic derive map direct Cases Layers Atomics Regions Direct memory access 

Section 30 says tables, trees, graphs, and nests are privileged standard-library abstractions with optional syntax sugar.

Historical language material describes listings, nests, nodes, branches, trees, graphs, and tables as first-class language citizens.

Impact: It is unclear which declarations a core parser must recognize and which are supplied by imported packages.

Required correction: Publish a normative feature-boundary matrix.

Recommended classification:

| Feature | Parser syntax | Core semantics | Library implementation |
|---|---:|---:|---:|
| Array | Yes | Yes | No |
| Slice | Yes | Yes | No |
| Listing | No, except literal sugar | No | Yes |
| Nest | Optional sugar | No | Yes |
| Tree | Optional sugar | No | Yes |
| Graph | Optional sugar | No | Yes |
| Table | Optional sugar | No | Yes |
| Case | Yes | Yes | Runtime support permitted |
| Layer | Yes | Yes | Runtime support permitted |
| Lock | No | Synchronization guarantee only | Yes |
| Channel | No | Happens-before contract only | Yes |

---

#### TC-BOUND-002 — “Optional syntax sugar” is incompatible with source portability

If one conforming compiler supports:

rg1 table Accounts {     ... } 

and another does not, the same Raygen source is not portable.

Required correction: Optional syntax cannot be part of the standard Raygen 1 source language.

Choose one of:

- Make the syntax mandatory for every implementation at the required conformance level.
- Remove it from the core grammar and use ordinary declarations and function calls.
- Place it behind an explicit standardized feature declaration.

Recommended feature-gated form:

rg1 raygen 1 feature system.table_syntax 

A compiler that does not support the feature must issue a stable feature diagnostic rather than a generic parse error.

---

#### TC-BOUND-003 — Privileged library replacement rules are missing

A privileged library may be optimized through “stable intrinsic traits,” but those traits are not defined.

Questions include:

- Can third-party types implement them?
- Can an implementation recognize only the official library type?
- Are intrinsic traits forgeable?
- Must optimized and ordinary implementations be observationally identical?
- Can the compiler depend on private representation?

Required correction: Define intrinsic interfaces as sealed semantic contracts.

Example:

text core.intrinsic.Table core.intrinsic.ContiguousSequence core.intrinsic.Channel 

The compiler may specialize an implementation only when:

- The trait identity is trusted and edition-matched.
- All preconditions are verified.
- The specialization preserves documented semantics.
- Private representation dependencies are versioned.

Third-party types should use ordinary public traits unless explicitly certified.

---

#### TC-BOUND-004 — Standard-library replacement must preserve denial and effect behavior

A compiler-recognized operation may lower differently from the library implementation.

For example:

rg1 sort values ascending 

might become an intrinsic rather than a library call.

The optimized form must preserve:

- Stability selection
- Ordering relation
- Allocation behavior where semantically constrained
- Denial behavior
- Cancellation behavior
- Determinism
- Element ownership
- Comparator effects

Required correction: Every privileged operation needs a semantic contract independent of its implementation.

---

#### TC-BOUND-005 — Imports shown in examples do not match syntax use

Examples import:

rg1 use system.io use system.math 

but directly use:

rg1 emit math loop sort 

It is unclear whether these names are:

- Core keywords
- Imported names
- Prelude facilities
- Contextual syntax enabled by imports

Required correction: Imports must not silently alter lexical grammar unless a standardized syntax-extension system exists.

A keyword such as emit should either always belong to core grammar or be expressed as an ordinary imported function:

rg1 io.emit(value) 

---

#### TC-BOUND-006 — A standard prelude is not defined

Examples use many unqualified names:

text emit sum minimum maximum average cast overlaps separate allocate zero 

Required correction: Define whether Raygen has:

- No implicit prelude
- One mandatory prelude
- A profile-selected prelude
- Module-specific implicit imports

For auditability, a small fixed core prelude is preferable. All other facilities should require explicit imports.

---

#### TC-BOUND-007 — emit lacks a boundary classification

emit appears throughout the language, including the bootstrap subset, but its destination and effects are undefined.

It may mean:

- Console output
- Debug event
- General stream emission
- VM host output
- Compiler-recognized logging

Required correction: Either define emit as a core abstract output effect with a host-provided sink, or move it to system.io.

For a general-purpose systems language, ordinary output should be library-based. A minimal VM may provide a testing host call without making it a universal language instruction.

---

#### TC-BOUND-008 — Sorting is simultaneously a language operation and library algorithm

Historical material declares sorting to be inherent language syntax. The Normative Core does not list sorting in the core boundary and does not define its grammar.

Required correction: Make sorting a standard-library operation with optional standardized sugar only if the sugar is mandatory.

Recommended ordinary form:

rg1 values.sort(     order = ascending,     stability = stable ) 

Compiler intrinsic recognition can still provide specialized lowering.

---

#### TC-BOUND-009 — Sort comparator semantics are missing

Sorting structured data requires an ordering relation. The specification does not define:

- Total versus partial ordering
- NaN behavior
- Comparator denial
- Comparator side effects
- Mutation during sorting
- Stability with equal keys
- Deterministic handling of equivalent elements

Required correction: Sorting should require a pure ordering contract.

For floating-point values, the user must select a total-order policy or explicitly permit partial-order denial.

---

#### TC-BOUND-010 — rapid sorting is not semantically defined

Terms such as:

rg1 sort rapid samples 

sound like optimization preferences but may imply instability or nondeterminism.

Required correction: Separate semantic requirements from optimization preferences:

rg1 samples.sort(     stability = unstable,     policy = rapid ) 

unstable is semantic. rapid is advisory.

---

#### TC-BOUND-011 — Pipeline syntax has no ownership or denial contract

A pipeline stage receives the prior result, but it is unclear whether it receives it by:

- Move
- Shared borrow
- Mutable borrow
- Copy
- Stream subscription

It is also unclear how a denied result flows through later stages.

Required correction: Pipeline lowering must preserve the called operation’s parameter mode.

For example:

text x |> f 

lowers to f(x) and ordinary call rules determine ownership.

There should be no automatic denial propagation unless the stage syntax explicitly requests it.

---

#### TC-BOUND-012 — Structural pipeline operators require scope rules

Syntax such as:

rg1 where row.active order row.name ascending group row.category 

introduces implicit names like row and rows.

Required correction: Define them as lambda sugar or avoid implicit bindings.

For example:

rg1 users |> filter(row => row.active) |> sort_by(row => row.name, ascending) 

Privileged query syntax may lower to these forms, but scope and capture must be identical.

---

#### TC-BOUND-013 — Lazy versus eager structural operations remain library semantics

The core correctly says the pipeline does not imply laziness. However, this means the type returned by each stage is essential.

Required correction: Collection and query APIs must expose whether they return:

- Materialized collections
- Borrowing views
- Iterators
- Streams
- Query plans
- Transactions

The compiler may fuse stages only when effect, ownership, and denial equivalence is proven.

---

#### TC-BOUND-014 — Table semantics are far larger than the stated lowering

The conceptual lowering:

rg1 type Accounts =     core.table.Table<AccountSchema> 

does not define:

- Key uniqueness
- Index behavior
- Transactions
- Persistence
- Query semantics
- Schema evolution
- Concurrency
- Distribution
- Serialization

Required correction: The core should specify only the optional declaration sugar and stable intrinsic interface. All table behavior belongs to the Standard Library Specification.

Normative Core examples must not imply unspecified database guarantees.

---

#### TC-BOUND-015 — Table storage policies cannot be compiler-selected without observability rules

Row, column, mapped, and distributed storage have different effects on:

- Address stability
- Iteration order
- Mutation cost
- Transaction boundaries
- Foreign access
- Persistence

Required correction: The table library must distinguish semantic storage requirements from optimizer-selectable physical layout.

The compiler may change layout only when no public operation exposes the difference.

---

#### TC-BOUND-016 — Table iteration order is unspecified

A query without an explicit order clause may be:

- In insertion order
- Key order
- Storage order
- Arbitrary order

This affects semantic and schedule determinism.

Required correction: Unordered table queries must explicitly have unspecified order. A deterministic profile must require explicit ordering before order-sensitive publication or reduction.

---

#### TC-BOUND-017 — Table keys and indexes need type-level constraints

The Normative Core lists table-key uniqueness as a possible deduction target, but the standard-library boundary owns table semantics.

Required correction: The table specification must define whether uniqueness is:

- Statically proven
- Checked on insertion
- Transactionally enforced
- Eventually enforced in distributed tables

The compiler must not remove checks without a valid table capability contract.

---

#### TC-BOUND-018 — Trees and graphs have no canonical identity model

A tree node may be identified by:

- Address
- Stable index
- Key
- Handle
- Structural path
- Value identity

Representation independence requires identity not to depend accidentally on pointer layout.

Required correction: Tree and graph libraries must define explicit stable handle types.

Compiler layout transformations may change physical location without changing logical node identity.

---

#### TC-BOUND-019 — Graph edge direction must not imply ownership

Raygen’s directional model could be misread as making an edge own its target.

Required correction: Graph link semantics must explicitly distinguish:

text owning edge non-owning handle weak link external reference 

Ordinary graph connectivity should not imply memory ownership unless the library type says so.

---

#### TC-BOUND-020 — Traversal mutation rules are missing

During tree or graph traversal, a program might mutate the same structure.

The library must define whether this:

- Invalidates the iterator
- Is prohibited by borrowing
- Uses a snapshot
- Applies transactionally
- Is reflected immediately

Required correction: Standard traversal APIs should rely on Raygen access modes:

text read traversal write traversal snapshot traversal transactional traversal 

---

#### TC-BOUND-021 — Nest semantics are not clearly distinct from records

A nest appears to provide hierarchical named structure, but this can already be represented through nested records.

Required correction: Define the unique semantic purpose of nests or remove them as a distinct privileged abstraction.

Potential distinguishing features might include:

- Schema-oriented declarative initialization
- Path-based access
- Partial configuration merging
- Generated adapters

Those belong in the standard library unless required by core typing.

---

#### TC-BOUND-022 — Listings and contiguous collections need one canonical standard type

The Normative Core mentions “contiguous listings,” while historical material alternates among:

text list<T> listing<T> array<T, N> slice<T> 

Required correction: Standardize the names and relationships.

Recommended model:

text array<T, N>     fixed-size owned inline sequence slice<T>        non-owning contiguous view buffer<T>       owned contiguous allocation listing<T>      dynamically sized sequence, contiguous by default 

list<T> should not be a synonym unless formally declared as an alias.

---

#### TC-BOUND-023 — Collection allocation and denial behavior are missing

Operations such as:

rg1 scores.add(88) 

may allocate and fail.

Required correction: Standard collection APIs must expose allocation failure according to the active memory policy.

A potentially growing operation cannot have an infallible signature in bounded or no-heap profiles unless capacity is proven sufficient.

---

#### TC-BOUND-024 — Collection iterator invalidation must be specified

Mutating a listing may invalidate:

- Element references
- Slices
- Iterators
- Addresses
- Derived dependencies

Required correction: The standard collection specification must define invalidation after growth, insertion, removal, sorting, and layout conversion.

The compiler may use this information for borrow rejection and proof invalidation.

---

#### TC-BOUND-025 — Locks are described as privileged core-library types but no minimum API is defined

The memory model defines synchronization guarantees while delegating fairness and poisoning. At least the following still need a standard semantic interface:

text acquire try_acquire release through guard destruction read/write guards where applicable 

Required correction: Publish a core lock capability trait or one mandatory standard lock type.

The memory model alone is insufficient for portable source.

---

#### TC-BOUND-026 — Channels need a standard semantic trait

The memory model refers to channels as though every implementation shares one concept, but buffering, closure, and ownership are library concerns.

Required correction: Define a privileged channel interface containing at least:

text send receive close ordering capacity ownership transfer cancellation behavior 

Only successful matching send/receive operations receive the core happens-before guarantee.

---

#### TC-BOUND-027 — JSON and serialization boundaries need validated type construction

JSON is correctly classified as a standard-library feature, but dynamic deserialization interacts with refined types, nominal types, choices, and ownership.

Required correction: The serialization library must construct values only through validated type constructors.

It must not populate arbitrary object memory and bypass invariants.

---

#### TC-BOUND-028 — Cryptography cannot rely only on optimization policies

The standard library includes cryptography, while constant_time appears as a direct-map policy.

Required correction: Cryptographic APIs need a broader security contract covering:

- Constant-time operations
- Secret-dependent memory access
- Secure zeroing
- Randomness
- Key ownership
- Side-channel profiles
- Hardware fallback

A dispatch policy alone is insufficient.

---

#### TC-BOUND-029 — GPU APIs need explicit asynchronous lifetime semantics

GPU APIs are standard-library facilities, but layers may lower onto GPU queues.

The boundary must define:

- Buffer ownership while commands are in flight
- Host/device synchronization
- Failure reporting
- Cancellation
- Queue lifetime
- Device memory domains

Required correction: GPU integration should expose command and fence capabilities that connect to the core memory and case models.

---

#### TC-BOUND-030 — Process and filesystem operations need effect classification

The standard library contains process management and filesystems, but the effect system currently uses broad examples such as storage.write.

Required correction: Standard-library APIs must publish machine-readable effect summaries, including:

text filesystem.read filesystem.write network.send network.receive process.spawn process.wait clock.read random.read device.submit 

These summaries are required for derivation purity, determinism, deployment restrictions, and optimization.

---

#### TC-BOUND-031 — Privileged syntax must have canonical lowering before semantic analysis

Syntax sugar is safe only when its lowering does not depend on later overload resolution in an ambiguous way.

Required correction: Each privileged syntax form must define:

- Parse tree
- Name bindings
- Type-checking environment
- Canonical semantic operation
- Ownership behavior
- Effect behavior
- Denial behavior
- Source-location mapping

The lowering may target intrinsic interfaces rather than concrete library functions.

---

#### TC-BOUND-032 — User shadowing of privileged names is undefined

Can user code declare:

rg1 function sort(...) function where(...) type Table = ... 

If privileged syntax lowers by name, shadowing may alter semantics.

Required correction: Privileged syntax must resolve to edition-defined intrinsic identities, not ordinary lexical names.

Ordinary function-call syntax should continue to follow normal name resolution.

---

#### TC-BOUND-033 — Standard-library versioning can alter language behavior

If pipeline or table syntax lowers into library interfaces, updating the library could change source semantics without changing the Raygen edition.

Required correction: Privileged interfaces require edition-pinned semantic versions.

A Raygen 1 compiler should bind its sugar to a specified major interface edition, such as:

text core.table.intrinsic/v1 

Behavioral breaking changes require a new interface or language edition.

---

#### TC-BOUND-034 — Library absence must not change parsing

A source file using unavailable standard-library facilities should still parse.

Required correction: Parsing depends only on the language edition and declared syntax features. Missing packages or intrinsic interfaces produce resolution diagnostics after parsing.

---

#### TC-BOUND-035 — Conformance levels need library profiles

Raygen Core, Native, Concurrent, Professional, and Safety-Critical refer to different feature sets, but the Standard Library Specification is treated as one broad document.

Required correction: Define library profiles aligned with conformance levels:

text Core prelude Native systems profile Concurrent profile Professional collections and structure profile Safety-critical bounded profile 

A compiler should declare exactly which profile editions it supports.

---

#### TC-BOUND-036 — Safety-critical library behavior needs bounded alternatives

General listings, channels, JSON parsers, process APIs, and graph structures may allocate or recurse without fixed bounds.

Required correction: The safety-critical profile must provide or require:

- Fixed-capacity collections
- Bounded channels
- Bounded parsers
- No implicit growth
- Bounded graph traversal
- Explicit recursion limits
- Static or declared allocation plans

Using an unbounded general implementation should be rejected.

---

#### TC-BOUND-037 — Standard-library denial types need stability rules

Compiler-recognized library operations may produce typed denials. These types form part of source and ABI contracts.

Required correction: Privileged denial types must have stable identities, variant meanings, and versioning.

Implementations may enrich diagnostics but cannot silently replace one denial category with another.

---

#### TC-BOUND-038 — Compiler intrinsic fallback must always exist where portability is claimed

A privileged operation may map to an optimized intrinsic on one target and a library implementation on another.

Required correction: Unless a deployment policy forbids fallback, every portable privileged operation must have a semantically equivalent nonintrinsic implementation.

Target-specific facilities must be marked as such.

---

#### TC-BOUND-039 — Standard-library replacement must preserve capability checks

A compiler must not bypass:

- Filesystem authority
- Network authority
- Device authority
- Audit hooks
- Allocation limits
- Cancellation

when replacing a library call with an intrinsic.

Required correction: Capability and effect checks apply to the semantic operation, not merely to the library function body.

---

#### TC-BOUND-040 — Compiler plugins and syntax extensions are not bounded

Historical material mentions runtime plugins but not compiler plugins. A future extension mechanism could threaten grammar and semantic stability.

Required correction: Raygen 1 should prohibit arbitrary package-defined syntax extensions.

Compile-time generation should operate on standardized parsed structures, and compiler plugins should not be required for portable source unless a future edition defines a capability-safe extension model.

---

### Recommended boundary model

Raygen should use four distinct categories:

text Language kernel     Grammar and semantics required to type-check and execute basic programs.  Core semantic interfaces     Sealed identities used by the compiler for locks, channels, collections,     tables, and other privileged operations.  Standard library     Portable implementations and public APIs built on those interfaces.  Optional platform libraries     Filesystems, networking, GPU, database, and operating-system facilities. 

### Recommended privileged-syntax rule

> Privileged syntax is part of the language edition, not an optional compiler convenience. Each syntax form must lower to a stable intrinsic semantic interface. The standard library may implement that interface, and the compiler may substitute an equivalent intrinsic implementation.

### Proposed Raygen 1 boundary

A practical first-edition boundary is:

text Core syntax and semantics:     modules     functions     records     units     choices     arrays     slices     control flow     outcomes     ownership and borrowing     regions     policies     derivation     deduction     cases     layers     atomics     direct memory  Standard-library types:     text storage implementation     dynamic listings     buffers     locks     channels     tables     trees     graphs     nests     sorting     iterators     files     networking     JSON     cryptography     processes     GPU APIs  Optional mandatory sugar for Professional conformance:     structural table declarations     structural query pipelines     tree and graph declarations 

### Status

The intended boundary is sensible but currently inconsistent across the supplied specifications.

A restricted implementation should avoid optional parser extensions and express tables, trees, graphs, sorting, channels, and locks through explicit standard-library APIs. Full Professional conformance requires a mandatory privileged-syntax catalog, stable intrinsic interfaces, a defined prelude, library profile versioning, effect summaries, and portable fallback behavior.



## 11. Conformance Levels, Diagnostics, and Testability

### Overall finding

Raygen’s proposed conformance ladder is useful, but the current levels are not yet independently testable. Several levels depend on artifacts and specifications that remain undefined, while feature dependencies and diagnostic obligations are not expressed as machine-verifiable requirements.

A viable conformance system must distinguish language acceptance, semantic validation, artifact generation, runtime behavior, interoperability, and profile certification. It must also define how invalid programs fail, which diagnostics are stable, and whether any undefined behavior is permitted.

### Critical issues

#### TC-CONFORM-004 — Conformance levels are described by feature lists rather than closed obligations

The levels currently reference broad capabilities such as:

text Raygen Core Raygen Native Raygen Concurrent Raygen Professional Raygen Safety-Critical 

but do not define:

- Required language edition
- Required dependent specifications
- Mandatory target profiles
- Required runtime services
- Required tests
- Permitted unsupported features
- Whether levels are cumulative
- Whether partial conformance is allowed

Required correction: Define every level as a closed conformance profile.

Example:

text Raygen Core 1.0:     Normative Core 1.0     Core Grammar 1.0     Core Semantics 1.0     Core VM 1.0     Core Test Suite 1.0 

Each level should reference exact specification editions and test-suite versions.

---

#### TC-CONFORM-005 — Cumulative conformance is not explicit

It is unclear whether:

text Concurrent 

necessarily includes all Native obligations, or whether:

text Safety-Critical 

is a separate restricted subset.

Required correction: Define the dependency graph.

Recommended model:

text Core   |   +-- Native   |     |   |     +-- Concurrent   |             |   |             +-- Professional   |   +-- Safety-Critical Core         |         +-- Safety-Critical Native         |         +-- Safety-Critical Concurrent 

Safety-critical conformance is better modeled as a restriction profile applied to another execution level rather than as the final level in one linear hierarchy.

---

#### TC-CONFORM-006 — Feature support and conformance claims are conflated

An implementation may support some Professional features while claiming only Native conformance.

Conversely, it may parse a feature but reject every use during lowering.

Required correction: Require separate declarations for:

text Conformance claims Supported optional features Experimental extensions Target capabilities Restricted or unavailable features 

An implementation must not imply conformance merely because it recognizes syntax.

---

#### TC-CONFORM-007 — Feature dependencies are not formally declared

Examples of hidden dependencies include:

- cases depend on ownership, cancellation, memory ordering, and typed outcomes.
- layers depend on effect analysis and publication rules.
- Dynamic derivation depends on graph storage, synchronization, and denial routing.
- Direct memory depends on provenance, layout, and target mappings.
- Reproducible builds depend on package, linker, and environment specifications.

Required correction: Publish a machine-readable feature dependency graph.

Example:

text feature concurrent.cases requires:     core.outcome     core.ownership     core.effects     memory.happens_before     runtime.structured_cancellation 

A compiler must deny enabling a feature when any mandatory dependency is unavailable.

---

#### TC-CONFORM-008 — Conformance of extensions is undefined

Implementations may add:

- Syntax
- Types
- Intrinsics
- Policies
- Memory orders
- Target directives
- Standard-library facilities

Extensions can make invalid standard programs valid or alter parsing.

Required correction: Extensions must be isolated through explicit namespaces or feature declarations.

A conforming mode must:

- Disable unrequested extensions.
- Preserve standard keyword behavior.
- Reject unknown standard-edition features predictably.
- Identify extension-dependent artifacts as nonportable.

---

#### TC-CONFORM-009 — Strict conformance mode is not required

Without a strict mode, compiler extensions, permissive recovery, or implementation defaults may conceal nonportable source.

Required correction: Every conforming compiler should provide a mode equivalent to:

text raygen build --conformance raygen-core-1.0 --extensions none 

The option spelling is implementation-specific, but the behavior must be testable.

---

#### TC-DIAG-001 — A “stable diagnostic” is not defined

The specification refers to stable diagnostics but does not state what must remain stable:

- Numeric code
- Symbolic identifier
- Severity
- Message text
- Source span
- Notes
- Ordering
- Exit status
- Structured metadata

Required correction: Only structured diagnostic identity should be normative.

Recommended diagnostic record:

text code: RG-TYPE-0042 category: type.mismatched_numeric_operands severity: error primary_span: ... related_spans: [...] parameters:     left_type: u8     right_type: u16 

Human-readable wording may vary without breaking conformance.

---

#### TC-DIAG-002 — Diagnostic terminology conflicts with language-level denial

The specification uses “compile-time denial” for invalid source and deny<E> for ordinary program failure.

This can confuse:

- Compiler diagnostics
- Constant-evaluation outcomes
- Runtime outcomes
- Build-policy rejection
- Linker rejection

Recommended correction: Reserve denial for language-level typed failure.

Use:

text compile-time error conformance error policy rejection target rejection link error runtime denial runtime trap 

The specification may retain philosophical wording, but test artifacts need distinct categories.

---

#### TC-DIAG-003 — Diagnostic categories are not enumerated

A test suite cannot require stable diagnostics without a registry.

Required correction: Create a versioned diagnostic taxonomy, such as:

text RG-LEX RG-PARSE RG-NAME RG-TYPE RG-OWN RG-LIFE RG-EFFECT RG-DERIVE RG-CONC RG-MEM RG-POLICY RG-LAYOUT RG-ABI RG-VM RG-TARGET RG-DEPLOY RG-CONFORM 

Each normative invalid condition should map to at least one required primary category.

---

#### TC-DIAG-004 — One error can legitimately match several categories

Example:

rg1 free borrowed_value 

could be classified as:

- Ownership violation
- Lifetime violation
- Invalid memory directive
- Type mismatch

Required correction: Define a required primary diagnostic category and optional related diagnostics.

For this example, the primary category should likely be ownership, with lifetime and directive notes allowed.

Conformance must not depend on arbitrary compiler pass ordering.

---

#### TC-DIAG-005 — Diagnostic ordering is unspecified

Parallel front ends or incremental compilers may emit errors in different orders.

Required correction: Ordinary diagnostic order should not be normative except where:

- One diagnostic suppresses dependent noise.
- Deterministic-build profiles require stable output.
- A test explicitly targets first-failure semantics.

For deterministic diagnostic mode, order by canonical source location and diagnostic code, not worker completion order.

---

#### TC-DIAG-006 — Error recovery can produce nonportable secondary diagnostics

After one syntax error, parsers may recover differently and emit different additional messages.

Required correction: Negative syntax tests should generally validate:

- Rejection
- Required primary diagnostic code
- Required primary span class

They should not require an exact complete error list unless recovery behavior is explicitly standardized.

---

#### TC-DIAG-007 — Source-span requirements are absent

Diagnostics should identify whether the primary span covers:

- The invalid token
- The complete expression
- The declaration
- The conflicting pair of locations

Required correction: Each diagnostic specification should define a span role rather than exact byte offsets where impractical.

Example:

text Primary span:     the second conflicting mutable borrow expression.  Related span:     the first still-live mutable borrow. 

UTF-8 byte offsets and user-facing columns must be clearly distinguished.

---

#### TC-DIAG-008 — Macro or generated-source diagnostics are not addressed

Future generation facilities may produce source or semantic declarations.

Required correction: Diagnostics must preserve an origin chain:

text generated location expansion or generation site original user declaration 

Conformance tests should validate structured origin metadata where generation is standardized.

---

#### TC-DIAG-009 — Policy conflict diagnostics need effective-policy evidence

A message saying “policy conflict” is insufficient for audit.

Required correction: Policy diagnostics should report:

text canonical policy key competing values source of each value scope and authority level mandatory status reason neither can be selected 

This should also appear in machine-readable build records.

---

#### TC-DIAG-010 — Failed deduction diagnostics must distinguish proof outcomes

An unproven condition due to budget exhaustion differs from a disproven condition.

Required correction: Deduction diagnostics should expose:

text proven disproven unproven_missing_fact unproven_budget_exhausted invalidated_after_lowering non_guardable 

The distinction is important for required optimization and safety certification.

---

#### TC-DIAG-011 — Runtime denial and trap reporting are underspecified

A runtime failure report may need:

- Denial type and variant
- Source location
- Case identity
- Layer identity
- Dependency version
- Policy profile
- Target
- Stack trace
- Ownership state
- Whether cleanup completed

Required correction: Define a minimum structured runtime event format for conformance tooling.

Production profiles may suppress sensitive details, but the semantic failure identity must remain available to the host or supervisor.

---

#### TC-DIAG-012 — Diagnostics cannot be silently downgraded by implementation options

A compiler option must not turn a required language error into a warning and still claim standard conformance.

Required correction: Diagnostic policy may promote warnings to errors or suppress explicitly suppressible notices. It may not weaken normative errors.

---

#### TC-DIAG-013 — Warning behavior should not be part of core conformance unless standardized

Warnings for style, suspicious code, or optimization opportunities vary legitimately.

Required correction: Separate:

text Normative errors Normative required notices Profile-required warnings Implementation advisory warnings 

Only the first three should appear in conformance requirements.

---

#### TC-TEST-001 — The conformance suite needs both positive and negative tests

Positive tests alone cannot verify rejection of unsafe or ambiguous programs.

Required correction: Every normative rule should be testable through one or more of:

text Positive compile test Negative compile test Runtime semantic test Artifact inspection test Cross-module test Cross-compiler interoperability test Profile rejection test Diagnostic structure test 

---

#### TC-TEST-002 — Negative tests need stable expected outcomes

A negative test should not merely expect “compilation fails.”

Required correction: Each negative test should declare:

text required result: reject required primary diagnostic: RG-OWN-0017 required phase: semantic analysis forbidden result: emitted conforming artifact 

Exact message wording should not be required.

---

#### TC-TEST-003 — Invalid programs must not proceed to artifact emission

A compiler might report an error but still emit stale or partial output.

Required correction: After a normative error, no artifact may be labeled conforming.

Tools may emit debugging IR or partial artifacts only when clearly marked invalid and unusable for conformance.

---

#### TC-TEST-004 — Tests must distinguish compile-time rejection from runtime guarding

For guardable conditions, two valid implementations may differ:

- One proves the condition.
- One inserts a runtime guard.

The test must not require one internal choice unless a policy mandates it.

Required correction: Semantic tests should validate allowed external behavior.

Artifact tests should require guard elimination only under a standardized proof or optimization profile.

---

#### TC-TEST-005 — Runtime tests need defined nondeterminism envelopes

Concurrent, floating-point, randomized, and externally interacting tests cannot always require one exact output.

Required correction: Tests must declare one of:

text Exact output Set of permitted outputs Trace constraints Partial-order constraints Statistical requirement Deterministic-profile exact output 

Unbounded “any behavior” is not a useful test oracle.

---

#### TC-TEST-006 — Memory-model litmus tests are required

Informal examples cannot validate atomics and happens-before.

Required correction: Publish litmus tests for at least:

text message passing store buffering load buffering independent reads of independent writes release/acquire publication release sequences sequential consistency relaxed modification order compare-exchange success and failure lock synchronization case spawn and join layer barriers channel send and receive 

Each test must define forbidden, required, and permitted outcomes.

---

#### TC-TEST-007 — Ownership tests need path-sensitive negative cases

The suite should include:

text use after move double free free through borrow move while borrowed partial move late initialization denial-path cleanup case cancellation cleanup region escape foreign retention returned borrow 

Required correction: Test both static rejection and required runtime behavior at direct or foreign boundaries.

---

#### TC-TEST-008 — Policy-resolution tests need a complete precedence matrix

Tests should cover:

text same-scope identical rules same-scope conflicts nested lexical override mandatory outer policy type invariant versus lexical policy build mandate versus module policy imported declaration policy target-selected policy optimization preference versus safety rule semantic behavior prohibited by safety profile 

A machine-readable expected effective-policy record should accompany each test.

---

#### TC-TEST-009 — Grammar tests must include newline and Unicode edge cases

The parser suite should cover:

text newline continuation closing-brace termination nested generics shift operators nested comments Unicode normalization confusable identifiers invalid UTF-8 string escapes typed literals contextual keywords 

Each grammar feature needs both accepted and rejected examples.

---

#### TC-TEST-010 — ABI tests need independently compiled components

Compiling and linking everything with one compiler does not prove ABI interoperability.

Required correction: ABI tests should compile producer and consumer modules separately, ideally with different conforming compilers or compiler versions.

Tests should cover:

text primitive calls records and units choices outcomes vectors rayword nominal wrappers callbacks ownership transfer foreign errors atomics at boundaries 

---

#### TC-TEST-011 — VM tests need malformed-module cases

A verifier must reject hostile or corrupted artifacts.

Required correction: The VM suite should include invalid modules with:

text bad opcode truncated operands invalid jump target type mismatch uninitialized register ownership duplication invalid denial edge region escape illegal atomic order missing capability invalid relocation malformed metadata 

The verifier must reject them before execution.

---

#### TC-TEST-012 — Reproducible-build tests need controlled perturbations

Building the same source twice on one machine is insufficient.

Required correction: Tests should vary permitted irrelevant inputs such as:

text filesystem path directory enumeration order parallel job count host locale timezone build timestamp temporary directory environment variable order 

Artifacts must remain identical under the declared reproducible profile.

---

#### TC-TEST-013 — Safety-critical tests require resource-bound evidence

Functional correctness does not establish bounded behavior.

Required correction: Safety-critical conformance should test or verify:

text maximum stack maximum heap or no heap bounded allocation count bounded retries bounded queue capacity bounded recursion atomic progress requirement scheduler response bound absence of unapproved foreign calls absence of unchecked direct blocks 

These may require static reports rather than ordinary runtime tests.

---

#### TC-TEST-014 — Test-suite self-versioning is essential

Changing a test expectation can alter who is conforming.

Required correction: Every test suite release needs:

text suite edition specification edition test identifier stability policy errata mechanism expected-result manifest cryptographic integrity record 

Conformance claims should name the exact suite edition passed.

---

#### TC-TEST-015 — Normative examples must become executable tests

Many examples currently use undefined or illustrative syntax.

Required correction: Each example should be labeled:

text normative-valid normative-invalid informative-pseudocode illustrative-library-code 

Only normative-valid examples should be automatically compiled as acceptance tests.

---

#### TC-TEST-016 — Implementation-defined behavior needs testable declarations

When the specification permits implementation choice, the implementation must disclose the choice.

Examples may include:

- Supported target profiles
- Native atomic widths
- Lock-free status
- Deduction budget limits
- Maximum object size
- Optional library profiles

Required correction: Tests should compare implementation behavior to its capability manifest, not assume one universal implementation choice.

---

#### TC-UB-001 — The specification does not clearly state whether undefined behavior exists

Raygen repeatedly promises safety, runtime guards, denials, and traps, but direct and foreign operations imply that some executions may fall outside ordinary guarantees.

Required correction: State the language policy explicitly.

Recommended rule:

> Conforming safe Raygen has no undefined behavior. Every well-formed safe program has behavior defined by the language, a typed denial, a specified trap, or an explicitly permitted nondeterministic result.

For direct and foreign boundaries:

> Violating a documented direct or foreign precondition produces behavior outside the safe-language guarantee. The specification must classify each such violation as a required trap, target-defined behavior, or unsafe undefined behavior.

---

#### TC-UB-002 — “Implementation-defined,” “unspecified,” and “undefined” are not distinguished

These terms must not be used interchangeably.

Required correction: Define:

text implementation-defined:     one documented choice made by an implementation.  unspecified:     one behavior from a defined permitted set; documentation is not required     unless a profile says otherwise.  nondeterministic:     behavior may vary between executions within a defined permitted set.  undefined:     no language requirements apply after the triggering violation.  trap:     execution terminates or transfers control through a specified trap mechanism. 

Raygen should minimize or eliminate undefined behavior outside explicitly unsafe boundaries.

---

#### TC-UB-003 — Erroneous safe programs must not become optimization poison

In C-like models, a possible data race or invalid pointer can allow arbitrary optimizer behavior. That conflicts with Raygen’s safe-code goals.

Required correction: Unresolved safety obligations in safe code must cause:

- Compile-time rejection
- A runtime guard
- Synchronization
- A specified trap

They must not authorize arbitrary transformations.

---

#### TC-UB-004 — Direct-code undefined behavior needs containment rules

If direct code violates provenance or layout assumptions, can that invalidate the entire program, or only the affected operation?

Required correction: Define containment where possible.

Recommended categories:

text Local trap violation:     invalid access traps before a value is produced.  Unsafe state corruption:     guarantees for reachable corrupted state are lost.  Process-level unsafe violation:     no further language guarantees. 

The specification should avoid claiming recoverability after arbitrary memory corruption.

---

#### TC-UB-005 — Foreign behavior must be contract-relative

A foreign function may violate its declared effects or ownership contract.

Required correction: If foreign code violates an audited contract, Raygen cannot guarantee ordinary safety. The boundary should be classified as unsafe contract violation, with optional runtime containment mechanisms.

Conformance tests validate correct behavior under a conforming foreign implementation, not arbitrary hostile native code unless sandboxing is a declared feature.

---

#### TC-UB-006 — Resource exhaustion behavior must be defined

Potential exhaustion includes:

- Memory
- Stack
- Case handles
- Threads
- Graph nodes
- File descriptors
- VM registers
- Symbol tables
- Compiler deduction budget

Required correction: Each resource class must map exhaustion to one of:

text typed denial compile-time rejection configured trap host failure outside language execution 

Silent corruption or arbitrary behavior is prohibited.

---

#### TC-UB-007 — Stack overflow cannot remain implicit

If recursion or large local storage exceeds the stack, behavior must be specified.

Required correction: General profiles may define stack exhaustion as a trap. Safety-critical profiles should require static bounds or prohibit unbounded recursion.

---

#### TC-UB-008 — Integer division edge cases need explicit outcomes

Cases include:

text division by zero signed minimum divided by -1 remainder sign oversized shifts negative shifts 

Required correction: Define each as a semantic result, typed denial, or trap under the active arithmetic policy. None should be undefined in safe code.

---

#### TC-UB-009 — Invalid enum or choice discriminants need defined behavior

Raw memory or foreign input may create invalid discriminants.

Required correction: Safe construction validates discriminants. Direct unchecked interpretation may trap on observation or be classified as unsafe invalid representation. The optimizer must not assume validity across arbitrary foreign mutation without revalidation.

---

#### TC-UB-010 — Uninitialized reads must have one clear status

The ownership and late rules imply they are invalid, but runtime behavior is not explicit.

Required correction: Safe code must reject statically possible uninitialized reads. At direct boundaries, an attempted uninitialized read should trap or be classified as unsafe invalid representation; it must not yield arbitrary safe values.

---

#### TC-INTEROP-001 — Cross-compiler interoperability target is unclear

Two compilers may agree on source semantics but disagree on:

- Object format metadata
- Name mangling
- Generic instantiation
- VM encoding
- Diagnostic codes
- Package manifests
- Runtime ABI

Required correction: Define interoperability tiers:

text Source interoperability VM-module interoperability Native ABI interoperability Package-interface interoperability Debug-information interoperability Build-metadata interoperability 

Conformance claims must identify which tiers are supported.

---

#### TC-INTEROP-002 — Common runtime ownership is required for native interoperability

A value allocated or retained by one compiler’s runtime may be destroyed by another module.

Required correction: Native ABI profiles must define:

- Allocator ownership
- Destructor entry points
- Region-handle ABI
- Shared ownership runtime
- Case-handle runtime
- Outcome cleanup
- Exception or trap boundary behavior

Without this, cross-compiler ownership transfer is unsafe.

---

#### TC-INTEROP-003 — Generic cross-compiler interoperability may require canonical IR

Different compilers may use incompatible monomorphization and trait-resolution metadata.

Required correction: Choose one of:

- Generics cannot cross native binary interfaces.
- Exported generics are instantiated by the consumer from canonical semantic metadata.
- A standardized generic ABI and mangling scheme is required.

The first option is simplest for Raygen 1 Native.

---

#### TC-INTEROP-004 — Inline functions and semantic policies need interface serialization

An imported inline or generic function must carry its defining:

- Semantic policies
- Effects
- Ownership contracts
- Layout assumptions
- Denial types
- Target restrictions

Required correction: Package interfaces need a canonical semantic representation and compatibility hash.

---

#### TC-INTEROP-005 — VM version negotiation is absent

A runtime loading a module needs to know whether it supports the module’s:

- VM edition
- Feature set
- Target profile
- Capability requirements
- Standard-library intrinsic versions

Required correction: VM modules should contain mandatory version and feature manifests. Unsupported requirements cause validation rejection before execution.

---

#### TC-INTEROP-006 — Symbol visibility and versioning are unspecified

Native libraries need stable rules for:

- Exported symbols
- Hidden symbols
- Symbol versions
- Weak symbols
- Replacement
- Dynamic loading
- Package identity

Required correction: The ABI and package specifications must define exported symbol identity independently of source filenames and local module paths.

---

#### TC-INTEROP-007 — Denial types need stable cross-module identity

Two modules may independently define structurally identical errors that are nominally distinct.

Required correction: Exported denial types require canonical package-qualified identities and version compatibility rules.

Cross-module propagation cannot depend only on structural similarity.

---

#### TC-INTEROP-008 — Trait coherence must be global across separately compiled packages

Two packages could provide conflicting implementations only discovered at final link.

Required correction: Package metadata must include implementation identities and coherence keys. The linker or package resolver must reject conflicting reachable implementations deterministically.

---

#### TC-INTEROP-009 — Target-profile compatibility must be checked at link and load time

Modules compiled for different:

- Endianness
- ISA features
- Atomic guarantees
- Floating modes
- ABIs
- Direct-memory domains

may not be safely combined.

Required correction: Every artifact must carry a structured target profile. Linkers and loaders must verify compatibility or invoke an explicitly specified adaptation boundary.

---

#### TC-INTEROP-010 — Reproducible artifacts require canonical metadata across compilers

Two different conforming compilers may not produce byte-identical output because code generation differs.

Required correction: Clarify that reproducible builds normally mean repeatability for the same declared toolchain identity, not identical artifacts across all conforming compilers.

Cross-compiler semantic equivalence is a separate test objective.

---

### Recommended conformance architecture

Use orthogonal claims rather than one overloaded level:

text Language edition:     Raygen 1  Front-end profile:     Core     Professional  Execution profile:     VM     Native     Hybrid  Concurrency profile:     Sequential     Structured Concurrent  Safety profile:     General Safe     Safety-Critical Bounded  Interoperability profile:     Source     VM     Native ABI     Package Interface  Determinism profile:     Semantic     Schedule     Numerical Bitwise     Reproducible Build 

A conformance claim could then be expressed as:

text Raygen 1 Professional Front End Native Execution Structured Concurrent General Safe Native ABI Interoperability Schedule Deterministic 

This is more precise than one cumulative label.

### Recommended conformance test manifest

Each test should contain machine-readable metadata similar to:

yaml id: RG1-OWN-0047 specification: RG-NCORE-1.0 profile:   front_end: core   execution: vm kind: negative_compile source: use_after_move.rg expected:   result: reject   primary_diagnostic: RG-OWN-0017   phase: ownership_validation 

Runtime tests should additionally define permitted outcomes, target requirements, and timeout or progress assumptions.

### Recommended diagnostic policy

text 1. Every normative invalid condition has a stable symbolic code. 2. Human-readable message wording is not normative. 3. Required diagnostics include structured source and related spans. 4. Secondary diagnostics may vary unless explicitly tested. 5. Normative errors cannot be suppressed or downgraded. 6. Deterministic diagnostic mode uses canonical ordering. 7. Runtime denials and traps have distinct structured event identities. 

### Recommended undefined-behavior policy

> Safe Raygen contains no undefined behavior. A safe execution produces a defined value, a typed denial, a specified trap, or a result within an explicitly defined nondeterministic set. Undefined behavior is permitted only inside explicitly unsafe direct or foreign contracts, and each such boundary must document the violated preconditions and the scope of lost guarantees.

### Status

Conformance certification is not yet possible.

The specification can support early implementation test suites, but formal conformance requires exact profile dependencies, a versioned diagnostic registry, executable positive and negative tests, a defined safe-code undefined-behavior policy, memory-model litmus tests, artifact manifests, and explicit cross-compiler interoperability tiers.



