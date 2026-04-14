# BIAN v14 Semantic APIs → GraphQL Schema
## Proof of Concept – Customer Workbench (Online Banking)

---

### Overview

This artefact demonstrates that the [BIAN Release 14.0.0](https://bian.org/) Semantic APIs
can be meaningfully represented as a GraphQL Schema with little or no loss of semantic
or structural information.

The scope is deliberately narrow — covering **eight BIAN Service Domains** that together
represent the core functionality of a customer-facing online banking portal (Customer
Workbench).  The design is production-like and intended to scale up to a broader BIAN
subset if the concept proves successful.

---

### Service Domain Scope

| Service Domain | BIAN Control Record | Behaviour Qualifiers |
|---|---|---|
| **CustomerWorkbench** | `CustomerWorkbenchOperatingSession` | Browsing, Contact, Broadcast, SWUpdate, ProductAndServiceAccess |
| **CurrentAccount** | `CurrentAccountFacility` | PaymentTransaction, Deposit, Withdrawal, DirectDebitMandate, IssuedDevice, AccountBalance, Sweep, ServiceModality |
| **SavingsAccount** | `SavingsAccountFacility` | PaymentTransaction, Deposit, Withdrawal, DirectDebitMandate, AccountBalance, Interest, Sweep, ServiceModality |
| **PaymentOrderInitiation** | `PaymentOrderInitiationTransaction` | Compliance, Confirmation, OrderInitiation |
| **CustomerAgreement** | `CustomerAgreement` | LegalTerms, RegulatoryTerms, PolicyTerms, ProductAgreement |
| **CustomerAccessEntitlement** | `CustomerAccessProfileAgreement` | AccessArrangement, Restrictions, Preferences |
| **CustomerEventHistory** | `CustomerEventLog` | Relationship, Sales, Servicing, ProductProcessing, Case, Fraud, Life |
| **CustomerPosition** | `CustomerPositionState` | Cashflow, CreditPosition, DelinquentAccount, ReportingArrangement |
| **CustomerProductAndServiceDirectory** | `CustomerProductAndServiceDirectoryEntry` | Product, Service |
| **SalesProductAgreement** | `SalesProductAgreement` | LegalTerm, RegulatoryTerm, PolicyTerm |
| **ServicingOrder** | `ServicingOrderProcedure` | *(single-procedure, no BQs)* |
| **CustomerProductAndServiceEligibility** | `CustomerEligibilityAssessment` | EligibilityCheck, NextBest |

---

### Schema Structure

```
release14.0.0/graphql/
├── README.md                          ← This file
├── schema/
│   ├── schema.graphql                 ← Root assembly + schema declaration
│   ├── scalars.graphql                ← Custom scalar types
│   ├── enums.graphql                  ← 146 enums from BIAN typevalues schemas
│   ├── common.graphql                 ← Shared structural types
│   ├── service-domains/
│   │   ├── CustomerWorkbench.graphql
│   │   ├── CurrentAccount.graphql
│   │   ├── SavingsAccount.graphql
│   │   ├── PaymentOrderInitiation.graphql
│   │   ├── CustomerAgreement.graphql
│   │   ├── CustomerAccessEntitlement.graphql
│   │   ├── CustomerEventHistory.graphql
│   │   └── CustomerPosition.graphql
│   └── operations/
│       ├── Query.graphql              ← 40 query operations
│       └── Mutation.graphql           ← 39 mutation operations
└── docs/
    ├── APPROACH.md                    ← Design rationale and transformation rules
    └── MAPPING.md                     ← Detailed BIAN → GraphQL type mapping
```

**Schema statistics:**
- **327** named GraphQL types defined
- **54** query fields (read operations)
- **62** mutation fields (write operations)
- **150** enum types (all BIAN `typevalues` schemas in scope)
- **6** custom scalar types

---

### Quick Start – Validating the Schema

The schema can be validated using [graphql-js](https://github.com/graphql/graphql-js):

```bash
# Install graphql-js
npm install graphql

# Validate (example Node script)
node -e "
const { buildSchema } = require('graphql');
const fs = require('fs');
const path = require('path');

const dir = './schema';
const files = [
  'scalars.graphql', 'enums.graphql', 'common.graphql',
  'service-domains/CustomerWorkbench.graphql',
  'service-domains/CurrentAccount.graphql',
  'service-domains/SavingsAccount.graphql',
  'service-domains/PaymentOrderInitiation.graphql',
  'service-domains/CustomerAgreement.graphql',
  'service-domains/CustomerAccessEntitlement.graphql',
  'service-domains/CustomerEventHistory.graphql',
  'service-domains/CustomerPosition.graphql',
  'operations/Query.graphql',
  'operations/Mutation.graphql',
];

const sdl = files.map(f => fs.readFileSync(path.join(dir, f), 'utf8')).join('\n');
buildSchema(sdl);
console.log('Schema is valid');
"
```

The schema can also be loaded by any GraphQL tool that supports SDL files,
including [Apollo Server](https://www.apollographql.com/docs/apollo-server/),
[GraphQL Yoga](https://the-guild.dev/graphql/yoga-server),
[Pothos](https://pothos-graphql.dev/), or schema exploration tools such as
[GraphQL Inspector](https://the-guild.dev/graphql/inspector).

---

### Example Queries

#### Retrieve a customer's product and service holdings

```graphql
query GetCustomerProducts($customerId: String!) {
  customerProductAndServiceDirectoryByCustomer(customerReference: $customerId) {
    customerProductAndServiceDirectoryId
    products {
      productId
      product {
        productName { name }
        productType
        productLifecycleStatus { productStatus }
      }
      productAgreementReference {
        agreementStatus
        agreementValidityPeriod { fromDateTime toDateTime }
      }
    }
    services {
      serviceId
      service {
        serviceName { name }
        serviceType
        serviceLifecycleStatus { reason }
      }
    }
  }
}
```

#### Retrieve sales product agreements for a customer

```graphql
query GetSalesAgreements($customerId: String!) {
  salesProductAgreementsByCustomer(customerReference: $customerId) {
    salesProductAgreementId
    agreementType
    agreementValidFromToDate { fromDateTime toDateTime }
    bankingProductReference {
      productName { name }
      productType
    }
    legalTerms {
      legalTermId
      jurisdiction { jurisdictionName }
    }
  }
}
```

#### List servicing orders for a customer

```graphql
query GetServicingOrders($customerId: String!) {
  servicingOrdersByCustomer(customerReference: $customerId) {
    servicingOrderId
    servicingOrderType
    servicingOrderDescription
    date
    servicingOrderWorkTaskResult {
      taskType
      taskStatus
      taskDateTime
    }
  }
}
```

#### Initiate a servicing order (e.g. change of address)

```graphql
mutation RaiseServicingOrder {
  initiateServicingOrder(input: {
    customerReference: "CUST-001"
    servicingOrderType: "ChangeOfAddress"
    servicingOrderDescription: "Customer requesting postal address update"
  }) {
    servicingOrderId
    servicingOrderType
    date
  }
}
```

#### Retrieve a customer's current account

```graphql
query GetCurrentAccount($accountId: ID!) {
  currentAccount(currentAccountId: $accountId) {
    currentAccountId
    currentAccountNumber {
      accountIdentificationValue
      accountIdentificationType
    }
    customerReference {
      partyReference
    }
    accountType
    accountCurrency {
      currencyCode
      currencyType
    }
    accountBalance {
      balanceAmount {
        amountValue
        currencyCode
      }
      balanceType
      balanceValueDate
    }
    accountStatus
  }
}
```

#### Initiate a payment

```graphql
mutation InitiatePayment {
  initiatePaymentOrder(input: {
    customerReference: "CUST-001"
    paymentType: Domesticpayment
    amount: "250.00"
    currency: "GBP"
    payerAccountReference: "ACC-CA-001"
    payeeAccountReference: "ACC-CA-002"
    paymentPurpose: "Rent payment"
  }) {
    paymentOrderInitiationId
    amount {
      amountValue
      currencyCode
    }
    paymentMechanism
    payerReference {
      payerReference {
        partyReference
      }
    }
  }
}
```

#### Retrieve a customer's consolidated position

```graphql
query GetCustomerPosition($positionId: ID!) {
  customerPosition(customerPositionId: $positionId) {
    customerPositionId
    customerReference {
      partyReference
    }
    cashflows {
      cashflowId
      reportPeriod {
        fromDateTime
        toDateTime
      }
      customerAccountBalance {
        balanceAmount {
          amountValue
          currencyCode
        }
        balanceType
      }
    }
    creditPositions {
      totalAvailableCredit {
        amountValue
        currencyCode
      }
      totalOutstandingDebt {
        amountValue
        currencyCode
      }
      creditUtilisationPercentage
    }
  }
}
```

---

### Design Principles

See [`docs/APPROACH.md`](docs/APPROACH.md) for full design rationale.

Key principles applied:

1. **Semantic fidelity** — BIAN terminology is preserved in type names, field names,
   and descriptions. No BIAN concepts are collapsed or renamed arbitrarily.

2. **BIAN structure → GraphQL structure** — Each Service Domain becomes a family of
   GraphQL types. The Control Record maps to the root object type; each Behaviour
   Qualifier maps to a nested object type.

3. **Operations preserved** — BIAN operations (Initiate, Retrieve, Update, Exchange,
   Execute, Evaluate, Control, Request, Grant) map explicitly to GraphQL queries and
   mutations, with comments recording the BIAN source endpoint.

4. **Enums from typevalues** — Every BIAN `typevalues` schema in scope is represented
   as a GraphQL enum, preserving all member values.

5. **Scalars for formats** — OpenAPI `format` hints (date-time, date, currency, decimal)
   are represented as custom GraphQL scalars rather than plain strings.

---

### Relationship to BIAN OpenAPI Specifications

Each GraphQL Service Domain file maps directly to a BIAN v14 OAS YAML:

| GraphQL file | BIAN OAS source |
|---|---|
| `CustomerWorkbench.graphql` | `CustomerWorkbench.yaml` |
| `CurrentAccount.graphql` | `CurrentAccount.yaml` |
| `SavingsAccount.graphql` | `SavingsAccount.yaml` |
| `PaymentOrderInitiation.graphql` | `PaymentOrderInitiation.yaml` |
| `CustomerAgreement.graphql` | `CustomerAgreement.yaml` |
| `CustomerAccessEntitlement.graphql` | `CustomerAccessEntitlement.yaml` |
| `CustomerEventHistory.graphql` | `CustomerEventHistory.yaml` |
| `CustomerPosition.graphql` | `CustomerPosition.yaml` |
| `CustomerProductAndServiceDirectory.graphql` | `CustomerProductandServiceDirectory.yaml` |
| `SalesProductAgreement.graphql` | `SalesProductAgreement.yaml` |
| `ServicingOrder.graphql` | `ServicingOrder.yaml` |
| `CustomerProductAndServiceEligibility.graphql` | `CustomerProductAndServiceEligibility.yaml` |

Source YAML files: `release14.0.0/semantic-apis/oas3 /yamls/`

---

### Extending the Schema

To add more BIAN Service Domains:

1. Create a new file under `schema/service-domains/<ServiceDomain>.graphql`
2. Define the Control Record type and all Behavior Qualifier types
3. Add corresponding query fields to `operations/Query.graphql`
4. Add corresponding mutation fields to `operations/Mutation.graphql`
5. Add any new enum types from that Service Domain's YAML to `enums.graphql`
6. Validate with `graphql-js` or your preferred schema validation tool

The `common.graphql` shared types cover the majority of BIAN base types and
should not need modification for most new Service Domains.

---

### References

- [BIAN Website](https://bian.org/)
- [BIAN Portal](https://portal.bian.org/)
- [BIAN v14 OAS Specifications (this repository)](../semantic-apis/oas3%20/yamls/)
- [GraphQL Specification](https://spec.graphql.org/)
- [GraphQL SDL Reference](https://graphql.org/learn/schema/)
