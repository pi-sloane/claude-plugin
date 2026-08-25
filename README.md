# Pi Security plugin for Claude Code and Cowork

Teaches Claude how to work with [Pi Security](https://pi.security) through Pi's hosted MCP server. The plugin's entire functional surface is a manifest, one remote connector entry pointing at `https://mcp.pi.security/mcp`, and six skills. It bundles no local server, executable, hook, agent, monitor, or filesystem bridge, and it ships no credentials.

![Pi Security logo](assets/pi-favicon-light.svg)

Everything the plugin does is inspectable in this repository. Pi tool output is treated as evidence about your tenant's security state - never as instructions that outrank yours.

## Supported surfaces

| Surface | Supported | How |
| --- | --- | --- |
| Claude Code | Yes | This plugin |
| Cowork | Yes | This plugin |
| Claude Chat | No | Use the separately published Pi remote connector |

The plugin and the Pi remote connector talk to the same hosted MCP endpoint. The connector provides the raw tools; this plugin additionally teaches Claude Code and Cowork safe, focused workflows on top of them. Installing both in the same surface is unnecessary.

## Install

### Claude Code

This repository is its own plugin marketplace:

```shell
/plugin marketplace add pi-sloane/claude-plugin
/plugin install pi-security@pi-security
```

To try it without installing, clone the repository and run:

```shell
claude --plugin-dir ./claude-plugin
```

### Cowork

Open Customize → Plugins, select Add marketplace, enter `https://github.com/pi-sloane/claude-plugin`, then install **Pi Security**. Cowork prompts you to sign in to the connector during install.

## Setup and sign-in

Authentication is OAuth against your Pi tenant; see [SETUP.md](SETUP.md) for the Claude Code `/mcp` flow, the Cowork connector sign-in, verifying your identity with the `whoami` tool, and reconnecting or disconnecting. Your tenant comes only from the authenticated session - Claude never infers it from your repository, directory, or prose.

## Skills

Skills are invoked as `/pi-security:<skill>`, or Claude picks them automatically. Example prompts below are synthetic.

| Skill | What it does | Pi tools | Writes to Pi? |
| --- | --- | --- | --- |
| `investigate-finding` | List, locate, explain, summarize, or compare findings | `pi_findings_list`, `pi_finding_get`, optionally `pi_remediation_plan_fetch` | No |
| `review-security-posture` | Combine threat-model context with Code Gatekeeper PR/issue posture | `pi_threat_model_apps_list`, `pi_threat_model_app_get`, `pi_threat_model_app_section_get`, `pi_gatekeeper_prs_list`, `pi_gatekeeper_issues_list`, `pi_gatekeeper_pr_get`, `pi_gatekeeper_issue_get` | No |
| `remediation-guidance` | Fetch existing finding/package remediation plans; optionally prepare a missing package plan | `pi_remediation_plan_fetch`, `pi_package_remediation_plan_fetch`; write: `pi_package_remediation_plan_prepare` | Only `prepare`, after confirmation |
| `security-design-review` | Start a design review from a Confluence/Notion URL or inline Markdown | write: `pi_design_review_url_create`, `pi_design_review_markdown_create` | Yes, after confirmation |
| `ingest-markdown-report` | Submit a security report supplied as inline Markdown | write: `pi_report_markdown_upload` | Yes, after confirmation |
| `secure-development-playbook` | Repo-specific secure-development guidance; optional targeted feedback | `pi_playbook_task_query`; optional threat-model context: `pi_knowledgebase_query`; write: `pi_playbook_comment_create` | Only feedback, after confirmation |

For a stable engineering task, the secure-development workflow queries the Pi playbook exactly once. If its coverage is missing or only adjacent, Claude surfaces the gap and continues from the user's instructions and current repository evidence instead of retrying, rephrasing, or falling back to generic tenant knowledge. A generic knowledge lookup is reserved for genuinely necessary threat-model context and is limited to one short non-sentence query.

Example prompts:

- "Explain FND-1042 and how it compares to FND-977."
- "What does our threat model say about the payments service, and are there open Gatekeeper issues touching it?"
- "Is there a remediation plan for lodash in the storefront repo? If not, have Pi prepare one."
- "Start a Pi design review of https://example.atlassian.net/wiki/spaces/ENG/pages/123 (a synthetic URL)."
- "Submit this pentest summary to Pi as pentest-q3.md."
- "How should I add a new webhook endpoint in this repo according to our playbook?"

## Write actions and confirmation

Five tools change state in Pi: `pi_package_remediation_plan_prepare`, `pi_design_review_url_create`, `pi_design_review_markdown_create`, `pi_report_markdown_upload`, and `pi_playbook_comment_create`. Every skill that uses one follows the same contract: validate inputs, show the exact target and a concise argument summary, explain what Pi will do, require an explicit yes in the current turn, call the tool once, report exactly the state Pi returned, and never retry automatically after an ambiguous failure. Pi starts hosted work (reviews, ingestion, plan generation); it never edits your local code.

## Identifiers Pi understands

- Findings: `FND-XXX` display IDs, finding UUIDs, or Pi finding URLs.
- Threat-model applications and sections: slugs.
- Repositories/codebases: slug or ID (stated by you, never inferred).
- Packages: package reference such as name@version, optionally with vulnerability ID and ecosystem.
- Gatekeeper PR records and issues: UUIDs from list results.

## Markdown and attachment boundary

Hosted submission accepts exactly two shapes: a supported Confluence or Notion document URL, or content pasted inline as Markdown/plain text. Chat attachments, local file paths, `file://` URIs, PDFs, DOCX files, and images are not converted into MCP inputs; use the Pi web app (or the Sloane CLI, for reports) for those.

## Update, uninstall, disconnect

- Claude Code: update via `/plugin` (marketplace updates), uninstall from the `/plugin` Installed tab, and manage authentication in `/mcp`.
- Cowork: update or uninstall the plugin under Customize → Plugins; manage sign-in on its connector page.
- Disconnecting or uninstalling removes access from that surface only; it does not change your Pi account or tenant data.

## Version compatibility

Requires a current Claude Code release (the `displayName` manifest field, used here, is recognized from v2.1.143). Validated with Claude Code 2.1.210. Cowork installs follow Cowork's published plugin limits.

## Legal and contact

- License: [MIT](LICENSE)
- Terms: <https://www.pi.security/terms-and-conditions>
- Data Processing Agreement: <https://www.pi.security/data-processing-agreement>
- Security reports: see [SECURITY.md](SECURITY.md)
- Contact and support: <contact@pi.security>
