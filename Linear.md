# Linear limitations: app and integrations

## App-wide limitations

### Workspace structure and plans

- Team limits are plan-dependent: Free 2, Basic 5, Business unlimited, Enterprise unlimited.
- Projects can belong to multiple teams, but an issue belongs to one team. Cycles are team-specific; Linear does not provide a single cycle view across multiple teams.
- Each issue has one main assignee, and each project has one lead. Linear does not provide native role slots such as owner, reviewer, approver, and contributor on the same issue.
- Linear does not currently provide a default assignee for every newly created issue. Teams need templates, triage rules, or automations for that routing.
- Issue relationships cover defined types such as blocking, blocked by, related, and duplicate; this is not a general dependency-management model with arbitrary dependency types or project-level dependency logic.
- Private teams are available only on Business and Enterprise. Sharing an issue from a private team to non-members is an Enterprise feature, and shared users cannot view or change the issue’s team, project, cycle, or milestone.
- Deleting a team also deletes its issues after the 30-day restore window; after that, Linear says the data cannot be recovered.
- Guests have restricted workspace visibility and cannot use workspace-wide features such as workspace views, customer requests, initiatives, or Code Intelligence.
- On the Free plan, all users are Administrators, which is a coarse permission model for organizations that need separation between ordinary members and workspace administrators.

### Import, export, and portability

- Workspace issue exports are admin-only; Enterprise limits this to owners. The emailed workspace-export link expires after 12 hours.
- Member issue exports are limited to 250 issues; admins/owners can export views up to 2,000 issues. Project and initiative exports are limited to 200 at a time.
- CSV exports do not include attachment files; descriptions may contain attachment links instead.
- There is no CSV export for teams.
- Google Sheets sync is not a complete historical warehouse: it refreshes hourly only when changes exist, exports public-team data only, and source-sheet edits are overwritten.
- Linear-to-Linear import excludes saved display preferences, favorites, reminders, drafts, inbox notifications, integrations, webhooks, OAuth clients, API keys, billing/plans, workspace/personal settings, and most roles. Integrations must be reinstalled and automations may not resume.
- Importers map source concepts to the closest Linear concepts; fields that do not translate may be renamed, remapped, or omitted. Re-importing the same source/team skips already imported issues unless the original import is deleted first, and import deletion is available only for a limited period.

### API and webhooks

- API-key requests: up to 2,500 requests/hour per authenticated user; OAuth apps: 5,000/hour per user/app user; unauthenticated calls: 600/hour per IP.
- Complexity budgets are also enforced: 3,000,000 points/hour for API keys, 2,000,000 for OAuth apps, 100,000 unauthenticated, with a maximum single-query complexity of 10,000 points.
- Some queries and mutations have lower endpoint-specific limits. Exceeding limits returns HTTP 429 with `RATELIMITED`; clients must honor `Retry-After`.
- Default pagination is up to 50 records. Broad nested queries can multiply complexity quickly; narrow fields, filters, and pagination are required for reliable extraction.
- Linear discourages polling; use webhooks for change notifications. Webhooks require admin permissions to create/manage and are scoped to supported entities/events, not an unrestricted event stream.
- Webhook delivery is not an ordered, permanent event log. Consumers must verify signatures, deduplicate by delivery ID, tolerate retries, and reconcile from the API after the retry window.
- API keys and webhooks can expose private-team data when authorized by a user/team owner. Treat them as data-export channels and scope keys to teams and permissions.

### Desktop app and access

- Linux is not currently supported as a desktop app; Linux users must use a supported browser.
- Passkeys are not currently supported in the desktop app.
- Offline mode can become stuck due to a Chromium issue; Linear’s documented workaround is toggling Wi-Fi off and on.
- Terminal-based custom coding tools require the Linear desktop app on a supported platform.

### AI, agents, and coding sessions

- Linear Agent cannot sign in to the app, access admin functionality, or manage users.
- Coding sessions require GitHub code access, a supported repository/environment, and AI credits. One environment contains one repository, and a repository can belong to only one active environment; archived environments cannot start new sessions.
- AI credits are pooled at workspace level. Failed runs, retries, and partial completions can be billable; funds are non-refundable and non-transferable.
- If the workspace runs out of credits, new credit-consuming work cannot begin. Admins cannot currently restrict which individual members spend enabled AI credits, although guest agent access can be prevented.
- Agent providers and third-party agents may have their own pricing, privacy, retention, and operational limits.
- Linear Agent and coding sessions do not replace human review: agents operate within available permissions and repository context, and generated changes still depend on the connected Git provider’s review and merge rules.

## Integration limitations

### GitHub

- GitHub Issues Sync is a separate feature from PR/commit linking and issue import. It is designed for linked repositories/teams and should not be assumed to synchronize every GitHub issue automatically.
- Historical GitHub issues require the GitHub Issues Importer; PR/commit linking alone does not pull GitHub Issues into Linear.
- Personal GitHub account connections are needed for identity-dependent features such as assignee mapping, mentions, and personal automations.
- Diffs/code review require repository code access. GitHub organization IP allow lists can block review actions even when the Linear GitHub App is installed.
- Draft/unsubmitted GitHub reviews do not sync to Linear; review state syncs when a review is submitted.
- Some GitHub inline comments cannot always be shown or created through Linear because of GitHub API limitations. Rich check-run annotations and detailed external-tool output are not currently displayed; Linear shows overall check status/basic details.
- GitHub cannot resolve a real GitHub mention for someone who has not connected/mapped their GitHub identity.
- Private-team issues posted to GitHub are restricted to linkbacks containing the issue ID and link only.
- Coding sessions require enabled GitHub code access, AI credits, and a repository-specific environment; they are not a general-purpose CI/CD replacement.

### GitLab

- Self-hosted GitLab is supported only when the installation is publicly available.
- Setup requires a GitLab access token; the token is used to query GitLab and post issue linkbacks.
- The core workflow is merge-request linking and automation; it should not be treated as a full GitLab issue migration unless using a separate import/API workflow.
- Private-team issues posted to merge requests are restricted to issue ID/link linkbacks.

### Jira

- Jira Sync is intended for transition/coexistence. Existing issues created before sync do not automatically appear unless they are updated, or are brought over through import.
- Jira has no direct Linear equivalent for issue types, constraints, components, and required fields. Components become Linear labels that cannot be grouped/deleted; removing one in Linear removes the component in Jira.
- Jira and Linear hierarchy rules differ. Parent/sub-issue changes can fail or become inconsistent, especially where Jira’s Epic → Story/Bug/Task → sub-issue hierarchy would be violated.
- Uni-directional sync creates Jira-created issues in Linear but does not create new Jira issues when a new Linear issue is created. Bi-directional sync is required for creation in both directions.
- Legacy uni-directional mappings created before March 2026 do not sync Linear property changes back to Jira. Switching them to current sync is irreversible.
- JQL filters used during import do not control Jira Sync; filtering must be configured on the Jira webhook.
- Deleted synced issues are not deleted in the other system. A Jira status that cannot be mapped to Linear leaves the Linear status unchanged.
- Assignee/creator mapping requires each user to connect their Jira account; otherwise Linear may leave the issue unassigned or attribute it to the configuring user.
- Labels must already exist in Linear to sync Jira → Linear. Some Jira metadata changes can make issues drift out of sync.
- Multiple Jira sites per Linear workspace are Enterprise-only.
- Jira setup requires high Jira permissions (`ADMINISTER`) or a service account with the documented scopes.

### Slack

- Workspace installation requires a Linear admin (or Enterprise owner); multiple Slack workspaces per Linear workspace are Enterprise-only.
- Only users with Linear accounts can create issues through the standard integration. Slack Guests cannot install/approve the app for this use.
- Private channels require inviting `@Linear`; group DMs require inviting Linear when the DM is created, and Linear cannot be added to an existing group DM.
- Slack Loops run only from selected public channels: private channels, direct messages, externally shared channels, and thread replies do not trigger runs.
- Slack link previews/unfurls follow Slack channel visibility and workspace unfurl settings, not necessarily the viewer’s Linear role; this can create data-exposure risk.
- `/linear` is not supported in Slack threads, for Slack sync, or for uploading files to issues.
- Rich Slack previews may not exactly match Linear; some formatting/details are visible only in Linear.
- Project-channel auto-creation requires appropriate Slack permissions and may require reauthorization if the integration was installed before the relevant permissions were added.

### Microsoft Teams

- Standard workspaces connect one Microsoft Teams tenant per Linear workspace; multiple tenants require Enterprise.
- Microsoft Teams Personal Use is not supported; Teams for Work and School are supported.
- A Linear admin/workspace owner and a Teams admin may both be needed. Organizations that centrally restrict third-party apps must allow Linear in the Teams Admin Center.
- Shared Teams channels are not supported for project-channel connections.
- Private-channel support depends on Microsoft’s developer-preview/Targeted Release availability; tenants not opted in may not be able to use Linear Agent in private channels.
- Guests in shared channels are not answered unless they have access to the Linear workspace.
- Each user may connect one Microsoft account from a single tenant.

### Discord

- Every Linear user must individually link their Discord account; workspace enablement alone is not enough.
- The documented commands are issue creation, issue search/display, and daily wrap. Project-update notifications are not supported.
- The integration requires Discord Read Message History to retrieve message data when linking a Discord message to an issue.
- Anyone using the commands must have a Linear account.


### Figma

- The installer of the Figma integration must have access to the file; a different user’s access does not guarantee previews work.
- Interactive in-app preview is currently supported for publicly shared Figma files only; private-file preview is not supported in the documented integration.
- Figma previews are snapshots and are not refreshed by default. Refresh is manual, so an embedded design can become stale.
- Figma embeds cannot be collapsed/hidden once a full Figma link is pasted; use a text hyperlink as a workaround.
- Browser/cross-site-cookie settings can block authenticated previews, and Linear recommends connecting in a browser rather than the desktop client.
- Figma plugin authentication transmits/stores file keys for linked files. Anyone with access to a Linear issue can access the associated file key, so issue permissions and Figma permissions must be reviewed together.


### Sentry

- Sentry integration supports cloud accounts only; self-hosted Sentry is not supported.
- Only one Sentry organization can be connected at a time.
- Issues can be created/linked only in public Linear teams. Converting a team to private breaks existing connections.
- Automatic Sentry resolution depends on the linked Linear issue being resolved; assignee sync works when the same email is used in both services.

### Intercom

- Available only on Business and Enterprise plans.
- Every Intercom user can create/link issues and see sidebar details, but only Linear users can open the linked issue in Linear.
- Intercom internal notes do not sync into the associated Linear issue. The integration can post selected Linear updates back as Intercom internal notes.
- Disconnect/reconnect does not retroactively sync changes made while disconnected; integration settings reset to defaults.
- The integration exposes customer conversation content, metadata, and attachments to Linear Agent when AI issue creation is enabled; review data handling and AI guidance before enabling.
- Private-team issues cannot be created or linked.
- At most 10 templates can be made available in the Intercom integration.


### Front

- Available only on Business and Enterprise plans.
- Every Front user needs a Linear account to create/view issues.
- The installation has three ordered steps: install/approve in Front, enable automation in Linear, then sign in from the Front widget.
- Front templates are not supported.
- Internal notes written in Front do not sync to Linear; Linear comments/status changes can optionally post as Front internal notes.
- Conversations in private Front inboxes do not support automated comments or reopening.
- Private-team behavior depends on the user’s access: users with access can create/link private-team issues, but the integration must still be reviewed for exposure.

### Zendesk

- Available only on Business and Enterprise plans.
- Each Zendesk agent who uses the integration needs a Linear account.
- Installation requires both the Zendesk Marketplace app and Linear-side automation configuration; missing either step prevents reopening/internal-note automations.
- Zendesk domain changes are not supported cleanly: tickets linked before the domain change may no longer appear in the Linear app, although some automations may continue.
- Internal notes are optional/configurable; customer-facing replies are not automatically mirrored as ordinary Linear comments.
- Private-team issues cannot be created or linked.
- At most 10 templates can be made available in the integration.
- AI issue creation analyzes the full conversation, metadata, and attachments; this can create multiple issues and should be reviewed before relying on automatic routing.

### Google Sheets

- One Google account/sheet connection per Linear workspace.
- Only issues in public teams, and projects associated with at least one public team, are exported. Data contained only in private teams is not accessible.
- There is no field-level selection beyond choosing issues, projects, and/or initiatives.
- Refresh is hourly when changes exist unless an admin manually refreshes; it is not real-time.
- Source-sheet edits are overwritten. Build analytics in a separate sheet using references such as `IMPORTRANGE` or `VLOOKUP`.
- Data is displayed in GMT.
- Only documented issue/project/initiative fields are synced; this is not a full-fidelity copy of every Linear object, event, attachment, or permission.

### Airbyte

- Available only on Enterprise.
- Linear supports Airbyte Open Source, not Airbyte Cloud.
- The integration grants read access to all supported tables and cannot exclude private teams.
- Incremental sync is not supported; available modes are Full Refresh and Append.
- Sync cannot run more frequently than every 12 hours.
- Setup requires running/operating Airbyte locally with Docker and configuring a custom Linear source connector.
- Airbyte receives information from all public and private teams, so destination access and retention become part of Linear’s security boundary.

### Zapier

- Zapier is a third-party automation layer; Zapier pricing, task limits, delays, retries, and app-specific limits apply in addition to Linear’s limits.
- Workflows are event/action automations, not guaranteed transactional two-way synchronization. Duplicate events, retries, throttling, and partial failures must be handled in the Zap design.
- Zapier can expose private-team names/titles through API/webhook authorization; review the connected user/key and destination app carefully.
- The Linear Zapier integration creates/updates supported Linear records and triggers; it should not be assumed to expose every Linear object, field, permission, or event.
- API rate limits and webhook behavior still apply underneath Zapier.

### Notion

- The standard integration is primarily a preview/linking integration, not a full two-way synchronization system.
- Each user must connect their own Notion account; there is no single workspace-wide authentication for all users.
- Previews update on reload or manual refresh, so they are not guaranteed to be real-time push notifications.
- The integration previews Linear issues, projects, initiatives, and views, but it does not make all Linear actions available inside Notion.
- Private-team previews can work because authentication is per user, but only users who belong to the private team should be able to see the content.
- Notion AI, Custom Agents, and Notion MCP workflows have separate Notion availability, AI, and usage limits.

### Gong

- Available only on Linear Enterprise.
- It processes customer-facing Gong recordings periodically, not necessarily immediately.
- Internal/private calls and recordings shorter than 10 minutes are skipped.
- The integration filters out non-actionable findings such as sales questions, beta-access requests, general inquiries, and internal commentary.
- One call can create multiple Linear issues if multiple distinct findings are detected, so human triage is still required.
- External call participants do not need Linear seats, but the integration can create issues and transcript excerpts from their calls.
- AI-generated titles, descriptions, excerpts, routing, and customer-request associations should be reviewed for accuracy and privacy.

### Salesforce

- Available only as an add-on to Linear Enterprise; contact Linear sales to purchase licenses.
- Salesforce users can create issues without Linear accounts, but Salesforce users need an assigned license for the Salesforce integration.
- The integration is designed around Salesforce cases and links cases to Linear issues/projects; it is not a general Salesforce data replication tool.
- Salesforce fields can be mapped only to the Linear customer-attribute data types supported by the integration.
- Salesforce properties can be used in Linear views, but are not supported in Insights or Dashboards.
- If reduced personal information is enabled, customer matching may return an unknown customer because Linear does not fetch additional Salesforce personal information.
- Closing a duplicate Linear issue cancels the linked issue and reopens the Salesforce case; Salesforce cases are not merged automatically.
- The integration supports Salesforce applications that support cases, including Service Cloud and Sales Cloud, but its behavior depends on Salesforce permissions and page configuration.

### Origin

- Origin is a pull-request linking/status-automation integration, not a general issue or repository migration tool.
- Issues link to Origin pull requests using branch names, PR titles, or magic words such as `Fixes LIN-123`.
- Status changes are controlled by team-level workflow automations; the integration does not automatically impose one global workflow across all teams.
- Personal automations require each member to configure their own preferences.
- Linear’s public documentation does not state an Origin subscription price or guarantee that every Origin plan supports every feature. Check Origin/Cursor’s current terms before treating it as free.

### Linear Agent and MCP integrations

- Linear Agent is included in Linear plans, but actions remain bounded by the user’s existing Linear permissions.
- Agents cannot sign in to the Linear app, access admin functionality, or manage users.
- MCP servers can read or write external data depending on the connected server and granted scopes. A connected MCP server is not automatically read-only.
- Workspace admins control MCP server access at workspace level; external providers retain their own permissions, billing, rate limits, model limits, and data-retention policies.
- Multiple Linear workspaces require separate authentication contexts for MCP sessions; reconnecting does not necessarily switch the workspace inside an existing session.
- Linear’s MCP server supports read-only endpoints and restricted OAuth/API-key access, but a default connection can provide read-write access.
- A bearer token or API key used with MCP inherits the permissions of that token. Use a read-only or team-scoped key for least privilege.

### Coding-tool integrations

Linear can open issues with context in Claude Code, Codex, Cursor, Conductor, GitHub Copilot, OpenCode, Replit, v0, Zed, and custom tools.

- Opening an issue through a deeplink is different from running an autonomous coding session. The deeplink mainly transfers issue context; the coding tool performs the work.
- The external tool’s own subscription, model credits, API limits, local installation, repository permissions, and privacy rules apply.
- Terminal-based tools require the Linear desktop app on a supported platform because Linear launches a local terminal.
- Custom URL tools can receive issue context in query parameters; treat those URLs as potentially sensitive because issue descriptions, comments, images, and links may be passed to the tool.
- Custom local scripts run with the user’s local permissions and are not sandboxed by Linear in the same way as Linear cloud coding sessions.
- Signed-commit requirements can prevent coding sessions from committing unless the user has configured the required signing key.
- Codex inside Linear requires an eligible ChatGPT Plus, Pro, Business, Enterprise, or Edu plan; it is not a free Linear-only feature.

### Other official directory integrations

Linear’s official documentation directory also lists integrations such as Notion, Gong, Salesforce, and Origin, plus an AI-client directory containing Claude, Cursor, Codex, VS Code, Windsurf, Zed, Replit, Figma Make, Notion Custom Agents, Postman Agent Mode, Qodo, CodeRabbit, Lovable, Builder.io, and other tools.

For these and other third-party directory entries, Linear does not publish a complete standardized table of external pricing, quotas, retention, supported fields, retries, or failure behavior. The reliable limitation is therefore provider-specific: the app may be listed in Linear, but the integration owner controls its plan requirements and operational limits.

## Public-review claims checked (2026-11-03)

The following review comments mix subjective usability feedback with objectively documented product constraints. The status below reflects current Linear documentation checked on 2026-11-03.

| Review claim | Current assessment | What the documentation confirms |
|---|---|---|
| “It’s difficult to navigate between teams.” | **Subjective, still plausible; partly mitigated.** | Teams have expandable sidebar sections, team home pages, an `O` → `T` team search shortcut, customizable sidebar ordering/hiding, and All teams/Exploring behavior. Teams remain separate work areas and cycles are team-specific, so multi-team work can still feel fragmented. |
| “Permission settings aren’t clearly explained.” | **Usability concern; underlying model is real.** | Permissions are split among workspace owner/admin/member/guest roles, team owners, team access, private teams, sub-teams, and workspace restrictions. Permission settings are not inherited from parent team to sub-team. This is a legitimate onboarding/documentation burden even though the controls exist. |
| “Linear depends on the team keeping it up to date.” | **Still applicable as a process limitation.** | Linear is a live planning system. Statuses, assignees, cycles, projects, triage rules, integrations, and customer links can become stale if people do not maintain them. Templates, triage, rules, Git automations, and integrations reduce the work but do not eliminate the need for ownership. |
| “Epics are too lightweight; I want modules inside epics, then tasks/subtasks.” | **Partly outdated in wording, but the hierarchy concern remains valid.** | Linear’s current model uses initiatives → projects → issues → sub-issues, plus parent/sub-issue relationships. Sub-issues can be nested, but Linear does not present a Jira-style “epic” object with arbitrary module layers. Converting a parent issue to a project removes the parent/sub-issue relationships and makes the former issue/sub-issues standalone project issues. |
| “Linear is less robust than Asana for custom fields and multiple project assignments.” | **Still applicable for teams needing those exact capabilities.** | An issue can be associated with only one project at a time. Linear’s documented workaround is to use sub-issues for tasks that need to appear under different projects. Official issue/display/filter docs list built-in properties rather than a general user-defined custom-field system. Public reviews and community discussions continue to identify custom fields and cross-project assignment as common gaps, but those comments are user evidence, not official product guarantees. |
| “Linear AI lacks antivirus, firewall, malware protection, network monitoring, traffic visualization, remote access, inventory, SAP, CRM, help desk, risk assessment, calendar management, and advanced BI.” | **Correct category distinction, but not an AI-specific defect.** | Linear is primarily product-development, issue-tracking, planning, customer-request, and engineering workflow software. It is not documented as an endpoint-security, network-management, ERP, full CRM, full help-desk, remote-access, IT-asset inventory, or full calendar product. It has integrations, dashboards/Insights, exports, and API/webhooks, but these do not turn Linear into those systems. |
| “Audit logs are Enterprise-only and retained 90 days.” | **Confirmed.** | Audit Log is available on Enterprise and retained for 90 days. Workspace owners can access it; advanced filtering is available through the API, and audit events can be streamed to a webhook/SIEM. |
| “Authentication mainly relies on Google/email login, passkeys, or SAML/SSO rather than built-in traditional 2FA.” | **Mostly applicable, but phrase carefully.** | Linear documents Google SSO, email codes, passkeys, SAML, SCIM, domain/login restrictions, and IP restrictions. Current documentation does not describe a separate built-in username/password-style TOTP 2FA product. Passkeys are not supported in the desktop app. Enterprise provides SAML/SCIM and advanced authentication controls. |
| “AI credits are shared and admins cannot restrict individual spend.” | **Confirmed, with one qualification.** | AI credits are pooled at workspace level. Linear says admins cannot currently restrict which individual members spend enabled AI credits. Admins can manage spend limits, and they can prevent guests from interacting with agents; this is not the same as per-user spending authorization. |



## Sources reviewed

- [Linear Docs](https://linear.app/docs)
- [Integration Directory](https://linear.app/docs/integration-directory)
- [Exporting Data](https://linear.app/docs/exporting-data)
- [Private teams](https://linear.app/docs/private-teams)
- [Members and roles](https://linear.app/docs/members-roles)
- [API and Webhooks](https://linear.app/docs/api-and-webhooks)
- [API Rate Limiting](https://linear.app/developers/rate-limiting)
- [Download Linear](https://linear.app/docs/get-the-app)
- [Agents in Linear](https://linear.app/docs/agents-in-linear)
- [Coding sessions](https://linear.app/docs/coding-sessions)
- [AI credits](https://linear.app/docs/ai-credits)
- [GitHub](https://linear.app/docs/github-integration)
- [Diffs and reviews](https://linear.app/docs/diffs)
- [GitLab](https://linear.app/docs/gitlab)
- [Jira](https://linear.app/docs/jira)
- [Slack](https://linear.app/docs/slack)
- [Microsoft Teams](https://linear.app/docs/microsoft-teams)
- [Discord](https://linear.app/docs/discord)
- [Figma](https://linear.app/docs/figma)
- [Sentry](https://linear.app/docs/sentry)
- [Intercom](https://linear.app/docs/intercom)
- [Front](https://linear.app/docs/front)
- [Zendesk](https://linear.app/docs/zendesk)
- [Google Sheets](https://linear.app/docs/google-sheets)
- [Airbyte](https://linear.app/docs/airbyte)
- [Zapier](https://linear.app/docs/zapier)
- [Notion](https://linear.app/docs/notion)
- [Gong](https://linear.app/docs/gong)
- [Salesforce](https://linear.app/docs/salesforce)
- [Origin](https://linear.app/integrations/origin)
- [AI clients directory](https://linear.app/integrations/ai-clients)
- [MCP](https://linear.app/docs/mcp)
- [Assign and delegate issues](https://linear.app/docs/assigning-issues)
- [Codex integration](https://linear.app/integrations/codex)
