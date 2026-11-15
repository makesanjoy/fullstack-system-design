# Week 1 · Day 2 — Client–Server Architecture

**Stack:** React → Load Balancer → Node.js/Express → Redis + MongoDB

**Status:** Completed

## 1. Client vs Server

| React (Client) | Node.js (Server) |
| --- | --- |
| Render UI and handle clicks | Validate requests and apply business rules |
| Send HTTP requests | Authenticate and authorize users |
| Show loading, errors, and retries | Perform database operations |
| Manage UI state | Return trusted API responses |

**Rule:** Never trust client-supplied prices, identity, or permissions. Fetch authoritative data server-side.

## 2. Add to Cart Request Flow

1. React sends POST /api/cart with product ID, quantity, and an idempotency key.
2. Load balancer routes the request to a Node.js instance.
3. Node.js authenticates the user's session using shared Redis storage.
4. Backend validates the request and business rules.
5. MongoDB atomically records the operation and updates the cart.
6. Node.js returns JSON and an appropriate HTTP status.
7. React updates its UI using the server response.

## 3. Optimistic UI and Rollback

React can immediately display an updated cart count before server confirmation.

- Success: confirm the server's updated cart state.
- Failure: roll back the optimistic change and show an error.
- Timeout: the update may have succeeded. Retry the same action with the same idempotency key.

## 4. Stateful vs Stateless Servers

A stateful server may depend on user sessions stored in its own memory.

A stateless API server can handle requests independently of which instance receives them, using request context and shared services.

**Example:** Store sessions in Redis so Server A and Server B can authenticate the same user.

## 5. Redis Atomic Operations

`SET idempotency:user123:abc123 processing NX EX 300`

- SET: Create a key.
- NX: Create only when absent.
- EX 300: Expire after 300 seconds.

Redis executes this as one atomic command. Even if multiple servers attempt it concurrently, only one succeeds while the key exists.

**Limitation:** Redis reservations alone do not guarantee exactly-once database updates.

## 6. Durable Database Idempotency

For reliable cart updates:

1. Scope the idempotency key to the user and operation.
2. Reject the same key when reused with different request data.
3. Use a unique idempotency record in MongoDB.
4. Commit the idempotency record and cart modification in the same database transaction.
5. Return the stored result when an already-completed operation is retried.

A MongoDB replica set or sharded deployment is required for multi-document transactions.

**Important distinction:** Retrying one action uses the same key. Two genuinely separate Add to Cart actions need separate keys.

## 7. Interview Takeaways

- Frontend validation improves UX; backend validation enforces correctness.
- Shared sessions support horizontally scaled API servers.
- Redis SET NX prevents concurrent reservation of the same key.
- Database transactions protect against partial updates and retries.
- Network failure does not prove database failure.
- Idempotency and optimistic UI rollback solve different problems.

## Day 2 Mastery

- Concept questions: 3/3 passed
- Practical Add to Cart architecture: passed with follow-up guidance
- Final mastery checks: 2/2 passed
- Improvement focus: Redis atomicity vs durable database idempotency

**Next:** Day 3 requires explicit approval.
