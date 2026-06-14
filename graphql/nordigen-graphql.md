# GoCardless (Nordigen) GraphQL Schema

This is a conceptual GraphQL schema for the GoCardless Bank Account Data API (formerly Nordigen). It models the open banking REST API that enables access to EU bank account data under PSD2 regulation.

## Source API

- **Provider**: GoCardless (formerly Nordigen)
- **API Version**: v2
- **Base URL**: `https://bankaccountdata.gocardless.com/api/v2/`
- **Documentation**: https://developer.gocardless.com/bank-account-data/quick-start-guide
- **Coverage**: All European Economic Area (EEA) countries under PSD2

## Schema File

- [nordigen-schema.graphql](nordigen-schema.graphql)

## Overview

The GoCardless Bank Account Data API (Nordigen) provides access to bank account information through a standardized open banking interface. The workflow involves:

1. **Authentication** - Obtain token pair using API credentials
2. **Institution Discovery** - Find banks available in a given country
3. **End-User Agreement** - Create an agreement defining data access scope
4. **Requisition** - Build an authentication link for the end user
5. **Account Data** - Retrieve account details, balances, and transactions

## Types Summary

### Authentication Types

| Type | Description |
|------|-------------|
| `TokenPair` | Combined access and refresh tokens returned on initial authentication |
| `AccessToken` | Short-lived token for API requests |
| `RefreshToken` | Long-lived token used to obtain new access tokens |
| `APIKey` | Credential pair (secret_id + secret_key) for generating tokens |

### Institution Types

| Type | Description |
|------|-------------|
| `Institution` | A financial institution (bank) available in the system |
| `InstitutionDetails` | Extended metadata about an institution including supported features |
| `InstitutionCountry` | Country association for an institution |
| `BIC` | Bank Identifier Code with parsed components |
| `Logo` | Institution logo image metadata |

### Agreement Types

| Type | Description |
|------|-------------|
| `EndUserAgreement` | Agreement defining data access scope and duration |
| `AgreementDetails` | Extended agreement metadata |
| `AccessScopeDetails` | Description of a specific access scope permission |

### Requisition Types

| Type | Description |
|------|-------------|
| `Requisition` | Authentication requisition linking a user to bank accounts |
| `RequisitionDetails` | Extended requisition metadata |
| `Link` | Hypermedia link with rel and method |

### Account Types

| Type | Description |
|------|-------------|
| `Account` | Bank account linked through a requisition |
| `AccountDetails` | Extended account information (IBAN, owner, product, etc.) |

### Balance Types

| Type | Description |
|------|-------------|
| `Balance` | A single balance record for an account |
| `BalanceDetails` | Extended balance metadata |
| `AccountBalances` | Collection of balances for an account |
| `TransactionAmount` | Monetary amount with currency code |

### Transaction Types

| Type | Description |
|------|-------------|
| `Transaction` | A single bank transaction (booked or pending) |
| `TransactionDetails` | Extended transaction metadata |
| `TransactionDate` | Booking and value date pair |
| `AccountTransactions` | Collection of booked and pending transactions |
| `Creditor` | Creditor party in a transaction |
| `Debtor` | Debtor party in a transaction |
| `RemittanceInfo` | Remittance/reference information |
| `MandateDetails` | SEPA mandate details |
| `Merchant` | Merchant associated with a transaction |
| `MerchantDetails` | Extended merchant information |

### Reference Types

| Type | Description |
|------|-------------|
| `IBAN` | International Bank Account Number with parsed components |
| `Currency` | Currency with code and symbol |
| `CurrencyCode` | ISO 4217 currency code |
| `Country` | Country with code and institutions |
| `CountryCode` | ISO 3166-1 alpha-2 country code |
| `Color` | Brand colors for a provider or institution |
| `Metadata` | Generic key-value metadata pair |
| `Permission` | Access permission with grant status |
| `Scope` | API scope definition |

### Provider Types

| Type | Description |
|------|-------------|
| `Provider` | API provider (GoCardless) details |
| `ProviderDetails` | Extended provider metadata |

### User Types

| Type | Description |
|------|-------------|
| `User` | API user/organization |
| `UserDetails` | Extended user metadata |

### Utility Types

| Type | Description |
|------|-------------|
| `Refresh` | Account data refresh status |
| `RefreshStatus` | Status detail for a refresh operation |
| `PaymentDetails` | Payment initiation details |
| `RateLimitInfo` | API rate limit information |
| `ErrorResponse` | API error envelope |
| `ErrorDetail` | Individual error detail within an error response |

## Enumerations

| Enum | Values |
|------|--------|
| `AccessScope` | `BALANCES`, `DETAILS`, `TRANSACTIONS` |
| `AgreementStatus` | `CREATED`, `GIVEN`, `REVOKED`, `EXPIRED` |
| `RequisitionStatus` | `CR`, `ID`, `LN`, `RJ`, `ER`, `SU`, `EX`, `GA`, `UA`, `SA` |
| `AccountStatus` | `DISCOVERED`, `PROCESSING`, `READY`, `DELETED`, `SUSPENDED`, `ERROR`, `BLOCKED` |
| `AccountType` | ISO 20022 cash account type codes (CACC, SACC, SVGS, etc.) |
| `BalanceType` | `CLOSING_BOOKED`, `INTERIM_AVAILABLE`, `OPENING_BOOKED`, etc. |
| `TransactionStatus` | `BOOKED`, `PENDING`, `INFORMATION` |
| `PaymentStatus` | `ACCEPTED`, `REJECTED`, `PENDING`, `CANCELLED` |

## Example Queries

### Get Institutions for a Country

```graphql
query GetInstitutions {
  institutions(country: "GB") {
    id
    name
    bic
    countries
    logo
    maxAccessValidForDays
    transactionTotalDays
  }
}
```

### Create Authentication Flow

```graphql
mutation Authenticate {
  createToken(secretId: "your-secret-id", secretKey: "your-secret-key") {
    access
    accessExpires
    refresh
    refreshExpires
  }
}
```

### Get Account Transactions

```graphql
query GetTransactions($accountId: ID!) {
  accountTransactions(id: $accountId, dateFrom: "2024-01-01", dateTo: "2024-12-31") {
    booked {
      transactionId
      bookingDate
      transactionAmount {
        amount
        currency
      }
      creditorName
      debtorName
      remittanceInformationUnstructured
    }
    pending {
      transactionId
      valueDate
      transactionAmount {
        amount
        currency
      }
    }
  }
}
```

### Get Account Balances

```graphql
query GetBalances($accountId: ID!) {
  accountBalances(id: $accountId) {
    balances {
      balanceType
      balanceAmount {
        amount
        currency
      }
      referenceDate
      lastChangeDateTime
    }
  }
}
```
