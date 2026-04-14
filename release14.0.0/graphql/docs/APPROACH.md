# BIAN v14 → GraphQL: Design Approach and Rationale

## Goal

Demonstrate that the BIAN Release 14.0.0 Semantic APIs can be translated into a
GraphQL Schema with **no meaningful loss of semantic or structural information**.

GraphQL's type system, rich description support, and composable SDL make it a
strong candidate for representing BIAN's structured service model in a way that:

- Preserves the BIAN domain model and terminology
- Enables strongly-typed, self-documenting APIs
- Reduces over-fetching common in REST (clients ask for exactly what they need)
- Supports aggregation of data from multiple Service Domains in a single request
- Integrates naturally with modern frontend frameworks

---

## BIAN Structural Concepts and Their GraphQL Equivalents

### Service Domain → Type Family

Each BIAN **Service Domain** is a bounded context managing a specific banking
capability. In GraphQL it maps to a **family of related types**:

| BIAN Concept | GraphQL Equivalent |
|---|---|
| Service Domain | A module/file; a family of types |
| Control Record (CR) | Root object type (e.g. `CurrentAccountFacility`) |
| Behaviour Qualifier (BQ) | Nested object type referenced from the CR |
| `Retrieve` operation | Query field |
| `Initiate`, `Update`, `Execute`, `Exchange`, `Evaluate`, `Control`, `Request`, `Grant` | Mutation fields |
| `typevalues` schema | GraphQL `enum` type |
| Shared base schemas | Shared `common.graphql` object types |
| OpenAPI format hints | GraphQL custom `scalar` types |

### BIAN Operations → GraphQL Operations

BIAN defines a standard vocabulary of operation types for each Service Domain.
Each maps to a GraphQL operation as follows:

| BIAN Operation | HTTP Method | GraphQL Root Type | Rationale |
|---|---|---|---|
| `Retrieve` | GET | `Query` | Read-only, idempotent |
| `Initiate` (InCR / InBQ) | POST | `Mutation` | Creates a new entity |
| `Update` (UpCR / UpBQ) | PUT | `Mutation` | Modifies an existing entity |
| `Execute` (ExBQ) | PUT | `Mutation` | Executes a function (side-effecting) |
| `Exchange` (EcCR / EcBQ) | PUT | `Mutation` | State transition (accept/reject) |
| `Evaluate` (EvCR) | POST | `Mutation` | Creates via rule evaluation |
| `Control` (CoCR) | PUT | `Mutation` | Lifecycle control (suspend/resume) |
| `Request` (RqBQ / RqCR) | PUT | `Mutation` | Request an action |
| `Grant` (GrCR) | PUT | `Mutation` | Grant authority |

Each mutation field includes a doc comment referencing the exact BIAN endpoint path.

---

## Naming Conventions

### Types

- **Control Records** use the BIAN entity name verbatim in PascalCase
  (e.g. `CurrentAccountFacility`, `CustomerAgreement`)
- **Behavior Qualifier types** are prefixed with the Service Domain name to avoid
  collisions (e.g. `CurrentAccountPaymentTransaction`, `SessionBrowsing`)
- **Shared base types** use BIAN names where unique; where BIAN names conflict with
  GraphQL keywords or other types, a disambiguating prefix is added
  (e.g. `BianDateTime` for the BIAN `Datetime` structured type, to avoid clashing
  with the `DateTime` scalar)

### Fields

- Field names use **camelCase**, derived directly from the BIAN property name
  (e.g. BIAN `ProductInstanceReference` → `productInstanceReference`)
- No semantic renaming is applied; the original BIAN terminology is preserved

### Enums

- Enum type names use the **full BIAN `typevalues` schema name** with the first
  letter capitalised (e.g. `Datetimetypevalues`, `Accounttypevalues`)
- This avoids all naming conflicts with object types, since no BIAN object type
  name ends in `typevalues`
- Enum member values are taken verbatim from BIAN, with any characters invalid
  in GraphQL identifiers (e.g. `/`, `-`) replaced by underscore

### Scalars

| Scalar | Maps to | Description |
|---|---|---|
| `DateTime` | OpenAPI `format: date-time` | ISO 8601 combined date-time |
| `Date` | OpenAPI `format: date` | ISO 8601 date only |
| `CurrencyCode` | `format: CurrencyCode` / `Currencycode` schema | ISO 4217 three-letter code |
| `Decimal` | `format: Amount` | Arbitrary-precision decimal as string |
| `URL` | `format: Object` (URL context) | URL string |
| `JSON` | `format: Object` (unstructured) | Arbitrary JSON payload |

---

## Information Fidelity

### What is preserved

- All BIAN Control Record properties
- All Behavior Qualifier properties
- All `typevalues` enumerations, including every member value
- BIAN field descriptions (carried in GraphQL type and field descriptions)
- BIAN operation semantics (operation type mapping documented per field)
- Structural composition: CR → BQ hierarchy is explicit in the schema
- BIAN references (each file documents its source YAML)

### Simplifications made

| Simplification | Rationale |
|---|---|
| Optional vs required: most fields are optional (`String` not `String!`) | BIAN OpenAPI specs rarely mark fields `required`; optionality is preserved conservatively |
| `JSON` scalar for open-ended objects | Some BIAN properties have `format: Object` with no sub-schema; `JSON` preserves the data without imposing false structure |
| Shared types defined once in `common.graphql` | BIAN duplicates many schemas across Service Domain YAMLs; GraphQL's type system benefits from a single canonical definition |
| Some convenience queries added | `<entity>ByCustomer` queries not present in BIAN REST; added as GraphQL pattern (BIAN allows this via search/filter operations) |

### What is not yet represented

| Gap | Notes |
|---|---|
| Subscriptions | BIAN async/event APIs (AsyncAPI 3.x) could map to GraphQL Subscriptions; out of scope for this POC |
| Pagination | BIAN list operations support pagination; GraphQL Relay-style cursor pagination could be added via connection types |
| Error detail types | BIAN HTTP error schemas (400, 401, 403, 404, 429, 500) could map to a GraphQL union error type pattern |
| Directives | `@deprecated`, `@auth`, `@constraint` directives could be added to enrich production use |
| Federation | Large BIAN deployments could use GraphQL Federation to compose Service Domains from separate subgraphs |

---

## Why GraphQL for BIAN?

### Strengths of the mapping

1. **BIAN's composable structure aligns with GraphQL's type system.**
   The CR + BQ hierarchy maps naturally to parent/child object types with nested
   field resolution.

2. **Single request for cross-domain data.**
   A customer dashboard needing data from CurrentAccount, SavingsAccount, and
   CustomerPosition can be served in one GraphQL query rather than three REST calls.

3. **Type safety and discoverability.**
   GraphQL's introspection allows tooling (IDEs, API explorers) to surface the full
   BIAN domain model without separately maintaining documentation.

4. **Description propagation.**
   GraphQL's description strings carry BIAN field and type descriptions directly
   into the schema, making it self-documenting.

5. **Enum precision.**
   BIAN's `typevalues` enumerations map exactly to GraphQL enums, preventing invalid
   values at the schema level.

### Challenges observed

1. **Name clashes between BIAN types and BIAN enums.**
   BIAN names `Accounttypevalues`, `Actiontypevalues`, `Arrangementtypevalues`, etc.
   produce enum names that clash with corresponding object type names if the
   `typevalues` suffix is stripped. Solved by retaining the full BIAN name.

2. **BIAN schemas are duplicated across Service Domains.**
   The same `Amount`, `Identifier`, `Datetime` schemas appear in every YAML.
   GraphQL requires a single canonical definition; `common.graphql` resolves this.

3. **Open-ended `format: Object` fields.**
   Some BIAN properties have no sub-schema. Represented as `JSON` scalar,
   which preserves the data but removes type safety for those fields.

4. **Large number of types.**
   BIAN v14 has 259 Service Domains with thousands of schemas. The full graph
   would be large but manageable with code generation from the OAS YAML files.

---

## Production Readiness Path

To evolve this POC toward production:

1. **Add authentication directives** — Use `@auth` or custom directives to enforce
   BIAN's access control model
2. **Add pagination** — Implement Relay cursor-based connections for list fields
3. **Add error types** — Map BIAN HTTP error schemas to GraphQL union error types
4. **Code-generate from BIAN YAML** — Automate schema generation from the 259
   OAS YAML files to cover the full BIAN surface
5. **Add resolvers** — Wire the schema to backend BIAN service implementations
6. **Add subscriptions** — Map BIAN AsyncAPI 3.x events to GraphQL Subscriptions
7. **Evaluate Federation** — Split into per-Service-Domain subgraphs for scale
