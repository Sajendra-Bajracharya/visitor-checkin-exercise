## Regression Subset — Minor Registration Form Update

**Update I am assuming:** Full Name gets validation added to reject special
characters and numbers — only letters (and probably spaces) are allowed,
enforced both client-side and server-side, with an error shown on invalid
input. I picked this specifically because it relates back to Defect 3
(the field currently accepts a string like `@#$#$^#$^%@#$@#$1234` with no
validation at all) — this update is essentially the fix for that defect.
That defect maps to **QA-013** and **QA-016**("Verify registration with invalid Name") and(" Confirm registration with a whitespace-only full name is rejected Confirm registration with a whitespace-only full name is rejected")which currently fails and should flip to pass once this update ships.

Because the rule is "letters (and spaces) only," it doesn't just fix
Defect 3 it also changes *how* several other inputs get handled.
Whitespace-only, `<script>...</script>`, and `' OR '1'='1`-style input
all contain zero letters or contain characters outside the allowed set,
so all of them would now be rejected at the validation layer itself,
rather than being accepted and then handled safely downstream (empty
check, escaped rendering, or parameterized query). Any test that
previously passed *because* the app safely handled a weird Full Name
value needs to be re-checked against a new question: does it still pass
now that the value is rejected before it ever reaches storage/rendering,
instead of being stored and neutralized there?

I went through every test in `qa-suite.md` and decided in/out based on
whether it actually touches the Full Name field or shares its validation
path not just whether it's "part of registration" in general.

### Tests Included

| Test | Why |
|------|-----|
| QA-001 | Basic happy-path registration goes straight through the Full Name field with a normal alphabetic name. Need to confirm valid names still pass once the new character check is added. |
| QA-010 | POST /api/visitors — registers a visitor with a valid full_name at the API level. Same validation path as QA-001 but bypassing the UI, so worth confirming valid names still pass server-side too. |
| QA-011 | Tests empty Full Name. Same validation layer as the new character rule — want to confirm the empty-name check still fires first/correctly and isn't overridden by the new rule. |
| QA-013 | Tests invalid/unsupported input in Full Name. This is the *exact* case the update targets — this test's expected result should flip from fail to pass, so it must be re-run. |
| QA-016 | Tests whitespace-only Full Name. Same validator — need to confirm this still gets rejected on its own terms (whitespace has no letters *or* special characters/numbers, so both rules could plausibly interact here). This is the same case as "Defect 5" (whitespace-only name), so no separate line item is needed for that. |
| QA-019 | Confirms script-tag input doesn't execute. `<script>alert("hacked")</script>` is made almost entirely of special characters, so under the new rule this input should now be rejected outright at validation — its expected result should still pass, but for a different reason (rejected at the form, not stored-then-neutralized). Need to confirm the error path is what actually fires, not a coincidental pass from the old escaping behavior no longer being exercised. |
| QA-020 |  malicious code  `' OR '1'='1`entered  directly into the Full Name field,It should now be rejected at validation instead of being accepted and safely stored as a literal string. Need to confirm it's caught there rather than passing for the old reason. |

### Tests I'd leave out

| Test | Why |
|------|-----|
| QA-002 | Checkout flow. Doesn't touch the registration form at all. |
| QA-003 | Timezone display. Different feature entirely, no shared code with Full Name validation. |
| QA-004, QA-005 | Deactivation behavior — different field/endpoint. QA-005 does exercise the Full Name autocomplete, but only for searching/selecting an existing visitor, not for validating new input, so it doesn't share the changed code path. |
| QA-006 | GET /api/hosts. Tests the host resource, unrelated to Full Name. |
| QA-007 | GET /api/visitors listing. Confirms response shape, not Full Name validation. |
| QA-008 | PATCH /api/visitors/:id/check_out. Checkout endpoint, no overlap with registration-time validation. |
| QA-009 | GET /api/visitors/search. Searches existing visitors by name; doesn't exercise new-registration validation. |
| QA-012 | Testing the Host field, not Full Name. |
| QA-014 | Testing Company, not Full Name — this field is explicitly unvalidated and out of scope for this update. |
| QA-015 | Testing Purpose, not Full Name — same reasoning as QA-014. |
| QA-017, QA-018 | Both are checkout-endpoint negative tests. No overlap with registration-time validation. |
| QA-020 | Hits the *search* endpoint with a SQLi payload, not the registration form's Full Name input. Different code path (existing visitors, not new-registration validation), so the new rule doesn't touch it — search still needs to accept arbitrary query strings safely. (See the SQLi note above for the separate Full-Name-side case, which is covered, just not under this ID.) |
| QA-021–QA-025 | All pagination / empty-list / single-record display tests. None of them care what the Full Name validation rules are. |