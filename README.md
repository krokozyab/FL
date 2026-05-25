# FusionLens

**Read-only investigation lens for Oracle Fusion Financials.**

> *"What exactly makes up this balance, and where did it come from?"*

<!-- TODO: record the 8–10s demo GIF (balance → journals → AP invoice → xlsx) and drop into docs/img/demo.gif -->
<p align="center">
  <img src="img/demo.gif" alt="FusionLens demo: balance → journals → SLA → AP invoice → xlsx export" width="720">
</p>
▶️ **[Watch the demo](https://youtu.be/tCHShFHWbZk)** &nbsp;·&nbsp; 📥 **[Setup & install →](#setup--install)**


[![GitHub Downloads](https://img.shields.io/github/downloads/krokozyab/ofjdbc/total?style=for-the-badge&logo=github)](https://github.com/krokozyab/FL/releases)


Setup is **one-time per Fusion tenant**, takes ~5 minutes, and is usually handled by a BI Publisher developer or admin. After that, every teammate just downloads the client and signs in.

## Who it's for

- **Accountants and controllers** investigating balance differences, unusual movements, unposted activity, reconciliation issues, and period-close exceptions — then exporting the evidence to Excel for workpapers and review.
- **Application support and helpdesk teams** resolving user tickets by inspecting the live accounting trail instead of relying on screenshots, partial exports, and back-and-forth with business users.
- **BI Publisher, OTBI, and reporting developers** validating report logic against the underlying Fusion accounting data, comparing expected results with actual journals and subledger records.
- **Finance systems and ERP teams** triaging cross-functional issues that span configuration, accounting rules, source transactions, reporting logic, and user access.

## Where it fits

FusionLens **complements** Oracle Smart View, Account Monitor, Account Inspector, and Fusion Data Intelligence — it does not replace any of them. Those tools govern, dashboard, monitor, and present. FusionLens does one different job: **rapid tactical investigation** — pivot or flatten GL balances any way you need, drill from any cell into the journal, the SLA line, and the source row, with local filter / sort / export built in. Wherever Fusion exposes a deep link for the source document, one more click jumps straight from the drilled row into the live Fusion page for that document — no copy-pasting transaction numbers between tabs.

## In 30 seconds

1. Sign in once — via your existing Fusion SSO, or with service-account credentials.
2. Pick a ledger, period, and chart of accounts — balances populate locally. Pivot them across two dimensions (e.g. natural account × period) or keep them flat.
3. Click any balance → drill into the journals behind it.
4. Drill again → land on the Subledger Accounting line and source row.
5. Need the document? Click the row → the AP invoice (or AR transaction) opens directly in Fusion.
6. Filter and sort on the cached result; export the slice to `.xlsx`.

No screenshots, no extracts from IT, no jumping across half a dozen Fusion pages to chase one number.

## Read-only by design

FusionLens never writes to Fusion. It uses your existing Fusion access — Oracle SSO (Bearer tokens cached in your OS keychain: macOS Keychain / Windows Credential Manager) or HTTP Basic for service accounts — and only ever issues `SELECT` against the BI Publisher reports you're already authorized to read. Nothing leaves your machine.

## Setup & install

FusionLens is a desktop client that talks to a small set of BI Publisher reports inside *your* Fusion tenant. The reports are how it asks Fusion for balances, journals, and SLA data — without them the client has nothing to call. Plan ~15 minutes for the first-time setup; after that, every user on your team just downloads the client.

### Prerequisite — deploy the BI Publisher reports *(one-time, per tenant)*

> **Already using [`ofjdbc`](https://github.com/krokozyab/ofjdbc)?** You can skip this step — FusionLens uses the same `DM_ARB` data model and `RP_ARB` report.

In your Fusion instance, un-archive these two catalog files into `/Shared Folders/Custom/Financials/`:

- `DM_ARB.xdm.catalog` — the BI Publisher data model
- `RP_ARB.xdo.catalog` — the report

Both files live in the [`otbireport/`](https://github.com/krokozyab/ofjdbc/tree/master/otbireport) folder of the `ofjdbc` repository. In Fusion, open **BI Publisher → Catalog**, then un-archive each file into the target folder.

You can use a different catalog folder if your tenant's structure requires it — just point **BI Publisher report path** at the matching location in the in-app Settings form (the default is `/Custom/Financials/RP_ARB.xdo`). A Fusion administrator or report developer with BIP catalog write access typically handles this step.

### 1. Download the desktop client

[Latest release for macOS & Windows](https://github.com/krokozyab/FL/releases/latest). Grab the build for your OS — macOS Apple Silicon `.app` (inside a `.zip`) or Windows `.exe` (inside a `.zip`). Drag the `.app` into `/Applications` (macOS) or extract the `.exe` anywhere (Windows). No `config.json` to place — settings live in their standard OS location and are managed through the in-app form.

### 2. First launch & Gatekeeper

- **macOS** — Gatekeeper blocks the app because the binary isn't signed by an Apple-registered developer. Right-click the app → **Open** → confirm in the dialog. macOS remembers your choice for future launches. (Alternative: **System Settings → Privacy & Security → "Open Anyway"** after the first blocked attempt.)
- **Windows** — SmartScreen shows *"Windows protected your PC"* for the same reason. Click **More info → Run anyway**.

### 3. Settings form (first run)

On a fresh install, FusionLens opens straight to the **Settings** page. Add your first environment:

- **Name** — short label like `dev1` / `prod`. Becomes the OS-keychain account for that environment's tokens.
- **Fusion host URL** — e.g. `https://fa-acme-dev1.fa.us2.oraclecloud.com`.
- **BI Publisher report path** — defaults to `/Custom/Financials/RP_ARB.xdo`; change only if you deployed the catalog files to a non-default folder (see Prerequisite).
- **Authentication** — pick one:
  - **SSO** (default) — Chrome opens on first sign-in, Bearer token is cached automatically and refreshed in the background. Requires Chrome installed on the machine.
  - **HTTP Basic** — enter a service-account username + password; the password is written to your OS keychain, never to disk.
- **Color** (optional) — pick an accent so dev / prod / sandbox stand out at a glance in the topbar dropdown.

Optional collapsible sections cover performance tuning (max balance rows, SOAP / REST / SSO timeouts), cache-freshness windows, and tenant-specific cosmetic settings like segment-label prefix stripping. Reasonable defaults are baked in — leave them empty unless you need to tune.

Click **Save & Connect** — FusionLens runs a quick `SELECT 1 FROM DUAL` against the Fusion host to verify the connection (this is also where Chrome opens for SSO sign-in on first save). On success it takes you straight to the main app; on a wrong host / bad credentials / SSO timeout, you stay on the settings form with the underlying error visible so you can fix and retry.

Revisit settings at any time via the **sliders icon** in the top-right corner.

### 4. Connect & investigate

- The top-right pill should turn green and say **authenticated**.
- The environment dropdown sits at the top-left of the topbar; click it to switch between saved tenants without going through settings.
- Pick a ledger in the left panel — the picker is filtered by your Fusion Data Access Set grants.
- Set the period range and any optional segment filters, then click **Submit**.
- Drill into journal lines or SLA detail by clicking cells in the result table.

### Multiple environments

Open **Settings** → **+ Add** to register more tenants (e.g. `dev1`, `dev4`, `prod`). Each one keeps its own:

- Fusion host, BI Publisher report path, auth mode, credentials.
- OS-keychain entry (tokens never mix between environments).
- SQLite metadata cache (one file per `fusion_host`, isolated under `~/Library/Application Support/ofglpivot/metadata-*.db`).
- Performance / cache-TTL settings.

Switch the live environment via the topbar dropdown — the app rebuilds its connection in-place, no restart. Deleting an environment purges its keychain entries and SQLite cache file automatically.

### Where data lives

For diagnostics or a forced clean state. Close the app first if you delete anything.

| Purpose                       | macOS                                                          | Windows                                                          |
|-------------------------------|----------------------------------------------------------------|------------------------------------------------------------------|
| Settings (saved environments) | `~/Library/Application Support/FusionLens/config.json`         | `%AppData%\FusionLens\config.json`                                |
| Metadata cache (SQLite)       | `~/Library/Application Support/ofglpivot/metadata-*.db`        | `%LocalAppData%\ofglpivot\metadata-*.db`                          |
| SSO tokens & Basic password   | macOS Keychain (service `ofglpivot`)                           | Windows Credential Manager (service `ofglpivot`)                  |
| Logs                          | stdout — visible only when launched from a terminal            | stdout — visible only when launched from `cmd` / PowerShell       |

Environment variables still work as overrides for headless / CI use — `OFGLPIVOT_FUSION_HOST`, `OFGLPIVOT_BI_REPORT_PATH`, `OFGLPIVOT_AUTH_MODE`, `FLENS_USER`, `FLENS_PASSWORD`. Anything set in the environment wins over the saved config file for the active environment.

### Notes & known behaviour

- **No drilldown from pivot rows.** A pivot row spans multiple column values (e.g. one row per natural account, one column per period), so it doesn't map to a single balance with a journal stack to drill into. Switch the result to flat output to follow a balance into its journal / SLA / source document.
- **"Access denied" banner** — selecting a ledger your Fusion role has no Data Access Set grant on shows *"Access denied — your Oracle Fusion role does not include this ledger"*. That's intended; not a bug.
- **First-window onboarding glow** — on a fresh install the Ledger picker pulses with a *"Pick a ledger to start"* callout. It disappears once you've picked any ledger.

---

FusionLens is an independent, unaffiliated tool. Oracle, Oracle Fusion, Smart View, Account Monitor, Account Inspector, BI Publisher, OTBI, and Fusion Data Intelligence are trademarks of Oracle Corporation.
