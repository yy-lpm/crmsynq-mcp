# CRMSynQ MCP server

Connect your AI agent to [CRMSynQ](https://crmsynq.com) and let it set up and run LinkedIn-based contact updates in HubSpot, Pipedrive, Zoho CRM, Zoho Bigin and Zoho Recruit.

```
https://app.crmsynq.com/api/mcp
```

> **This repository contains documentation; the server is hosted by CRMSynQ.** There is no source code here and nothing to install or run. You add the URL above to your MCP client and sign in. The files are this README, [`server.json`](server.json) (the entry published to the official MCP Registry as `com.crmsynq/crmsynq`) and the [license](LICENSE) for the configuration snippets.

## What CRMSynQ does

CRMSynQ keeps the contacts in your CRM up to date from their LinkedIn profiles, on a schedule.

- **Supported CRMs:** HubSpot contacts, Pipedrive persons, Zoho CRM Contacts and Leads, Zoho Bigin Contacts, and Zoho Recruit Candidates and Contacts.
- **What gets updated:** the CRM fields you map, such as job title, company, location, industry and skills, from the LinkedIn profile URL stored on each record. Unmapped fields are never touched, and the recommended mapping never includes names, emails or phone numbers.
- **Companies:** for Contacts, CRMSynQ links each person to their current employer, searching your existing companies first and creating one only when none matches (you can turn that off).
- **Your team's edits are kept:** by default, a field someone changed after the person started their current LinkedIn job isn't overwritten. A per-field "only if empty" option writes a field only while it's empty.
- **Every change is visible and reversible:** each update is logged with before and after values and can be reverted. In review mode, a schedule checks contacts but writes nothing until you apply the changes.
- **Schedules, not real time:** a schedule processes a small first batch within minutes, then the rest, then re-checks the audience on its interval (1 to 180 days when set up through an agent; 30 by default).
- **Without a CRM:** look up to 25 LinkedIn profile URLs at a time, or refresh a list of rows (CSV text or URLs), to get each person's current title, company, location and headline.

CRMSynQ reads public LinkedIn profile data. It doesn't need Sales Navigator and never asks for your LinkedIn login. It doesn't find email addresses or phone numbers, and it doesn't add new contacts to your CRM.

### Pricing

- **1 credit per contact checked** in a CRM sync, and updated if anything changed. A credit is used even when nothing changed, because the profile still has to be checked.
- **1 credit per profile found** for lookups and list refreshes. Invalid URLs and profiles that aren't found are free.
- **50 free credits** for new accounts, no credit card.
- **One-time packs, no subscription:** 1,000 credits for $100, 3,000 for $200, 5,000 for $300, 10,000 for $500 or 20,000 for $800. **Credits never expire.**
- Using CRMSynQ through MCP costs nothing extra. Agents use the same credits as the app. See [pricing](https://crmsynq.com/pricing).

## Server details

| | |
|---|---|
| Endpoint | `https://app.crmsynq.com/api/mcp` |
| Transport | Streamable HTTP (stateless, POST) |
| Authentication | OAuth 2.1, authorization code with PKCE (S256), dynamic client registration, refresh tokens. You sign in with your CRM or with an email address. No API keys. |
| Scopes | `read`: see setup, schedules and activity. `write`: also change setup, run schedules and lookups (uses credits), apply or revert changes. `write` includes `read`. |
| Protected resource metadata | `https://app.crmsynq.com/.well-known/oauth-protected-resource` |
| Authorization server metadata | `https://app.crmsynq.com/.well-known/oauth-authorization-server` |
| REST API (same features, same sign-in) | `https://app.crmsynq.com/api/v1`, OpenAPI spec at [`/api/v1/openapi.json`](https://app.crmsynq.com/api/v1/openapi.json) |

A request without a token gets `401` with a `WWW-Authenticate` header pointing to the protected resource metadata, so clients that support MCP authorization start the sign-in on their own.

## Connect your client

Client menus change often. The steps below follow each vendor's documentation as of November 2026. If a menu name differs, look for "custom connector" or "MCP server" in your client's settings.

### Claude (web and desktop)

1. Open **Customize > Connectors**.
2. Click **+ Add**, then **Add custom connector**.
3. Name it `CRMSynQ`, paste `https://app.crmsynq.com/api/mcp` and click **Continue**.
4. Review the detected authentication settings, choose **Sign in now** and click **Add**. Sign in to CRMSynQ when the page opens.

CRMSynQ registers the client automatically, so you don't need a client ID, client secret or request headers. On Team and Enterprise plans, an owner adds the connector under **Organization settings > Connectors** first, and members then connect it from **Customize > Connectors**. See Anthropic's [custom connectors article](https://support.claude.com/en/articles/11175166-getting-started-with-custom-connectors-using-remote-mcp) for which plans include custom connectors.

### Claude Code

```bash
claude mcp add --transport http crmsynq https://app.crmsynq.com/api/mcp
```

Then run `/mcp` in Claude Code and choose CRMSynQ to sign in.

### Cursor

Add CRMSynQ to `.cursor/mcp.json` in your project, or to `~/.cursor/mcp.json` to use it in every project:

```json
{
  "mcpServers": {
    "crmsynq": {
      "url": "https://app.crmsynq.com/api/mcp"
    }
  }
}
```

Cursor handles the OAuth sign-in, including client registration, when it first connects.

### VS Code (GitHub Copilot)

Add this to `.vscode/mcp.json` in your workspace:

```json
{
  "servers": {
    "crmsynq": {
      "type": "http",
      "url": "https://app.crmsynq.com/api/mcp"
    }
  }
}
```

Start the server and sign in when VS Code asks. You can also add it from VS Code's MCP server settings by choosing an HTTP server and entering the URL. See VS Code's [MCP configuration reference](https://code.visualstudio.com/docs/agents/reference/mcp-configuration) for other config locations.

### ChatGPT

In ChatGPT on the web, add CRMSynQ as a custom MCP server:

1. Open **Plugins**, select **+** and choose **Add custom MCP server**. (OpenAI's help article also describes an older path, **Settings > Apps > Create**.)
2. Enter the name `CRMSynQ` and the server URL `https://app.crmsynq.com/api/mcp`.
3. Choose **OAuth** for authentication, select **I understand and want to continue**, then **Create as a plugin**.
4. Sign in to CRMSynQ when ChatGPT asks, and install the plugin before using it in a chat.

Which plans can add custom MCP servers, and which can use tools that make changes, depends on your ChatGPT plan and workspace settings. See OpenAI's [help article](https://help.openai.com/en/articles/12584461-developer-mode-apps-and-full-mcp-client-support-in-chatgpt).

### Other clients

Any client that supports remote MCP servers over Streamable HTTP with OAuth can connect. Use the endpoint URL; no headers or keys are needed.

## Sign in and approve access

1. Your client opens the CRMSynQ sign-in page. Sign in with your CRM (HubSpot, Pipedrive, Zoho CRM, Zoho Bigin or Zoho Recruit) or with an email address. You don't need a CRMSynQ account first; signing in creates one with 50 free credits.
2. A consent screen shows the app's name, the address you'll be returned to, and the access it asks for: **read**, or **read and write**. Choose **Allow** or **Deny**.
3. You're back in your client, connected.

If you signed in with an email address and haven't connected a CRM yet, the account, credit and LinkedIn profile tools work, and CRM tools return `NO_CRM_CONNECTED`. Connect your CRM in the [CRMSynQ app](https://app.crmsynq.com) and ask again.

To disconnect an agent, open **AI agents** in the CRMSynQ app and choose **Disconnect** next to it. Its tokens are deleted immediately.

## Tools

29 tools. The 13 read tools are available with the `read` scope. The 16 write tools are only offered to tokens with the `write` scope. Tools that write to your CRM, spend credits or delete something are annotated as destructive, so clients that respect MCP annotations ask you before calling them.

### Read tools (13)

| Tool | What it does |
|---|---|
| `get_setup_status` | Connected CRM, credit balance, setup progress per contact type, schedules, the main sync's status and the recommended next step. |
| `list_contact_types` | Contact types (for example Contacts, Leads, Candidates) with their layouts, LinkedIn URL field and current field mappings. |
| `list_crm_fields` | All CRM fields for a contact type: id, label, type, whether it can be updated or searched, and what it's mapped from. |
| `list_linkedin_fields` | LinkedIn data points that can be mapped into CRM fields. |
| `list_crm_lists` | Saved lists or segments a schedule can target (HubSpot and Pipedrive). |
| `count_contacts` | How many contacts an audience matches, to estimate credits before creating a schedule. |
| `list_schedules` | All schedules with status, interval, matching contacts, progress and what the last update did. |
| `get_schedule` | One schedule with live progress and its last update's results. |
| `list_activity` | Recent contact updates with before and after values for each changed field, plus the mapped fields left alone and why. |
| `list_pending_changes` | Changes waiting for review from schedules in review mode: before and after per field, the company link, photo and profile PDF, and values kept as they were. |
| `get_lookup_results` | Results of a LinkedIn profile lookup: status per URL and, once checked, the current title, company, location and headline. |
| `get_list_status` | Progress of a list refresh: status, rows, profiles checked and not found, and credits used. |
| `get_list_results` | Rows of a list refresh with the current title, company, location, headline, whether it changed, check date and status. Paged. |

### Write tools (16)

| Tool | What it does |
|---|---|
| `set_linkedin_url_field` | Choose the CRM field that stores each contact's LinkedIn URL. |
| `apply_recommended_mapping` | Map LinkedIn job title, company, location, industry and skills onto matching CRM fields. Never overwrites existing mappings and never maps names, emails or phones. |
| `set_field_mapping` | Set which LinkedIn data fills a CRM field, or stop updating it, optionally only while the field is empty. Applies to schedules created afterwards. |
| `set_enrichment_options` | Turn company linking, company creation, adding or replacing photos, the profile PDF and "keep your team's edits" on or off for new schedules. |
| `create_schedule` | Start updating an audience: a small first batch within a minute, then the rest, then a refresh every 1 to 180 days. Uses 1 credit per contact checked. Takes `applyMode`: `auto` or `review`. |
| `set_schedule_apply_mode` | Switch a schedule between writing changes automatically and keeping them for review. Changes already pending stay pending. |
| `apply_pending_changes` | Write pending changes to the CRM, for whole contacts or only some fields. Each contact is read again first, and a value changed in the CRM since the check is left alone. |
| `dismiss_pending_changes` | Drop pending changes without writing anything. Credits spent on the checks aren't refunded. |
| `run_schedule_now` | Run a schedule now instead of waiting for its interval. Contacts it already checked within the interval are skipped. Uses credits. |
| `pause_schedule` | Stop a schedule until it's resumed. |
| `resume_schedule` | Resume a paused schedule. It runs within a minute if it's due, its audience changed or its last run failed; otherwise it continues on its interval. |
| `delete_schedule` | Permanently delete a schedule. Past activity stays. |
| `revert_activity` | Write the previous values back to the CRM for an update, for every field or only some, including undoing a photo change. |
| `lookup_linkedin_profiles` | Check up to 25 LinkedIn profile URLs for each person's current job title, company, location and headline. 1 credit per profile found. |
| `refresh_list` | Create a list refresh from CSV text with a header row, or from a list of URLs (up to 2,000 rows), and preview the cost. Nothing is spent unless it's started. |
| `start_list_refresh` | Start a draft list refresh. 1 credit per distinct profile found. If credits run out, the remaining rows wait until you buy more. |

## Example prompts

- "Make CRMSynQ update all my contacts from LinkedIn every 30 days."
- "How many of my contacts have a LinkedIn URL, and how many credits would a full refresh use?"
- "Refresh everyone in my 'Prospects' list every week, but let me review the changes before anything is written to the CRM." (Saved lists work with HubSpot and Pipedrive.)
- "Show me the pending changes and apply only the job titles."
- "What did CRMSynQ update this week? List the contacts whose company changed."
- "Also fill in industry from LinkedIn for new schedules, but only where the field is empty."
- "Pause the schedule for my old leads."
- "Undo the last change CRMSynQ made to Jane Cooper's job title."
- "Look up these three LinkedIn profiles and tell me where each person works now: ..."
- "Here's a CSV of event attendees. Check where each of them works now, but tell me the credit cost first."

The server's instructions tell agents to confirm with you before creating schedules that will spend many credits, and before applying, dismissing, reverting or deleting.

## Security and data

- **No keys or passwords in chats.** Clients connect through OAuth 2.1 with PKCE. The agent never receives your CRM or CRMSynQ sign-in credentials.
- **Short-lived tokens.** Access tokens are valid for one hour and refresh tokens for 60 days, and every renewal replaces both. CRMSynQ stores only one-way hashes of tokens.
- **Two access levels.** A read-only connection isn't offered the tools that change anything.
- **Limited writes.** Write access lets an agent cause the same writes you can cause in the app: LinkedIn profile data written into the fields you mapped, or previous values restored by a revert. Agents can't use CRMSynQ to create or delete contact records, read whole contact records or export your contact list.
- **Logs.** Server logs record the path and method of each request, not the content of tool calls.
- **Your AI provider.** Tool results go to the client you connected and the AI model it uses, under your own agreement with that provider.

Details are in section 10, "AI Agents and the MCP Server", of the [privacy policy](https://crmsynq.com/privacy-policy). See also the [sub-processors](https://crmsynq.com/sub-processors), [data processing agreement](https://crmsynq.com/dpa) and [terms and conditions](https://crmsynq.com/terms-and-conditions).

## Links

- Setup guide: [How to connect CRMSynQ to Claude, ChatGPT and Cursor (MCP)](https://crmsynq.com/guide/how-to-connect-crmsynq-to-ai-agents-mcp)
- CRMSynQ for AI agents: https://crmsynq.com/ai-agents
- Website: https://crmsynq.com
- App: https://app.crmsynq.com
- Machine-readable summary: https://crmsynq.com/llms.txt

## Support

Email [support@crmsynq.com](mailto:support@crmsynq.com).

## License

The documentation and configuration snippets in this repository are released under the [MIT License](LICENSE). The CRMSynQ service is not open source; using it is subject to the CRMSynQ [terms and conditions](https://crmsynq.com/terms-and-conditions).
