# Phase 1 — Domain & Business Requirements

## Project Overview

**Project:** Event-Driven FinTech Transaction Platform

**Goal:**

Build a production-oriented digital wallet and financial transaction platform.

---

# 1. Account & Identity

## User Registration

- A user can create an account.
- `Username` must be unique.
- `Phone Number` must be unique.
- An OTP is sent to the user's phone number during registration.
- The account is not created until the OTP is successfully verified.

## Profile

- User can view their own profile.
- User can edit their profile.
- User can view other users' public profiles.
- Some profile information may have visibility rules.

## Username

- Username is unique.
- Username can be changed.
- Changing a username does not affect existing transactions.
- Transactions are associated with the user, not with the username value itself.

---

# 2. Wallet

- Every user has one wallet.
- Wallet starts with a balance of `0`.
- A user can receive money without having a linked card.
- A user's wallet has:
    - `Total Balance`
    - `Held Amount`
    - `Available Balance`

### Balance Rule

`Available Balance = Total Balance - Held Amount`

### Example

```
Total Balance = 500
Held Amount   = 300
Available     = 200
```

Money held by pending transfers cannot be spent.

---

# 3. Card & Top-up

## Card

- A card is optional.
- A user can add a card after creating their account.
- A user can have only one linked card at a time.
- The user can replace their existing card.
- Card replacement does not affect wallet balance or previous transactions.
- A card is used as a funding source for the wallet.
- A card is not required to receive money.

## Top-up

**Top-up = Card → Wallet**

- A successful top-up increases the wallet balance.
- Failed card payment operations are currently recorded with status `Failed`.

## Recovery Requirement

If:

1. Payment provider reports `Success`
2. System crashes before wallet update is completed

The system must be able to determine whether the operation's wallet update was already applied.

Each operation has a unique ID and tracks its progress/state.

The system must prevent the same successful operation from adding money to the wallet more than once.


---

# 4. Send Money

A user can send money to another user on the same application.

## Receiver

- Receiver is identified using their username.
- Username must exist before creating the transaction.
- If the username does not exist:
    - User is informed that the username does not exist.
    - No financial transaction is created.

## Balance Validation

A transfer amount must not exceed the sender's `Available Balance`.

If:

```
Transfer Amount > Available Balance
```

Then:

- Transfer is rejected.
- User receives an insufficient balance message.
- No financial transaction is created.

---

# 5. Transfer Approval

Each user has an `Auto-Accept` setting for incoming transfers.

## Auto-Accept Enabled

Incoming transfer is completed immediately.

```
Sender → Transfer → Completed
```

No Pending Approval state is required.

## Auto-Accept Disabled

Incoming transfer requires receiver approval.

```
Sender → Transfer → Pending
                     ↓
             Accept / Reject
```

The receiver has 20 minutes to respond.

---

# 6. Pending Transfer

When a transfer becomes `Pending`:

- Sender's money is held.
- Held money cannot be spent.
- Sender's total balance does not decrease yet.
- Receiver does not receive the money yet.
- Sender can still spend their remaining Available Balance.
- Receiver sees the transaction in `Pending Approval`.
- Sender sees the transaction as `Pending`.

### Example

```
Balance   = 500
Held      = 300
Available = 200
```

The sender can still spend up to `200`.

---

# 7. Transfer State Transitions

## Pending → Completed

Triggered when the receiver accepts.

### Sender

```
Balance = Balance - Amount
Held    = Held - Amount
```

### Receiver

```
Balance = Balance + Amount
```

The transaction moves to History for both users.

---

## Pending → Rejected

Triggered when the receiver rejects.

- Sender's held amount is released.
- Amount becomes available again.
- Transaction is removed from Pending.
- Transaction does not appear in the user's final History.

---

## Pending → Cancelled

Triggered when the sender cancels.

- Sender's held amount is released.
- Amount becomes available again.
- Transaction is removed from Pending for both users.
- Transaction does not appear in the user's final History.

---

## Pending → Expired

Triggered when the 10-minutes approval window expires without a response.

- Sender's held amount is released.
- Amount becomes available again.
- Transaction is removed from Pending for both users.
- Transaction does not appear in the user's final History.
- Receiver receives a notification that the approval window expired.

---

# 8. Transaction Finality

Once a transfer becomes `Completed`:

- Sender cannot cancel it.
- Receiver cannot reject it.
- The transaction is final.

---

# 9. Transaction History

## User-facing History

History currently contains completed financial transactions.

### Pending

- Pending transfers are temporarily visible as Pending.
- They are not considered final History entries.

### Rejected / Cancelled / Expired

- They are removed from the user-facing Pending/History views.
- They do not appear in the final user-facing History.

### Completed

- Completed transactions are added to History.

> Note: Internal retention of rejected, cancelled, and expired transaction records is not finalized yet. This is separate from what the user sees in History.
> 

---

# 10. Notifications

Every transfer operation generates notifications for **both parties**.

| Event | Sender | Receiver |
| --- | --- | --- |
| Pending | Notification | Notification |
| Completed | Notification | Notification |
| Rejected | Notification | Notification |
| Cancelled | Notification | Notification |
| Expired | Notification | Notification |

Notifications are associated with the corresponding operation/transaction event.

---

# 11. Weekly Summary

The Weekly Summary currently includes:

## Money Added

Total amount of successful Top-ups during the week.

## Money Spent

Total amount of completed outgoing transfers during the week.

Spending is aggregated by receiver.

### Example

```
Weekly Summary

Money Added
1,000

Money Spent
Bob       → 350
Carol     → 150
----------------
Total     → 500
```

Only `Completed` transfers are included in spending.

Rejected, Cancelled, and Expired transfers are not included.

> Note: The Weekly Summary requirements may evolve in future versions.
> 

---

# 12. Account Closure

A user can close their account only if:

```
Balance = 0
AND
No Pending Operations
```

The account cannot be closed while:

- The user has money remaining.
- The user has pending operations.

---

# 13. Current Business Rules

| # | Rule |
| --- | --- |
| 1 | Username must be unique |
| 2 | Phone number must be unique |
| 3 | OTP verification is required before account creation |
| 4 | Wallet starts with zero balance |
| 5 | Card is optional |
| 6 | User can have only one linked card |
| 7 | Card can be replaced |
| 8 | Card is not required to receive money |
| 9 | Receiver username must exist |
| 10 | Invalid transfer attempts do not create financial transactions |
| 11 | User can only spend from Available Balance |
| 12 | Pending transfers hold money |
| 13 | Auto-Accept causes immediate completion |
| 14 | Manual approval creates a Pending transfer |
| 15 | Receiver has one hour to respond |
| 16 | Accept → Completed |
| 17 | Reject → money released |
| 18 | Sender Cancel → money released |
| 19 | Timeout → money released |
| 20 | Completed transfers are final |
| 21 | Completed transfers enter History |
| 22 | Notifications are sent to both parties |
| 23 | Weekly spending includes Completed transfers only |
| 24 | Weekly spending is aggregated by receiver |
| 25 | Weekly Summary includes successful money added |
| 26 | Account closure requires zero balance |
| 27 | Account closure requires no pending operations |

---

# 14. Open Questions

These have **not been finalized yet**.

- OTP expiration and retry rules.
- Maximum OTP attempts.
- Minimum/maximum transfer amount.
- Whether a user can send money to themselves.
- What happens if a receiver closes their account while a transfer is Pending.
- Maximum number of Pending transfers.
- Card replacement while a Top-up is in progress.
- Internal retention policy for Rejected / Cancelled / Expired transactions.
- Notification behavior if notification delivery fails.
- Exact account closure behavior after the account passes the business conditions.

These should be decided only when they become relevant to the next phase.

---
