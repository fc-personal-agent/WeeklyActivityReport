---
name: handwritten-pdf-to-word
description: Prompt the user for a handwritten PDF, transcribe the handwriting into typed text page by page, and produce a Word (.docx) document with the transcription. Use when the user wants a handwritten scan converted into typed/editable text.
---

# Handwritten PDF to Word

Converts a handwritten PDF into a typed Word document. This is a faithful
transcription task, not a summary — the goal is to turn handwriting into
typed text, preserving the original content and structure as closely as
possible.

## Workflow

1. **Get the PDF path.** If the user already gave a file path when invoking
   this skill, use it. Otherwise ask them for the full path to the
   handwritten PDF (a plain question is fine, or use AskUserQuestion if it
   helps). Confirm the file exists and ends in `.pdf` before continuing.

2. **Read the PDF.** Use the Read tool on the PDF. If the document has more
   than ~10 pages, Read will require a `pages` range and caps at 20 pages per
   call — read it in sequential chunks (e.g. `1-20`, `21-40`, ...) until the
   whole document is covered.

3. **Transcribe each page verbatim.** For every page, produce the typed
   equivalent of the handwritten content, preserving line/paragraph breaks
   where they carry meaning (e.g. list items, separate thoughts). Do not
   summarize, rephrase, or reorganize — this is a transcription, not an
   executive summary.
   - Mark a word or short phrase you're not fully sure of as
     `[unclear: best guess]`.
   - Mark a span you cannot read at all as `[illegible]`.
   - Keep a running list of pages/spots that had unclear or illegible text,
     for the notes section.
   - Never invent content that isn't legible or supported by the page.

4. **Build the JSON payload** describing the transcription:
   ```json
   {
     "title": "Transcription of <source filename>",
     "source_pdf": "<absolute path to the source PDF>",
     "pages": [
       {"page_number": 1, "paragraphs": ["...", "..."]}
     ],
     "notes": ["Page 2: 'amount' guessed from context, ink smudged"]
   }
   ```
   Write it to a temp file in the scratchpad directory.

5. **Generate the Word document** by running the helper script:
   ```
   python "<repo>/.claude/skills/handwritten-pdf-to-word/scripts/build_docx.py" <input.json> <output.docx>
   ```
   Save the output `.docx` next to the source PDF (same name, `.docx`
   extension) unless the user asked for a different location.

6. **Report back**: the path to the generated `.docx`, and a short summary of
   any `[unclear]`/`[illegible]` items so the user knows what to double-check
   against the original scan.

## Guardrails

- Never guess at content and present it as certain — flag it instead.
- Don't summarize or condense; this skill produces a typed transcript, not a
  digest. If the user actually wants a summary instead of a transcription,
  confirm that with them before proceeding.
- If a page is entirely illegible, say so explicitly in that page's section
  rather than leaving it blank with no explanation.
