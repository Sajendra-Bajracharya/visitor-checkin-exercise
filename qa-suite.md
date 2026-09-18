# QA Test Suite — Visitor Check-in

## Happy Path Tests

### QA-001 — Verify successful visitor registration

**Type:** Happy Path

**Precondition:**
- The visitor registration page is open.
- An active host employee is available.

**Steps:**
1. Confirm the visitor registration form is displayed.
2. Enter a valid full name.
3. Enter a valid company name.
4. Select an active host employee.
5. Enter a valid purpose of visit.
6. Click the Submit button.
7. Check the Active Visitors list.

**Expected Result:**
The visitor is registered successfully and appears in the
Active Visitors list with the correct visitor information
and check-in time.

**Execution:** [pass]

### QA-002 — Verify visitor checkout

**Type:** Happy Path

**Precondition:**
- At least one visitor is present in the Active Visitors list.

**Steps:**
1. Confirm the visitor appears in the Active Visitors list.
2. Click the Check Out button for the visitor.
3. Check the Active Visitors list.

**Expected Result:**
The visitor is successfully checked out and no longer appears
in the Active Visitors list.

**Execution:** [pass]

### QA-003 — Confirm check-in time is displayed in receptionist's local timezone

**Type:** Happy Path

**Precondition:**
- The visitor registration page is open.
- The receptionist's browser/system timezone is set to Asia/Kathmandu.

**Steps:**
1. Register a new visitor with valid details.
2. Note the current local time (Asia/Kathmandu).
3. Check the check-in time displayed in the Active Visitors list.

**Expected Result:**
The displayed check-in time matches the current Asia/Kathmandu local time, not UTC or another timezone.

**Execution:** [fail]

### QA-004 — Confirm deactivated visitor does not appear in the active list

**Type:** Happy Path

**Precondition:**
- A visitor is currently in the Active Visitors list.

**Steps:**
1. Confirm the visitor appears in the Active Visitors list.
2. Deactivate the visitor's record as an administrator.
3. Check the Active Visitors list.

**Expected Result:**
The deactivated visitor no longer appears in the Active Visitors list.

**Execution:** [fail]

### QA-005 — Confirm end-to-end deactivation removes visitor from active list and repeat-visit selection

**Type:** Happy Path

**Precondition:**
- A visitor record exists and is currently active (not deactivated).

**Steps:**
1. Confirm the visitor appears in the Active Visitors list.
2. Confirm the visitor is selectable via the Full Name autocomplete when
   starting a new registration.
3. Deactivate the visitor's record (via administrator action or, in the
   absence of an admin UI, PATCH /api/visitors/:id/deactivate).
4. Confirm the deactivation was applied (e.g. "active":false in the
   response, or a status indicator if a UI exists).
5. Check the Active Visitors list.
6. Attempt to search/select the same visitor again via the Full Name
   autocomplete for a new registration.

**Expected Result:**
After deactivation: the visitor no longer appears in the Active Visitors
list, and does not appear as a selectable option in the repeat-visit
autocomplete.

**Execution:** [fail]

### QA-006 — Confirm GET /api/hosts returns the full list of host records

**Type:** Happy Path

**Precondition:**
- Host records exist in the database (verified via `GET /api/hosts`, e.g.
  12 records returned in current dataset: id 1 "Alice Mercer" through id
  12 "Liam Fitzpatrick").

**Steps:**
1. Send GET /api/hosts directly (e.g. via curl or API client).
2. Check the HTTP status code.
3. Check the response body structure and field names for each host record.

**Expected Result:**
The API returns 200 OK with a JSON array of host objects. Each object
contains `id` and `name` only — there is no `active`/`status` field on
the host resource, confirming hosts have no enable/disable state in the
current data model (relevant to QA-024: an "inactive host" is not a
reachable state; only an empty hosts table is).

**Execution:** [pass]

### QA-007 — Confirm GET /api/visitors returns the list of visitors

**Type:** Happy Path

**Precondition:**
- At least one visitor exists.

**Steps:**
1. Send GET /api/visitors.
2. Check the HTTP status code.
3. Check the response body.

**Expected Result:**
Returns 200 OK with a JSON array of visitor records, each including id, full_name, company_name, purpose, checked_in_at, checked_out_at, active, host_id, and host_name.

**Execution:** [pass]

### QA-008 — Confirm PATCH /api/visitors/:id/check_out successfully checks out a visitor

**Type:** Happy Path

**Precondition:**
- At least one currently active (not checked out) visitor exists, with a known id.

**Steps:**
1. Send PATCH /api/visitors/:id/check_out for that visitor's id.
2. Check the HTTP status code.
3. Check the response body.

**Expected Result:**
Returns 200 OK with the visitor's updated record, including a populated checked_out_at timestamp.

**Execution:** [pass]

### QA-009 — Confirm GET /api/visitors/search returns correct results for a valid query

**Type:** Happy Path

**Precondition:**
- At least one visitor with a known, unique full name exists.

**Steps:**
1. Send GET /api/visitors/search?q=<visitor's name>.
2. Check the HTTP status code.
3. Check the response body.

**Expected Result:**
Returns 200 OK with a JSON array containing the matching visitor(s) and their details (id, full_name, company_name, host_id).

**Execution:** [pass]

### QA-010 — Confirm POST /api/visitors successfully registers a new visitor

**Type:** Happy Path

**Precondition:**
- A valid host_id exists in the system.

**Steps:**
1. Send POST /api/visitors with a valid full_name, company_name, host_id, and purpose in the request body.
2. Check the HTTP status code.
3. Check the response body.
4. Send GET /api/visitors to confirm the new record is present.

**Expected Result:**
Returns a success status (200/201) with the newly created visitor record, including a generated id and checked_in_at timestamp. The visitor also appears in the visitor list afterward.

**Execution:** [pass]

---
## Negative Tests

### QA-011 — Verify registration with empty full name

**Type:** Negative

**Precondition:**
- The visitor registration form is open.

**Steps:**
1. Confirm the Full Name field is empty.
2. Enter a valid company name.
3. Select an active host employee.
4. Enter a valid purpose.
5. Click the Submit button.
6. Check the result.

**Expected Result:**
The visitor should not be registered and an appropriate
validation message should be displayed.

**Execution:** [pass ]

### QA-012 — Verify registration with empty Host field

**Type:** Negative

**Precondition:**
- The visitor registration form is open.

**Steps:**
1. Enter a valid full name.
2. Enter a valid company name.
3. Confirm that no host is selected.
4. Enter a valid purpose.
5. Click the Submit button.
6. Check the result.

**Expected Result:**
The visitor should not be registered and an appropriate
validation message should be displayed.

**Execution:** [pass]

### QA-013 — Verify registration with invalid Name

**Type:** Negative

**Precondition:**
- The visitor registration form is open.

**Steps:**
1. Enter invalid or unsupported data in the visitor fields.
2. Select an active host employee if required.
3. Click the Submit button.
4. Check the result.

**Expected Result:**
The application should reject invalid input and display
an appropriate validation message.

**Execution:** [fail]

### QA-014 — Verify registration with empty Company

**Type:** Negative

**Precondition:**
- The visitor registration form is open.

**Steps:**
1. Enter a valid visitor name.
2. Leave the Company field empty.
3. Enter valid data in the remaining required visitor fields.
4. Select an active host employee if required.
5. Click the Submit button.
6. Check the result.

**Expected Result:**
The application should accept the registration because the Company is not marked required.
The visitor is registered successfully and appears in the
Active Visitors list with the correct visitor information
and check-in time.

**Execution:** [pass]

### QA-015 — Verify registration with empty Purpose

**Type:** Negative

**Precondition:**
- The visitor registration form is open.

**Steps:**
1. Enter a valid visitor name.
2. Enter a valid company name.
3. Leave the Purpose field empty.
4. Select an active host employee if required.
5. Click the Submit button.
6. Check the result.

**Expected Result:**
The application should accept the registration because the Purpose field is not marked required.
The visitor is registered successfully and appears in the
Active Visitors list with the correct visitor information
and check-in time.

**Execution:** [pass]

### QA-016 — Confirm registration with a whitespace-only full name is rejected

**Type:** Negative

**Precondition:**
- The visitor registration form is open.

**Steps:**
1. Enter only spaces into the Full Name field.
2. Enter valid data in the remaining fields.
3. Select an active host employee.
4. Click the Submit button.
5. Check the result.

**Expected Result:**
The visitor should not be registered and an appropriate validation message should be displayed.

**Execution:** [fail]

### QA-017 — Confirm checking out an already-checked-out visitor is handled correctly

**Type:** Negative

**Precondition:**
- A visitor has already been checked out.

**Steps:**
1. Attempt to trigger checkout again for the same visitor (e.g., via direct API call or stale UI state).
2. Check the result.

**Expected Result:**
The system should reject the duplicate checkout or handle it gracefully, without error or duplicate side effects.

**Execution:** [fail]

### QA-018 — Confirm checking out a non-existent visitor ID returns an appropriate error

**Type:** Negative

**Precondition:**
- No visitor exists with the ID to be tested (e.g., a deleted or invalid ID).

**Steps:**
1. Send a checkout request for an invalid/non-existent visitor ID (via API).
2. Check the response.

**Expected Result:**
The API returns an appropriate error (e.g., 404) rather than a server error or silent success.

**Execution:** [pass]

### QA-019 — Verify stored script tags in Full Name are not executed when displayed (UI and API)
**Type:** Negative

**Precondition:**
- The visitor registration form is open.

**Steps:**
1. Enter `<script>alert("hacked")</script>` into the Full Name field.
2. Fill remaining fields with valid data and submit.
3. Check the Active Visitors list where this visitor's name is displayed.
4. Check whether an alert popup appears, or whether the text is shown as plain, harmless text.

**Expected Result:**
The script tag should either be rejected at input, or stored and displayed as plain visible text and should never actually execute in the browser.

**Execution:** [pass]

### QA-020 — Verify stored SQL injection payload in Full Name does not alter query behavior or get executed (UI and API)

**Type:** Negative

**Precondition:**
- The visitor registration form is open.

**Steps:**
1. Enter `' OR '1'='1` into the Full Name field.
2. Fill remaining fields with valid data and submit.
3. Check the Active Visitors list where this visitor's name is displayed.
4. Check whether the payload altered query behavior (e.g. an unexpectedly large/complete set of records, or a SQL exception), or whether it is stored and displayed as plain, harmless text with no change to normal application behavior.

**Expected Result:**
The payload should either be rejected at input, or stored and displayed as plain visible text, treated as a literal string with no special meaning — it should never alter query behavior, expose unintended data, or cause a SQL error.

**Execution:** [pass]

---
## Boundary Testing

### QA-021 — Verify active visitor pagination at 20 records

**Type:** Boundary

**Precondition:**
- At least 20 active visitors exist.

**Steps:**
1. Confirm the Active Visitors list is displayed.
2. Check the number of visitors displayed on the first page.

**Expected Result:**
A maximum of 20 visitors should be displayed on one page.
Pagination should behave correctly according to the number
of active visitors.

**Execution:** [pass]

### QA-022 — Verify pagination when the 21st visitor is added

**Type:** Boundary

**Precondition:**
- 20 active visitors already exist.

**Steps:**
1. Confirm that 20 active visitors are available.
2. Register one additional visitor.
3. Check the Active Visitors list.
4. Move to the next page.
5. Check the visitors displayed on the second page.

**Expected Result:**
The first page should contain a maximum of 20 visitors and
the additional visitor should appear on the second page.

**Execution:** [pass]

### QA-023 — Verify that the active list renders an empty state with zero active visitors

**Type:** Boundary

**Precondition:**
- No visitors are currently active (all checked out or none registered).

**Steps:**
1. Confirm no active visitors exist.
2. Check the Active Visitors list display.

**Expected Result:**
The list displays a clear empty state rather than an error, broken layout, or blank screen.

**Execution:** [pass]

### QA-024 — Confirm active list displays correctly with exactly one active visitor

**Type:** Boundary

**Precondition:**
- Exactly one visitor is currently active.

**Steps:**
1. Confirm only one visitor exists in the active list.
2. Check the list display and pagination controls.

**Expected Result:**
The single visitor displays correctly with no pagination controls shown.

**Execution:** [pass]

### QA-025 — Confirm pagination shows no "next page" control at exactly 20 active visitors

**Type:** Boundary

**Precondition:**
- Exactly 20 active visitors exist.

**Steps:**
1. Confirm the Active Visitors list is displayed.
2. Check the number of visitors on the page.
3. Check whether a next-page control is present.

**Expected Result:**
All 20 visitors display on a single page, and no next-page control is shown (since there is no second page).

**Execution:** [fail]