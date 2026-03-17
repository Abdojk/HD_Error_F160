# D365 F&O ETL Trace Analysis — LedgerJournalTransDaily Customization Detection

**Form:** LedgerJournalTransDaily
**Trace File:** Trace.etl
**Repo:** https://github.com/Abdojk/HD_Error_F160
**Analysis Date:** 2026-03-17

---

## Phase 1 — Pre-Analysis: ETL Readability Check

### File Properties

| Property | Value |
|----------|-------|
| File name | `Trace.etl` |
| File size | 13,200,388 bytes (13 MB) |
| Format | Windows Event Trace Log (ETL v2), binary |
| Extractable ASCII strings | ~211,000 fragments |
| Extractable UTF-16LE strings | ~12,400 fragments |

### D365 Identifiers Found in Binary Scan

| Identifier | Encoding | Occurrences | Notes |
|------------|----------|-------------|-------|
| `Microsoft-Dynamics-AX-FormServer` | ASCII | 12+ | ETW provider name — confirms this is a D365 F&O trace |
| `AOS` | UTF-16LE | 1+ | Application Object Server reference |
| `Batch` | UTF-16LE | 1+ | Batch framework reference |
| `SessionState` | UTF-16LE | 2+ | Session lifecycle events |
| `@tzres.dll` | UTF-16LE | 2 | Timezone resource references |
| `2026-3-17` | UTF-16LE | 1 | Trace capture date |

### X++ Execution Data — NOT Found

The following critical identifiers were searched in **both ASCII and UTF-16LE** encodings and were **NOT found**:

- `LedgerJournalTransDaily` — target form name
- `LedgerJournalTrans` — parent table
- `LedgerJournalEngine` — journal engine class
- `FormRun` / `xFormRun` — form runtime classes
- `XppRuntime` — X++ runtime markers
- `DimensionAttribute` — financial dimension classes
- `MainAccount` — chart of accounts class
- `_Extension` — extension class suffix
- Any customer/partner prefixed classes (`AK_`, `CUS_`, `ISV_`, `CON_`)

### Verdict

**The ETL file is binary and cannot be parsed directly.** The X++ method-level execution data (class names, method calls, form lifecycle events, SQL queries) is stored in structured binary event payloads that require the ETL manifest/schema to decode. Standard string extraction utilities cannot access this data.

**Analysis Phases 2–4 cannot proceed until the file is converted to a readable format.**

---

## Phase 1b — Conversion Guide: ETL to Readable Format

### Overview

The D365 **Trace Parser** tool decodes the binary ETL file using the D365 ETW provider manifests, producing human-readable output that includes X++ class names, method calls, SQL statements, and form lifecycle events.

### Step 1 — Obtain the Trace Parser

**Option A — From LCS (Lifecycle Services):**
1. Log in to [LCS](https://lcs.dynamics.com)
2. Navigate to **Shared asset library** → **Tools**
3. Download **Trace Parser** (look for the latest version)
4. Install on a machine with .NET Framework 4.7.2+

**Option B — From a D365 Dev/Build VM:**
The Trace Parser is pre-installed on D365 development and build VMs at:
```
K:\PerfSDK\PerfTools\TraceParser\
```
or
```
C:\PerfSDK\PerfTools\TraceParser\
```
Run `Microsoft.Dynamics.AX.Tracing.TraceParser.exe` from that directory.

**Option C — From the D365 installation media:**
Check the following path on your D365 environment:
```
J:\AosService\PackagesLocalDirectory\Bin\
```
Look for `Microsoft.Dynamics.AX.Tracing.TraceParser.exe`.

### Step 2 — Import the ETL File

1. Launch **Trace Parser**
2. Select **File → Open Trace** (or the "Import trace" button)
3. Browse to the ETL file:
   ```
   C:\Users\User\OneDrive - Info-sys\Documents\GitHub\HD_Error_F160\Trace.etl
   ```
4. The tool will process the file (this may take a few minutes for a 13MB trace)
5. Once imported, the Trace Parser will display the decoded events in its grid view

### Step 3 — Export to a Readable Format

**Recommended: CSV export** (easiest to process programmatically)

1. After import, select **all events** in the grid (Ctrl+A)
2. Go to **File → Export** or right-click → **Export to CSV**
3. Save as:
   ```
   C:\Users\User\OneDrive - Info-sys\Documents\GitHub\HD_Error_F160\Trace_Export.csv
   ```

**Alternative: Copy to clipboard**
1. Select all events (Ctrl+A)
2. Right-click → **Copy**
3. Paste into a new text file and save as `Trace_Export.txt`

**Alternative: XML export** (if available in your Trace Parser version)
1. File → Export → XML
2. Save as `Trace_Export.xml`

### Step 4 — Verify the Export

Before committing the export, open it and verify you can see entries like:
- X++ method names (e.g., `FormRun.init`, `FormDataSource.executeQuery`)
- Class names (e.g., `LedgerJournalEngine`, `LedgerJournalCheckPost`)
- Form names (e.g., `LedgerJournalTransDaily`)
- SQL statements (if the trace captured data layer events)
- Timestamps for each event

### Step 5 — Add to the Repository

1. Place the exported file in the repo root:
   ```
   C:\Users\User\OneDrive - Info-sys\Documents\GitHub\HD_Error_F160\
   ```
2. Accepted file names (any of these):
   - `Trace_Export.csv`
   - `Trace_Export.txt`
   - `Trace_Export.xml`
3. Commit and push to the repo, or simply let the next analysis session pick it up from the local folder

### Step 6 — Re-run Analysis

Once the converted file is available in the repo, re-run this analysis. Phases 2–4 will then:
1. **Phase 2** — Isolate all `LedgerJournalTransDaily` form session events
2. **Phase 3** — Detect customizations (extensions, CoC overrides, custom tables/fields, financial dimension logic, security objects)
3. **Phase 4** — Produce the structured Customization Analysis Report

---

## Current Status

| Phase | Status |
|-------|--------|
| Phase 1 — ETL Readability Check | COMPLETE — file is binary, not directly parseable |
| Phase 1b — Conversion Guide | COMPLETE — see instructions above |
| Phase 2 — Form Isolation | BLOCKED — awaiting converted trace file |
| Phase 3 — Customization Detection | BLOCKED — awaiting converted trace file |
| Phase 4 — Findings Report | BLOCKED — awaiting converted trace file |
