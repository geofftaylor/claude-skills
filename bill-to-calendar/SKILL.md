---
name: bill-to-calendar
description: Parses bill/payment due information from copied email text and creates a BusyCal event on the Bills calendar. Use this skill whenever the user pastes or shares email text that contains a bill, payment due, or statement balance — even if they don't explicitly say "create a calendar event." Trigger phrases include: "add this to my calendar," "here's my bill email," "can you schedule this payment," or any time the user pastes text that looks like a billing statement or payment reminder.
---

# Bill to Calendar

Parses three values from copied billing email text and creates an all-day BusyCal event on the Bills calendar with reminders.

## What to extract

| Field | Notes |
|---|---|
| **Merchant** | Who the bill is from. For credit card statements, include both the issuer and the card name (e.g., `Chase Sapphire Preferred`, `Citi Double Cash`, `Amex Gold`). For other bills (utilities, subscriptions, etc.), use the company or service name (e.g., `Seattle City Light`, `Comcast`). Look for card names in the subject line, greeting, or account label. |
| **Amount due** | The amount the user owes. If both a statement balance and a minimum payment are present, use the **statement balance**. Format as a dollar amount: `$142.50`. |
| **Due date** | The payment due date. Parse natural language dates and convert to `YYYY-MM-DD`. |

## Handling missing or ambiguous values

- **Merchant not found**: Ask the user — "I couldn't determine who this bill is from. Who is the merchant or creditor?"
- **Amount not found**: Ask the user — "I couldn't find an amount due. What is the amount owed?"
- **Due date not found or ambiguous**: Ask the user — "I couldn't determine the due date. When is this payment due?"
- **Statement balance vs. minimum payment**: Always prefer the statement balance. Only use the minimum payment if no statement balance is present.

Resolve all missing fields before creating the event. If multiple values are missing, ask for all of them in a single message.

## Event format

Once all three values are confirmed, create the event with:

- **Title**: `[Merchant] $[Amount]` — e.g., `Chase Sapphire Preferred $1,204.38` or `Comcast $89.99`
- **Calendar**: Bills (`CALENDAR_ID`)
- **Date**: The due date as an all-day event (`allDay: true`, date-only `startDate`)
- **Reminders**: Three alarms — 2 days before, 1 day before, and on the due date, each at 9 AM:
  - `alarms: [-2340, -900, 540]` (in minutes; all-day events start at midnight, so +540 min = 9 AM)

## After creating the event

Confirm to the user with a brief summary:
> Added **[Merchant] $[Amount]** to your Bills calendar on **[due date]** with reminders 2 days before, 1 day before, and on the due date.
