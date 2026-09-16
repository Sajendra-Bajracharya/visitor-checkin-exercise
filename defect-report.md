# Defect Report

## Defects

### Defect 1
**Summary:**  Check-in time displayed in raw UTC instead of Asia/Kathmandu local time
**Type:** Functional

**Description:** The API correctly returns check-in timestamps in UTC with proper ISO 8601 formatting (e.g., "2026-09-11T09:24:56Z"). However, the frontend displays this raw UTC time directly without converting it to Asia/Kathmandu (UTC+5:45), the timezone the app is designed for per the README. This causes the "Checked In" column to show a time roughly 5 hours 45 minutes behind the receptionist's actual local time.

**Steps to Reproduce:**
1. At a known local time (e.g. 10:55 Asia/Kathmandu), register a new visitor via the registration form (e.g. "Sam Bahadur").
2. Inspect the network response (or GET /api/visitors) for this visitor's checked_in_at value — e.g. "2026-09-15T05:10:57Z" (UTC).
3. Confirm the conversion: 05:10 UTC + 5:45 (Kathmandu offset) = 10:55, matching the actual local registration time  confirming the backend stored the correct UTC-equivalent value.
4. Compare against the "Checked In" time shown in the Active Visitors table in the UI.

**Expected Result:** Checked-in time displayed in UI should be converted to Asia/Kathmandu local time (10:55), per spec: "All times are displayed in the receptionist's local timezone."
 
**Actual Result:** UI displays the raw UTC value unconverted (05:10), roughly 5h45m behind actual local time.

---

### Defect 2 

**Summary:** "Next" button appears/is clickable at exactly 20 active visitors, but page 2 loads empty

**Type:** Functional

**Description:** When the active visitor count reaches exactly 20 (the stated page size), the "Next" pagination button becomes enabled. Clicking it navigates to a second page with no records, even though page 1 already displayed all existing visitors. This suggests the pagination logic enables "Next" based on the count reaching the page size threshold, rather than checking whether records actually exist beyond the current page.

**Steps to Reproduce:**
1. Register exactly 20 active visitors.
2. Observe the active visitor list; confirm all 20 appear on page 1.
3. Confirm the "Next" button is enabled/clickable.
4. Click "Next."

**Expected Result:** With exactly 20 total records, "Next" should be disabled/absent.

**Actual Result:** "Next" is enabled at exactly 20 records; clicking it loads an empty page.

---

### Defect 3

**Summary:** Deactivated visitors remain selectable via repeat-visit search

**Type:** Functional

**Description:** The requirements state deactivated visitors "must not be selectable for repeat visits." A deactivated visitor was observed still present in the Full Name autocomplete dropdown and remained selectable. This matches the underlying search behavior directly: the search query does not filter by the `active` field, so deactivated visitors are never excluded from results. (Note: the same search also returns checked-out visitors, but that is expected/correct behavior, since a returning visitor should still be findable for a repeat visit — only the deactivated-visitor case violates the spec.)

**Steps to Reproduce:**
1. Register a visitor and note their full name.
2. Deactivate that visitor's record via PATCH /api/visitors/:id/deactivate.
3. In the Full Name field, type the name of the deactivated visitor.
4. Observe the autocomplete dropdown results.

**Expected Result:** The deactivated visitor should not appear in the search/autocomplete results and should not be selectable for a repeat visit.

**Actual Result:** The deactivated visitor still appeared in the search results and remained selectable.

---

### Defect 4

**Summary:** Deactivated visitors are not excluded from the active visitor list

**Type:** Functional

**Description:**
Per the requirements, deactivated visitors "must not appear in the active
list." However, GET /api/visitors (the endpoint backing the Active
Visitors list) does not filter out records where active is false. A
visitor deactivated via PATCH /api/visitors/:id/deactivate continues to
be returned by this endpoint, with active:false clearly present in the
response, confirming the record itself is correctly flagged but simply
not being filtered.

**Steps to Reproduce:**
1. Register a visitor (or use an existing one) and confirm their id via
   GET /api/visitors.
2. Deactivate that visitor: PATCH /api/visitors/:id/deactivate. Confirm
   the response shows "active":false.
3. Send GET /api/visitors again.
4. Check whether the deactivated visitor's record is still present.

**Expected Result:**
The deactivated visitor should not be present in the GET /api/visitors
response (or should at minimum not surface in the Active Visitors UI list).

**Actual Result:**
The deactivated visitor remains present in the response with
"active":false, meaning the UI's Active Visitors list would also
continue displaying them, since it consumes this same endpoint.

 
### Defect 5
 
**Summary:** Duplicate checkout silently succeeds and overwrites checkout timestamp
 
**Type:** Functional
 
**Description:** The check_out endpoint (PATCH /api/visitors/:id/check_out) does not guard against checking out a visitor who has already been checked out. Calling it a second time on the same visitor ID returns 200 OK and overwrites checked_out_at with a new timestamp, instead of rejecting the request or leaving the original checkout record unchanged.
 
**Steps to Reproduce:**
1. Send PATCH /api/visitors/86/check_out (visitor successfully checked out, checked_out_at set to 2026-09-13T08:52:06Z).
2. Immediately send PATCH /api/visitors/86/check_out again for the same ID.
3. Observe the response.
**Expected Result:** The second request should be rejected (e.g. 422 Unprocessable Entity) with a clear error message, or otherwise leave the original checked_out_at value unchanged.
 
**Actual Result:** The second request returns 200 OK and checked_out_at is overwritten with a new timestamp (2026-09-13T08:52:30Z), silently allowing a duplicate checkout.

## Assumptions / Open Questions

- **No administrator UI exists for deactivating visitors.** The README states deactivation is performed "by an administrator" but doesn't specify whether this requires a dedicated UI. The only working path found is the API endpoint (PATCH /api/visitors/:id/deactivate) — there is no visible admin section in the frontend. Unclear whether this is an intentional scope limitation for the exercise or a missing feature. Recommend confirming with the Product Owner whether an admin UI was expected as part of this deliverable.

- **No maximum character length is specified for Full Name, Company, or Purpose fields.** The spec doesn't state a hard limit, so it's unclear whether one should be enforced server-side (e.g. 255 characters) or whether unbounded input is intentional and only the display layer is expected to handle it gracefully. Recommend confirming with the Product Owner.

- **Company and Purpose fields appear fully unvalidated** (accept empty, whitespace-only, and arbitrary special characters). This is consistent with the working assumption that these fields are optional but it's worth confirming with the Product Owner whether any format expectations exist for these fields at all, or whether "accept anything, including nothing" is intentional.

- **Full name field accepts non-name input with no validation.** The visitor registration form's full name field accepts arbitrary special-character strings (e.g., "@#$#$^#$^%@#$@#$1234") with no client-side or server-side validation rejecting non-name input, and the record is saved and appears in the active visitor list as entered. Unclear whether strict name-format validation was expected or whether free-text input is acceptable for this field. Recommend confirming with the Product Owner whether a minimum bar (e.g., at least some alphabetic characters) should be enforced.

- **Duplicate checkout silently succeeds and overwrites the checkout timestamp.** The check_out endpoint (PATCH /api/visitors/:id/check_out) does not guard against checking out a visitor who has already been checked out; calling it a second time on the same visitor ID returns 200 OK and overwrites checked_out_at with a new timestamp rather than rejecting the request or preserving the original value. Unclear whether idempotency/duplicate-guard behavior was an explicit requirement or an implicit expectation. Recommend confirming with the Product Owner whether this should be rejected (e.g. 422) or left as-is.

- **Whitespace-only full name is accepted and persisted.** Evidence exists in the current dataset: visitor id 108 has full_name equal to a single space character (" "), confirming this was previously accepted rather than being a purely theoretical case. This overlaps with the broader open question above about whether any name-format validation is expected; recommend confirming with the Product Owner whether whitespace-only values should be explicitly rejected.

- **Long input text breaks Active Visitors table layout.** When a visitor is registered with a very long Full Name (500+ characters), the Active Visitors table renders the entire string unbounded in the Name column, with no truncation, ellipsis, or word-wrap containment, which causes the row to expand and misaligns the other columns for that row. Since no maximum length is specified for the field (see above), it's unclear whether this is a defect in its own right or a downstream consequence of the open question about field length limits. Recommend confirming with the Product Owner whether visual containment (truncation/wrapping) is expected regardless of input length.

- **No defined handling for script/HTML tags in the Full Name field.** Entering `<script>alert("Sajendra");</script>` in the Full Name field is stored and displayed as plain text in the UI (the script does not execute), while the API returns the value with Unicode escaping (`\u003cscript\u003ealert(\"Sajendra\");\u003c/script\u003e`). The spec doesn't state whether script/HTML-like input should be rejected outright, stripped/sanitized, stored as-is, or escaped differently than currently observed. Recommend confirming with the Product Owner which behavior is intended: reject script/HTML tags entirely, strip/sanitize them, store as-is (current behavior), or apply different escaping rules.