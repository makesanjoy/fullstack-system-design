# Full-Stack System Design

Practical, engineering-focused guides to designing reliable web applications with **React, Node.js, HTTP APIs, and databases**.

The material follows realistic application flows and explains not only how components communicate, but also what happens when requests fail, data changes concurrently, or services scale beyond one server. Examples are intentionally small; the design principles apply to larger systems.

## Start here

| Guide | What you'll learn |
| --- | --- |
| [HTTP Request Lifecycle](week-01-foundations/day-01-request-lifecycle.md) | Trace an Add to Cart request from the browser to the database and back; understand HTTP methods, status codes, and safe retries. |
| [Client–Server Architecture](week-01-foundations/day-02-client-server-architecture.md) | Define client/server responsibilities, build a trust boundary, share sessions across servers, and prevent duplicate writes. |

## Architecture in scope

```text
Browser (React)
    │ HTTPS / JSON
    ▼
Load balancer / API entry point
    │
    ▼
Node.js + Express API instances
    ├── Shared infrastructure (for example, Redis sessions)
    └── Persistent data store (for example, PostgreSQL or MongoDB)
```

The guides use **PostgreSQL** in the request-lifecycle example and **MongoDB** in the multi-server example. These are alternative illustrative data stores, not a requirement to deploy both for the same cart.

## Engineering principles

- **Trust the server, not the browser.** Authenticate users, authorize operations, and validate business rules on the API.
- **Define API behavior precisely.** Request shapes, response codes, and retry semantics are part of the contract.
- **Treat failures as normal.** A timeout does not tell the client whether a database write committed.
- **Design for concurrency.** Multiple API instances may receive the same operation simultaneously.
- **Choose storage guarantees deliberately.** Caches and short-lived locks are not substitutes for durable data integrity.

## Repository layout

```text
fullstack-system-design/
├── README.md
└── week-01-foundations/
    ├── day-01-request-lifecycle.md
    └── day-02-client-server-architecture.md
```

Each guide can be read independently and includes an architecture flow, practical examples, failure scenarios, and engineering trade-offs. The numbered filenames provide a reading order; the articles themselves are general-purpose technical references.
