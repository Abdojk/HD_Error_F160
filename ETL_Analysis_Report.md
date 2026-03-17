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

## Phase 1b — Quick Conversion: `tracerpt.exe` (No Install Required)

`tracerpt.exe` is **built into every Windows installation** — no download or setup needed.

### Step 1 — Open an Administrator Command Prompt

Right-click **Command Prompt** (or **PowerShell**) → **Run as administrator**.

### Step 2 — Run the Conversion Command

**Option A — XML output (recommended, richer structure):**
```cmd
tracerpt "C:\Users\User\OneDrive - Info-sys\Documents\GitHub\HD_Error_F160\Trace.etl" -o "C:\Users\User\OneDrive - Info-sys\Documents\GitHub\HD_Error_F160\Trace_Export.xml" -of XML -lr
```

**Option B — CSV output:**
```cmd
tracerpt "C:\Users\User\OneDrive - Info-sys\Documents\GitHub\HD_Error_F160\Trace.etl" -o "C:\Users\User\OneDrive - Info-sys\Documents\GitHub\HD_Error_F160\Trace_Export.csv" -of CSV -lr
```

**Key flags:**
- `-of XML` / `-of CSV` — output format
- `-lr` — use local resources for decoding event payloads
- `-summary Trace_Summary.txt` — (optional) produces a summary file alongside the export

### Step 3 — Check the Output

1. Open the exported file
2. **Search for `LedgerJournalTransDaily`** — if you find it, the conversion captured the X++ payloads successfully
3. **If the output is mostly hex/GUIDs and no X++ class names are visible**, the D365 ETW provider manifests aren't registered on your machine. In that case:
   - Try running the same command **on the machine the trace was captured on** or **on a D365 dev VM**
   - Or try **PerfView** as a backup (free single-exe download from [github.com/microsoft/perfview](https://github.com/microsoft/perfview/releases))

### Step 4 — Add to the Repository

1. Place the exported file in the repo root folder
2. Accepted file names: `Trace_Export.csv`, `Trace_Export.xml`, or `Trace_Export.txt`
3. If the file is **larger than 100MB**, either:
   - Filter it to only include rows containing `LedgerJournal` or `FormServer`
   - Or zip it before committing
4. Commit and push, or simply start a new analysis session

### Important Notes

- **Best results:** Run on the same machine the trace was captured on, or on a D365 F&O dev/build VM where the D365 ETW providers are registered
- **On a plain Windows PC:** Event structure will decode, but X++ payload fields (class names, method names) may appear as hex values
- **Expected output size:** 50–200 MB for a 13 MB ETL (XML is verbose, CSV is smaller)

---

## Phase 1c — Fallback Options (If `tracerpt` Output Is Insufficient)

If `tracerpt` output is mostly hex, here are backup approaches:

| Option | Tool | Install? | Notes |
|--------|------|----------|-------|
| **PerfView** | Free Microsoft tool | No install — single .exe | Good at resolving payloads even without registered manifests |
| **D365 Trace Parser** | Official D365 tool | From LCS or pre-installed on dev VMs | Best decoding, but requires D365 environment |
| **Query D365 AOT directly** | Visual Studio on dev VM | N/A | Skips the trace entirely — finds *all* deployed customizations on the form |

### PerfView Quick Steps
1. Download `PerfView.exe` from [github.com/microsoft/perfview/releases](https://github.com/microsoft/perfview/releases)
2. Run it → **File → Open** → select `Trace.etl`
3. Go to the **Events** view
4. Filter for `Microsoft-Dynamics` providers
5. Export matching events to CSV

### Query D365 AOT Directly (No Trace Needed)
If no conversion method works, you can find all customizations on `LedgerJournalTransDaily` directly from the D365 development environment:
1. Open **Visual Studio** on a D365 dev VM
2. Open **Application Explorer**
3. Search for `LedgerJournalTransDaily` → right-click → **Find all references**
4. This shows ALL extensions, event handlers, and CoC classes targeting the form
5. Provide the results list and I'll produce the customization report

---

### D365 Trace Parser (Original Method)

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
| Phase 1b — `tracerpt.exe` Guide | COMPLETE — recommended first attempt (no install needed) |
| Phase 1c — Fallback Options | COMPLETE — PerfView, Trace Parser, or direct AOT query |
| Phase 2 — Form Isolation | BLOCKED — awaiting converted trace file |
| Phase 3 — Customization Detection | BLOCKED — awaiting converted trace file |
| Phase 4 — Findings Report | BLOCKED — awaiting converted trace file |
