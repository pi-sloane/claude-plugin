---
name: ingest-markdown-report
description: Submit a security report to Pi as inline Markdown with a .md or .markdown filename, after explicit user confirmation. Reports the queued ingestion workflow state truthfully.
---

# Submit a Markdown security report to Pi

Use this skill only when the user explicitly asks to submit or ingest a security report into Pi Security, and the report content is available inline as Markdown.

## Tool

State-changing, confirmation required:

- `pi_report_markdown_upload` - uploads Markdown report content and starts the tenant report-ingestion workflow. Inputs: `fileName` (must end in `.md` or `.markdown`) and `content` (non-empty Markdown, up to 5 MiB).

## Accepted input

Report content provided inline in the conversation as Markdown, plus a filename ending in `.md` or `.markdown`. If either is missing, ask for it instead of inventing one.

## Rejected inputs

- Chat attachments, local file paths, and `file://` URIs
- PDFs, DOCX files, images, or any binary content

Those workflows belong in the Pi web app or the Sloane CLI, not this skill.

## Before calling the tool

1. Validate: `fileName` ends in `.md` or `.markdown`, and `content` is non-empty Markdown within the size limit.
2. Summarize what will be submitted - filename, approximate size, and a one-line description of the report's scope. Do not echo sensitive report content back into the conversation beyond what is needed to confirm the target.
3. Explain the effect: Pi will store the report and start its ingestion workflow in the authenticated tenant.
4. Ask for confirmation and proceed only on an explicit yes in the current turn.
5. Call the tool once.
6. Report exactly the upload and workflow state Pi returned. Queued or started means ingestion is in progress, not that findings exist yet. Never invent ingestion results.
7. If the call fails or times out ambiguously, report that and stop. Do not retry automatically, since a retry could submit a duplicate report.
