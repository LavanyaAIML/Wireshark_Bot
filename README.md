# Wireshark Automation Bot

Automated PCAP analysis and reporting tool built on Kali Linux — ingests packet captures and produces structured HTML/JSON traffic reports without manual Wireshark digging.

## Overview

Manually triaging packet captures in Wireshark is slow and doesn't scale. This bot automates the first pass of traffic analysis: it parses a `.pcap` file, extracts protocol distribution, top talkers, port activity, and conversation flows, and renders the results as a clean, shareable HTML report (plus a machine-readable JSON export for downstream tooling).

It was built and tested against a real-world traffic-analysis PCAP (3,053 packets) and used to surface indicators consistent with MITRE ATT&CK technique **T1071 (Application Layer Protocol)**, demonstrating its use as a lightweight DFIR triage aid, not just a protocol counter.

## Features

- **One-command analysis** — `python3 main.py --mode analyze --pcap <file>` and get a full report in seconds
- **Dual output formats** — polished HTML report for human review, JSON for scripting/SIEM ingestion
- **Summary statistics** — total packets, average/max packet size, alert count at a glance
- **Protocol breakdown** — packet counts by protocol (TCP, TLS, HTTP, DNS, ARP, NBNS, LLMNR, DHCPv6, etc.)
- **Top source IPs** — ranked by packets sent, to quickly spot high-volume or anomalous hosts
- **Top destination ports** — surfaces common and unusual service ports in the capture
- **Top conversations** — bidirectional flow pairs ranked by packet count, useful for spotting beaconing or data-transfer patterns
- **Extensible alerting** — report includes an alert counter, designed as a hook point for custom detection rules

## Tech Stack

| Component | Purpose |
| --- | --- |
| `tshark` | Packet capture parsing (CLI companion to Wireshark) |
| `pyshark` | Python wrapper around `tshark` for programmatic packet access |
| `Jinja2` | HTML report templating |
| Python 3 | Core automation logic |

Developed and tested on **Kali Linux**.

## Installation

```bash
# Clone the repo
git clone https://github.com/LavanyaAIML/wireshark_bot.git
cd wireshark_bot

# Set up a virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

**Prerequisites:** `tshark` must be installed and available on your `PATH` (comes bundled with Wireshark, or install via `sudo apt install tshark` on Debian/Kali).

## Usage

```bash
python3 main.py --mode analyze --pcap /path/to/capture.pcap
```

This generates timestamped reports in `./reports/`:

```
[*] Analyzing: /path/to/capture.pcap
[+] Analysis complete: 3053 packets, 0 alerts
[+] HTML report → ./reports/report_20260610_112925.html
[+] JSON report → ./reports/report_20260610_112925.json
```

Open the HTML report directly in a browser:

```bash
firefox ./reports/report_20260610_112925.html
```

## Sample Report

Running the bot against a 3,053-packet capture produced:

| Metric | Value |
| --- | --- |
| Total packets | 3,053 |
| Avg packet size | 819.7 bytes |
| Max packet size | 1,514 bytes |
| Alerts fired | 0 |

**Top protocols:** TCP (2,500), TLS (373), HTTP (41), NBNS (24), DNS (22), ARP (19), and smaller volumes of DATA-TEXT-LINES, LLMNR, DHCPv6, and IMAGE-GIF.

The report also breaks down top source IPs, top destination ports (port 80 dominating at 611 packets), and the top 10 host-pair conversations — all rendered in a clean card-and-table layout suitable for sharing with a team or attaching to an incident report.

## Analysis Highlights

Using this tool against the sample capture, traffic patterns consistent with **MITRE ATT&CK T1071 – Application Layer Protocol** were identified, suggesting command-and-control-style communication hidden within legitimate-looking application traffic. This illustrates the bot's role as a first-pass triage layer: it won't replace manual deep-dive analysis, but it dramatically cuts the time needed to decide *where* to look.

## Project Structure

```
wireshark_bot/
├── main.py              # Entry point — CLI argument parsing and orchestration
├── reports/             # Generated HTML/JSON reports (timestamped)
├── requirements.txt     # Python dependencies
└── README.md
```

## Roadmap

- [ ] Configurable alert/detection rules (beaconing, unusual ports, DNS tunneling heuristics)
- [ ] CLI flag for custom report output directory
- [ ] Batch mode for analyzing multiple PCAPs in one run
- [ ] Optional MITRE ATT&CK technique tagging in the report itself

## License

MIT License — free to use, modify, and distribute with attribution.

---

*Built as part of ongoing DFIR and network traffic analysis practice.*
