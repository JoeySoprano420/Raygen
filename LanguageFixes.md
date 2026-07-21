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



