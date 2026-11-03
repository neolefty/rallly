# Homework phase: approach and mechanics

Working notes for the take-home portion of the interview. Companion to
`interview-candidates.md`, which lists which backlog threads to turn into
tasks. The live-interview phase is out of scope here.

Status as of 2026-11-03: approach agreed in discussion, nothing built yet. No
interview org or base repo exists; `neolefty/rallly` is only a scratch space.

## What we are assessing

Candidates use an AI assistant of their choice (Claude Code, Opus or similar)
in whatever harness they like. The core signal is the ability to take a large,
under-specified problem and choose a slice worth building, then build it well.
Setup friction and raw typing speed are not signals.

## Deliverable: a pull request inside a per-candidate private repo

A PR, not a bare branch. It gives reviewers the diff view, a description
field, inline comments for the live follow-up, and CI if we enable it.

The PR must include:

- **A scope comment on the task issue before any code.** The task is posted
  as an issue in the candidate's repo, written the way the upstream threads
  are written, with the maintainer's ambiguity left in. The candidate replies
  with their proposed scope, ideally within the first day. This single comment
  is the clearest evidence of the skill above and mirrors how contributors
  actually work in the upstream repo.
- **A `HOMEWORK.md` on the branch**, roughly one page, linked from the PR
  body rather than duplicated there. Required sections:
  - What they built and, more importantly, what they cut and why.
  - How to run it from a clean clone, including the test command.
  - How they used the assistant: what they accepted, rejected, and verified
    by hand. A paragraph, not a prompt log.
  - What they would do next with another day.
  - Answers to our fixed questions (to be written per task).
- **Unsquashed commit history.** The sequence shows how they explored, and
  the AI co-author trailers show assistant usage without us asking.

Decide and state up front which gates are graded. These all exist in the
repo: `pnpm check` (Biome), `pnpm type-check`, `pnpm check:structure`
(feature-folder vocabulary and layering), `pnpm test:unit`. CLAUDE.md is
strict about layering and lint-enforces it; candidates need to know whether
a red run costs them anything.

## Privacy: one private repo per candidate, created by mirror push

Candidates must not see each other's work. The constraints that shape this:

- On a shared private repo every collaborator sees every PR, and a PR from a
  candidate's fork lands in the shared base. So the PR has to live in a repo
  only that candidate can access.
- **GitHub allows one fork per owner per upstream.** You cannot fork the base
  ten times into the same org, and there is no bulk fork. Forking is out.
- **Template repos flatten history to a single commit.** Candidates and their
  assistants rely on `git log` and `git blame` for context, so we want the
  full upstream history. Templates are out.
- **Mirror push preserves history and drops the fork link**, so GitHub will
  not advertise the candidate repo under the upstream's network graph.

So the mechanism is: create a private repo in the interview org, then
`git push --mirror` from a pinned base tag. The candidate gets write access,
branches inside that repo, and opens the PR there. When the process ends,
archive or delete the repo and remove the collaborator. The candidate keeps
their clone; their work never sits in a repo they control.

### Create on demand, not a pre-made supply

Ten pre-created repos go stale the moment the issue text or base commit
changes. A script that takes a candidate number and finishes in seconds gives
the interview manager the same "draw one when needed" experience. Sketch,
not yet written:

```
new-candidate.sh 07
  gh repo create <org>/rallly-hw-07 --private
  git push --mirror from the pinned base tag
  gh issue create from the task template
  add the candidate as a collaborator with write access
  print the repo URL and issue URL
```

### Decisions still open

- **GitHub plan of the interview org.** Free orgs have unlimited private
  repos and collaborators. Team and Enterprise charge a seat per outside
  collaborator on private repos, so ten concurrent candidates could be ten
  seats for the month. Removing them when done releases the seats.
- **Numbering vs. names.** Numbered repos allow blind grading; a spreadsheet
  maps numbers to people.
- **Actions.** Workflows copied by the mirror will try to run on the PR, and
  minutes on private repos are metered. Likely delete the heavy ones
  (Playwright integration tests, Docker image build, release) from the base
  before mirroring and keep lint, type-check and unit tests.
- **Base tag.** Push from a tag such as `hw-v1`, never a branch, so every
  candidate in a cohort starts identical and the assignment is versioned.
- **Streamlined setup.** Suggest, do not require, the Docker path. A short
  `SETUP.md` or a devcontainer in the base repo is cheap and removes setup
  from the grade.

## Environment facts a future agent will need

- **Node is pinned to 24** (`.nvmrc`, `engines`), and pnpm 12 refuses to
  install on any other major. Homebrew's unversioned `node` drifts ahead;
  use `node@24` or a version manager.
- **Dev dependencies run in Docker, the app runs natively.**
  `docker-compose.dev.yml` (via `pnpm docker:up`) starts Postgres on host
  port **5450** (not 5432), Redis plus a REST shim for rate limiting, Garage
  for S3-compatible storage, and Mailpit on 8025 for catching email. The
  full self-hosted stack in `docker-compose.yml` builds the app image and is
  not needed for development.
- **Three env files** are copied from samples in the Getting Started block
  of CLAUDE.md. `pnpm db:reset` seeds demo data.
- **Husky pre-commit runs `pnpm lint-staged`.** A broken Node kills every
  commit at the hook, which is how the Node pin above was discovered.
- **Commits use gitmoji prefixes** (see CLAUDE.md). Notes-only commits on
  this branch use `📝`.
- The upstream backlog's large asks are in GitHub Discussions, category
  Ideas, not in issues. `gh api graphql` is needed to list them; the
  `gh issue` commands do not see discussions.

## Next steps

1. Create the interview org and base repo; pick the plan.
2. Choose the first task from `interview-candidates.md` and write its issue
   text plus the fixed questions.
3. Write `HOMEWORK.md` template and `SETUP.md` into the base.
4. Tag `hw-v1`. Write `new-candidate.sh`. Dry-run with one internal person.
