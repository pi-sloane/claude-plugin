---
name: security-design-review
description: Start a Pi Security design review from a supported Confluence or Notion document URL or from inline Markdown, after explicit user confirmation. No attachments, local files, or binaries.
---

# Start a Pi design review

Use this skill only when the user explicitly asks to start a Pi Security design review of a design document.

## Tools

Both are state-changing and require confirmation:

- `pi_design_review_url_create` - starts a design review for a supported Confluence or Notion document URL (`url`, http/https only).
- `pi_design_review_markdown_create` - starts a design review from inline Markdown design-document content (`content`, up to 5 MiB).

## Accepted inputs

- A Confluence or Notion document URL, or
- Design-document content provided inline as Markdown or plain text in the conversation.

## Rejected inputs

Do not accept, convert, or promote any of the following into a design-review call:

- Local file paths or `file://` URIs
- Chat attachments
- PDFs, DOCX files, images, or any binary content
- URLs other than supported Confluence or Notion document URLs

If the user offers one of these, explain that this workflow takes a supported Confluence/Notion URL or inline Markdown, and that other document types are handled in the Pi web app.

## Before calling either tool

1. Validate the input: a well-formed https Confluence/Notion document URL, or non-empty inline Markdown.
2. Show the user exactly what will be submitted: the URL, or the document title plus approximate length for inline Markdown.
3. Explain the effect: Pi will start a design review of this document in the authenticated tenant.
4. Ask for confirmation and proceed only on an explicit yes in the current turn.
5. Call the tool once.
6. Report exactly what Pi returned: if the review started, give its status and review ID; otherwise report the returned status as-is. Started or queued means the review is in progress, not finished. Never invent results or completion.
7. If the call fails or times out ambiguously, report that and stop. Do not retry automatically, since a retry could start a duplicate review.
