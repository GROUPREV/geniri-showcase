# The invoice waits for a person

**Status:** in force. **Owners:** the product and finance side.

## Context

Geniri sells to creative professionals and small studios in Algeria. For a
business purchase in that market, a CCP transfer — BaridiMob, or at the post
office — is the ordinary instrument; a card checkout is the exception, and a card
checkout with an invoice that has to match the company's accounting is a bigger
promise than it looks.

Two paths were possible: take the money automatically with a card processor, or
accept CCP transfers and confirm them by hand. The first is faster for the customer
and hands the failure modes to a third party: chargebacks, processor holds, refund
timing, and a reconciliation job that has to trust a webhook.

## Decision

Accept payment by CCP transfer, by BaridiMob or at the post office, against a
`GNR-…` reference, and have a person confirm the match in `/admin`. Confirmation
releases the credits and issues the French invoice.

Invoice numbers come from one gapless counter, assigned inside the confirming
transaction, so a double click cannot skip a number or produce a second invoice
for one order.

## Consequences

- Delivery is not instant. The customer waits for a person, and the queue makes
  that wait visible instead of hiding it: unmatched transfers have their own view,
  and a transfer with no usable reference is matched by hand.
- The fraud surface is smaller and the disputes are conversations. Nobody has to
  argue with a chargeback.
- Someone has to be at the desk. This is the cost we accepted, and the release
  notes say when the desk is not staffed.
- The ledger has one entry path. Credits granted by a confirmation look exactly
  like credits granted any other way, which is what makes a disputed balance
  answerable.

## What we would reconsider

If volume outgrows one person's morning, the answer is a processor with a proper
invoice integration, not a faster queue. Until then, the manual step is doing real
work: it catches the transfer that arrives without a reference, and that is the
one an automated system would misapply.
