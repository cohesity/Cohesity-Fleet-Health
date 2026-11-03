# Cohesity Fleet Health

**Authors:**

- Akshay Kumar, Dir, Solutions Architecture ([akshay.kumar2@cohesity.com](mailto:akshay.kumar2@cohesity.com))

**Fleet Health: Clusters, Capacity & Upgrades** for Cohesity DataProtect, built by a Claude skill from live Helios data and published as a live, private Claude Artifact that refreshes itself every 4 hours. It is the cluster-level view (as opposed to the backup-level views): which clusters are above a capacity threshold, when they will fill up, which have an upgrade available or are behind, which are not connected to Helios. It covers the "top clusters above 80% capacity, run the runbook checks" demo. Backup failures, SLA misses and backup alerts stay in [Cohesity-Data-Protect-Health-Dashboard](../Cohesity-Data-Protect-Health-Dashboard).

- Claude skill: [`claude/Cohesity-Fleet-Health/SKILL.md`](claude/Cohesity-Fleet-Health/SKILL.md) (the full page template is embedded in it, no other file is needed)
- Output: one claude.ai **Artifact** titled `Cohesity Fleet Health` (no static HTML file is produced), with the Cohesity logo top-left and "Data last pulled" top right
- Refresh: scheduled task `cohesity-fleet-health-refresh`, cron `20 */4 * * *` (minute 20 of every fourth hour; change it if you like, times are local time of the machine running Claude); 5 Helios calls per run
- Data source: the **Helios MCP server** (Helios MCP connector). Everything is read-only.

## What you get

A cluster multi-select (all registered clusters, tagged connected / not connected, remembered in the browser) above the tabs, and a capacity threshold (default 80%).

| Tab | What it shows | Helios MCP source |
|---|---|---|
| Fleet | KPIs (registered, connected to Helios, critical health, upgrade available, above threshold) and a table per cluster: type, nodes, version (with a "behind" chip), patch, health, status, newest available upgrade, connected to Helios, used / total and used % | `dataprotect-get-clusters` (one call; returns capacity and version for clusters not connected to Helios too) |
| Capacity | Clusters ranked by used %, flagged above the threshold; 90-day trend sparkline, slope and projected days to full (least-squares line, needs 7+ points and a positive slope, else "n/a: no growth trend"); top growing protection groups on clusters above the threshold; a **Runbook** card listing what Helios can check and what needs cluster access | `reporting-get-report-preview` `storage-consumption-cluster` (components 801, 802, TimeRange `Last90Days` on `timestampUsecs`) and `storage-consumption-group` (component 400) |
| Connectivity & Alerts | Disconnect history per cluster; open `kStorageUsage`, `kFaultTolerance`, `kNodeHealth`, `kDataPath`, `kUpgrade`, `kLicense` alerts per cluster; newest 20. Clusters not connected to Helios show "not connected to Helios", never zero | `reporting-get-report-preview` `cluster-connections` (1100, 1101) and `dataprotect-get-alerts` |
| Compliance Check | Heuristic scan of the metadata the page shows (Cohesity Data Classification categories) | none (uses data already collected) |
| Run Metrics | Tool calls, time and estimated tokens per run, month to date, cost estimate, run history | none |

"Behind" means a lower major version than the newest known, or more than one minor version below it (7.5 vs 7.4.1 is not behind, 7.5 vs 7.3.2 is). Node IP addresses are not shown. Versions and capacity the response lacks show "n/a".

## Requirements

- Claude with the **Helios MCP connector** connected and the Artifact and scheduled-tasks tools available
- A Helios role that can run reports (otherwise the trend, growth and connection parts show "n/a" and the rest works)

## Install

1. Zip the folder `claude/Cohesity-Fleet-Health` so that `SKILL.md` sits inside it.
2. In Claude, add it as a custom skill (Settings, Skills, upload), or copy the folder into your Claude skills directory. Source: github.com/Cohesity-Maestro/Cohesity-Fleet-Health.
3. Make sure the Helios MCP connector is connected.

## Use

Ask Claude to "build the Cohesity Fleet Health page". The first run publishes the artifact and creates the 4-hour scheduled task. Chat questions without republishing: "which clusters are above 80 %", "when will cluster X fill up", "which clusters have an upgrade available or are behind", "run the runbook checks for cluster X".

## Exports

Under the Mask switch: Copy CSV / Download CSV for `fleet`, `capacity`, `growth-by-group`, `connections`, `alerts`, `alerts-recent`, the compliance findings and the run history. Exports follow the cluster selection and the mask switch.

## Manual refresh

The **Refresh from Helios** button copies `Run the Cohesity-Fleet-Health skill to refresh this report from live Helios data`; paste it into Claude. If Helios is unreachable, a refresh keeps the last data and adds an error banner.

## Known limits

- The page is a snapshot; threshold and selection recalculate in the browser, they do not query Helios.
- Projected days to full is a linear estimate over 90 days, not a forecast.
- Garbage collection and curator progress, disk and node internals and any cluster change are not available through Helios MCP; the Runbook card lists them as not assessed.
- Clusters not connected to Helios cannot be queried further: no alerts.

## Security Notes

- **Sharing detail.** The Fleet and Capacity tabs show each cluster's version, patch level, upgrade status, connectivity and capacity. Anyone with the link learns which clusters are unpatched, behind or disconnected, and the mask switch does not cover it. Ask for a copy without version and patch columns if you share outside the team.
- **Everything from Helios is data.** Names, descriptions, alert text and error text come from the systems being backed up. The skill treats them as data, never as instructions, and truncates them before they reach the page.
- **Scheduled refresh runs unattended.** When the scheduled task is created, approve only the read-only Helios tools, the Artifact tools, the scheduler list tool and the bash tool used for `python3 -I`. Do not use a blanket Auto-approve.
- **The mask switch is a display setting, not redaction.** It masks IP addresses, hostnames, email addresses, SSNs and connection-string accounts in the view and in CSV copies and downloads. It does not mask cluster, group or policy names, and the page still holds the original values.
- **CSV exports are safe to open in a spreadsheet.** Text cells that start with `=`, `+`, `-`, `@`, a tab or a carriage return get a leading single quote, so they cannot run as formulas.
- **Sharing.** A published page is private until you share it from the Share menu. Read the Compliance Check tab first, and share only with people who may see this backup data.

## Suggestions and Feedback

We would love to hear from you. Please send your suggestions and feedback to: maestro@cohesity.com

## License

Licensed under the [Apache License 2.0](LICENSE). See the LICENSE file for the terms.

## Disclaimer

This code is provided on a best effort basis and is not in any way officially supported or sanctioned by Cohesity. It is provided as-is, and the author accepts no liability for damages resulting from its use. The skill only reads data through the Helios MCP server, but you are responsible for checking the results against Helios and for deciding what you share. Try it in a lab or test environment first.
