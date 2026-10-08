| Concept | Entity / Value Object / Role / Enum | Why? |
| --- | --- | --- |
| User | entity  | because it has identity |
| Profile | value object   | because as business wise we interest with the use entity not profile , the user have profile if the user changes the profile will change |
| Wallet | Entity | Has identity and lifecycle |
| Money/Balance | value objects | these values related to the wallet entity and these are values will change mostly for the wallet of user  |
| Card | entity | card is entity because it has life cycle and identity and can be replaced to another card and may be not found  |
| Transaction | Entity | The object has **identity across state changes**. |
| Transaction Status | Enum | it take one value from some values in every state  |
| History | value object | **collection/view of transactions belonging to a User** |
| History Entry | entity | is a transaction |
| Notification | value object | because it generated from transactions  |
| Weekly Summary | value object  | generated from history  |

# Phase 0.1 — OOP Domain Implementation

## Goal

Implement the first version of the FinTech Transaction Platform using **plain Java and object-oriented programming**.

The objective of this phase is to translate the approved business/domain model into working objects and enforce the important business rules through the domain model.

Do **not** use Spring Boot, databases, REST APIs, Kafka, or external payment providers yet.

---

## 1. Implementation Scope

Implement the following domain concepts:

- User
- Profile
- Wallet
- Card
- TransactionParticipant
- Transaction
- ParticipantSnapshot
- TransactionEvent
- Notification

Enums:

- TransactionType
- TransferStatus
- TopUpStatus
- TransactionEventType

---

## 2. Package Structure

Start with a simple domain-oriented structure.

```
src/
└── main/
    └── java/
        └── ...
            └── domain/
                ├── user/
                ├── wallet/
                ├── card/
                ├── transaction/
                ├── notification/
                └── common/
```

The exact package names are your decision.

The important part is to organize the code according to domain concepts rather than putting every class into one package.

---

## 3. User

### Responsibilities

User represents the account and identity of a person using the platform.

### Data

- name
- username
- phoneNumber
- email
- hashedPassword
- autoAccept
- profile
- wallet
- current card

### Important Rules

- username is unique
- username cannot be changed
- phone number is unique
- one User has exactly one Wallet
- one User has exactly one Profile
- one User has at most one currently linked Card

### Avoid

Do not make User responsible for every application use case.

Avoid turning User into a "God Object".

---

## 4. Profile

### Data

- visibility
- name
- username
- personalImage

### Responsibilities

- edit profile information
- change visibility

### Relationship

```
User 1 ◆── 1 Profile
```

---

## 5. Wallet

### Data

- totalBalance
- heldAmount

### Derived Data

```
availableBalance = totalBalance - heldAmount
```

Do not maintain `availableBalance` as an independently mutable value unless you have a strong reason.

### Responsibilities

The Wallet must protect balance-related business rules.

It should eventually support operations such as:

- add money
- hold money
- release held money
- spend money

### Important Invariants

A Wallet must never allow:

```
heldAmount > totalBalance
```

and must never allow available balance to become negative.

---

## 6. Card

### Data

- cardId
- maskedCardNumber
- phoneNumber
- createdAt

### Rules

- a User can have at most one current Card
- Card is optional
- Card can be replaced
- Card can be removed
- replacing a Card does not change Wallet balance
- historical transactions must remain understandable after Card replacement

Never store real sensitive card data in the learning project.

---

## 7. TransactionParticipant

Create an abstraction representing something that can participate in a financial transaction.

```
TransactionParticipant
----------------------
+ getIdentifier()
```

Implementations:

```
Wallet
Card
```

Do not make User a TransactionParticipant.

Examples:

```
Transfer:
Wallet → Wallet

TopUp:
Card → Wallet
```

---

## 8. ParticipantSnapshot

A transaction stores historical participant information instead of depending on the current User, Wallet, or Card state.

### Data

- identifier
- displayName
- participantType
- snapshotTimestamp

### Rule

The snapshot represents the participant information relevant to the transaction at that point in time.

It should not contain sensitive information such as passwords or full card numbers.

---

## 9. Transaction

### Data

- id
- senderSnapshot
- receiverSnapshot
- amount
- timestamp
- type
- status

### Transaction Types

```
Transfer
TopUp
```

### Transfer State Machine

```
Pending
   ├── Completed
   ├── Rejected
   ├── Cancelled
   ├── Expired
   └── Failed
```

### Top-Up State Machine

```
Pending
   ├── Completed
   ├── Cancelled
   └── Failed
```

### Responsibilities

The Transaction should protect valid state transitions.

Examples:

```
Pending → Completed     valid
Pending → Cancelled     valid
Pending → Rejected      valid
Pending → Expired       valid
Pending → Failed        valid

Completed → Cancelled   invalid
Completed → Rejected    invalid
```

Do not allow arbitrary status mutation from outside the object.

---

## 10. TransactionEvent

### Data

- id
- stateChange
- timestamp

### Relationship

```
Transaction 1 ◆── 1..* TransactionEvent
```

A TransactionEvent represents something that happened during the transaction lifecycle.

Example:

```
Pending
Failed
Failed
Completed
```

The Transaction keeps its current state.

TransactionEvents provide the historical sequence of state changes.

---

## 11. Notification

### Data

- id
- recipient
- sender
- receiver
- amount
- status
- timestamp

### Rule

Every TransactionEvent produces exactly two Notifications.

Example:

```
TransactionEvent
       │
       ├── Notification → sender
       │
       └── Notification → receiver
```

Notification contains the information necessary to communicate what happened at that point in time.

---



