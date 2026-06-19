# Workflows

Reusable workflows for the `nova-experiments` org. Module repos opt in via starter workflow templates surfaced in the GitHub Actions tab — no copy-pasting required.

---

## What module repos should set up

Every module repo should add both caller workflows from the org starter templates:

| Starter template | What it does |
|---|---|
| `call-post-issue-map-fields` | On `issues: opened` — adds the new issue to the roadmap project and sets Team and Type fields. |
| `call-pr-parent-check` | On PR open/edit/push — warns if any linked issue has no parent in the Roadmap repo, or if the parent is unscheduled or in the Parking Lot. |

To add them: go to **Actions → New workflow** in the repo and select the templates from the `nova-experiments` org section.

The Roadmap repo should also add `call-post-issue-map-fields` so roadmap-level issues are mapped to the project on creation.

---

## Workflows

### `post-issue-map-fields.yml`

Called by `call-post-issue-map-fields` in each repo on `issues: opened`.

Adds the new issue to the org roadmap project and sets the **Team** and **Type** single-select project fields from the issue form body and the issue's type frontmatter. Silently skips any field that has no value or no matching option.

**Requires:** `ORG_PROJECT_TOKEN` secret · `ROADMAP_PROJECT_NUMBER` org variable

---

### `pr-parent-check.yml`

Called by `call-pr-parent-check` in each repo on PR open, edit, and push.

Reads every issue linked in the PR body (`closes #n`, `fixes #n`, `resolves #n`, `part of #n`). For each linked issue, checks that:

1. A native sub-issue parent is set on the issue.
2. The parent lives in the `Roadmap` repo.
3. The parent is assigned to a scheduled project (not the Parking Lot, not unassigned).

Posts a single warning comment on the PR if any check fails; updates the comment in place on subsequent pushes.

**Requires:** `ORG_PROJECT_TOKEN` secret · `ROADMAP_PROJECT_NUMBER` org variable · `PARKING_LOT_PROJECT_NUMBER` org variable

> **Note:** Project membership is fetched with `items(first: 100)`. If either project exceeds 100 items, add cursor-based pagination to `getProjectIssueIds`.

---

### `roadmap-sweeper.yml`

Runs on a schedule (daily at 6 AM UTC) and on manual dispatch. Lives in the `.github` repo — no caller needed elsewhere.

1. **Misplaced issue detection:** searches all org repos for open issues with roadmap-type labels (`feature-set`, `feature`, `design`, `major-task`, `other-work`) that are not in the Roadmap repo, and transfers them there.
2. **Parking Lot backfill:** finds open Roadmap issues not assigned to any project and adds them to the Parking Lot, posting a one-time comment on the issue to notify the author.

**Requires:** `ORG_PROJECT_TOKEN` secret · `ROADMAP_PROJECT_NUMBER` org variable · `PARKING_LOT_PROJECT_NUMBER` org variable

> **Note:** Project membership is fetched with `items(first: 100)`. Add pagination if the roadmap project exceeds 100 items.