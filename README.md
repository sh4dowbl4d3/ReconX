# ReconX

<p align="center">
  <img src="Screenshots/Screenshot1.png" alt="ReconX Screenshot" width="800">
</p>

ReconX is a network reconnaissance CLI tool for host discovery, port scanning, CVE lookups, risk assessment, and report generation with Nmap.

## Features

- Host discovery using ping sweeps and ARP scans
- Port scanning with SYN, TCP connect, and full 65,535-port scans
- Service version fingerprinting and OS detection with confidence scoring
- Evasion options including decoys, packet fragmentation, MAC spoofing, custom TTLs, and timing templates
- Vulnerability scanning with the Nmap Scripting Engine (NSE)
- CVE lookups via the CIRCL API with local caching
- Multi-factor risk scoring for individual hosts and scan aggregates
- Report export in HTML and PDF formats
- Recurring scans via cron schedules and daemon mode
- Terminal menu (TUI) and standalone HTML dashboard for viewing scan results

## Installation

### Prerequisites

- Python 3.8 or newer
- Nmap 7.x installed and available on PATH
- pip or pipx

### 1. Install Nmap

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install -y nmap

# Fedora / RHEL
sudo dnf install -y nmap

# Arch Linux
sudo pacman -S nmap

# macOS
brew install nmap

# Windows
winget install InsecureCommunity.Nmap
# or download from https://nmap.org/download.html
```

### 2. Install ReconX

#### Option A: pipx (recommended)

pipx installs ReconX into an isolated environment and exposes the `reconx` command globally.

```bash
# Install pipx if needed
python3 -m pip install --user pipx
python3 -m pipx ensurepath

# Clone and install
git clone https://github.com/sh4dowbl4d3/ReconX.git
cd ReconX
pipx install .
```

After running `pipx ensurepath`, restart your terminal or reload your shell profile.

#### Option B: pip user install

Installs ReconX into your user site-packages directory.

```bash
git clone https://github.com/sh4dowbl4d3/ReconX.git
cd ReconX
pip install --user .
```

Make sure `~/.local/bin` is in your `PATH`:

```bash
# Linux
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Windows (PowerShell)
# Add %APPDATA%\Python\Scripts to your PATH environment variable
```

#### Option C: editable install for development

```bash
git clone https://github.com/sh4dowbl4d3/ReconX.git
cd ReconX
pip install -e .
```

This links the repository directly to your Python environment so local edits take effect immediately while keeping the `reconx` command available globally.

### 3. Verify installation

```bash
reconx --help
```

The `reconx` command is ready to use directly from any terminal session without activating a virtual environment.

### 4. Optional PDF report support

```bash
pip install fpdf2
# or if using pipx:
pipx run reconx pip install fpdf2
```

Scan results, reports, and cached CVE data are saved in `~/.local/share/reconx/` on Linux, `~/Library/Application Support/reconx/` on macOS, or `%APPDATA%/reconx/` on Windows.

## Quick start

```bash
# Show help
reconx --help

# Show version
reconx --version

# Scan a target directly (default action without subcommands)
reconx example.com
reconx 192.168.1.1
reconx https://example.com

# Interactive terminal menu
reconx menu

# Full scan summary
reconx all
```

## Usage

### Scanning

Running `reconx` with a target starts a scan directly:

```bash
# Basic scan (SYN scan on top 1,000 ports, service version detection, and OS detection)
reconx example.com
reconx 192.168.1.0/24

# Quick scan for host discovery only
reconx {Target} --quick

# Deep scan across all 65,535 ports
reconx {Target} --deep

# Explicit standard scan
reconx {Target} --standard

# Scan with service banner grabbing
reconx {Target} --banners
```

### Stealth scanning

```bash
# Stealth mode with SYN scanning, slow timing, randomized decoys, packet fragmentation, and randomized MAC
reconx target.com --stealth

# Custom decoy IP addresses
reconx 10.0.0.1 --decoy 10.0.0.2,10.0.0.3,10.0.0.4

# Packet fragmentation with custom source port
reconx {Target} --fragment --source-port 53

# MAC spoofing, custom TTL, and timing template
reconx {Target} --spoof-mac 0 --ttl 64 --timing 1

# Bad checksum scan
reconx {Target} --badsum

# Combined stealth scan options
reconx target.com --stealth --decoy RND:5 --source-port 1234 --data-length 100 --ttl 128

# Stealth vulnerability scan
reconx vuln-scan {Target} --stealth
```

### Vulnerability scanning

```bash
# Run Nmap NSE vulnerability scripts
reconx vuln-scan {Target}
```

### CVE lookup

```bash
# Lookup CVEs for all discovered services
reconx cve-lookup --all

# Lookup CVEs for a specific service
reconx cve-lookup --service ssh --version "OpenSSH 7.4"
```

### Risk assessment

```bash
# Show overall and per-host risk scores
reconx risk-score
```

### Report generation

```bash
# HTML report
reconx report --html

# PDF report (requires fpdf2)
reconx report --pdf

# Custom output path
reconx report --html --output ./my_report.html
```

### Scheduled scanning

```bash
# List schedules
reconx schedule list

# Add a daily scan
reconx schedule add {Target} daily

# Weekly scan with deep profile
reconx schedule add {Target} weekly --profile deep

# Remove schedule
reconx schedule remove 1

# Toggle schedule on/off
reconx schedule toggle 1

# Run daemon (checks every 60s for due scans)
reconx schedule daemon
```

### Results display

```bash
reconx status        # Scan summary
reconx hosts         # Live hosts
reconx ports         # Open ports
reconx services      # Service versions
reconx os            # OS fingerprints
reconx vulns         # Vulnerability findings
reconx phases        # Scan phase breakdown
reconx all           # Full report
reconx menu          # Interactive menu
```

### Data management

```bash
# Clear all cached scan data, raw output, reports, and CVE cache
reconx clear
```

### Uninstall

```bash
# Remove ReconX and optionally clean up scan data
reconx uninstall
```

Remove manually:

```bash
pip uninstall reconx
rm -rf ~/.local/share/reconx    # Linux (adjust for your OS)
```

## Project structure

```
ReconX/
├── pyproject.toml          # Package config & entry point
├── reconx/
│   ├── __init__.py         # Package init, version
│   ├── cli.py              # Entry point & argument parsing
│   ├── display.py          # Terminal UI, tables, show commands, menu
│   ├── scanner.py          # Nmap orchestration, parsers, cache I/O
│   ├── paths.py            # XDG-compliant data directory paths
│   ├── cve_lookup.py       # CVE database querying (CIRCL API)
│   ├── report_gen.py       # HTML & PDF report generation
│   ├── risk_scoring.py     # Multi-factor risk assessment engine
│   └── scheduler.py        # Cron-based scan scheduler
├── scans/                  # (legacy directory; data now stored in XDG directory)
├── reports/                # (legacy directory; data now stored in XDG directory)
├── requirements.txt        # Python dependencies
└── README.md
```

## Requirements

- Python 3.8 or newer
- Nmap 7.x on PATH
- fpdf2 (optional, required for PDF report exports)
- pipx (optional, recommended for isolated CLI installation)

### Data storage

| Platform | Data directory |
|---|---|
| Linux | `~/.local/share/reconx/` (or `$XDG_DATA_HOME/reconx/`) |
| macOS | `~/Library/Application Support/reconx/` |
| Windows | `%APPDATA%/reconx/` |

## Workflow

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Host        │      │  Port/       │      │  Service     │
│  Discovery   │────▶ │  Service     │────▶ │  Version     │
│  (nmap -sn)  │      │  Scan        │      │  Detection   │
└──────────────┘      └──────────────┘      └──────────────┘
                                                   │
┌──────────────┐     ┌──────────────┐              │
│  OS          │     │  NSE Vuln    │  ◀───────────┘
│  Fingerprint │     │  Scan        │
└──────────────┘     └──────┬───────┘
                            │
                    ┌───────▼───────┐
                    │  CVE          │
                    │  Enrichment   │
                    └───────┬───────┘
                            │
                    ┌───────▼───────┐
                    │  Risk         │
                    │  Assessment   │
                    └───────┬───────┘
                            │
                    ┌───────▼───────┐
                    │  Report       │
                    │  (HTML/PDF)   │
                    └───────────────┘
```

## Security notes

- Scan only networks and systems that you have explicit authorization to test.
- Stealth features are intended for authorized security assessments and CTF environments.
- Scan data is stored locally. Service names and version strings query the public CIRCL API during CVE lookups.
- Target inputs are validated before execution to prevent command injection.

## License

[GNU GPLv3](LICENSE)
