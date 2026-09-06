---
name: weekly-report
description: Turn a handwritten PDF (or an existing verbatim transcription / executive summary of one) into a polished, submission-ready "Weekly SRE Update" Word document with a date-range title, summary, key findings & actions, follow-up items, and room for additional hand-written notes. Use when the user wants the final weekly status report they'll actually send/share, as the last step after transcription and executive summary.
---

# Weekly Report

Produces the final, shareable weekly status report Word document from a
handwritten PDF. This is the last step of the handwritten-pdf-workflow: it
reuses the content already extracted by [[handwritten-pdf-to-word]] and/or
[[handwritten-pdf-executive-summary]] rather than re-reading the source PDF,
and formats it into the "Weekly SRE Update" layout used for the actual
submitted/shared report.

## Workflow

1. **Get the source content.**
   - Reuse the transcription and/or executive summary already produced in
     this conversation for this document — do not re-read the PDF from
     scratch if that content is available.
   - If neither exists yet, get verbatim content first (ask for the PDF path
     and/or run `handwritten-pdf-to-word`) so this skill has real material to
     draw from. It does not invent findings on its own.

2. **Determine the reporting period and title.** Work out the date range
   covered by the source notes (the first and last dated entries) and the
   project/client name (e.g. "AssetMark") from context or the source
   filename. Build a title in this form:
   `Weekly SRE Update - <Project> - <Month> <StartDay>-<EndDay>, <Year>`
   (e.g. `Weekly SRE Update - AssetMark - August 24-28, 2026`). Ask the user
   if the project name isn't obvious from context.

3. **Extract and organize**, conservatively — same standard as the
   executive-summary skill:
   - A brief (3-6 sentence) summary of the week.
   - Key Findings & Actions — short bullet points covering what was done,
     decided, or blocked.
   - Follow-up Items / Open Questions — outstanding items needing action or
     confirmation.
   - Additional Notes — other noteworthy personal observations or
     accomplishments that don't fit the sections above (e.g. a new skill
     picked up, a process followed for the first time). Fine to leave this
     list short or empty — the generated document reserves blank lines here
     regardless, for the user to jot more by hand.
   - Never invent information not legible or supported by the source.

4. **Build the JSON payload**:
   ```json
   {
     "title": "Weekly SRE Update - AssetMark - August 24-28, 2026",
     "source": "<absolute path to the source PDF or transcription>",
     "summary": "...",
     "key_findings": ["...", "..."],
     "follow_up_items": ["...", "..."],
     "additional_notes": ["...", "..."]
   }
   ```
   Write it to a temp file in the scratchpad directory.

5. **Generate the Word document**:
   ```
   python "<repo>/.claude/skills/weekly-report/scripts/build_weekly_report_docx.py" <input.json> <output.docx>
   ```
   The script appends a handful of blank bulleted lines at the end of
   "Follow-up Items / Open Questions" and "Additional Notes" so the user has
   room to add more, directly in Word, before sending the report out.
   Save the output as `<source basename> - WeeklyReport.docx` next to the
   source file unless the user asked for a different location.

6. **Report back** the path to the generated `.docx`.

## Guardrails

- This is the final, externally-shareable deliverable in the workflow — keep
  it polished and professional, matching the tone of a status report sent to
  a manager or stakeholder, not a raw transcript or an internal summary.
- Don't fabricate a project name, date range, or content the source doesn't
  support — ask rather than guess if something is missing.
- Run this only after (or by reusing the output of) `handwritten-pdf-to-word`
  — and typically `handwritten-pdf-executive-summary` too — since both feed
  it real content; it isn't meant to run standalone against a bare PDF path
  without that groundwork.
