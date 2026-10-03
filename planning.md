# Receipt Splitting App

A collaborative expense-splitting application inspired by Splitwise, focused on making restaurant and group expense splitting faster through **receipt scanning, item claiming, and automatic calculations**.

## Core Idea

Instead of manually entering how much each person owes:

1. One person uploads or takes a photo of the receipt.
2. Gemini analyzes the receipt and extracts the items and totals.
3. The receipt owner verifies the extracted information.
4. A room is created for the expense.
5. Friends join using a QR code or invite link.
6. Everyone claims the items they purchased or shared.
7. The application automatically calculates each person's share.
8. The application determines who owes whom.

---

# Core Features

## 1. User Accounts

Users should be able to:

- Create an account.
- Sign in with Google.
- Have a display name and profile.
- View previous expenses.
- View groups they belong to.
- View money they owe.
- View money owed to them.

### Guest Users

People should not need an account to participate in a single expense.

A guest can:

- Scan a QR code or open an invite link.
- Enter their name.
- Join the expense.
- Claim items.
- View their final amount.

Guests can optionally create an account afterward to save their expense history.

---

# 2. Create an Expense

A user can create a new expense by:

- Taking a picture of a receipt.
- Uploading an existing receipt image.
- Manually creating an expense.

The creator becomes the host of the expense.

---

# 3. Gemini Receipt Scanner

Gemini Vision will analyze receipt images.

Gemini should extract:

- Restaurant/store name.
- Item names.
- Item quantities.
- Individual item prices.
- Subtotal.
- Tax.
- Tip, if present.
- Discounts, if present.
- Final total.

Example:

```json
{
  "merchant": "Restaurant",
  "items": [
    {
      "name": "Burger",
      "quantity": 1,
      "unit_price": 18.00,
      "total": 18.00
    },
    {
      "name": "Beer",
      "quantity": 3,
      "unit_price": 8.00,
      "total": 24.00
    }
  ],
  "subtotal": 42.00,
  "tax": 5.04,
  "tip": 8.00,
  "total": 55.04
}
```

---

# 4. Receipt Verification

Gemini's output should never immediately become the final bill.

Before continuing, the host can:

- Correct item names.
- Correct prices.
- Change quantities.
- Add missing items.
- Delete incorrectly detected items.
- Correct tax.
- Correct tip.
- Correct the total.

The backend should also verify that the receipt calculations are valid.

---

# 5. Expense Rooms

Every expense creates a temporary collaborative room.

Example:

```text
Dinner at Restaurant

Room Code: X7K2

[Show QR Code]
[Copy Invite Link]
```

People can join through:

- QR code.
- Invite link.
- Room code.

---

# 6. Item Claiming

Each participant sees the extracted receipt.

Example:

```text
What did you have?

□ Burger                 $18.00
□ Fries                   $7.00
□ Coke                    $4.00
□ Wings                  $16.00
□ Beer                    $8.00
```

Users tap items to claim them.

---

# 7. Shared Items

Multiple people can claim the same item.

Example:

```text
Nachos — $18.00

✓ Gurman
✓ Alex
✓ Sam
□ Jake

$6.00 each
```

There should also be an:

```text
[Everyone]
```

option for items shared by the entire group.

---

# 8. Quantity Claiming

Items with multiple quantities should support individual quantities.

Example:

```text
Beer × 4

Gurman     0
Alex       1
Sam        2
Jake       1

4 / 4 claimed
```

This prevents four identical items from needing to appear separately.

---

# 9. Unclaimed Item Detection

The application should detect items nobody has claimed.

Example:

```text
⚠ 2 items still need to be claimed

Fries        $7.00
Beer         $8.00
```

Settlement cannot be finalized until all items have been resolved.

---

# 10. Real-Time Updates

Item claiming should update live.

If Gurman claims:

```text
Burger
```

everyone else immediately sees:

```text
Burger — Gurman
```

Possible implementation:

- Supabase Realtime.

---

# 11. Automatic Tax Splitting

Tax should be distributed proportionally based on how much each person purchased.

Example:

```text
Gurman's items:       $30
Receipt subtotal:    $100

Gurman's share = 30%
```

If tax is $12:

```text
Gurman's tax = $3.60
```

---

# 12. Tip Splitting

Support multiple tip methods:

- Proportional to items purchased.
- Split equally.
- Custom split.

Default:

**Proportional split.**

---

# 13. Final Breakdown

Each person gets a detailed breakdown.

Example:

```text
Gurman

Burger                 $18.00
½ Wings                  $8.00
Coke                     $4.00
──────────────────────────────
Items                   $30.00
Tax                      $3.60
Tip                      $6.00
──────────────────────────────
TOTAL                   $39.60
```

---

# 14. Settlement

The application determines who owes whom.

Example:

```text
Alex
owes Gurman
$24.50

Sam
owes Gurman
$17.25
```

The application should simplify debts where possible to reduce the number of payments required.

---

# 15. Groups

Registered users can create persistent groups.

Examples:

- Roommates
- StormHacks
- Whistler Trip
- Family
- Vacation

Groups contain:

- Members.
- Expenses.
- Running balances.
- Expense history.

---

# 16. Dashboard

Registered users have a dashboard showing their overall financial position.

Example:

```text
You are owed:      $84.20
You owe:           $31.40
─────────────────────────
Net balance:      +$52.80
```

It should also show recent expenses and groups.

---

# 17. Expense History

Users can view previous expenses.

Example:

```text
Recent Expenses

Oct 3
Dinner
You paid $120
You are owed $74

Oct 2
Uber
Alex paid $32
You owe $8

Sep 29
Groceries
You paid $86
You are owed $43
```

---

# Gemini Features

## Required

### Receipt Understanding

Gemini converts receipt images into structured data.

### Receipt Item Cleanup

Gemini converts abbreviated receipt descriptions such as:

```text
CHX BCN BRG
LG FF
D/COKE
```

into readable descriptions such as:

```text
Chicken Bacon Burger
Large French Fries
Diet Coke
```

---

## Stretch Feature: Natural-Language Claiming

Allow users to say or type:

> I had the burger and Coke. Sam and I shared the wings, and Jake had two beers.

Gemini converts the statement into item claims.

---

## Stretch Feature: Shared Item Suggestions

Gemini can recognize items that are commonly shared:

```text
Nachos
Large Pizza
Pitcher
Appetizer Platter
```

and suggest:

```text
This looks like it may have been shared.

[Choose People]
[Everyone]
[One Person]
```

Gemini should only suggest. The users make the final decision.

---

# Technology Stack

## Frontend

- Next.js
- React
- Tailwind CSS

## Backend

- Python
- FastAPI

## Database

- Supabase
- PostgreSQL

## Authentication

- Supabase Auth
- Google OAuth

## Real-Time Communication

- Supabase Realtime

## AI

- Google Gemini

## Additional

- QR code generation
- Camera/image upload
- Python `Decimal` for financial calculations

---

# Important Architecture Rule

Gemini should **interpret information**, not perform financial calculations.

```text
Gemini
├── Understand receipts
├── Clean item names
├── Understand natural language
└── Suggest classifications

Python
├── Calculate item shares
├── Calculate tax
├── Calculate tips
├── Calculate balances
├── Validate totals
└── Determine settlements
```

All financial calculations should be deterministic.

---

# MVP

The minimum version required for the hackathon is:

- [ ] Create/sign into account.
- [ ] Google authentication.
- [ ] Create expense.
- [ ] Upload/take receipt photo.
- [ ] Gemini receipt extraction.
- [ ] Verify/edit extracted receipt.
- [ ] Generate expense room.
- [ ] Generate QR code/invite link.
- [ ] Join as guest.
- [ ] Item claiming.
- [ ] Shared item claiming.
- [ ] Quantity claiming.
- [ ] Real-time claim updates.
- [ ] Detect unclaimed items.
- [ ] Calculate proportional tax.
- [ ] Calculate tip.
- [ ] Show individual totals.
- [ ] Show who owes whom.
- [ ] Finalize expense.

---

# Stretch Goals

Only begin these after the