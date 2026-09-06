---
name: handwritten-pdf-executive-summary
description: Turn a handwritten PDF (or an existing verbatim transcription of one) into a polished executive-summary Word document, with key findings, follow-up items, and unclear-handwriting flags called out separately. Use when the user wants a management/stakeholder-ready summary rather than a raw transcription.
---

# Handwritten PDF Executive Summary

Produces a concise, executive-friendly Word document from a handwritten PDF.
This is a summary/interpretation task, not a transcription — condense,
prioritize, and organize, but never invent facts that aren't in the source.

This skill is commonly run right after [[handwritten-pdf-to-word]] on the
same document, reusing the transcription instead of re-reading the PDF from
scratch, but it can also be run standalone directly against a PDF.

## Workflow

1. **Get the source content.**
   - If a verbatim transcription of this document was already produced in
     this conversation (e.g. by the `handwritten-pdf-to-word` skill), reuse
     that text instead of re-reading the PDF.
   - Otherwise, ask the user for the handwritten PDF path and read it with
     the Read tool, chunking through `pages` if it's long (same approach as
     `handwritten-pdf-to-word`).

2. **Extract and organize**, conservatively:
   - A brief (3-6 sentence) summary of what the document covers.
   - Key findings, decisions, and action items — as short bullet points.
   - Follow-up items or open questions raised by the content.
   - Anything that needs human confirmation because the handwriting was
     unclear or illegible in the source (pull these from `[unclear]` /
     `[illegible]` markers if a prior transcription exists, or note them
     directly if reading the PDF fresh).
   - Never invent information not legible or supported by the document.
     Prefer conservative interpretation over risky assumptions.

3. **Build the JSON payload**:
   ```json
   {
     "title": "Executive Summary of <source filename>",
     "source": "<absolute path to the source PDF or transcription>",
     "summary": "...",
     "key_findings": ["...", "..."],
     "follow_up_items": ["...", "..."],
     "confirmation_needed": ["...", "..."]
   }
   ```
   Write it to a temp file in the scratchpad directory.

4. **Generate the Word document**:
   ```
   python "<repo>/.claude/skills/handwritten-pdf-executive-summary/scripts/build_summary_docx.py" <input.json> <output.docx>
   ```
   Save the output as `<source basename> - Executive Summary.docx` next to
   the source file unless the user asked for a different location.

5. **Report back** the path to the generated `.docx` and a one-line note on
   whether any items need human confirmation due to unclear handwriting.

## Guardrails

- Conservative over confident: if in doubt, flag it in
  "Areas Requiring Human Confirmation" rather than asserting it as fact.
- Keep the tone professional, clear, and concise — this is for executive or
  stakeholder review, not a full transcript.
- Don't overload the document with detail that belongs in a full
  transcription instead — that's what `handwritten-pdf-to-word` is for.
