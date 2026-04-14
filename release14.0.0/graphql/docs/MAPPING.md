# BIAN v14 → GraphQL: Type Mapping Reference

This document provides a detailed mapping between BIAN v14 OpenAPI schemas
and their GraphQL equivalents in this proof-of-concept schema.

---

## Custom Scalars

| GraphQL Scalar | BIAN/OAS Format | Example Value |
|---|---|---|
| `DateTime` | `format: date-time` | `"2024-03-15T10:30:00Z"` |
| `Date` | `format: date` | `"2024-03-15"` |
| `CurrencyCode` | `Currencycode` schema | `"GBP"` |
| `Decimal` | `format: Amount` | `"1250.00"` |
| `URL` | `format: Object` (URL context) | `"https://bank.example.com"` |
| `JSON` | `format: Object` (unstructured) | `{"key": "value"}` |

---

## Common Types (shared across Service Domains)

| GraphQL Type | BIAN Source Schema | Notes |
|---|---|---|
| `Identifier` | `Identifier` | Identifier value + issuer + start/end dates |
| `BianDateTime` | `Datetime` | Structured date-time with timezone and type |
| `DateTimePeriod` | `Datetimeperiod` | from/to datetime range |
| `Amount` | `Amount` | Decimal value + currency code + amount type |
| `AccountBalance` | `Accountbalance` / `AccountBalance` | Balance amount, type, date, indicator |
| `AccountCurrency` | `Accountcurrency` | Currency code + currency type |
| `InvolvedParty` | `Involvedparty` | Party reference + involvement reference |
| `Status` | `Status` | Reason + datetime + validity + party |
| `ActivityLog` | `Log` | Log type, period, date, identification |
| `Name` | `Name` | Name string wrapper |
| `Address` | `Address` | Address type + description |
| `Location` | `Location` | Full location details |
| `Action` | `Action` | Action type, event details |
| `Arrangement` | `Arrangement` | Arrangement action, dates, status, type |
| `ArrangementstatusType` | `Arrangementstatus` | Status value + type |
| `Account` | `Account` | Account identification + datetime + type + status |
| `AccountIdentification` | `Accountidentification` | Account ID value + type (IBAN, BBAN, etc.) |
| `AccountDateTime` | `Accountdatetime` | Account datetime + purpose type |
| `AccountstatusType` | `Accountstatus` | Status value + datetime + validity + reason |
| `Party` | `Party` | Party identification, legal structure, type, address |
| `PartyIdentification` | `Partyidentification` | Identifier + identification type |
| `ContactPoint` | `Contactpoint` | Contact value + type |
| `Organisation` | `Organisation` | ID, name, legal structure, industry code, address |
| `DocumentReference` | `DocumentDirectoryEntry` / `Document` | Document ID, name, type, date, content |
| `Product` | `Product` | Product ID, type, status, datetime |
| `ProductAgreement` | `ProductAgreement` | Agreement ID, type, dates, status |
| `FeeArrangement` | `FeeArrangement` / `Feearrangement` | Fee type, plan, calculation basis/frequency, amount |
| `Schedule` | `Schedule` | Description, start/end dates, frequency |
| `Branch` | `Branch` | Branch ID, type, address |
| `RatePlan` | `Rateplan` | Rate plan type, interest rate type and value |
| `Jurisdiction` | `Jurisdiction` | Jurisdiction name, legal entity type, start date |
| `Mandate` | `Mandate` | Mandate ID, type, status, duration, tracking |
| `PaymentIdentification` | `Paymentidentification` | Payment ID + identification type |
| `PaymentDateTime` | `Paymentdatetime` | Payment datetime + datetime type |
| `PaymentTransactionStatus` | `Paymenttransactionstatus` | Payment transaction status value |
| `PaymentAmountAndCurrency` | `Paymentamountandcurrency` | Payment amount + type |
| `FinancialFacility` | `FinancialFacility` | Facility reference + type |
| `LimitArrangement` | `LimitArrangement` / `Limitarrangement` | Limit type, status, start/end dates |
| `PartyObligationOrEntitlement` | `PartyObligationOrEntitlement` | Party, involvement type, description |
| `AccountRestriction` | `AccountRestriction` | Status, reason, application period |
| `AccountInvolvement` | `AccountInvolvement` | Party, involvement type, obligation/entitlement |
| `Device` | `Device` | Device ID, type, OS |
| `Feature` | `Feature` | Feature type, description |

---

## Service Domain Type Mappings

### CustomerWorkbench

| GraphQL Type | BIAN Schema | Description |
|---|---|---|
| `CustomerWorkbenchSession` | `CustomerWorkbenchOperatingSession` | CR - Active device/portal session |
| `SessionBrowsing` | `Browsing` | BQ - Web browsing interaction |
| `SessionContact` | `Contact` | BQ - Customer contact interaction |
| `SessionBroadcast` | `Broadcast` | BQ - Bank-to-customer broadcast |
| `SessionSoftwareUpdate` | `SWUpdate` | BQ - Device software update |
| `SessionProductAndServiceAccess` | `ProductandServiceAccess` | BQ - Product/service access |

### CurrentAccount

| GraphQL Type | BIAN Schema | Description |
|---|---|---|
| `CurrentAccountFacility` | `CurrentAccountFacility` | CR - Current account facility |
| `CurrentAccountPaymentTransaction` | `PaymentTransaction` | BQ - Payment transaction |
| `CurrentAccountDeposit` | `Deposit` | BQ - Deposit transaction |
| `CurrentAccountWithdrawal` | `Withdrawal` | BQ - Withdrawal transaction |
| `CurrentAccountDirectDebitMandate` | `DirectDebitMandate` | BQ - Direct debit mandate |
| `CurrentAccountIssuedDevice` | `IssuedDevice` | BQ - Card/device issued |
| `CurrentAccountBalanceRecord` | `AccountBalance` | BQ - Balance snapshot |
| `CurrentAccountSweep` | `Sweep` | BQ - Sweep arrangement |
| `CurrentAccountServiceModality` | `ServiceModality` | BQ - Service mode config |

### SavingsAccount

| GraphQL Type | BIAN Schema | Description |
|---|---|---|
| `SavingsAccountFacility` | `SavingsAccountFacility` | CR - Savings account facility |
| `SavingsAccountPaymentTransaction` | `PaymentTransaction` | BQ - Payment transaction |
| `SavingsAccountDeposit` | `Deposit` | BQ - Deposit |
| `SavingsAccountWithdrawal` | `Withdrawal` | BQ - Withdrawal |
| `SavingsAccountDirectDebitMandate` | `DirectDebitMandate` | BQ - Direct debit mandate |
| `SavingsAccountBalanceRecord` | `AccountBalance` | BQ - Balance snapshot |
| `SavingsAccountInterest` | `Interest` | BQ - Interest accrual |
| `SavingsAccountSweep` | `Sweep` | BQ - Sweep arrangement |
| `SavingsAccountServiceModality` | `ServiceModality` | BQ - Service mode config |

### PaymentOrderInitiation

| GraphQL Type | BIAN Schema | Description |
|---|---|---|
| `PaymentOrderInitiation` | `PaymentOrderInitiationTransaction` | CR - Payment order record |
| `Payer` | `Payer` | Payer party details |
| `Payee` | `Payee` | Payee party details |
| `PaymentTransactionDetail` | `PaymentTransaction` (in POI) | Full payment transaction detail |
| `PaymentArrangementRef` | `PaymentArrangement` (reference) | Recurring payment arrangement ref |
| `PaymentComplianceCheck` | `Compliance` | BQ - Compliance check |
| `PaymentFundingCheck` | `FundingCheck` | Funding availability check |
| `PaymentOrderConfirmation` | `Confirmation` | BQ - Payment confirmation |
| `PaymentOrderInitiationRecord` | `OrderInitiation` | BQ - Submitted payment order |

### CustomerAgreement

| GraphQL Type | BIAN Schema | Description |
|---|---|---|
| `CustomerAgreement` | `CustomerAgreement` | CR - Master customer agreement |
| `PartyRelationshipLifecyclePhase` | `PartyRelationshipLifecyclePhase` | Lifecycle phase |
| `CustomerLegalTerms` | `LegalTerms` | BQ - Legal terms |
| `CustomerRegulatoryTerms` | `RegulatoryTerms` | BQ - Regulatory terms |
| `CustomerPolicyTerms` | `PolicyTerms` | BQ - Internal policy terms |
| `CustomerProductAgreement` | `ProductAgreement` (BQ of CustomerAgreement) | BQ - Product agreement |

### CustomerAccessEntitlement

| GraphQL Type | BIAN Schema | Description |
|---|---|---|
| `CustomerAccessProfile` | `CustomerAccessProfileAgreement` | CR - Access profile |
| `CustomerAccessAgreementRecord` | `CustomerAccessAgreement` | Access agreement details |
| `AccessArrangementInvolvementRecord` | `AccessArrangementInvolvement` | Access agreement signatory |
| `CustomerAccessArrangement` | `AccessArrangement` | BQ - Channel/device access arrangement |
| `AccessRestriction` | `Restrictions` / `AccessRestrictionArrangement` | BQ - Access restriction |
| `AccessPreference` | `Preferences` / `AccessPreferenceArrangement` | BQ - Access preference |
| `ChannelUsageRecord` | `ChannelUsage` | Channel/device usage history |

### CustomerEventHistory

| GraphQL Type | BIAN Schema | Description |
|---|---|---|
| `CustomerEventLog` | `CustomerEventLog` | CR - Master event log |
| `CustomerHistoryEvent` | `Event` | Base event record |
| `CustomerRelationshipEvent` | `Relationship` | BQ - Relationship event |
| `CustomerSalesEvent` | `Sales` | BQ - Sales/lead event |
| `CustomerServicingEvent` | `Servicing` | BQ - Servicing event |
| `CustomerProductProcessingEvent` | `ProductProcessing` | BQ - Product event |
| `CustomerCaseEvent` | `Case` | BQ - Case/complaint event |
| `CustomerFraudEvent` | `Fraud` | BQ - Fraud event |
| `CustomerLifeEvent` | `Life` | BQ - Life event |

### CustomerPosition

| GraphQL Type | BIAN Schema | Description |
|---|---|---|
| `CustomerPositionState` | `CustomerPositionState` | CR - Position state |
| `CustomerPositionSummary` | `CustomerPosition` | Position type + event reference |
| `CustomerPositionReport` | `Report` | Position report payload |
| `CustomerCashflow` | `Cashflow` | BQ - Cash flow analysis |
| `CustomerCreditPosition` | *(derived from Cashflow BQ credit fields)* | BQ - Credit position |
| `CustomerDelinquentAccount` | *(derived from CustomerPosition pattern)* | BQ - Delinquent account |
| `CustomerReportingArrangement` | `ReportingArrangement` | BQ - Reporting arrangement |

---

## Enum Naming Convention

BIAN OAS defines enums via `typevalues` schemas. The mapping to GraphQL enum names is:

```
BIAN schema name (lowercase)     →  GraphQL enum name (first letter capitalised)
-------------------------------------------------------------------
Datetimetypevalues               →  Datetimetypevalues
Accounttypevalues                →  Accounttypevalues
Paymenttypevalues                →  Paymenttypevalues
Currentaccounttypevalues         →  Currentaccounttypevalues
Directdebitstatustypevalues      →  Directdebitstatustypevalues
```

The full `typevalues` suffix is retained to avoid naming collisions with
GraphQL object types that share a similar root name (e.g. `Account` object type
vs `Accounttypevalues` enum type).

**Enum member normalisation:** Characters invalid in GraphQL identifiers
(forward slash `/`, hyphen `-`, space ` `) are replaced with underscore `_`.

Example:
```graphql
enum Deposittypevalues {
  CashDeposit
  SecurityDeposit
  PawnDeposit
  Call_NoticeDeposit    # was "Call/NoticeDeposit" in BIAN
  FixedTermDeposit
  DemandDeposit
  TimeDeposit
  CardDeposit
}
```

---

## Operation Mapping

### Query Operations (40 fields)

| GraphQL Query | BIAN Operation | BIAN Path Pattern |
|---|---|---|
| `customerWorkbenchSession` | ReCR | `GET /CustomerWorkbench/{id}/Retrieve` |
| `customerWorkbenchBrowsing` | ReBQ | `GET /CustomerWorkbench/{id}/Browsing/{bqId}/Retrieve` |
| `customerWorkbenchContact` | ReBQ | `GET /CustomerWorkbench/{id}/Contact/{bqId}/Retrieve` |
| `customerWorkbenchSoftwareUpdate` | ReBQ | `GET /CustomerWorkbench/{id}/SWUpdate/{bqId}/Retrieve` |
| `customerWorkbenchProductAndServiceAccess` | ReBQ | `GET /CustomerWorkbench/{id}/ProductandServiceAccess/{bqId}/Retrieve` |
| `currentAccount` | ReCR | `GET /CurrentAccount/{id}/Retrieve` |
| `currentAccountPaymentTransaction` | ReBQ | `GET /CurrentAccount/{id}/PaymentTransaction/{bqId}/Retrieve` |
| `currentAccountBalance` | ReBQ | `GET /CurrentAccount/{id}/AccountBalance/{bqId}/Retrieve` |
| `currentAccountDeposit` | ReBQ | `GET /CurrentAccount/{id}/Deposit/{bqId}/Retrieve` |
| `currentAccountWithdrawal` | ReBQ | `GET /CurrentAccount/{id}/Withdrawal/{bqId}/Retrieve` |
| `currentAccountDirectDebitMandate` | ReBQ | `GET /CurrentAccount/{id}/DirectDebitMandate/{bqId}/Retrieve` |
| `currentAccountSweep` | ReBQ | `GET /CurrentAccount/{id}/Sweep/{bqId}/Retrieve` |
| `currentAccountIssuedDevice` | ReBQ | `GET /CurrentAccount/{id}/IssuedDevice/{bqId}/Retrieve` |
| `savingsAccount` | ReCR | `GET /SavingsAccount/{id}/Retrieve` |
| `savingsAccountDeposit` | ReBQ | `GET /SavingsAccount/{id}/Deposit/{bqId}/Retrieve` |
| `savingsAccountBalance` | ReBQ | `GET /SavingsAccount/{id}/AccountBalance/{bqId}/Retrieve` |
| `savingsAccountInterest` | ReBQ | `GET /SavingsAccount/{id}/Interest/{bqId}/Retrieve` |
| `paymentOrderInitiation` | ReCR | `GET /PaymentOrderInitiation/{id}/Retrieve` |
| `paymentOrderComplianceCheck` | ReBQ | `GET /PaymentOrderInitiation/{id}/Compliance/{bqId}/Retrieve` |
| `paymentOrderConfirmation` | ReBQ | `GET /PaymentOrderInitiation/{id}/Confirmation/{bqId}/Retrieve` |
| `paymentOrderInitiationRecord` | ReBQ | `GET /PaymentOrderInitiation/{id}/OrderInitiation/{bqId}/Retrieve` |
| `customerAgreement` | ReCR | `GET /CustomerAgreement/{id}/Retrieve` |
| `customerAgreementLegalTerms` | ReBQ | `GET /CustomerAgreement/{id}/LegalTerms/{bqId}/Retrieve` |
| `customerAgreementRegulatoryTerms` | ReBQ | `GET /CustomerAgreement/{id}/RegulatoryTerms/{bqId}/Retrieve` |
| `customerAgreementPolicyTerms` | ReBQ | `GET /CustomerAgreement/{id}/PolicyTerms/{bqId}/Retrieve` |
| `customerAccessProfile` | ReCR | `GET /CustomerAccessEntitlement/{id}/Retrieve` |
| `customerAccessArrangement` | ReBQ | `GET /CustomerAccessEntitlement/{id}/AccessArrangement/{bqId}/Retrieve` |
| `customerEventLog` | ReCR | `GET /CustomerEventHistory/{id}/Retrieve` |
| `customerRelationshipEvent` | ReBQ | `GET /CustomerEventHistory/{id}/Relationship/{bqId}/Retrieve` |
| `customerProductProcessingEvent` | ReBQ | `GET /CustomerEventHistory/{id}/ProductProcessing/{bqId}/Retrieve` |
| `customerPosition` | ReCR | `GET /CustomerPosition/{id}/Retrieve` |
| `customerCashflow` | ReBQ | `GET /CustomerPosition/{id}/Cashflow/{bqId}/Retrieve` |
| `customerReportingArrangement` | ReBQ | `GET /CustomerPosition/{id}/ReportingArrangement/{bqId}/Retrieve` |
| Plus convenience queries | — | `ByCustomer` variants for common lookups |

### Mutation Operations (39 fields)

| BIAN Operation | Count | Examples |
|---|---|---|
| `Initiate` (InCR/InBQ) | 17 | `initiateCurrentAccount`, `initiatePaymentOrder`, `initiateCustomerWorkbenchBrowsing` |
| `Update` (UpCR/UpBQ) | 8 | `updateCurrentAccount`, `updatePaymentOrder`, `updateCustomerAgreement` |
| `Execute` (ExBQ) | 3 | `executeCustomerWorkbenchBrowsing`, `executeCustomerWorkbenchContact`, `executeCustomerWorkbenchProductAndServiceAccess` |
| `Exchange` (EcCR/EcBQ) | 3 | `exchangeCustomerAgreement`, `exchangePaymentOrderInitiationRecord` |
| `Evaluate` (EvCR) | 1 | `evaluateCustomerAgreement` |
| `Control` (CoCR) | 3 | `controlCurrentAccount`, `controlCustomerAgreement`, `controlCustomerWorkbenchSession` |
| `Request` (RqBQ/RqCR) | 4 | `requestCustomerAgreementCheck`, `requestCustomerWorkbenchContact`, `requestCustomerPositionReport` |
| `Grant` (GrCR) | 1 | `grantCustomerAgreementAuthority` |
