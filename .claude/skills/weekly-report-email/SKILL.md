---
name: weekly-report-email
description: Send the content of an already-generated Weekly SRE Update report as a plain email body (never as an attachment) to the configured recipient, using the report's title as the subject. Use as the final step after the weekly-report skill, once the user has confirmed they've reviewed the document.
---

# Weekly Report Email

Sends the already-generated Weekly SRE Update report as an email — pasting
its content directly into the email body, not as an attachment — using the
report's title as the subject line. This is the final step of the
handwritten-pdf-workflow, run only after the user has reviewed the Weekly
Report Word document produced by [[weekly-report]].

## Workflow

1. **Confirm the document has been reviewed.** Before doing anything else,
   ask the user (e.g. via AskUserQuestion) whether they have reviewed the
   generated Weekly Report document. If they haven't yet, stop here and wait
   — do not send anything until they explicitly confirm it's been reviewed.

2. **Get the report content.** Reuse the title, summary, key findings,
   follow-up items, and additional notes already produced in this
   conversation for the Weekly Report (the same content passed to
   `weekly-report`'s JSON payload) — do not regenerate, reinterpret, or
   re-read the source PDF. If that content isn't available in this
   conversation (e.g. this skill is being run standalone), ask the user for
   the path to the generated `.docx` and read it to reconstruct the
   sections instead of guessing.

3. **Resolve the recipient from the environment variable.** The address to
   send to is stored in the `WEEKLY_REPORT_RECIPIENT` environment variable —
   never hardcode an email address in this file or any script. It's defined
   in this project's `.claude/settings.json` under the `env` key. Read it at
   runtime, e.g. `$env:WEEKLY_REPORT_RECIPIENT` (PowerShell) or
   `echo "$WEEKLY_REPORT_RECIPIENT"` (Bash). If it's unset or empty, ask the
   user for the recipient address directly rather than guessing one.

4. **Compose and send the email** using the Gmail send tool available in
   this session:
   - `to`: the resolved recipient address.
   - `subject`: the Weekly Report's title, verbatim (e.g.
     `Weekly SRE Update - AssetMark - August 31-September 4, 2026`).
   - `body`: the report content as plain text, using the same section
     headings and order as the document — Summary, Key Findings & Actions,
     Follow-up Items / Open Questions, Additional Notes — each heading
     followed by its paragraph or bullet items.
   - Do not attach the `.docx` file. The content goes directly in the
     message body; this is a deliberate, user-specified requirement, not an
     oversight.

5. **Report back**: confirm the email was sent, to whom, and the subject
   line used.

## Guardrails

- Never send without an explicit "yes, I've reviewed it" from the user in
  step 1 — this is the last, externally-visible action in the workflow and
  must not fire automatically just because the document was generated.
- Never hardcode the recipient address — always resolve it from
  `WEEKLY_REPORT_RECIPIENT` at runtime, falling back to asking the user if
  it isn't set.
- Send the actual report content in the body, not a summary of a summary,
  and never as an attachment — the recipient should be able to read the full
  report without opening anything.
- Don't regenerate or reinterpret the findings — reuse exactly what
  `weekly-report` already produced so the emailed content matches the
  reviewed document word for word.
