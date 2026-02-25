Cybernicol
A SOC-oriented task manager, calendar, and safe demo security console for Kaggle notebooks.

Overview
Cybernicol is a Gradio-based web application designed to run effortlessly in Kaggle notebooks. It provides:

🏠 Command Center — animated KPI dashboard with rich visualizations (risk timeline, attack flow Sankey, asset treemap, MITRE heatmap, network graph)
✅ Operations — task/project management with due dates, progress tracking, ownership, and Gantt-like timeline
📡 Intelligence — threat intel feed + synthetic telemetry explorer (source/severity/asset filtering)
🧩 Automations — safe workflows for defensive enrichment (VirusTotal hash lookup, Webhook notification)
🔌 Integrations — optional connectors for VirusTotal, Shodan, and generic Webhooks (enrichment-only; no offensive actions)
📤 Reports & State — export/import full app state (JSON) for session persistence across Kaggle runs
⚙️ Settings — configurable organization name, risk threshold, simulation seed, and telemetry generation rate
📊 Complex Graphics — Plotly animated charts, Sankey diagrams, treemaps, heatmaps, network graphs, task timelines
Key Features
Safe by Design
✅ No offensive scanning, exploitation, or network probing
✅ Synthetic telemetry and demo scans (hygiene-style findings only)
✅ Defensive integrations only (enrichment + notification)
✅ No credentials stored in code; optional API keys in app state (JSON)
Kaggle-Optimized
✅ Always launches with share=True — reliable public link, no iframe/localhost issues
✅ State persists to /kaggle/working/cybernicol/cybernicol_state.json
✅ Export/import state between sessions
✅ Single self-contained file (cybernicol.py) — easy to copy to any Kaggle notebook
✅ Automatic server cleanup on re-runs (no "No interface is running" errors)
Rich Visuals & Animations
Animated risk timeline (Plotly line chart with markers)
Attack flow Sankey (Source → Tactic → Severity)
Asset pressure treemap (hierarchical event severity)
Technique × Tactic heatmap (MITRE-inspired matrix)
Signal network graph (Source ↔ Asset bipartite layout)
Task Gantt timeline (Created → Due date)
Pulsing KPI cards (custom CSS animations)
Gradient hero banner with floating glow effects
Efficient & Responsive
Capped array sizes (5000 events, 200 scans, 2000 tasks, 500 intel items, 200 audit entries) to keep UI snappy
O(n) synthetic telemetry generation
Lazy-loaded plots (render on-demand, not every callback)
Hardened JSON schema (auto-repair corrupted state)
Installation & Usage
Quick Start (Kaggle Notebook)
Cell 1 — Install dependencies:

python
!pip -q install gradio>=4.0 pandas plotly requests numpy
Cell 2 — Write the app:

python
%%writefile /kaggle/working/cybernicol.py
# (paste the full cybernicol.py code from below)
Cell 3 — Launch:

python
import sys, importlib
if "/kaggle/working" not in sys.path:
    sys.path.insert(0, "/kaggle/working")

import cybernicol
importlib.reload(cybernicol)

cybernicol.launch()
You'll get a public Gradio link (e.g., https://xxxxx.gradio.live) that works for 7 days.

Local Development (outside Kaggle)
bash
git clone https://github.com/your-username/cybernicol.git
cd cybernicol
pip install -r requirements.txt
python cybernicol.py
Then open http://localhost:7860 in your browser.

Architecture
File Structure
text
cybernicol.py
├── Config (paths, API endpoints, limits)
├── Core helpers (time, state, safety checks)
├── State schema + persistence (load/save/harden)
├── Safe simulation engine (telemetry, scans, MITRE tactics)
├── Defensive integrations (VirusTotal, Shodan, Webhook)
├── Workflows (safe orchestration)
├── Visualization helpers (Plotly figures, HTML panels, CSS)
├── Gradio callbacks (state mutations + persistence)
├── Build demo (Blocks + tabs + wiring)
└── Launch helper (always share=True)
State Schema
The app state is a single JSON dict (persisted to /kaggle/working/cybernicol/cybernicol_state.json):

json
{
  "settings": {
    "org_name": "My SOC",
    "risk_threshold": 70,
    "seed": 12345,
    "telemetry_rate": 250,
    "ui_density": "rich"
  },
  "integrations": {
    "virustotal": { "enabled": false, "api_key": "" },
    "shodan": { "enabled": false, "api_key": "" },
    "webhook": { "enabled": false, "url": "" }
  },
  "tasks": [
    {
      "id": "abc123...",
      "title": "Daily Vulnerability Review",
      "type": "SOC Task",
      "category": "Vuln Mgmt",
      "priority": "High",
      "due": "2025-01-15",
      "progress": 25,
      "status": "Open",
      "owner": "Tier-1",
      "tags": ["daily", "vuln"],
      "created_at": "2025-01-13T14:30:00Z",
      "notes": "Review scanner output..."
    }
  ],
  "scans": [
    {
      "id": "scan_xyz...",
      "ts": "2025-01-13T14:30:00Z",
      "facts": { "platform": "Linux", ... },
      "risk_score": 65,
      "findings": [ ... ],
      "status": "Completed"
    }
  ],
  "events": [
    {
      "event_id": "evt123...",
      "ts": "2025-01-13T14:30:00Z",
      "severity": "High",
      "source": "EDR",
      "asset": "vpn-gw-01",
      "tactic": "Privilege Escalation",
      "technique": "T1110 Brute Force",
      "score": 78,
      "message": "..."
    }
  ],
  "intel": [ ... ],
  "workflows": [ ... ],
  "audit_log": [ ... ]
}
Key Callbacks (State Mutations)
All callbacks follow this pattern:

python
def cb_example(state: Dict[str, Any], ...inputs):
    # 1. Mutate state
    state["tasks"].append(...)
    
    # 2. Log event
    log_event(state, "task", "Task created", {...})
    
    # 3. Persist
    save_state(state)
    
    # 4. Return updated state + UI outputs
    return state, updated_output1, updated_output2, ...
This ensures:

Atomicity: state and UI stay in sync
Auditability: every action logged
Durability: disk persisted after each action
Recoverability: state can be exported/imported
Tabs & Features
1. Command Center
KPI cards (pulsing if risk ≥ threshold)
Last Risk Score
Open Tasks
Events (synthetic telemetry)
Intel Items
Generate Telemetry — create N synthetic SOC events (EDR, IAM, Firewall, Proxy, Email, CloudTrail, DNS, WAF sources; MITRE tactics & techniques)
Run Safe Demo Scan — compute pseudo-risk score + hygiene findings
Complex Graphs (6 tabs worth):
Risk Timeline (animated by hour)
Attack Flow (Sankey: Source → Tactic → Severity)
Asset Pressure (treemap)
Technique/Tactic Heatmap
Signal Network (bipartite source/asset graph)
Latest Scan Findings (HTML panel)
Scan History (last 50 scans)
2. Operations
Create Task
Title, type, category, priority
Due date, progress %, status
Owner, tags, notes
Task List (all tasks, interactive dataframe)
Edit/Delete (select task, modify, save or delete)
Task Timeline (Gantt-like: Created → Due date, colored by priority)
3. Intelligence
Threat Intel Feed
Add demo intel (severity + message)
Persistent list (most recent first)
Event Explorer
Filter by severity, source, asset
Browse last 300 synthetic telemetry events
See tactic, technique, score
4. Automations
Create Workflow
Name, trigger type (manual)
Optional steps: VirusTotal hash lookup, Webhook notify
Run Workflow
Select workflow, provide SHA256 (optional)
Returns JSON result of each step execution
Safe by design: only enrichment steps, no exploitation
5. Integrations
VirusTotal
Enable, set API key
Test hash lookup (defensive enrichment)
Shodan
Enable, set API key
Test host lookup (only authorized IPs)
⚠️ User is responsible for authorization
Webhook
Enable, set URL
Test POST payload delivery
Great for SIEM/SOAR integration
6. Reports & State
Export Full State (download JSON snapshot)
Import State (upload JSON to restore session)
Useful for moving state between Kaggle sessions
Auto-repairs schema on import
Audit Log (latest 50 events: timestamp, kind, message, metadata)
7. Settings
Organization / SOC Name (display label)
Risk Threshold (0–100; affects scan finding severity)
Simulation Seed (reproducible telemetry generation)
Telemetry Batch Size (default N events to generate per click)
Save Settings or Reset to Defaults
Simulation Engine
Telemetry Generation
python
generate_telemetry(state, n=250)
Creates N synthetic events per call:

Severity: Info, Low, Medium, High, Critical (weighted distribution)
Source: EDR, IAM, Firewall, Proxy, Email, CloudTrail, DNS, WAF
Asset: vpn-gw-01, ad-dc-01, mail-01, db-01, k8s-node-02, workstation-17, jumpbox-01, proxy-01, gitlab-01, siem-01
MITRE Tactic/Technique: 11 tactics × 10 techniques
Score: 0–100 (influenced by severity, with jitter)
Deterministic: seed-based, so same settings generate reproducible telemetry
Safe Demo Scan
python
simulated_security_scan(state)
Computes risk score from:
Mean score of recent 200 events
Platform bump (Windows +8, Kaggle −4)
Time-based jitter (±9)
Result: 0–100
Produces 3 findings:
High: Risk threshold exceeded (if risk ≥ threshold)
Medium: MFA & access policy recommendations
Low: Logging & retention review
Safe: no actual scanning, no network probes, no exploitation.

Defensive Integrations
VirusTotal (Hash Enrichment)
python
ok, data = virustotal_hash_lookup(api_key, sha256_hash)
Passive: queries https://www.virustotal.com/api/v3/files/{hash}
Returns: metadata, detection counts, analysis results
Use case: triage suspicious file hashes
Shodan (Host Reconnaissance)
python
ok, data = shodan_host_lookup(api_key, ip_address)
Passive: queries https://api.shodan.io/shodan/host/{ip}
Returns: open ports, services, banners, location
⚠️ User responsibility: only query authorized IPs
Use case: enrich incident response with host details
Webhook (Notifications)
python
ok, resp = send_webhook(url, payload)
POSTs a JSON payload to your webhook
Great for: alerting SIEMs, SOAR platforms, Slack, teams, custom APIs
Payloads include: workflow ID, timestamp, enrichment results
UI Styling & Animations
CSS Features
Gradient hero banner with floating glow effects (7s animation loop)
Pulsing KPI cards (2s pulse animation when risk ≥ threshold)
Rounded corners, soft borders, opacity layers for modern look
Badges & small text for hierarchy and readability
All CSS is inlined in command_center_html() for portability.

Persistence & Export
State Lifecycle
Load (on app start or import)

Read from /kaggle/working/cybernicol/cybernicol_state.json
If missing/corrupted, initialize defaults
Auto-repair schema (add missing keys)
Mutate (via callbacks)

User adds task, generates telemetry, runs scan, etc.
State dict updated in memory
log_event() appends audit entry
save_state() persists to disk
Export (user initiates)

Download full state JSON from Reports tab
Perfect for backing up state across sessions
Import (user initiates)

Upload JSON file
Auto-repair schema
Merge into current state (overwrites on ID conflicts)
Log import event
Performance Tuning
Array Size Limits
python
MAX_EVENTS = 5000      # synthetic telemetry
MAX_AUDIT = 200        # audit log entries
MAX_SCANS = 200        # security scans
MAX_TASKS = 2000       # tasks/projects
MAX_INTEL = 500        # threat intel items
Configured to keep UI snappy; adjust if you want larger notebooks.

Computation Strategy
Telemetry generation: O(n), no database, pure list ops
Visualization: lazy (only render on demand, not every callback)
Dataframe ops: .tail() to keep last N rows (avoid full scans)
Network calls: best-effort (60s timeout); failures logged, not fatal
Development & Extending
Adding a New Integration
Write the API function (e.g., custom_api_lookup(...))

python
def my_api_lookup(api_key: str, param: str) -> Tuple[bool, Any]:
    # Your API call here
    return ok, data
Add to integrations dict

python
state["integrations"]["my_api"] = {"enabled": False, "api_key": ""}
Create a callback

python
def cb_test_my_api(state, param):
    api_config = state.get("integrations", {}).get("my_api", {})
    if not api_config.get("enabled"):
        return {"ok": False, "error": "Not enabled"}
    ok, data = my_api_lookup(api_config["api_key"], param)
    log_event(state, "integration_test", "My API tested", {"ok": ok})
    save_state(state)
    return {"ok": ok, "data": data}
Wire it in Gradio

python
btn_test_my_api.click(cb_test_my_api, inputs=[state, my_param], outputs=[my_output_json])
Adding a New Visualization
Write the Plotly function

python
def fig_my_chart(state):
    df = ...  # build dataframe
    fig = px.scatter(df, ...)
    fig.update_layout(height=360, ...)
    return fig
Add to Command Center callback

python
def cb_refresh_command_center(state):
    ...
    my_fig = fig_my_chart(state)
    return state, ..., my_fig, ...
Wire a new gr.Plot() in the layout

python
my_plot = gr.Plot()
...
btn_refresh.click(..., outputs=[..., my_plot, ...])
Local Testing (non-Kaggle)
python
# Run locally (no Kaggle):
python cybernicol.py

# Or in a Python REPL:
from cybernicol import demo, launch
launch()  # Opens http://localhost:7860
Troubleshooting
"No interface is running right now"
The Gradio server crashed or didn't start.
Fix: Re-run the launch cell. It calls demo.close() first to clean up stale servers.
Check Kaggle cell output for error messages.
"Rerunning server..."
You re-ran the launch cell while a server was still running.
Expected: Gradio auto-restarts. Wait ~3–5 seconds, then refresh the browser.
State not persisting between runs
By default, Kaggle notebooks are ephemeral.
Solution: Use Reports & State → Export to download your state JSON before the session ends.
Then re-import it in a new notebook run.
Or: ask Kaggle support to use persistent storage (paid tier).
Plots not rendering / showing blank
Large datasets (>3000 events) can slow Plotly.
Fix: Reduce MAX_EVENTS or filter older events: df.tail(1000).
API integration test fails
Check: API key is correct, API is reachable, you have auth.
VirusTotal: ensure key has file lookup permission.
Shodan: ensure key is valid and you have IP query credits.
Webhook: ensure URL is reachable and accepts POST.
Security & Privacy
What is stored?
Task descriptions, notes, owner names (internal data)
Integration API keys (if you enable them) in the local JSON state file
Audit log of all actions (user can view/export)
Synthetic telemetry (no real network data)
Best practices:
✅ Keep /kaggle/working/cybernicol/ private — it contains state + API keys
✅ Export state before session ends if you want to preserve it
✅ Use environment variables for API keys in production (override in code if needed)
✅ Don't commit state JSON to version control — add to .gitignore
✅ Review the Audit Log regularly to catch unauthorized changes
License
MIT License. See LICENSE file for details.

Contributing
Pull requests welcome! To contribute:

Fork the repo
Create a feature branch (git checkout -b feature/my-feature)
Make changes (ensure code is well-commented)
Test locally or on Kaggle
Push and open a PR
Authors
Built for Kaggle notebook users who need a safe, no-setup SOC dashboard.
Designed to be copy-paste-ready — single file, all dependencies minimal.
Changelog
v1.0.0 (2025-01-14)
Initial release
7 tabs (Command Center, Operations, Intelligence, Automations, Integrations, Reports, Settings)
6 complex visualizations (animated timeline, Sankey, treemap, heatmap, network, Gantt)
Synthetic telemetry generation
Safe demo scans
Task management with timeline
Defensive integrations (VT, Shodan, Webhook)
Workflow automation
Full state export/import
Audit logging
FAQ
Q: Can I use this in production?
A: Not directly. This is a Kaggle notebook demo tool. For production SOC dashboards, integrate the concepts into a proper web framework (FastAPI, Django, etc.) with a real database.

Q: Is the "scan" actually scanning my system?
A: No. It's a simulated demo scan that produces pseudo-risk scores and hygiene findings. No network probes, no exploits, no real vulnerabilities.

Q: Can I add real threat intel feeds (MISP, OTX, etc.)?
A: Yes. Add an intel_fetch() function that calls your feed API, and trigger it via a workflow or button callback.

Q: How do I share this with my team?
A: Export your state JSON, send it to teammates, they import it in their Kaggle notebook. Or deploy to Hugging Face Spaces using gradio deploy (permanent public link).

Q: Can I run this locally (not Kaggle)?
A: Yes. Download cybernicol.py, pip install -r requirements.txt, then python cybernicol.py. It will open at http://localhost:7860.

Questions? Open an issue on GitHub or reach out via [contact info].