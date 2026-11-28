---
name: Weekly Report Status
description: Publish a concise weekly repository activity report as a new issue.
intent: Keep maintainers informed about repository activity over the previous seven days.
on:
  schedule:
    - cron: "0 9 * * 1"
  workflow_dispatch:
permissions:
  contents: read
  issues: read
  pull-requests: read
  copilot-requests: write
engine: copilot
checkout: false
network:
  allowed: []
tools:
  github:
    mode: gh-proxy
    toolsets: [repos, issues, pull_requests]
safe-outputs:
  create-issue:
    title-prefix: "[weekly-report] "
    max: 1
---

# Weekly Report Status

Generate a concise activity report for the repository that triggered this run.
Use a UTC window covering the previous seven days, ending at the workflow run's
start time. Include the exact start and end timestamps in the report.

Use read-only GitHub queries through `gh` to collect activity in that window.
Follow pagination so totals are not limited to the first page:

- **Commits:** Count commits on the default branch and summarize notable changes,
  linking to representative commits.
- **Issues:** Summarize issues opened, closed, or updated during the window.
  Exclude pull requests from issue totals and exclude this workflow's report
  issues, identified by the `[weekly-report] ` title prefix.
- **Pull requests:** Summarize pull requests opened, updated, merged, or closed
  during the window, distinguishing merged from closed without merging.

Group the report into `### Summary`, `### Commits`, `### Issues`, and
`### Pull Requests`. Include counts and a few notable items with links; do not
dump exhaustive lists or count an item multiple times within the same category.
State "No activity occurred during this reporting period." when all three
categories are empty, and explicitly identify any empty category otherwise.
Do not mistake failed queries or unavailable data for zero activity; report
missing evidence through `report_incomplete` instead of publishing a misleading
report.

Publish exactly one new issue using the configured `create-issue` safe output,
including when no activity occurred. Use a title describing the UTC reporting
date range; the configured prefix is added automatically. Keep earlier reports
as history, and do not modify or close them. Include a link to this workflow run
for traceability. Never create issues through `gh` or direct API writes.
