# QA Notes — Visitor Check-in

## Highest-Risk Area

I'd consider **deactivation enforcement** (Defects 5 and 6) the highest-risk area I found.

Most of the other issues are related to validation or display, such as the wrong timezone, missing character limits, or the pagination issue when there are exactly 20 records. These are still valid bugs, but their impact is relatively contained.

Deactivation was different when I looked into it more closely. The requirement clearly states that deactivated visitors **should not appear in the active list or be available for repeat visits**. However, neither of these worked as expected. `GET /api/visitors` still returns deactivated visitors with `active:false`, and the Full Name autocomplete used for repeat visits also doesn't appear to check the `active` field. I confirmed this by looking at what the search query actually filters.

What made me rank this above the others is that the same issue — not checking the `active` field — appears in two different places independently. To me, that suggests the `active` status isn't being enforced in one central place. Instead, each endpoint seems to be responsible for remembering to apply the check, and two of them missed it. If that's the case, future features that work with visitor records, such as a reports page or host notifications, could potentially have the same problem. So even though the issue doesn't look very serious from the UI, I think it's worth flagging before sign-off rather than treating it as just another functional bug.

## Question for the Product Owner

> Is visitor deactivation supposed to have an admin UI at launch, or is it
> okay for v1 that deactivation only happens through the API
> (PATCH /api/visitors/:id/deactivate)?

I'm asking because right now there's no admin screen anywhere in the app
for this — the only way I could trigger it during testing was hitting the
API directly. If that's intentional for this release, Defects 5 and 6
still need fixing regardless of how deactivation gets triggered. But if
an admin UI was actually expected to be part of this feature, that's a
separate missing-piece finding on top of the two defects, and it
probably changes what "done" looks like for sign-off.