---
name: cr
description: Generate copy-paste-ready content for a Jira Change Request (CR) from a Jira ticket link and a GitHub PR link. Produces a one-line summary, a description (PR link first, then bullets), business justification bullets, and impact assessment bullets. Use whenever the user invokes /cr, or says they need a change request, CR, CR Jira, or CR content for a PR or ticket, even if they only paste a Jira link and a PR link with no other instructions.
---

# CR: Change Request content generator

Turn a Jira ticket and a GitHub PR into the four fields needed to raise a Change Request in Jira, formatted so the user can copy and paste each one directly.

## Inputs

The user provides:
- A Jira link (or issue key) for the work item
- A GitHub PR link

If either is missing, ask for it in one short line. Do not proceed with guesses. If the user gives several PRs or tickets, produce one combined CR and list every PR link at the top of the description.

## Gathering context

Read the sources before writing anything. Never write from the links alone.

1. Jira ticket: use the Atlassian/Jira MCP tools if available (load them with tool_search if deferred), otherwise fall back to acli or ask the user to paste the ticket text. Pull the summary, description, acceptance criteria, linked issues, and any comments that explain the "why".
2. PR: use gh pr view <url> --json title,body,files,additions,deletions,baseRefName,headRefName,labels,reviews and gh pr diff <url>. If gh is unavailable, use the GitHub MCP if present, or ask the user to paste the PR description and diff summary.
3. Read the diff for real. Note which services, modules, configs, CRDs, database migrations, toggles, dependencies, and infrastructure resources are touched. Impact claims must come from the diff, not from the PR title.

If the tickets or PR are in the tyro-private GitHub organisation, that is expected. Do not pull code from public sources.

## Output format

Output exactly these four sections, in this order, each in its own clearly separated block so each can be copied independently. Use plain Jira-friendly text: simple "-" bullets, no tables, no nested bullets deeper than one level, no headers inside a field. Do not use em dashes anywhere. Use commas, colons, or parentheses instead.

### 1. Summary
One line, under about 100 characters, in plain language. Lead with the change, not the ticket key. Example style: Add retry with backoff to settlement webhook publisher

### 2. Description
- First line: the PR link by itself (and the Jira key on the next line if helpful).
- Then a blank line, then bullets covering:
  - What is changing, in plain terms
  - Which components, services, or resources are touched
  - Key technical details a reviewer or approver should know (new config, toggles, migrations, new dependencies, new AWS resources)
  - How it was tested (unit tests, integration tests, verification in development and staging, if the PR or ticket says so)
  - Rollout and rollback approach (see below)
- 5 to 8 bullets. Each bullet is one sentence.

### 3. Business justification
Bullets explaining why the change is needed, drawn from the Jira ticket. Cover whichever apply: customer or merchant problem solved, incident or defect addressed, security or compliance driver (CVE, PCI, audit), cost or performance benefit, risk of not doing it, commercial or regulatory deadline. 3 to 5 bullets. Do not invent business drivers. If the ticket does not state one, state the technical driver honestly (for example "dependency upgrade to remediate CVE-XXXX-YYYY") rather than padding.

### 4. Impact assessment
Bullets covering:
- Systems and services affected, including downstream consumers
- Customer or merchant impact, including expected downtime (state "no downtime expected" only if the diff supports it)
- Data impact: schema changes, migrations, PII or PCI data handling touched
- Security impact: auth, secrets, permissions, new external calls
- Risk level (Low, Medium, High) with a one-line reason
- Rollback plan: how to revert (revert PR and redeploy via Drydock promotion, toggle off, etc.)
- Dependencies or sequencing: other changes that must go first, migrations before app deploy, etc.

3 to 7 bullets, combining the above where it keeps things tight.

## Rollout wording

When describing deployment, align with the change management process: changes go via reviewed PRs, are deployed and verified in order development, staging, then production, and production deployment needs an approved Change Request. Only state what the change actually involves. If the PR touches Schooner, Tide, or Slipway definitions, or Jetstream AWS resources, say so. If the PR does not make rollout details clear, say "Standard promotion: development, staging, production" rather than inventing specifics. If the user asks about the official process or who must approve, use the technology-foundations:change-management skill, which reads the current source of truth from Confluence.

## Quality rules

- Be factual and specific. Name the real services, files, or resources from the diff.
- Flag anything uncertain in a short "Needs your confirmation" line after the four sections (for example, test evidence not found, rollback unclear, risk level is a judgement call). Keep it to 1 to 3 items, and omit it if there is nothing to flag.
- Do not include secrets, tokens, or customer data that happen to appear in the diff or ticket. If hardcoded secrets or PII exposure are spotted in the diff, call that out in the "Needs your confirmation" line.
- Keep the tone neutral and professional, suitable for approvers who may not know the code.
- Do not add commentary, preamble, or a recap. Output the four sections, plus the optional confirmation line, and stop.
- Never perform any action against production environments. This skill only reads and writes text.

## Example output shape

Summary:
Upgrade Spring Boot to 3.3.5 to remediate CVE-2025-XXXXX in payments-api

Description:
https://github.com/tyro-private/payments-api/pull/123
Jira: PAY-456

- Upgrades Spring Boot BOM from 3.2.4 to 3.3.5 and aligns dependent Spring libraries
- Regenerates the Gradle lock file
- No application code changes beyond a deprecated API replacement in WebClientConfig
- Unit and integration tests pass in CI; verified in development
- Rollout via standard promotion: development, staging, production
- Rollback by reverting the PR and promoting the previous image with Drydock

Business justification:
- Remediates a high severity CVE flagged in the latest dependency scan
- Keeps payments-api compliant with internal vulnerability remediation SLAs
- Reduces risk of exploit in a PCI-scoped service

Impact assessment:
- Affects payments-api only; no API contract changes for downstream consumers
- No downtime expected; rolling deployment
- No data or schema changes
- No change to authentication or secrets handling
- Risk: Low, framework minor upgrade with full regression coverage
- Rollback: revert PR and redeploy previous image via Drydock
