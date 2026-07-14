# Deep vs Shallow Module Examples

Concrete examples demonstrating the difference between deep and shallow module design.

## Example 1: File Storage

### Shallow Design (Avoid)

```typescript
interface FileStorage {
  // Caller must manage every detail
  openConnection(config: ConnectionConfig): Connection
  closeConnection(conn: Connection): void
  createBucket(conn: Connection, name: string, region: string): Bucket
  uploadChunk(conn: Connection, bucket: Bucket, key: string,
              chunk: Buffer, offset: number): void
  finalizeUpload(conn: Connection, bucket: Bucket, key: string): void
  setMetadata(conn: Connection, bucket: Bucket, key: string,
              metadata: Metadata): void
  downloadChunk(conn: Connection, bucket: Bucket, key: string,
                offset: number, length: number): Buffer
  listObjects(conn: Connection, bucket: Bucket,
              prefix: string, marker: string, limit: number): ObjectList
}
```

**Problems**:
- 9 methods, all require connection management
- Caller must understand chunking, offsets, pagination
- Error handling pushed to caller
- Implementation details (chunks, markers) leaked

### Deep Design (Target)

```typescript
interface FileStorage {
  // Simple interface hides all complexity
  save(key: string, content: Buffer | Stream): Promise<void>
  load(key: string): Promise<Buffer>
  delete(key: string): Promise<void>
  list(prefix?: string): Promise<string[]>
}
```

**Hidden complexity**:
- Connection pooling and management
- Automatic chunking for large files
- Retry logic with exponential backoff
- Pagination handled internally
- Metadata management
- Regional routing

**Depth ratio**: 4 methods hide ~500 lines of implementation

---

## Example 2: User Authentication

### Shallow Design (Avoid)

```typescript
interface AuthService {
  hashPassword(password: string, algorithm: string,
               salt: Buffer, iterations: number): Buffer
  verifyPasswordHash(password: string, hash: Buffer,
                     algorithm: string, salt: Buffer,
                     iterations: number): boolean
  generateToken(userId: string, claims: Claims,
                algorithm: string, secret: string,
                expiresIn: number): string
  verifyToken(token: string, secret: string,
              algorithms: string[]): TokenPayload | null
  createSession(userId: string, deviceId: string,
                ip: string, userAgent: string): Session
  validateSession(sessionId: string): Session | null
  refreshSession(sessionId: string, newExpiry: Date): void
  invalidateSession(sessionId: string): void
  invalidateAllSessions(userId: string): void
}
```

**Problems**:
- 9 methods with complex parameters
- Security-sensitive details exposed (algorithms, iterations)
- Caller must coordinate multiple operations
- Easy to misuse (wrong algorithm, weak iterations)

### Deep Design (Target)

```typescript
interface AuthService {
  // Core operations only
  register(email: string, password: string): Promise<User>
  login(email: string, password: string): Promise<AuthResult>
  logout(): Promise<void>
  validateRequest(request: Request): Promise<User | null>
}

interface AuthResult {
  user: User
  token: string  // Opaque to caller
}
```

**Hidden complexity**:
- Password hashing with secure defaults (bcrypt, proper rounds)
- Token generation with secure algorithms
- Session management and storage
- Token refresh handling
- Rate limiting and brute force protection
- Device tracking

**Depth ratio**: 4 methods hide ~800 lines of security-critical code

---

## Example 3: Database Query Builder

### Shallow Design (Avoid)

```typescript
interface QueryBuilder {
  select(columns: string[]): QueryBuilder
  from(table: string): QueryBuilder
  where(column: string, operator: string, value: any): QueryBuilder
  and(column: string, operator: string, value: any): QueryBuilder
  or(column: string, operator: string, value: any): QueryBuilder
  join(table: string, type: string, leftCol: string,
       rightCol: string): QueryBuilder
  groupBy(columns: string[]): QueryBuilder
  having(column: string, operator: string, value: any): QueryBuilder
  orderBy(column: string, direction: string): QueryBuilder
  limit(count: number): QueryBuilder
  offset(count: number): QueryBuilder
  build(): { sql: string, params: any[] }
  execute(connection: Connection): Promise<Row[]>
}

// Usage - caller must build everything
const query = db.select(['id', 'name', 'email'])
  .from('users')
  .where('status', '=', 'active')
  .and('created_at', '>', lastMonth)
  .join('orders', 'LEFT', 'users.id', 'orders.user_id')
  .groupBy(['users.id'])
  .having('COUNT(orders.id)', '>', 5)
  .orderBy('name', 'ASC')
  .limit(10)
  .offset(20)
  .build();
```

**Problems**:
- 13 chainable methods
- SQL concepts directly exposed
- No protection against SQL injection in raw usage
- Caller manages connection
- Complex for simple queries

### Deep Design (Target)

```typescript
interface UserRepository {
  // Domain-focused interface
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  findActive(options?: { limit?: number, offset?: number }): Promise<User[]>
  findWithOrders(minOrderCount: number): Promise<UserWithOrders[]>
  save(user: User): Promise<User>
  delete(id: string): Promise<void>
}
```

**Hidden complexity**:
- Query building and optimization
- Connection management
- SQL injection prevention
- Result mapping to domain objects
- Caching strategies
- Query batching (N+1 prevention)

**Alternative: Balanced depth with query interface**

```typescript
interface Repository<T> {
  find(criteria: Criteria<T>): Promise<T[]>
  findOne(criteria: Criteria<T>): Promise<T | null>
  save(entity: T): Promise<T>
  delete(id: string): Promise<void>
}

// Criteria is still simpler than raw query builder
const activeUsers = await users.find({
  where: { status: 'active' },
  orderBy: { name: 'asc' },
  limit: 10
});
```

---

## Example 4: HTTP Client

### Shallow Design (Avoid)

```typescript
interface HttpClient {
  createRequest(method: string, url: string): Request
  setHeader(request: Request, name: string, value: string): Request
  setBody(request: Request, body: any, encoding: string): Request
  setTimeout(request: Request, timeout: number): Request
  setRetryPolicy(request: Request, retries: number,
                 backoff: BackoffStrategy): Request
  send(request: Request): Promise<RawResponse>
  parseResponse(response: RawResponse, format: string): any
  handleRedirect(response: RawResponse): Request | null
}

// Usage
const req = client.createRequest('POST', url);
client.setHeader(req, 'Content-Type', 'application/json');
client.setHeader(req, 'Authorization', `Bearer ${token}`);
client.setBody(req, data, 'utf-8');
client.setTimeout(req, 5000);
client.setRetryPolicy(req, 3, exponentialBackoff);
const rawResp = await client.send(req);
const redirect = client.handleRedirect(rawResp);
if (redirect) { /* handle */ }
const result = client.parseResponse(rawResp, 'json');
```

### Deep Design (Target)

```typescript
interface HttpClient {
  get<T>(url: string, options?: RequestOptions): Promise<T>
  post<T>(url: string, data: any, options?: RequestOptions): Promise<T>
  put<T>(url: string, data: any, options?: RequestOptions): Promise<T>
  delete<T>(url: string, options?: RequestOptions): Promise<T>
}

interface RequestOptions {
  headers?: Record<string, string>
  timeout?: number  // Has sensible default
  // Other options optional with defaults
}

// Usage
const user = await client.post<User>('/users', userData);
```

**Hidden complexity**:
- Request building and encoding
- Response parsing based on content-type
- Redirect handling (configurable limits)
- Retry with exponential backoff
- Connection pooling
- Timeout management
- Error normalization

---

## Key Takeaways

### Shallow Module Symptoms
- Many methods for related operations
- Implementation concepts in interface (chunks, offsets, algorithms)
- Caller must coordinate multiple calls
- Configuration required for basic usage
- Easy to misuse

### Deep Module Characteristics
- Few methods covering common use cases
- Domain concepts in interface (users, files, not rows, chunks)
- Single call for complete operations
- Sensible defaults, optional configuration
- Hard to misuse

### Depth Metrics

| Aspect | Shallow | Deep |
|--------|---------|------|
| Methods | 10+ | 3-5 |
| Parameters per method | 5+ | 0-2 |
| Required config | Most | None |
| Implementation leak | High | None |
| Lines hidden per method | 10-20 | 100+ |
| Time to learn | Hours | Minutes |
