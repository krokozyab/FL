# FusionLens

**Read-only investigation lens for Oracle Fusion Financials.**

> *"What exactly makes up this balance, and where did it come from?"*

<!-- TODO: record the 8–10s demo GIF (balance → journals → AP invoice → xlsx) and drop into docs/img/demo.gif -->
<p align="center">
  <img src="img/demo.gif" alt="FusionLens demo: balance → journals → SLA → AP invoice → xlsx export" width="720">
</p>

▶️ **[Watch the demo](https://youtu.be/tCHShFHWbZk)** &nbsp;·&nbsp; 📥 **[Setup & install →](#setup--install)**

No instant download here — FusionLens needs a one-time setup on your Fusion tenant before the desktop client can talk to it. Watch the demo first to decide if it's worth the 5 minutes; the install steps below are concrete.

## Who it's for

- **Accountants and controllers** investigating balance differences, unusual movements, unposted activity, reconciliation issues, and period-close exceptions — then exporting the evidence to Excel for workpapers and review.
- **Application support and helpdesk teams** resolving user tickets by inspecting the live accounting trail instead of relying on screenshots, partial exports, and back-and-forth with business users.
- **BI Publisher, OTBI, and reporting developers** validating report logic against the underlying Fusion accounting data, comparing expected results with actual journals and subledger records.
- **Finance systems and ERP teams** triaging cross-functional issues that span configuration, accounting rules, source transactions, reporting logic, and user access.

## Where it fits

FusionLens **complements** Oracle Smart View, Account Monitor, Account Inspector, and Fusion Data Intelligence — it does not replace any of them. Those tools govern, dashboard, monitor, and present. FusionLens does one different job: **rapid tactical investigation** — pivot or flatten GL balances any way you need, drill from any cell into the journal, the SLA line, and the source document, with local filter / sort / export built in.

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

You can use a different catalog folder if your tenant's structure requires it — just update `bi_report_path` in your `config.json` to match (`/Custom/Financials/RP_ARB.xdo` by default). A Fusion administrator or report developer with BIP catalog write access typically handles this step.

### 1. Download the desktop client

[Latest release for macOS & Windows](https://github.com/krokozyab/FL/releases/latest). Each release bundles:

- The application binary — macOS Apple Silicon `.app` (in a `.zip`), or Windows `.exe`.
- `config.example.json` — sample configuration to use as a template.

Place the binary and your `config.json` in the same folder (e.g. `C:\FusionLens\` on Windows, `~/Applications/FusionLens/` on macOS).

### 2. Configure the connection

Copy `config.example.json` to `config.json` and edit:

| Field            | Value                                                                |
|------------------|----------------------------------------------------------------------|
| `fusion_host`    | Your tenant URL, e.g. `https://fa-acme-dev1.fa.us2.oraclecloud.com`. |
| `bi_report_path` | Where you deployed the report, e.g. `/Custom/Financials/RP_ARB.xdo`. |
| `auth_mode`      | `sso` (default) — or `basic` for service accounts. See below.        |

**Authentication modes**

- **`sso` (default)** — at launch FusionLens opens Google Chrome, you sign in to Fusion once, and the Bearer token is captured and reused (with automatic refresh). Requires Chrome installed on the machine.
- **`basic`** — HTTP Basic against a Fusion service-account user; no Chrome needed. Set credentials via environment variables (preferred):

  ```powershell
  # Windows (PowerShell, current session)
  $env:FLENS_USER     = "service_account"
  $env:FLENS_PASSWORD = "..."
  ```

  ```bash
  # macOS / Linux
  export FLENS_USER=service_account
  export FLENS_PASSWORD=...
  ```

  For a permanent setting, use **System Properties → Environment Variables** (Windows) or your shell's rc file (macOS / Linux). Alternative: set `basic_user` / `basic_password` directly in `config.json`. Environment variables override the file when both are set.

### 3. First launch

- **macOS** — Gatekeeper will block the app because the binary isn't signed by an Apple-registered developer. Right-click the app → **Open** → confirm in the dialog. macOS remembers your choice for future launches. (Alternative: **System Settings → Privacy & Security → "Open Anyway"** after the first blocked attempt.)
- **Windows** — SmartScreen will show *"Windows protected your PC"* for the same reason. Click **More info → Run anyway**.

In the FusionLens window:

1. The top-right pill should turn green and say **authenticated**. The environment chip (`DEV1` / `PROD` / …) is derived from your `fusion_host`.
2. Pick a ledger in the left panel — the picker is filtered by your Fusion data-access roles.
3. Set the period range and any optional segment filters, then click **Submit**.
4. Drill into journal lines or SLA detail by clicking cells in the result table.

### Where data lives

For diagnostics or a forced clean state. Close the app first if you delete anything.

| Purpose                  | macOS                                                  | Windows                                            |
|--------------------------|--------------------------------------------------------|----------------------------------------------------|
| Metadata cache (SQLite)  | `~/Library/Application Support/ofglpivot/metadata-*.db` | `%LocalAppData%\ofglpivot\metadata-*.db`          |
| SSO / refresh tokens     | macOS Keychain (service `ofglpivot`)                   | Windows Credential Manager (service `ofglpivot`)  |
| Logs                     | stdout — visible only when launched from a terminal    | stdout — visible only when launched from `cmd` / PowerShell |

### Notes & known behaviour

- **No drilldown from pivot rows.** A pivot row spans multiple column values (e.g. one row per natural account, one column per period), so it doesn't map to a single balance with a journal stack to drill into. Switch the result to flat output to follow a balance into its journal / SLA / source document.
- **"Access denied" banner** — selecting a ledger your Fusion role has no Data Access Set grant on shows *"Access denied — your Oracle Fusion role does not include this ledger"*. That's intended; not a bug.
- **First-window onboarding glow** — on a fresh install the Ledger picker pulses with a *"Pick a ledger to start"* callout. It disappears once you've picked any ledger.

---

FusionLens is an independent, unaffiliated tool. Oracle, Oracle Fusion, Smart View, Account Monitor, Account Inspector, BI Publisher, OTBI, and Fusion Data Intelligence are trademarks of Oracle Corporation.
