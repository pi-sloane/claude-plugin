# Setup

The plugin connects Claude to Pi Security's hosted MCP server at `https://mcp.pi.security/mcp`. The connection is authenticated with OAuth; the plugin ships no credentials, tokens, or headers.

## Prerequisites

- A Pi Security account with access to at least one tenant.
- Claude Code, or Cowork with plugin support. Claude Chat is not supported; use the separately published Pi remote connector there.

## Claude Code

1. Install the plugin (see [README](README.md#install)).
2. Run `/mcp` inside Claude Code, select the `pi-security` server, and choose to authenticate. Claude Code opens Pi's OAuth sign-in in your browser.
3. Sign in and, if your account belongs to more than one tenant, pick the tenant during Pi's sign-in flow.

## Cowork

1. Install the plugin from the Plugins page (see [README](README.md#install)).
2. When a plugin includes a connector that needs authentication, Cowork prompts you to sign in during install. Complete Pi's OAuth sign-in there, choosing your tenant if prompted.
3. Manage the connector from the installed plugin's connector page under Customize → Plugins.

## Verify the connection

Ask Claude to call the Pi `whoami` tool. It returns the authenticated subject, tenant, and granted scopes without exposing any credentials. Confirm the tenant is the one you expect before running other workflows.

## Tenant selection and ambiguity

Your tenant is determined entirely by the authenticated OAuth session. Claude cannot infer or switch tenants from your working directory, repository, git remotes, or anything you type. If `whoami` shows the wrong tenant, disconnect and re-authenticate with the right one.

## Reconnect and disconnect

- Claude Code: run `/mcp`, select `pi-security`, and use the authentication options to re-authenticate or clear authentication. Uninstalling the plugin removes the server configuration.
- Cowork: sign out or back in from the plugin's connector page; uninstall the plugin to remove the connector.
