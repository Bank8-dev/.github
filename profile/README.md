# Bank8 - Finance Vertical

**Secure Post-Quantum-Safe Core Banking, ISO 20022 Payments Platform That Heals and Scales Itself.**

Bank8 is the finance vertical of the Sentin8 platform - a complete retail banking stack: core ledger, payment orchestration (domestic + cross-border), open banking composed from the OpenFin8 vertical, prudential risk measurement, and AML/CTF compliance. All built as six independent Rust services composed on top of the Sentin8 platform for identity, events, workflows, scheduling, and storage.

## Services

| Service | Domain | What it does | Tables | Endpoints |
|---|---|---|---|---|
| **Bank8:Ledger** | Core Banking | Double-entry ledger with 13 configurable product types, accounts, products, postings, balances (3-D), statements, checksum-chained audit trail | 44 domain tables + outbox + audit journal | 91 endpoints |
| **Bank8:Payments** | Payment Orchestration | ISO 20022 scheme flows (B8 on-us, SEPA, TIPS, SWIFT AGI), card issuing/lifecycle/3DS, mandates, payees, routing rules, recurring payments, sandbox simulators | 55 domain tables + outbox + audit journal | 90 endpoints |
| **OpenFin8** (composed vertical) | Open Finance | The open-finance vertical ([OpenFin8](https://openfin8.dev/) - org: [OpenFin8](https://github.com/OpenFin8)): consent ledger, recipient registry, CSDF minimization, per-jurisdiction adapters (CA/US serving FDX v6.x; UK designed). bank8 composes the provider role by enabling `openfin8:ca` in its environment; OpenFin8 consumes Ledger/Payments via public APIs + the `bank8::` SDK contracts only (the OBVERT invariant) | Owned by the OpenFin8 vertical | see the [OpenFin8](https://github.com/OpenFin8) org |
| **Bank8:Risk** | Prudential Risk | Exposure measurement from posting/payment events, Basel capital, LCR/NSFR liquidity, IFRS 9 ECL staging, limit monitoring with breach events, stress scenarios | Event-driven (consumes ledger/payments events) | Embedded worker |
| **Bank8:Reporting** | Regulatory Reporting | Jurisdiction registry, filing calendar, return generation (XBRL/CSV), submission/amendment lifecycle, AML/CTF filing executor (STR/SAR/CTR/EFTR/LCTR) | Event-driven (consumes risk/compliance events) | Embedded worker |
| **Bank8:Compliance** | AML/CTF | KYC/CDD, beneficial ownership, PEP/sanctions screening, transaction monitoring, alert/case lifecycle - detection only, filing via Reporting | Event-driven (consumes ledger/payments events) | Embedded worker |

## Flutter Apps

| Repository | Description |
|---|---|
| `bank8-ebank` | Retail customer banking app |
| `bank8-eteller` | Branch teller terminal |
| `bank8-admin` | Vertical admin console (RFW) |

The recipient app moved with the open-finance vertical: the former
`bank8-efintech` is now **eOpenCA**
([`OpenFin8/openfin8-eopenca`](https://github.com/OpenFin8/openfin8-eopenca)).

## SDKs & Libraries

| Path | Purpose |
|---|---|
| `sdk/rust/bank8` | Unified Rust SDK facade - typed clients + contracts (`bank8::ledger`, `bank8::payments`, etc.) |
| `crates/ledger/contracts` | Wire DTOs for ledger domain |
| `crates/payments/contracts` | Wire DTOs for payments domain |
| `openfin8-core` ([OpenFin8](https://github.com/OpenFin8/openfin8-core)) | The open-finance kernel library - traits + shared implementation (the OpenFin8 vertical) |
| `crates/fdx/contracts` | Wire DTOs for the FDX (open-banking) domain |
| `crates/regulatory/contracts` | Wire DTOs for regulatory reporting domain |

## Platform Dependencies

Bank8 depends on these Sentin8 platform services for cross-cutting concerns:

| Platform Service | What Bank8 gets from it | Dependency direction |
|---|---|---|
| **Sentin8:Auth** | Identity, sign-in, token issuance (user + M2M), JWKS, RBAC authorization | bank8 → Auth (verification + M2M identity) |
| **Sentin8:BFF** | Proxy, delegation gateway, client session custody, token attach | bank8 ← BFF (user tokens proxied to services) |
| **Sentin8:Bus** | Durable event transport, producer outbox relay, consumer DLQ, SSE push | bank8 → Bus (produce + consume domain events) |
| **Sentin8:Flow** | Multi-step workflow orchestration with retries and wait states | bank8 → Flow (payment instruction workflows) |
| **Sentin8:Cron** | Recurring job scheduling (statement runs, schedule-driven postings) | bank8 → Cron (schedule registration) |
| **Sentin8:Func** | Tenant-authored WASM function execution | bank8 ← Func (proxy-back handlers for hooks) |
| **Sentin8:Storage** | Object/file storage (statement documents, filing packages) | bank8 → Storage (document PUT/GET) |
| **Sentin8:Stat** | Business metrics aggregation and telemetry | bank8 → Stat (event ingestion) |
| **Sentin8:Sys** | Organization/environment lifecycle, service enablement | bank8 → Sys (env-init events) |

## Key Invariants

- **Double-entry atomicity** - posting batches that don't balance per denomination are rejected before any row is written
- **One money-movement executor** - all financial movement realizes through Bank8:Ledger via the public API; no service writes another's balances
- **Idempotent by construction** - every payment/posting is idempotent; retries can never double-post
- **Terminal states are final** - completed/cancelled/revoked states never regress
- **No consent, no data** - external data access exists only under explicit, revocable customer consent with field-level minimization

---

**Architecture & Design:** bank8 and its underlying concepts were designed by [Hamid Jahanzad](https://www.linkedin.com/in/hamidjahanzad/).
