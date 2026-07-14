# Module Design Patterns

Patterns for creating deep modules that hide complexity and provide simple interfaces.

## Pattern 1: Facade Module

**Problem**: Multiple related operations require coordinating several lower-level components.

**Solution**: Create a facade module that orchestrates the components behind a simple interface.

**Structure**:
```
┌─────────────────────────────────────┐
│           Facade Module             │
│  ┌─────────────────────────────┐    │
│  │     Simple Interface        │    │ ← Callers interact here
│  │  - doOperation()            │    │
│  │  - configure(options)       │    │
│  └─────────────────────────────┘    │
│              │                      │
│    ┌─────────┼─────────┐            │
│    ▼         ▼         ▼            │
│  ┌───┐    ┌───┐    ┌───┐            │ ← Hidden complexity
│  │ A │    │ B │    │ C │            │
│  └───┘    └───┘    └───┘            │
└─────────────────────────────────────┘
```

**Example**: File upload service
- Interface: `upload(file)`, `download(id)`
- Hidden: Chunking, encryption, cloud storage API, retry logic, caching

**Depth achieved by**: Hiding coordination complexity, error handling, and multi-component orchestration.

## Pattern 2: Configuration Absorber

**Problem**: Operations have many possible configurations, leading to complex interfaces.

**Solution**: Absorb configuration internally with sensible defaults; expose minimal options.

**Before (shallow)**:
```
processDocument(doc, encoding, compression, format,
                errorHandling, timeout, retries,
                cacheStrategy, logLevel)
```

**After (deep)**:
```
processDocument(doc)
processDocument(doc, options)  // optional overrides
```

**Implementation**:
- Define sensible defaults for 90% of use cases
- Allow optional override object for edge cases
- Validate and normalize options internally
- Handle conflicts and incompatibilities internally

**Depth achieved by**: Making common cases trivial while still supporting advanced use cases.

## Pattern 3: Error Absorber

**Problem**: Many edge cases and error conditions complicate the interface.

**Solution**: Handle errors internally with defined behavior; eliminate unnecessary exceptions.

**Techniques**:

1. **Idempotent operations**: Make it safe to retry or repeat
   - Delete non-existent item → succeed silently
   - Create existing item → return existing or update

2. **Bounded return values**: Always return valid data
   - Substring beyond length → return empty string
   - Index out of bounds → return default value

3. **Self-healing operations**: Recover automatically
   - Connection lost → reconnect transparently
   - Cache miss → populate and continue

**Depth achieved by**: Eliminating error handling burden from callers.

## Pattern 4: State Machine Encapsulator

**Problem**: Complex state transitions must be managed correctly.

**Solution**: Encapsulate all state management within the module; expose only actions and queries.

**Structure**:
```
┌────────────────────────────────────┐
│        State Machine Module        │
│  ┌──────────────────────────────┐  │
│  │  Interface (actions/queries) │  │
│  │  - start()                   │  │
│  │  - pause()                   │  │
│  │  - isRunning()               │  │
│  └──────────────────────────────┘  │
│           │                        │
│           ▼                        │
│  ┌──────────────────────────────┐  │
│  │    Hidden State Machine      │  │
│  │  IDLE ──► STARTING ──► RUN   │  │
│  │   ▲           │         │    │  │
│  │   └───────────┴─────────┘    │  │
│  └──────────────────────────────┘  │
└────────────────────────────────────┘
```

**Callers don't need to know**:
- Valid state transitions
- Transition side effects
- Intermediate states
- Recovery from invalid states

**Depth achieved by**: Hiding state complexity and ensuring correct transitions.

## Pattern 5: Protocol Translator

**Problem**: External systems use different protocols or data formats.

**Solution**: Create a translation layer that presents a consistent internal interface.

**Structure**:
```
┌─────────────────────────────────────┐
│       Protocol Translator           │
│  ┌─────────────────────────────┐    │
│  │   Internal Interface        │    │ ← Consistent API
│  │  - getData(): InternalModel │    │
│  │  - sendData(InternalModel)  │    │
│  └─────────────────────────────┘    │
│              │                      │
│    ┌─────────┴─────────┐            │
│    ▼                   ▼            │
│  ┌─────────┐      ┌─────────┐       │
│  │ Format A│      │ Format B│       │ ← Hidden translation
│  │ (JSON)  │      │ (XML)   │       │
│  └─────────┘      └─────────┘       │
└─────────────────────────────────────┘
```

**Depth achieved by**: Hiding protocol complexity and format variations.

## Pattern 6: Resource Manager

**Problem**: Resources (connections, handles, memory) require careful lifecycle management.

**Solution**: Manage resource lifecycle internally; callers get/use resources without managing them.

**Techniques**:
- Pooling: Maintain reusable resource pool
- Lazy initialization: Create resources on demand
- Automatic cleanup: Release resources automatically
- Reference counting: Track usage, cleanup when zero

**Interface**:
```
// Caller doesn't manage connection lifecycle
result = database.query(sql)

// Not this (shallow)
connection = database.getConnection()
try:
    result = connection.execute(sql)
finally:
    connection.release()
```

**Depth achieved by**: Hiding resource management complexity.

## Pattern 7: Batch Coordinator

**Problem**: Operations on multiple items require complex coordination.

**Solution**: Accept collections, handle batching/parallelization internally.

**Interface**:
```
// Deep: handles batching, parallelization, error aggregation
results = processor.processAll(items)

// Shallow: caller must handle iteration and errors
results = []
for item in items:
    try:
        results.append(processor.process(item))
    except Error as e:
        handle_error(e)
```

**Hidden complexity**:
- Optimal batch sizing
- Parallel execution
- Error aggregation and partial failure
- Progress tracking
- Rate limiting

**Depth achieved by**: Hiding collection processing complexity.

## Combining Patterns

Real modules often combine multiple patterns:

**Example: Document Service**
- **Facade**: Simple `save(doc)`, `load(id)` interface
- **Configuration Absorber**: Optional formatting/compression options
- **Error Absorber**: Auto-retry on network failures
- **Protocol Translator**: Works with multiple storage backends
- **Resource Manager**: Connection pooling to storage
- **Batch Coordinator**: `saveAll(docs)` with optimal batching

## Anti-Patterns to Avoid

### Pass-Through Module
Module adds no value, just forwards to another:
```
class UserService:
    def getUser(id):
        return userRepository.getUser(id)  # No added value
```
**Fix**: Either add value (caching, validation, transformation) or eliminate the layer.

### Leaky Abstraction
Internal details leak through the interface:
```
class Cache:
    def get(key, shard_hint):  # Why does caller need to know about shards?
        ...
```
**Fix**: Handle sharding internally, remove from interface.

### Configuration Explosion
Too many parameters exposed:
```
def send(msg, retry, timeout, backoff, maxRetries,
         encoding, compression, priority, ttl, ...)
```
**Fix**: Use configuration objects with defaults, or absorb configuration internally.

### Incomplete Abstraction
Module handles some cases but not others:
```
class FileHandler:
    def read(path):  # Works
    def write(path, content):  # Works
    # But caller must handle directories, permissions, encoding...
```
**Fix**: Complete the abstraction or clearly document what's excluded.
