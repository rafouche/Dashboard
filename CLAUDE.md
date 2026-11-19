# Dashboard repo notes for Claude

## Merging

Always merge finished work into `main` for this repo without waiting to be asked
(standing instruction from the repo owner, 2026-11-19). `main` is what gets
downloaded and deployed, so a change left on a feature branch is not delivered.

Workflow: commit on the session branch, push, open a pull request against
`main`, and squash-merge it.

## Versioning

- Bump `APP_VERSION` in `dashboard.html` on every meaningful change so the
  version badge confirms which build is running.
- Start the commit title and the PR title with the version, e.g.
  `v1.9: auto-hide Show/Sort chip rows`. GitHub shows that title next to
  `dashboard.html` in the file list, and the owner uses it to confirm which
  version is on `main` before downloading. Squash-merge so that title is kept.
