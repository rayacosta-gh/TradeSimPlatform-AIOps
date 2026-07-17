# Claude Code Task: GitHub Setup for TradeSim Compliance Platform

Scope: this task sets up repo tracking infrastructure only (labels, milestones, project board, custom field). It does not write application code and does not file individual issues yet, that's a separate follow-up task once this scaffolding exists.

## Prerequisites

Confirm before starting:
- `gh` CLI is authenticated (`gh auth status`)
- Current directory is the TradeSim repo, or repo is specified explicitly with `--repo owner/name` on each command
- Repo is public (matches OrderFlow convention)

## Step 1: Enable Issues

Confirm Issues is enabled under repo Settings > General > Features. This is on by default for new repos, but verify:

```bash
gh repo edit --enable-issues
```

## Step 2: Create Labels

Create exactly these labels. Use `gh label create`, and check with `gh label list` first to avoid duplicate-creation errors on any that already exist (new repos ship with some defaults like `bug` and `documentation`, decide whether to delete those or leave them alongside the new taxonomy, defaulting to leaving them alongside is fine).

```bash
gh label create "type: feature" --color "0E8A16" --description "New functionality"
gh label create "type: bug" --color "D73A4A" --description "Something broken"
gh label create "type: chore" --color "FBCA04" --description "Maintenance, tooling, non-feature work"
gh label create "type: docs" --color "0075CA" --description "Documentation"

gh label create "area: auth" --color "5319E7" --description "Authentication and roles"
gh label create "area: trading" --color "1D76DB" --description "Trade execution, positions, delegation"
gh label create "area: compliance" --color "B60205" --description "Flags, incidents, notes, access grants"
gh label create "area: aiops" --color "0E8A16" --description "LLM intake and observability"
gh label create "area: infra" --color "C5DEF5" --description "Terraform, CI/CD, cloud deployment"
```

Colors above are suggestions, keep them distinct enough to scan visually on the issues list. Adjust if they clash with repo defaults already in place.

Do not create `milestone-0` through `milestone-8` labels. Native GitHub Milestones are used instead (see Step 3). If native Milestones for some reason cannot be used, fall back to creating these as labels and flag that decision back to Ray rather than silently deciding.

## Step 3: Create Milestones

Use `gh api` since `gh` has no native `milestone create` subcommand. Repeat for each milestone below, POSTing to the issues/milestones endpoint.

```bash
gh api repos/:owner/:repo/milestones -f title="Milestone 0: Scoping and Setup" -f description="Finalize roles/permissions matrix. Design full data model including delegation scaffolding (TradingEntity, TradingAuthorization). Confirm market data source and rate limits. Set up GitHub Issues, Projects, labels, and milestones. Decide backend language/framework. Confirm GCP account setup."

gh api repos/:owner/:repo/milestones -f title="Milestone 1: Auth and Roles" -f description="User registration and login. Three role types enforced at the API layer. Administrator can create/manage users and assign roles. Every new user gets a corresponding individual-type TradingEntity at registration. No trading functionality yet."

gh api repos/:owner/:repo/milestones -f title="Milestone 2: Trader Experience, Read-Only Market Data" -f description="Trader dashboard pulls live public market data from the selected free API. No trade execution yet. Validates the market data pipeline and trader-facing UI."

gh api repos/:owner/:repo/milestones -f title="Milestone 3: Simulated Trade Execution" -f description="Trader can place simulated trades against current public market price. Trade status lifecycle implemented (submitted, filled, cancelled, rejected); each transition emits a distinct AuditEvent. Trade and position data persisted, position computed on read. Establishes the audit-event pattern for later milestones."

gh api repos/:owner/:repo/milestones -f title="Milestone 4: Compliance and Regulatory Dashboard" -f description="Compliance analyst can query trade history filtered by user, date, symbol. Anomaly detection creates TradeFlag records. ComplianceIncident case management: open incidents, link trades (IncidentTrade) and link flags (IncidentTradeFlag, many-to-many). ComplianceNote support, never visible to traders, enforced at API layer. AccessGrant mechanism for temporary, view-level, time-limited admin access, logged to AccessGrantAuditLog."

gh api repos/:owner/:repo/milestones -f title="Milestone 5: AIOps Layer" -f description="Trader-facing free-text prompt for feature requests and bug reports, classified automatically and drafted into GitHub Issues via the API, tagged Filed via: AI-intake. Every LLM interaction emits an LLMInteractionEvent. Compliance dashboard surfaces LLM accuracy/error rate, hallucination rate proxy, intake volume."

gh api repos/:owner/:repo/milestones -f title="Milestone 6: Backend Regulatory Reporting Service (Stretch)" -f description="Separate service periodically packages flagged trade activity, incident summaries, and LLM metrics into a report artifact simulating a regulator-facing report. Reuses the event-driven audit pattern."

gh api repos/:owner/:repo/milestones -f title="Milestone 7: Infrastructure, Observability, and Cloud Deployment" -f description="Terraform provisioning on GCP (Cloud Run or GKE). Metrics via Cloud Monitoring or self-hosted Prometheus/Grafana. CI/CD via GitHub Actions targeting GCP. Security scanning integrated into the pipeline."

gh api repos/:owner/:repo/milestones -f title="Milestone 8: Delegated Trading (Stretch)" -f description="Full delegation logic on top of Milestone 0/1 schema scaffolding: users trading on behalf of a desk or group via TradingAuthorization, permission checks for delegated execution, position and audit attribution split between executed_by and trading_entity_id."
```

Verify all nine were created:

```bash
gh api repos/:owner/:repo/milestones --jq '.[].title'
```

## Step 4: Create the Project Board

```bash
gh project create --owner :owner --title "TradeSim Compliance Platform"
```

Note the project number returned, it's needed for Step 5. Use the Board/Kanban template if prompted (columns: Backlog, In Progress, In Review, Done). If `gh project create` doesn't support template selection directly, create the project and then set up columns/views manually via the returned project URL, and note back to Ray that manual setup was needed.

## Step 5: Add the "Filed via" Custom Field

```bash
gh project field-create <PROJECT_NUMBER> --owner :owner --name "Filed via" --data-type SINGLE_SELECT --single-select-options "manual,AI-intake"
```

Replace `<PROJECT_NUMBER>` with the number from Step 4.

## Step 6: Add the Wiki Page

The companion file `TradeSim-Wiki-Home.md` should become the repo wiki's Home page. GitHub wikis are themselves git repos, cloned separately:

```bash
gh repo clone :owner/:repo.wiki wiki-tmp
cp TradeSim-Wiki-Home.md wiki-tmp/Home.md
cd wiki-tmp
git add Home.md
git commit -m "Add TradeSim data model, roles matrix, and roadmap"
git push
cd ..
rm -rf wiki-tmp
```

Note: the wiki repo only exists once at least one wiki page has been created through the GitHub UI at least once. If `gh repo clone :owner/:repo.wiki` fails with a not-found error, visit the repo's Wiki tab in the browser first, click "Create the first page" with any placeholder content to initialize it, then retry the clone.

## Step 7: Confirm and Report Back

After running the above, report back to Ray:
- Confirmation of labels created (and any that already existed and were skipped)
- Confirmation of all 9 milestones created, with their GitHub-assigned numbers
- Project board URL and confirmation the "Filed via" field was added
- Whether the wiki push succeeded or needed manual initialization first

Do not proceed to filing individual Milestone 0 issues in this task. That's a distinct follow-up once this scaffolding is confirmed working.
