# Server Health & Hardware Telemetry Monitor

A zero-dependency Python & Linux CLI hardware telemetry utility built for proactive data center server diagnostics, thermal monitoring, and storage threshold alerting.

## Key Capabilities
- **Thermal Sensor Polling:** Interfaces directly with Linux kernel `/sys/class/thermal` nodes to detect CPU thermal throttling risks before hardware shutdowns occur.
- **Storage & I/O Health Auditing:** Tracks root/array partition utilization (`shutil.disk_usage`) and flags capacity thresholds above 85%.
- **Kernel Load & Uptime Telemetry:** Monitors 1-minute, 5-minute, and 15-minute Unix load averages alongside structured JSON alert logging (`server_telemetry_alerts.json`) for automated incident triage.

## Tech Stack
- **Language:** Python 3.x (Standard Library: `os`, `shutil`, `platform`, `json`)
- **Target Systems:** Enterprise Linux Servers & Bare-Metal Nodes
