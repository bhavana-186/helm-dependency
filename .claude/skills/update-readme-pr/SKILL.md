---
name: update-readme-pr
description: 'Triggers the update-readme-pr GitHub Actions workflow with a given piece of text; the workflow appends
  that text to README.md on a new branch and opens a draft PR against main. Use when the user gives a message/text and
  wants it added to the README via an automated PR rather than a direct edit.'
argument-hint: 'the text to add to README.md'
---

# update-readme-pr

Dispatches `.github/workflows/update-readme-pr.yaml` (workflow_dispatch, input `text`). The workflow itself - not this
skill - does the work: checks out `main`, appends the text to `README.md` on a new branch, and opens a **draft** PR
back to `main` using the run's built-in `GITHUB_TOKEN`.

## Procedure

1. Get the text to add from the user if it wasn't given.
2. Dispatch the workflow:
   ```bash
   gh workflow run update-readme-pr.yaml -f text="<the text>"
   ```
3. Find the run that was just queued (dispatch doesn't return a run id directly):
   ```bash
   gh run list --workflow=update-readme-pr.yaml --limit 1
   ```
4. Wait for it to finish and show the log if it fails:
   ```bash
   gh run watch <run-id> --exit-status
   ```
5. Report the PR it opened:
   ```bash
   gh pr list --search "update-readme-pr-bot"
   ```
   or, since branches are named `update-readme-<run-number>`, `gh pr view update-readme-<run-number>` for the exact run.
   Give the user the PR URL. Do not mark it ready for review or merge it - that's a human decision.

## Notes

- The workflow passes the input through an `env:` var before writing it to the file, specifically to avoid
  shell-injection from special characters in the text (never interpolate `${{ inputs.* }}` directly into a `run:` line).
- Each dispatch creates its own branch (`update-readme-<run number>`), so repeated runs don't collide.
- This only ever touches `README.md`. It won't work until the workflow file is pushed to `main` (GitHub only lists/
  dispatches `workflow_dispatch` workflows that exist on the default branch).

## Don't

- Don't call `gh workflow run` for anything other than this one workflow from this skill.
- Don't merge or approve the PR it opens; that's for the user to do after reviewing the diff.
