## Summary

<!-- What does this change do and why? 1-3 sentences. -->

## Changes

<!-- Bullet the concrete changes -->
-

## Test plan

<!-- How was this verified? Check every box that applies. -->
- [ ] Targeted unit/integration tests added or updated
- [ ] Full test suite run locally before push
- [ ] Cloud Build (or other CI) green on this PR
- [ ] Live smoke test passed against the deployed service (for backend/API changes)
- [ ] Exercised in the browser end-to-end (for UI/frontend changes)

## Rollback plan

<!-- How do we undo this if it breaks production? Migration revert? Feature flag? Revert commit? -->

## Universal rules check

- [ ] No secrets, credentials, or `.env` contents committed
- [ ] DB migrations (if any) are reversible — and `--rolled-back <migration_name>` was added to the migrate script
- [ ] No `git push --force` to `main` was used to land this work
- [ ] No `rejectUnauthorized: false` introduced on any HTTPS call
- [ ] No card data stored (Stripe Elements only, where applicable)

## Linked issue

<!-- Closes #XXX, refs #YYY -->
