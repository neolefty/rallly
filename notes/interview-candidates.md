# Interview candidates from the Rallly backlog

Working notes for using Rallly as the basis for take-home and live technical
interviews where the candidate is expected to use an AI assistant. This fork
(`neolefty/rallly`) is a private scratch space; the actual interview basis will
be a separate fork.

Surveyed 2026-11-03 against upstream `lukevella/rallly` at commit `2e3c5db81`.

## How this list was built

- The upstream repo has ~323 issues, almost all closed as completed. The large
  or out-of-scope asks live in the **Ideas** discussion category (~175 threads).
- Many of the oldest, most-upvoted asks have since shipped: SSO, a public REST
  API (`app/api/v1`, OpenAPI), webhooks (`features/webhook`), a health
  endpoint (`app/api/status`), event types with `capacity`.
- Filtered to what is still unbuilt at the pinned commit, favoring threads where
  the maintainer said "too hard / not now" so the candidate has to narrow scope.

Links: `https://github.com/lukevella/rallly/discussions/<n>` for Ideas,
`https://github.com/lukevella/rallly/issues/<n>` for issues.

## Take-home candidates (large, must be scoped down)

| Thread | Title | Upvotes | Comments | Why it works |
|---|---|---|---|---|
| Ideas #148 | Use for non-date polls | 21 | 6 | Oldest open ask (2017). `Option` is hard-wired to `startTime` + `duration`. Candidate must decide: new `PollKind`, polymorphic options, or a separate entity. Touches schema, tRPC, poll form, results grid, CSV export, v1 API. |
| Ideas #171 | Date range as a votable option | 11 | 3 | Maintainer flagged the hard part in-thread: how ranges interact with week view. Tests whether the candidate reads the discussion before coding. |
| Ideas #1388 + #155 + #139 | Cap votes per option / first-come slot booking | 7 + 3 + 2 | 2 + 3 + 1 | Three threads, one feature (teacher–parent slots). Concurrency: two participants racing for the last seat. Event types already have `capacity`; a strong candidate finds that precedent and decides whether to reuse it. |
| Ideas #144 + #149 | Anonymous voting / hide answers until you vote | 7 + 6 | 3 + 1 | `hideParticipants` and `hideScores` already exist, so this is a permission-model question. Where does the rule live: CASL ability, loader, or data layer? |
| Ideas #443 + #163 + #916 | Duplicate poll / recurring polls / minimum attendees | 2 + 2 + 1 | 2 + 2 + 0 | Maintainer: "difficult without overcomplicating." Minimum is a duplicate action that shifts dates forward; maximum is a recurrence engine. The gap is the scoping call to observe. |
| Ideas #535 + #1933 | iCal subscription feed per poll / per space | 5 + 2 | 1 + 2 | Bounded but not small: tokenized unauthenticated route, ICS generation over tentative options, time zones, revocation. One-off `.ics` download exists in `features/scheduled-event`; feeds do not. |
| Ideas #3474 | In-app notification inbox | 1 | 0 | Recent; author already inventoried what exists (`PollActivity`, email prefs, webhooks). "Here is the brief, you pick the first slice." |
| Issue #2513 | Calendar view for answering polls | 0 | 1 | Closed not-planned because the poll page is about to split into participant and admin UIs. Front-end-leaning homework that must work around a pending refactor. |

## Live-session candidates (60–90 min, with an assistant)

- **Issue #1677 (still open): participants can vote on options already in the past.**
  Looks trivial. Two contributors already got it wrong by fixing the creation
  form instead of the invite page. Time zones and all-day floating dates make
  "in the past" non-obvious. Watch how the candidate verifies what the
  assistant proposes.
- **Ideas #284 + #2358: rank best dates, then find complementary times.**
  Pure scoring over yes / ifNeedBe / no votes; fits in one `utils.ts` with
  tests. Complementary times (set cover) is the stretch goal.
- **Ideas #3475: "Copy summary" button.** Small, reuses the scoring helper,
  plus i18n and clipboard. Pairing-style task for navigating `features/poll`.
- **Ideas #1017 + #1387: configurable terms/imprint links for self-hosted.**
  Tiny feature with a strong CLAUDE.md rule: no new `isSelfHosted` forks, add
  an instance-policy field. Tests whether they read conventions before letting
  the assistant pattern-match.
- **Ideas #896: anyone can comment under any name.** Maintainer: "works as
  expected for now." Design prompt: identity vs. display name, guest vs.
  account, what to enforce server-side.
- **Ideas #3109: reschedule a finalized event.** State machine (scheduled →
  canceled → rescheduled), notifications, calendar sync. Discussion-heavy.

## FHIR / Medplum tie-in

Rallly's domain maps almost one-to-one onto FHIR scheduling resources:

| Rallly | FHIR R4 |
|---|---|
| Poll | `Appointment` (status `proposed`) |
| Option | `Slot` |
| Participant | `Appointment.participant` |
| Vote (yes / ifNeedBe / no) | `AppointmentResponse.participantStatus` (accepted / tentative / declined) |

Homework variants: "export a finalized poll as a FHIR R4 Appointment bundle"
or "import Slots from a Medplum instance as poll options." Both stay inside the
existing `scheduled-event` and `calendars` features and test learning an
unfamiliar standard.

## Cautions

- Upstream moves fast and CLAUDE.md is demanding, with lint-enforced layering.
  Pin candidates to a commit and say whether those conventions are graded; an
  assistant only follows them if the candidate lets it read the file.
- Several old threads were closed as "moved to discussions" with no code
  change; check the current checkout before assuming anything is unbuilt.

## Other threads worth a second look

Ideas #374 (availability windows, crab.fit style), #1194 (participants add
options), #2096 (finalize several dates), #1244 (core-group availability
filter), #134 (automatic expiry), #2052 (poll for location), #1029 / #1071
(use account identity when voting), #352 (only admins create polls),
#306 (closed; screen-reader accessibility, ongoing).
