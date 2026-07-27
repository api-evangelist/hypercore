---
name: Route an edit through maker/checker approval
description: Use Hypercore's maker/checker (dual-control) change-request workflow to submit, review, and approve or reject a sensitive change.
api: graphql/hypercore-schema.graphql
endpoint: https://api.hypercore.ai/graphql
operations: [requestChangeApproval, updateChangeRequestAsMaker, updateChangeRequestAsChecker, approveChangeRequest, rejectChangeRequest]
---

# Route an edit through maker/checker approval

Hypercore governs sensitive edits with a maker/checker (dual-control) workflow so a
second party approves a change before it is applied. All calls go to
`POST https://api.hypercore.ai/graphql` with a bearer access token.

## Auth & conventions
- Bearer access token on every request; unauthenticated → `extensions.code = "UNAUTHENTICATED"`.
- Errors in `errors[]` with `extensions.code`.
- List/query change requests with the `changeRequests` / `changeRequest` queries.

## Steps (maker)
1. **Submit for approval.** Call `requestChangeApproval` to open a change request
   for the intended edit.
2. **Amend as maker.** If reviewers ask for changes, call
   `updateChangeRequestAsMaker` to revise the pending request.

## Steps (checker)
3. **Amend as checker.** Use `updateChangeRequestAsChecker` to adjust reviewer-side
   fields.
4. **Decide.** Call `approveChangeRequest` to apply the change, or
   `rejectChangeRequest` to decline it. `undoRejectChangeRequest` reverses a
   rejection if needed.

## Notes
- Comment on a request with `createEntityChangeRequestComment` /
  `editEntityChangeRequestComment` for an audit trail.
- The maker and checker must be different principals for true dual control;
  enforce role separation with the appropriate access tokens.
