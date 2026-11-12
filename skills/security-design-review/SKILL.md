---
name: security-design-review
description: Start a Pi Security design review from a supported Confluence or Notion document URL or from inline Markdown, read its gaps, and record a verdict on each gap, every write after explicit user confirmation. No attachments, local files, or binaries.
---

# Pi design reviews

Use this skill only when the user explicitly asks to start a Pi Security design review of a design document, or to read or decide the gaps of an existing review.

## Tools

Read-only:

- `pi_design_reviews_list` - lists design reviews visible to the user.
- `pi_design_review_get` - gets one review by its full UUID. A completed review lists its open and accepted gaps in `result.gaps` (each with `id`, `severity`, `title`, `mitigation`, and `status`) and its accepted-risk and false-positive gaps, with their `reason`, in `result.dismissedGaps`.

State-changing, require confirmation:

- `pi_design_review_url_create` - starts a design review for a supported Confluence or Notion document URL (`url`, http/https only).
- `pi_design_review_markdown_create` - starts a design review from inline Markdown design-document content (`content`, up to 5 MiB).
- `pi_design_review_gap_update_status` - records a verdict on one gap (`reviewId`, `gapId`, `status`, optional `reason`).

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

## Before starting a review

1. Validate the input: a well-formed https Confluence/Notion document URL, or non-empty inline Markdown.
2. Show the user exactly what will be submitted: the URL, or the document title plus approximate length for inline Markdown.
3. Explain the effect: Pi will start a design review of this document in the authenticated tenant.
4. Ask for confirmation and proceed only on an explicit yes in the current turn.
5. Call the tool once.
6. Report exactly what Pi returned: if the review started, give its status and review ID; otherwise report the returned status as-is. Started or queued means the review is in progress, not finished. Never invent results or completion.
7. If the call fails or times out ambiguously, report that and stop. Do not retry automatically, since a retry could start a duplicate review.

## Recording gap verdicts

Gap `status` values:

- `open` - no verdict yet. Setting `open` reopens a gap that has a verdict.
- `accepted` - accept Pi's proposed fix for the gap.
- `accepted_risk` - keep the design as it is and accept the risk, with an optional `reason`.
- `false_positive` - the gap does not apply, with an optional `reason`.

A gap with a verdict cannot move straight to another verdict: reopen it first, then record the new one. Only users whose Pi role can edit design reviews can record verdicts.

`pi_design_review_get` returns the gap's `mitigation`, not the exact document edit that `accepted` accepts. If the user asks to see that edit, say it is shown in the Pi web app.

1. Get the review with `pi_design_review_get` and use gap IDs only from its result. Never guess or construct a gap ID.
2. Show the user each verdict you will record: review title, gap title and severity, current status, new status, and the reason if any.
3. Ask for confirmation and proceed only on an explicit yes in the current turn. One yes can cover the list you showed, and nothing beyond it.
4. Call `pi_design_review_gap_update_status` once per gap.
5. Report exactly what Pi returned for each gap. If Pi rejects a change, report its message as-is.
6. If a call fails or times out ambiguously, report that and stop. Call `pi_design_review_get` to check the gap's status instead of retrying.
