# Economics proof boundary

This directory accepts only public, redacted bundle/signature pairs produced
by `bin/wywa-receipt` and explicitly allowlisted by bundle ID in
`platform/public-state.json`.

The publisher rejects inline amounts. A Stripe bundle qualifies only while its
signature and 24-hour verification window are current and the live provider
check found a succeeded captured charge, the complete refund set, and
reconciled `available` balance transactions. The public bundle contains
provider object IDs, request IDs, amounts, scopes, and receipt links, but no
secret key, customer identity, billing address, email, or payment method.

There are currently no qualifying bundles. The earlier Hugging Face CPU Basic
job is completed usage, but Hugging Face issues compute invoices at the
beginning of the following month. Until an invoice settles that cost, it stays
outside Product Hub sums.
