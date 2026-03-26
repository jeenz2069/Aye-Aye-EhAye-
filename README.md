Open WebUI Knowledge Sync
A robust, stateful PowerShell 7 script designed for bulk documentation injection into Open WebUI knowledge collections.

When building out a comprehensive local AI assistant platform, injecting thousands of files—spanning code repositories, operational manuals, and training documents—can lead to timeouts, duplicate entries, and vectorization crashes. This script acts as a resilient pipeline, synchronizing local directory trees into Open WebUI while bypassing common failure points.

✨ Key Features
Stateful Synchronization: Utilizes a state.json ledger to track uploads. On subsequent runs, it skips unmodified files (checked via size and LastWriteTime, or optionally SHA256 hashing), saving massive amounts of time and backend processing.

Master Error Resilience: Wrapped in a global try/catch block. If a file causes an out-of-memory exception, contains bizarre character encoding, or hits a locked state, the script logs the failure and continues to the next file without halting the entire job.

Duplicate Content Bypass: Gracefully intercepts Open WebUI's 400 Bad Request duplicate hash errors. If a file's exact content already exists in the vector database, the script tags it as a success and moves on, keeping your context window lean.

RAG Crash Prevention (Size Filter): Automatically skips phantom files (under 100 bytes by default). This prevents the Open WebUI embedding engine from crashing when trying to vectorize empty boilerplate files (like empty .xml or .config files).

Universal Whitelist & Junk Mover: Only evaluates explicitly allowed file extensions. Unsupported files are automatically moved to a designated Junk folder to keep your target directory clean.

Comprehensive Logging: Outputs detailed telemetry to three formats:

sync-summary.csv: Easy-to-read tabular overview of every file's outcome.

sync-log.jsonl: Deep programmatic logging for automated parsing.

sync-errors.txt: A dedicated, human-readable error log capturing stack traces for quick troubleshooting.

📋 Prerequisites
PowerShell 7+: This script utilizes modern PowerShell syntax and strict error handling. It must be executed via pwsh.exe, not the legacy Windows PowerShell 5.1 (powershell.exe).

Open WebUI & Ollama: A running instance of Open WebUI (presumably backed by Ollama or another LLM provider).

API Token: A valid API JWT from your Open WebUI instance.

Knowledge Collection ID: The specific GUID of the Knowledge Base you are targeting.

⚙️ Configuration
Before running the script, open opewebuiKnowledgeUploader.ps1 and modify the # CONFIGURE HERE section to match your environment:

PowerShell
$OpenWebUIBaseUrl     = "http://localhost:3000"
$OpenWebUIToken       = "<YOUR_API_TOKEN>"
$KnowledgeId          = "<YOUR_KNOWLEDGE_GUID>"
$RootPath             = "C:\Path\To\Your\Documents"
$LogRoot              = "C:\Path\To\Logs"
$JunkPath             = "C:\Path\To\Junk"
⚠️ Security Warning: API Credentials
Never commit this script to a public or shared repository with your $OpenWebUIToken hardcoded in plain-text. Storing service credentials in plain-text configuration files poses a severe security risk.

Recommended Practice: Modify the script to pull the token from a secure local environment variable before scheduling or sharing the code:

PowerShell
$OpenWebUIToken = $env:OPENWEBUI_API_TOKEN
Optional Tuning
$MinSizeBytes: Adjust the minimum file size (default 100).

$UseHash: Set to $true to force SHA256 hash comparisons for change detection instead of Date/Size (slower, but highly accurate).

$IncludeExtensions: Add or remove file extensions from the whitelist array depending on the codebase or documents you are parsing.

🚀 Usage
Open your terminal and execute the script using PowerShell 7:

PowerShell
pwsh -File .\opewebuiKnowledgeUploader.ps1
The console will display a real-time progress bar and a color-coded output feed indicating if files are being [UPLOADED], [SKIPPED], or moved to [JUNK].

📂 Output & State Management
All operational data is stored in the directory defined by $LogRoot:

state.json: Do not delete this file unless you want to force a complete re-upload of every document. It acts as the script's memory.

sync-summary.csv: Check this file in Excel or a CSV viewer to audit the success/failure of a large batch run.

sync-errors.txt: If the console reports "FATAL EXCEPTION ENCOUNTERED", check here for the full script stack trace to identify the exact file and cause.
