---
description: Audit dependencies, apply fixes, and verify with tests
---

Audit the project's npm dependencies and verify that automatic fixes don't break the test suite. Perform these three steps in order:

1. Run `npm audit` and report any vulnerabilities found (severity, package, advisory).
2. Run `npm audit fix` to apply automatic patch/minor updates for the vulnerable packages.
3. Run `npm test` to confirm the updates didn't break anything.

If step 3 fails, surface the failing tests and stop — do not attempt further fixes without confirmation.
