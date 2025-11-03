# SQLite Command Execution Flow (Based on Amalgamation Source)

## Complete Execution Pipeline

```md
SQL Command: "SELECT name FROM users WHERE id = 5"
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 1: APPLICATION INTERFACE LAYER                                  ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 1. Application Call: sqlite3_prepare_v2()                             ║
║    └─> Entry point for SQL statement preparation                      ║
║    └─> sqlite3LockAndPrepare()                                        ║
║        ├─> sqlite3BtreeEnterAll() - Acquire database mutexes          ║
║        │   └─> Ensure thread-safe access to database                  ║
║        ├─> sqlite3BtreeSchemaLocked() - Check schema locks            ║
║        │   └─> Verify no other connection is modifying schema         ║
║        └─> sqlite3Prepare()                                           ║
║            ├─> Allocate Parse structure (parser state)                ║
║            ├─> Initialize Parse->db, Parse->pVdbe                     ║
║            ├─> Check SQL statement length limits                      ║
║            └─> Allocate memory for tokenization/parsing               ║
╚═══════════════════════════════════════════════════════════════════════╝
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 2: LEXICAL ANALYSIS (TOKENIZATION)                              ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 2. Tokenizer (Lexer)                                                  ║
║    └─> sqlite3RunParser() calls sqlite3GetToken() in loop             ║
║        ├─> Scan SQL string character by character                     ║
║        ├─> Identify token boundaries                                  ║
║        ├─> Generate token stream:                                     ║
║        │   • TK_SELECT  - "SELECT" keyword                            ║
║        │   • TK_ID      - "name" (column identifier)                  ║
║        │   • TK_FROM    - "FROM" keyword                              ║
║        │   • TK_ID      - "users" (table identifier)                  ║
║        │   • TK_WHERE   - "WHERE" keyword                             ║
║        │   • TK_ID      - "id" (column identifier)                    ║
║        │   • TK_EQ      - "=" (equality operator)                     ║
║        │   • TK_INTEGER - "5" (integer literal)                       ║
║        ├─> Skip whitespace (TK_SPACE) and comments (TK_COMMENT)       ║
║        ├─> Check for query interrupts (sqlite3_interrupt)             ║
║        └─> Validate SQL length within limits                          ║
╚═══════════════════════════════════════════════════════════════════════╝
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 3: SYNTAX ANALYSIS (PARSING)                                    ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 3. Lemon Parser (Generated from parse.y)                              ║
║    └─> sqlite3ParserAlloc() - Initialize LALR(1) parser state machine ║
║    └─> Feed token stream to parser                                    ║
║        ├─> Apply grammar rules from parse.y                           ║
║        ├─> Build Abstract Syntax Tree (AST)                           ║
║        │   └─> Tree structure representing SQL semantics              ║
║        ├─> Create Select structure:                                   ║
║        │   ├─> pEList (ExprList) - columns to select (name)           ║
║        │   ├─> pSrc (SrcList) - tables involved (users)               ║
║        │   ├─> pWhere (Expr) - WHERE condition tree                   ║
║        │   └─> flags, ordering, limits, etc.                          ║
║        ├─> Resolve identifiers:                                       ║
║        │   ├─> Table name resolution (users → table ID)               ║
║        │   ├─> Column name resolution (name, id → column numbers)     ║
║        │   └─> Check column/table existence in schema                 ║
║        ├─> Build expression tree for WHERE clause:                    ║
║        │   └─> Expr(OP_EQ): left=id, right=5                          ║
║        ├─> Type checking and validation                               ║
║        ├─> Semantic analysis (constraints, permissions)               ║
║        └─> Syntax error detection and reporting                       ║
╚═══════════════════════════════════════════════════════════════════════╝
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 4: QUERY OPTIMIZATION                                           ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 4. Query Optimizer                                                    ║
║    └─> sqlite3WhereBegin() - WHERE clause optimizer                   ║
║        ├─> Analyze WHERE clause predicates:                           ║
║        │   └─> Extract: "id = 5" (equality on indexed column)         ║
║        ├─> Examine table schema:                                      ║
║        │   ├─> Check for indexes on 'users' table                     ║
║        │   ├─> Identify primary key, unique indexes                   ║
║        │   └─> Gather table statistics (row count estimates)          ║
║        ├─> Cost-based analysis (whereLoopAddBtree):                   ║
║        │   ├─> Option 1: Full table scan                              ║
║        │   │   └─> Cost = N rows * scan_cost_per_row                  ║
║        │   ├─> Option 2: Index scan on id                             ║
║        │   │   └─> Cost = log(N) + lookup_cost                        ║
║        │   └─> Compare costs, select optimal strategy                 ║
║        ├─> Join order optimization (if multiple tables):              ║
║        │   ├─> Estimate join costs                                    ║
║        │   └─> Reorder joins for minimum total cost                   ║
║        ├─> Index selection:                                           ║
║        │   └─> Choose best index for WHERE predicates                 ║
║        └─> Generate WhereInfo structure:                              ║
║            ├─> Execution plan details                                 ║
║            ├─> Loop strategy (scan/seek)                              ║
║            └─> Index usage information                                ║
╚═══════════════════════════════════════════════════════════════════════╝
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 5: VDBE BYTECODE GENERATION                                     ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 5. Code Generator                                                     ║
║    └─> sqlite3FinishCoding() - Convert AST to VDBE bytecode           ║
║        ├─> Create Vdbe structure (virtual machine instance)           ║
║        ├─> Generate initialization code:                              ║
║        │   └─> OP_Init (jump to main program entry)                   ║
║        ├─> Generate transaction setup:                                ║
║        │   └─> OP_Transaction (start read transaction)                ║
║        ├─> Generate table access code:                                ║
║        │   ├─> OP_OpenRead (cursor 0 → users table)                   ║
║        │   └─> Register cursor in VDBE cursor array                   ║
║        ├─> Generate WHERE loop code:                                  ║
║        │   ├─> OP_Rewind (position at first row)                      ║
║        │   │   OR OP_SeekGE (if using index seek)                     ║
║        │   ├─> Loop body:                                             ║
║        │   │   ├─> OP_Column (read id → register r[1])                ║
║        │   │   ├─> OP_Ne (if r[1] ≠ 5, jump to next iteration)        ║
║        │   │   ├─> OP_Column (read name → register r[2])              ║
║        │   │   ├─> OP_ResultRow (output r[2] to result set)           ║
║        │   │   └─> OP_Next (advance cursor, loop back)                ║
║        │   └─> Loop exit                                              ║
║        ├─> Generate cleanup code:                                     ║
║        │   ├─> OP_Close (close cursors)                               ║
║        │   └─> OP_Halt (terminate program)                            ║
║        ├─> Schema validation code:                                    ║
║        │   └─> OP_VerifyCookie (check schema version)                 ║
║        ├─> resolveP2Values():                                         ║
║        │   └─> Convert symbolic jump labels → absolute addresses      ║
║        └─> sqlite3VdbeMakeReady() - Prepare VDBE for execution:       ║
║            ├─> Allocate memory registers (aMem array)                 ║
║            ├─> Allocate cursor array (apCsr)                          ║
║            ├─> Allocate variable bindings (aVar for prepared params)  ║
║            ├─> Set up return value registers                          ║
║            └─> sqlite3VdbeRewind() - Initialize PC to 0               ║
╚═══════════════════════════════════════════════════════════════════════╝
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 6: STATEMENT PREPARATION COMPLETE                               ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 6. Return to Application                                              ║
║    └─> sqlite3Prepare() completes                                     ║
║        ├─> sqlite3VdbeSetSql() - Store original SQL in statement      ║
║        ├─> Schema validation - schemaIsValid()                        ║
║        │   └─> Compare schema cookies (detect schema changes)         ║
║        ├─> Set statement state → VDBE_READY                           ║
║        ├─> Store statement handle (*ppStmt)                           ║
║        └─> Return SQLITE_OK to application                            ║
║                                                                        ║
║ Application now has prepared statement ready for execution            ║
╚═══════════════════════════════════════════════════════════════════════╝
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 7: BYTECODE EXECUTION                                           ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 7. Application Call: sqlite3_step()                                   ║
║    └─> sqlite3Step() - Execute next step of prepared statement        ║
║        ├─> Increment db->nVdbeActive (track active statements)        ║
║        ├─> Check statement state (must be READY or RUN)               ║
║        └─> sqlite3VdbeExec() - VDBE execution engine                  ║
║            ├─> Enter execution loop                                   ║
║            ├─> Fetch opcode: pOp = &aOp[pc]                          ║
║            ├─> Execute opcodes sequentially:                          ║
║            │                                                           ║
║            │   [Addr 0] OP_Init                                       ║
║            │   └─> Initialize VDBE state, jump to start               ║
║            │                                                           ║
║            │   [Addr N] OP_Transaction (p1=0, read-only)              ║
║            │   └─> Begin read transaction on database                 ║
║            │   └─> Acquire SHARED lock via Pager                      ║
║            │                                                           ║
║            │   [Addr N+1] OP_VerifyCookie                             ║
║            │   └─> Check schema cookie hasn't changed                 ║
║            │   └─> If changed → SQLITE_SCHEMA error                   ║
║            │                                                           ║
║            │   [Addr N+2] OP_OpenRead (p1=cursor_id, p2=root_page)    ║
║            │   └─> Open B-tree cursor on users table                  ║
║            │   └─> Store cursor in apCsr[cursor_id]                   ║
║            │   └─> Call sqlite3BtreeCursor()                          ║
║            │                                                           ║
║            │   [Addr N+3] OP_Rewind (p1=cursor_id, p2=exit_label)     ║
║            │   └─> Position cursor at first row                       ║
║            │   └─> If table empty, jump to exit_label                 ║
║            │   └─> Call sqlite3BtreeFirst()                           ║
║            │                                                           ║
║            │   [Loop Start]                                           ║
║            │   [Addr N+4] OP_Column (p1=cursor, p2=col_num, p3=reg)   ║
║            │   └─> Read column 'id' from current row → r[1]           ║
║            │   └─> Call sqlite3VdbeSerialGet() to deserialize         ║
║            │                                                           ║
║            │   [Addr N+5] OP_Ne (p1=r[1], p2=next_iter, p3=5)         ║
║            │   └─> Compare r[1] with literal 5                        ║
║            │   └─> If r[1] ≠ 5, jump to next iteration                ║
║            │                                                           ║
║            │   [Addr N+6] OP_Column (p1=cursor, p2=col_num, p3=reg)   ║
║            │   └─> Read column 'name' from current row → r[2]         ║
║            │                                                           ║
║            │   [Addr N+7] OP_ResultRow (p1=r[2], p2=num_cols)         ║
║            │   └─> Set up result row for application                  ║
║            │   └─> RETURN SQLITE_ROW to application                   ║
║            │   └─> Execution pauses here (yield to app)               ║
║            │                                                           ║
║            │   [Addr N+8] OP_Next (p1=cursor, p2=loop_start)          ║
║            │   └─> Advance cursor to next row                         ║
║            │   └─> If more rows exist, jump to loop_start             ║
║            │   └─> Call sqlite3BtreeNext()                            ║
║            │   └─> Else, fall through to exit                         ║
║            │   [Loop End]                                             ║
║            │                                                           ║
║            │   [Addr N+9] OP_Close (p1=cursor_id)                     ║
║            │   └─> Close B-tree cursor                                ║
║            │   └─> Release cursor resources                           ║
║            │                                                           ║
║            │   [Addr N+10] OP_Halt (p1=SQLITE_OK)                     ║
║            │   └─> Terminate VDBE execution                           ║
║            │   └─> RETURN SQLITE_DONE to application                  ║
║            │                                                           ║
║            ├─> Update program counter (pc++)                          ║
║            ├─> Check for interrupts (sqlite3_interrupt)               ║
║            └─> Handle errors (rollback if needed)                     ║
╚═══════════════════════════════════════════════════════════════════════╝
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 8: STORAGE LAYER (B-TREE & PAGER)                               ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 8. Backend Storage Operations (during VDBE execution)                 ║
║                                                                        ║
║    A. B-TREE LAYER                                                    ║
║    ├─> sqlite3BtreeCursor() - Create B-tree cursor                    ║
║    │   ├─> Allocate BtCursor structure                                ║
║    │   ├─> Initialize cursor state                                    ║
║    │   └─> Link to root page of table/index                           ║
║    ├─> sqlite3BtreeFirst() - Position at first entry                  ║
║    │   └─> Navigate to leftmost leaf node                             ║
║    ├─> sqlite3BtreeNext() - Advance to next entry                     ║
║    │   ├─> Move within current leaf node                              ║
║    │   └─> Or traverse to next leaf node                              ║
║    └─> sqlite3BtreeKey/Data() - Extract row data                      ║
║        └─> Read payload from B-tree cell                              ║
║                                                                        ║
║    B. PAGER LAYER                                                     ║
║    ├─> sqlite3PagerGet() - Fetch database page                        ║
║    │   ├─> Check page cache (PCache):                                 ║
║    │   │   ├─> Cache HIT: Return page from memory                     ║
║    │   │   └─> Cache MISS: Continue to disk read                      ║
║    │   ├─> Read from disk (if not cached):                            ║
║    │   │   ├─> Calculate page offset: page_num * page_size            ║
║    │   │   ├─> Call VFS layer: xRead()                                ║
║    │   │   ├─> Load page into memory buffer                           ║
║    │   │   └─> Add page to cache                                      ║
║    │   └─> Return page pointer to B-tree layer                        ║
║    ├─> Page caching strategy:                                         ║
║    │   ├─> LRU (Least Recently Used) eviction                         ║
║    │   ├─> Configurable cache size (PRAGMA cache_size)                ║
║    │   └─> Shared cache mode (multi-connection)                       ║
║    └─> sqlite3PagerRef/Unref() - Reference counting                   ║
║        └─> Track page usage for safe eviction                         ║
║                                                                       ║
║    C. VFS (VIRTUAL FILE SYSTEM) LAYER                                 ║
║    ├─> OS-specific file I/O abstraction                               ║
║    ├─> xRead() - Read bytes from database file                        ║
║    ├─> xLock/xUnlock() - File locking primitives                      ║
║    └─> Platform portability (Unix, Windows, custom)                   ║
║                                                                       ║
║    D. LOCKING & CONCURRENCY                                           ║
║    ├─> Lock escalation during read:                                   ║
║    │   └─> UNLOCKED → SHARED (allow concurrent reads)                 ║
║    ├─> Lock modes:                                                    ║
║    │   ├─> SHARED: Multiple readers allowed                           ║
║    │   ├─> RESERVED: Prepare for write (readers still allowed)        ║
║    │   ├─> PENDING: Block new readers                                 ║
║    │   └─> EXCLUSIVE: Single writer, no readers                       ║
║    └─> Deadlock prevention via lock ordering                          ║
║                                                                       ║
║    E. DATA DECODING                                                   ║
║    └─> Raw page data → Structured records                             ║
║        ├─> Parse page header (page type, cell count)                  ║
║        ├─> Locate cell in cell pointer array                          ║
║        ├─> Decode record header (serial types)                        ║
║        ├─> Extract column values (sqlite3VdbeSerialGet)               ║
║        └─> Store in VDBE registers (aMem array)                       ║
╚═══════════════════════════════════════════════════════════════════════╝
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 9: RESULT DELIVERY TO APPLICATION                               ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 9. Application Interaction Loop                                       ║
║                                                                        ║
║    while ((rc = sqlite3_step(stmt)) == SQLITE_ROW) {                  ║
║        ├─> SQLITE_ROW returned:                                       ║
║        │   ├─> Result row is ready in VDBE registers                  ║
║        │   ├─> Application calls column access functions:             ║
║        │   │   ├─> sqlite3_column_text(stmt, 0)                       ║
║        │   │   │   └─> Returns "name" value as text                   ║
║        │   │   ├─> sqlite3_column_int(stmt, N)                        ║
║        │   │   ├─> sqlite3_column_blob(stmt, N)                       ║
║        │   │   └─> sqlite3_column_type(stmt, N)                       ║
║        │   ├─> Type conversion if needed:                             ║
║        │   │   └─> INTEGER → TEXT, REAL → INTEGER, etc.               ║
║        │   └─> Application processes row data                         ║
║        │                                                               ║
║        └─> Next sqlite3_step() call:                                  ║
║            └─> Resume VDBE execution at next opcode                   ║
║    }                                                                   ║
║                                                                        ║
║    ├─> SQLITE_DONE returned:                                          ║
║    │   ├─> All rows have been processed                               ║
║    │   ├─> No more data to return                                     ║
║    │   ├─> Transaction automatically committed (if needed)            ║
║    │   └─> Statement remains prepared (can be reset/reused)           ║
║    │                                                                   ║
║    ├─> SQLITE_ERROR returned:                                         ║
║    │   ├─> Error occurred during execution                            ║
║    │   ├─> Application calls sqlite3_errmsg() for details             ║
║    │   ├─> Statement state → ERROR                                    ║
║    │   └─> Must reset before reuse                                    ║
║    │                                                                   ║
║    └─> SQLITE_SCHEMA returned:                                        ║
║        ├─> Schema changed since preparation                           ║
║        ├─> Auto-reprepare with sqlite3Reprepare()                     ║
║        └─> Retry execution with new schema                            ║
╚═══════════════════════════════════════════════════════════════════════╝
        |
        v
╔═══════════════════════════════════════════════════════════════════════╗
║ PHASE 10: CLEANUP & FINALIZATION                                      ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 10. Statement Cleanup                                                 ║
║     └─> Application calls sqlite3_finalize(stmt)                      ║
║         ├─> sqlite3VdbeFinalize()                                     ║
║         │   ├─> Close all open cursors                                ║
║         │   ├─> Release B-tree locks                                  ║
║         │   ├─> Free VDBE memory:                                     ║
║         │   │   ├─> aMem array (registers)                            ║
║         │   │   ├─> aOp array (bytecode)                              ║
║         │   │   ├─> apCsr array (cursors)                             ║
║         │   │   └─> aVar array (bindings)                             ║
║         │   ├─> Decrement db->nVdbeActive                             ║
║         │   └─> Release database connection resources                 ║
║         └─> Statement handle invalidated                              ║
╚═══════════════════════════════════════════════════════════════════════╝
```

## Key Components & Functions Reference

### Architecture Layers

| Layer | Component | Responsibility |
|-------|-----------|----------------|
| **Interface** | SQL API | Application entry points (sqlite3_prepare_v2, sqlite3_step, etc.) |
| **Compiler** | Tokenizer + Parser | SQL text → Abstract Syntax Tree |
| **Optimizer** | Query Planner | AST → Optimal execution plan |
| **Code Gen** | VDBE Compiler | Execution plan → Bytecode |
| **VM** | VDBE Engine | Execute bytecode instructions |
| **Backend** | B-tree Manager | Navigate table/index structures |
| **Storage** | Pager | Page-level I/O and caching |
| **OS** | VFS | Platform-specific file operations |

### Key Functions by Phase

| Phase | Key Functions | Purpose | Approx. Line |
|-------|---------------|---------|--------------|
| **Interface** | `sqlite3_prepare_v2()` | Main API entry for SQL preparation | ~144676 |
| | `sqlite3LockAndPrepare()` | Acquire locks and initiate preparation | ~144581 |
| | `sqlite3BtreeEnterAll()` | Lock all attached databases | - |
| **Tokenizer** | `sqlite3GetToken()` | Extract next token from SQL string | Called at ~181949 |
| | `sqlite3RunParser()` | Main parser loop driver | ~181949 |
| **Parser** | `sqlite3ParserAlloc()` | Create Lemon parser instance | - |
| | `sqlite3Parser()` | Feed tokens to Lemon grammar | - |
| | `sqlite3Select()` | Process SELECT statement | - |
| **Resolver** | `sqlite3ResolveExprNames()` | Resolve column/table names | - |
| | `sqlite3ResolveSelfReference()` | Handle self-referential queries | - |
| **Optimizer** | `sqlite3WhereBegin()` | Initialize WHERE clause optimizer | ~152964 |
| | `whereLoopAddBtree()` | Analyze B-tree scan options | - |
| | `wherePathSolver()` | Choose optimal query path | - |
| **Code Gen** | `sqlite3FinishCoding()` | Finalize bytecode generation | ~123144 |
| | `sqlite3VdbeAddOp*()` | Add opcode to VDBE program | - |
| | `resolveP2Values()` | Resolve jump addresses | ~86766 |
| **VDBE Prep** | `sqlite3VdbeMakeReady()` | Allocate VDBE runtime structures | ~88538 |
| | `sqlite3VdbeRewind()` | Reset program counter to start | - |
| **Execution** | `sqlite3_step()` | Execute next step (public API) | - |
| | `sqlite3Step()` | Internal step implementation | ~92350 |
| | `sqlite3VdbeExec()` | Main VDBE execution loop | ~95154 |
| **B-tree** | `sqlite3BtreeCursor()` | Create B-tree cursor | - |
| | `sqlite3BtreeFirst()` | Move to first entry | - |
| | `sqlite3BtreeNext()` | Advance to next entry | - |
| | `sqlite3BtreeData()` | Read payload data | - |
| **Pager** | `sqlite3PagerGet()` | Fetch page (cached or from disk) | - |
| | `sqlite3PagerWrite()` | Mark page for writing | - |
| | `sqlite3PagerCommitPhaseOne()` | Prepare transaction commit | - |
| **Schema** | `schemaIsValid()` | Validate schema cookie | ~144237 |
| | `sqlite3BtreeSchemaLocked()` | Check schema lock status | ~144497 |
| **Cleanup** | `sqlite3_finalize()` | Free prepared statement (public API) | - |
| | `sqlite3VdbeFinalize()` | Internal cleanup implementation | - |

## Critical Substeps & Mechanisms

### 1. Mutex & Lock Management

- **Function**: `sqlite3BtreeEnterAll()`
- **Purpose**: Acquire mutexes for all attached databases before parsing
- **Why Critical**: Prevents race conditions during schema access
- **Lock Hierarchy**: Database mutex → B-tree mutex → Page mutex

### 2. Schema Validation System

- **Schema Cookie**: Integer stored in database header (offset 40)
- **Check Points**:
  1. Before parsing: `sqlite3BtreeSchemaLocked()`
  2. Before execution: `schemaIsValid()`
  3. During execution: `OP_VerifyCookie`
- **Auto-Reprepare**: On SQLITE_SCHEMA error, `sqlite3Reprepare()` regenerates bytecode

### 3. Memory Allocation Strategy

- **Parse Structure**: Temporary storage during compilation
  - Contains: Table/column resolution maps, AST nodes, error state
- **VDBE Structures** (persistent until finalize):
  - `aMem[]`: Register array for runtime values
  - `aOp[]`: Bytecode instruction array
  - `apCsr[]`: Cursor array for table/index access
  - `aVar[]`: Bound parameter storage
- **Allocation**: Uses `sqlite3DbMalloc()` with error checking

### 4. Jump Label Resolution

- **Function**: `resolveP2Values()`
- **Purpose**: Convert symbolic labels to absolute bytecode addresses
- **Process**:
  1. First pass: Collect label definitions
  2. Second pass: Replace label references with offsets
  3. Validate: Ensure all jumps are within bounds

### 5. VDBE State Machine

```md
States: INIT → READY → RUN → HALT
        ↓      ↓       ↓     ↓
Actions:       ↓       ↓     └─> Clean up, return result
        ↓      ↓       └─> Execute opcodes
        ↓      └─> Allocate runtime resources
        └─> Parse and compile
```

- **Transitions**: Controlled by return codes (OK, ROW, DONE, ERROR)
- **Error Recovery**: Can reset to READY state

### 6. Page Cache Architecture

- **Structure**: Hash table + LRU list
- **Cache Levels**:
  1. **Local Cache**: Per-connection page cache
  2. **Shared Cache**: Multiple connections share cache (optional)
- **Eviction Policy**: LRU (Least Recently Used)
- **Pinning**: Active pages cannot be evicted
- **Configuration**: `PRAGMA cache_size = N` (N = pages)

### 7. B-tree Navigation

- **Cursor State**:
  - Current page number
  - Cell index within page
  - Key value (for indexes)
  - Validity flag
- **Navigation Operations**:
  - `MoveToFirst()`: Descend to leftmost leaf
  - `MoveToNext()`: In-order traversal
  - `MoveToLast()`: Rightmost leaf
  - `Seek()`: Binary search for key
- **Page Types**: Internal nodes vs. Leaf nodes

### 8. Locking Protocol (5 States)

```md
UNLOCKED (no access)
   ↓
SHARED (read access, multiple allowed)
   ↓
RESERVED (intent to write, reads still allowed)
   ↓
PENDING (waiting for readers to finish)
   ↓
EXCLUSIVE (writing, no other access)
```

- **Read Transaction**: UNLOCKED → SHARED
- **Write Transaction**: UNLOCKED → SHARED → RESERVED → PENDING → EXCLUSIVE
- **Deadlock Prevention**: Timeout on lock acquisition

### 9. Transaction Journaling

- **Rollback Journal Mode** (default):
  1. Before modifying page: Write original to journal
  2. Modify page in memory
  3. On commit: Delete journal
  4. On rollback: Restore pages from journal
- **WAL Mode** (Write-Ahead Log):
  1. Write changes to WAL file
  2. Original database unchanged
  3. Readers see old data until checkpoint
  4. Periodic checkpoint merges WAL to database
- **Advantages of WAL**:
  - Concurrent reads during write
  - Faster commits (append-only)
  - Better crash recovery

### 10. Opcode Execution Details

- **Opcode Structure**: `{opcode, p1, p2, p3, p4, p5, comment}`
- **Register System**: Stack-based with named registers
- **Cursor System**: Array of B-tree cursors
- **Jump Instructions**: Use p2 for target address
- **Error Handling**: Each opcode can abort execution

### 11. Result Row Lifecycle

1. **Assembly**: `OP_ResultRow` packages registers
2. **Pause**: VDBE yields to application (SQLITE_ROW)
3. **Access**: App calls `sqlite3_column_*()` APIs
4. **Type Conversion**: Automatic if needed
5. **Resume**: Next `sqlite3_step()` continues execution

### 12. String Encoding

- **Internal**: UTF-8 (default) or UTF-16
- **Conversion**: Automatic based on database encoding
- **Functions**: `sqlite3_column_text()` vs `sqlite3_column_text16()`

### 13. Prepared Statement Reuse

- **Benefits**: Avoid re-parsing, faster execution
- **Reset**: `sqlite3_reset()` clears bindings and state
- **Rebind**: `sqlite3_bind_*()` for new parameters
- **Finalize**: `sqlite3_finalize()` frees all resources

### 14. Error Propagation

```md
VDBE error → sqlite3VdbeExec() → sqlite3Step() → sqlite3_step()
                     ↓
              sqlite3_errmsg() provides description
```

- **Error Codes**: Extended codes provide detail (e.g., SQLITE_IOERR_READ)
- **Error Messages**: Stored in connection object

### 15. Interrupt Handling

- **Mechanism**: `sqlite3_interrupt()` sets flag
- **Check Points**: Between opcodes, during I/O
- **Result**: SQLITE_INTERRUPT error code
- **Cleanup**: Automatic rollback of active transaction

## Example: Detailed VDBE Bytecode

### SELECT Statement Bytecode

For `SELECT name FROM users WHERE id = 5`:

```assembly
Addr  Opcode         P1    P2    P3    P4             P5  Comment
----  -------------  ----  ----  ----  -------------  --  -----------------
0     Init           0     11    0                    0   Start here; jump to 11
1     OpenRead       0     2     0     3              0   Open cursor 0 on users (root page 2, 3 cols)
2     Rewind         0     10    0                    0   Go to addr 10 if table empty
3     Column         0     0     1                    0   r[1] = users.id (column 0)
4     Ne             5     9     1     BINARY         82  if r[1] != 5 goto 9
5     Column         0     1     2                    0   r[2] = users.name (column 1)
6     ResultRow      2     1     0                    0   Output r[2] (1 column)
7     Next           0     3     0                    1   Advance cursor, goto 3 if more rows
8     Goto           0     10    0                    0   Exit loop
9     Goto           0     7     0                    0   Continue to Next (filter failed)
10    Close          0     0     0                    0   Close cursor 0
11    Halt           0     0     0                    0   End of program
12    Transaction    0     0     1     0              1   Start read transaction on DB 0
13    Goto           0     1     0                    0   Jump to main logic
```

### Opcode Parameter Summary

| Parameter | Meaning | Common Uses |
|-----------|---------|-------------|
| **P1** | Integer operand | Cursor ID, database number, literal value |
| **P2** | Jump target / Integer | Bytecode address for jumps, column count |
| **P3** | Register number | Source/destination register |
| **P4** | Pointer / String | Table name, literal string, collation |
| **P5** | Flags | Opcode-specific flags |

### Key Opcodes Explained

#### Transaction Control

- `OP_Transaction`: Begin transaction, acquire locks
- `OP_Commit`: Commit changes
- `OP_Rollback`: Abort transaction

#### Cursor Operations

- `OP_OpenRead/OpenWrite`: Create B-tree cursor
- `OP_Close`: Release cursor resources
- `OP_Rewind`: Move to first entry
- `OP_Next/Prev`: Iterate through entries

#### Data Access

- `OP_Column`: Read column from current row
- `OP_Rowid`: Get current rowid
- `OP_MakeRecord`: Create database record
- `OP_Insert/Delete`: Modify table data

#### Control Flow

- `OP_Goto`: Unconditional jump
- `OP_If/IfNot`: Conditional jumps
- `OP_Eq/Ne/Lt/Le/Gt/Ge`: Comparison jumps
- `OP_Halt`: Terminate execution

#### Results

- `OP_ResultRow`: Return row to application
- `OP_Integer/String8/Real`: Load literal values
