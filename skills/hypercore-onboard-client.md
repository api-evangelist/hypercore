---
name: Onboard a client and create a loan
description: Create a borrower/counterparty client in Hypercore, then originate a loan for them and move it through approval to active servicing.
api: graphql/hypercore-schema.graphql
endpoint: https://api.hypercore.ai/graphql
operations: [createClient, createLoan, approveLoan, activateLoan]
---

# Onboard a client and create a loan

Hypercore is a GraphQL API for private-credit loan management. All calls go to a
single endpoint: `POST https://api.hypercore.ai/graphql`.

## Auth
- Send a bearer access token with every request (`Authorization: Bearer <token>`).
- A missing/invalid token returns a GraphQL error with
  `extensions.code = "UNAUTHENTICATED"` and no data.

## Conventions
- Errors arrive in the top-level `errors[]` array; check `extensions.code`.
- List queries use offset pagination (`skip`, `limit`) and return
  `Paginated<Entity>` wrappers.
- Sensitive edits may be governed by a maker/checker change-request workflow
  (see the change-request-approval skill).

## Steps
1. **Create the client.** Call the `createClient` mutation with the borrower /
   counterparty details. Capture the returned client id.
2. **Create the loan.** Call the `createLoan` mutation, referencing the client id,
   with the loan terms/product. Capture the returned loan id.
3. **Approve the loan.** Call the `approveLoan` mutation with the loan id. If your
   tenant requires dual control, this may enqueue a change request instead of
   applying immediately.
4. **Activate the loan.** Call the `activateLoan` mutation with the loan id to move
   it into active servicing.
5. **Verify.** Query `loan(id: ...)` to confirm the loan status, or `loans` to list.

## Notes
- Preview/calculation queries (e.g. `getLoanApplicationPreview`,
  `getLoanTermsPreview`, `calculateLoanApplicationGrossAmount`) let you validate
  terms before committing writes.
- Mutations are not documented as idempotent — do not blindly retry on timeout;
  re-query state first.
