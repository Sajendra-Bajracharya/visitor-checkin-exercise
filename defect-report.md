# Defect Report

## Defects

### Defect 1

**Summary:** Check-in time displayed in raw UTC instead of Asia/Kathmandu local time

**Type:** Functional

**Severity:** Medium

**Description:** The API correctly returns check-in timestamps in UTC with proper ISO 8601 formatting (e.g., "2026-09-11T09:24:56Z"). However, the frontend displays this raw UTC time directly without converting it to Asia/Kathmandu (UTC+5:45), the timezone the app is designed for per the README. This causes the "Checked In" column to show a time roughly 5 hours 45 minutes behind the receptionist's actual local time.

**Steps to Reproduce:**
1. At a known local time (e.g. 10:55 Asia/Kathmandu), register a new visitor via the registration form (e.g. "Sam Bahadur").
2. Inspect the network response (or GET /api/visitors) for this visitor's checked_in_at value — e.g. "2026-09-15T05:10:57Z" (UTC).
3. Confirm the conversion: 05:10 UTC + 5:45 (Kathmandu offset) = 10:55, matching the actual local registration time — confirming the backend stored the correct UTC-equivalent value.
4. Compare against the "Checked In" time shown in the Active Visitors table in the UI.

**Expected Result:** Checked-in time displayed in UI should be converted to Asia/Kathmandu local time (10:55), per spec: "All times are displayed in the receptionist's local timezone."

**Actual Result:** UI displays the raw UTC value unconverted (05:10), roughly 5h45m behind actual local time.

---

### Defect 2

**Summary:** "Next" button appears/is clickable at exactly 20 active visitors, but page 2 loads empty

**Type:** Functional

**Severity:** Medium

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

**Summary:** Full name field accepts non-name input with no validation

**Type:** Functional

**Severity:** Medium

**Description:** The visitor registration form's Full Name field accepts arbitrary special-character strings (e.g., "@#$#$^#$^%@#$@#$1234") with no client-side or server-side validation rejecting non-name input. The record is saved and appears in the active visitor list as entered. Since Full Name is meant to capture a person's name, input that contains no alphabetic characters at all does not satisfy the field's evident purpose.

**Steps to Reproduce:**
1. Open the visitor registration form.
2. Enter "@#$#$^#$^%@#$@#$1234" in the Full Name field.
3. Fill remaining required fields with valid data.
4. Submit the form.

**Expected Result:** The form should reject input that doesn't resemble a valid name (e.g., require at least some alphabetic characters), showing a validation error.

**Actual Result:** The form accepts the input without error, and the visitor is registered and appears in the active list with the garbled name.

---

### Defect 4

**Summary:** Duplicate checkout silently succeeds and overwrites checkout timestamp

**Type:** Functional

**Severity:** High

**Description:** The check_out endpoint (PATCH /api/visitors/:id/check_out) does not guard against checking out a visitor who has already been checked out. Calling it a second time on the same visitor ID returns 200 OK and overwrites checked_out_at with a new timestamp, instead of rejecting the request or leaving the original checkout record unchanged. Since the app is responsible for accurately recording when a visitor checked out, silently overwriting that record on a repeat call corrupts historical data the app has already committed to tracking.

**Steps to Reproduce:**
1. Send PATCH /api/visitors/86/check_out (visitor successfully checked out, checked_out_at set to 2026-09-13T08:52:06Z).
2. Immediately send PATCH /api/visitors/86/check_out again for the same ID.
3. Observe the response.

**Expected Result:** The second request should be rejected (e.g. 422 Unprocessable Entity) with a clear error message, or otherwise leave the original checked_out_at value unchanged.

**Actual Result:** The second request returns 200 OK and checked_out_at is overwritten with a new timestamp (2026-09-13T08:52:30Z), silently allowing a duplicate checkout.

---

### Defect 5

**Summary:** Whitespace-only full name is accepted and persisted

**Type:** Data

**Severity:** Medium

**Description:** The visitor registration form (and/or its underlying API validation) does not reject a full name consisting only of whitespace. This is a direct inconsistency with the app's own established behavior: a truly empty Full Name is correctly rejected (confirmed working), but a whitespace-only value — which conveys no name information either — is accepted. Evidence of this exists in the current dataset: visitor id 108 has full_name equal to a single space character (" "), confirming this was previously accepted rather than being a purely theoretical case.

**Steps to Reproduce:**
1. Open the visitor registration form.
2. Enter only space characters into the Full Name field.
3. Fill remaining fields with valid data and submit.
   (Alternatively: query GET /api/visitors/search?q=%20%20%20 and observe that a record with full_name " " (id 108) already exists in the dataset.)

**Expected Result:** A whitespace-only full name should be rejected with a validation error, both client-side and server-side, consistent with how a fully empty name is already rejected.

**Actual Result:** The name is accepted; a record with a whitespace-only full_name already exists in the database (id 108).

---

### Defect 6

**Summary:** Long input text breaks Active Visitors table layout

**Type:** Usability

**Severity:** Low

**Description:** When a visitor is registered with a very long Full Name (500+ characters), the Active Visitors table renders the entire string unbounded in the Name column. There is no truncation, ellipsis, or word-wrap containment, which causes the row to expand dramatically and visually breaks the alignment of the Company, Host, Purpose, Checked In, and Action columns for that row. This is independent of whether a maximum character limit should exist (see Assumptions/Open Questions) — regardless of how long input is permitted to be, the table should not visually break when it receives long input.

**Steps to Reproduce:**
1. Open the visitor registration form.
2. Enter a Full Name of 500+ characters (e.g. a long paragraph of text).
3. Fill remaining fields with valid data and submit.
4. Check the Active Visitors list.

**Expected Result:** Long text should be visually contained (e.g. truncated with an ellipsis, or wrapped within a fixed column width) so the table layout and row alignment remain intact regardless of input length.

**Actual Result:** The full text renders unbounded, expanding the row height significantly and misaligning the row's other columns relative to the rest of the table.

---

### Defect 7

**Summary:** Deactivated visitors remain selectable via repeat-visit search

**Type:** Functional

**Severity:** High

**Description:** The requirements state deactivated visitors "must not be selectable for repeat visits." A deactivated visitor was observed still present in the Full Name autocomplete dropdown and remained selectable. This matches the underlying search behavior directly: the search query does not filter by the `active` field, so deactivated visitors are never excluded from results. (Note: the same search also returns checked-out visitors, but that is expected/correct behavior, since a returning visitor should still be findable for a repeat visit — only the deactivated-visitor case violates the spec.)

**Steps to Reproduce:**
1. Register a visitor and note their full name.
2. Deactivate that visitor's record via PATCH /api/visitors/:id/deactivate.
3. In the Full Name field, type the name of the deactivated visitor.
4. Observe the autocomplete dropdown results.

**Expected Result:** The deactivated visitor should not appear in the search/autocomplete results and should not be selectable for a repeat visit.

**Actual Result:** The deactivated visitor still appeared in the search results and remained selectable.

---

### Defect 8

**Summary:** Deactivated visitors are not excluded from the active visitor list

**Type:** Functional

**Severity:** High

**Description:** Per the requirements, deactivated visitors "must not appear in the active list." However, GET /api/visitors (the endpoint backing the Active Visitors list) does not filter out records where active is false. A visitor deactivated via PATCH /api/visitors/:id/deactivate continues to be returned by this endpoint, with active:false clearly present in the response, confirming the record itself is correctly flagged but simply not being filtered.

**Steps to Reproduce:**
1. Register a visitor (or use an existing one) and confirm their id via GET /api/visitors.
2. Deactivate that visitor: PATCH /api/visitors/:id/deactivate. Confirm the response shows "active":false.
3. Send GET /api/visitors again.
4. Check whether the deactivated visitor's record is still present.

**Expected Result:** The deactivated visitor should not be present in the GET /api/visitors response (or should at minimum not surface in the Active Visitors UI list).

**Actual Result:** The deactivated visitor remains present in the response with "active":false, meaning the UI's Active Visitors list would also continue displaying them, since it consumes this same endpoint.

---

## Security Testing — Confirmed Safe (Not Defects)

These were specifically tested for and found to be handled correctly. Included here for completeness, since verifying an app is *not* vulnerable is a meaningful part of the testing effort, not just an absence of findings.

- **Stored XSS via Full Name:** Entering `<script>alert("hacked")</script>` and `<img src=x onerror=alert('xss')>` into Full Name was accepted and stored, but rendered as plain, inert text in both the Active Visitors table and the search autocomplete dropdown — no script execution occurred in either location.
- **SQL injection via Full Name and search query:** Entering `' OR '1'='1` and similar payloads into Full Name, and into the search query parameter, was treated as a literal string in both cases — no SQL errors were raised, no unintended records were exposed, and no query behavior was altered.

## Assumptions / Open Questions

- **No administrator UI exists for deactivating visitors.** The README states deactivation is performed "by an administrator" but doesn't specify whether this requires a dedicated UI. The only working path found is the API endpoint (PATCH /api/visitors/:id/deactivate) — there is no visible admin section in the frontend. Unclear whether this is an intentional scope limitation for the exercise or a missing feature. Recommend confirming with the Product Owner whether an admin UI was expected as part of this deliverable.

- **No maximum character length is specified for Full Name, Company, or Purpose fields.** The spec doesn't state a hard limit, so it's unclear whether one should be enforced server-side (e.g. 255 characters) or whether unbounded input is intentional. Recommend confirming with the Product Owner. (Note: this is distinct from Defect 6 above — the *lack of a limit* is not itself a defect, but the UI's failure to visually contain long input regardless of limit is.)

- **Company and Purpose fields appear fully unvalidated** (accept empty, whitespace-only, and arbitrary special characters). This is consistent with the working assumption that these fields are optional (empty submissions succeed), but it's worth confirming with the Product Owner whether any format expectations exist for these fields at all, or whether "accept anything, including nothing" is intentional.