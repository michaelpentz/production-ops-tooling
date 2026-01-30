# Live Operations & Production Automation

This repository documents automation tools and workflows designed to improve system reliability, reduce operational toil, and mitigate failure modes in live production environments.

The projects included here demonstrate practical approaches to common infrastructure challenges: **race conditions**, **resource contention**, and **unattended system recovery**.

## Operational Principles Demonstrated
Each tool in this repository was built with specific live-operations constraints in mind:

*   **Idempotency & Self-Healing:** Ensuring scripts can run repeatedly or restart automatically without corrupting state (e.g., cleaning stale locks on boot).
*   **Risk-Aware Scheduling:** Decoupling high-CPU tasks from real-time broadcast windows to prevent resource contention.
*   **Failure Mode mitigation:** Designing systems that fail deterministically rather than degrading silently.
*   **Toil Reduction:** Automating error-prone manual tasks to ensure consistency across environments.

---

## Project Overview

### 1. [Raspberry Pi Kiosk: Unattended Display Recovery](./raspberry-pi-kiosk)
**The Challenge:** Maintaining 24/7 uptime for a headless digital signage node with zero on-site staff.
**The Solution:** A self-healing Bash wrapper that manages X11 session locks, handles memory leaks via scheduled power cycles, and ensures deterministic recovery after power loss.

### 2. [Docker Shared Volumes: Solving Race Conditions](./docker-shared-volumes)
**The Challenge:** Non-deterministic "Directory Not Found" errors during the concurrent startup of multiple game server containers.
**The Solution:** Diagnosed an I/O race condition between asset ingest and tokenization processes. Implemented a "Read-Once, Write-Many" architecture with flattened directory structures to eliminate filesystem contention.

### 3. [Media Ingest Tooling: Operator-Controlled Encoding](./python-video-converter)
**The Challenge:** Reducing the manual effort of standardizing video codecs for web playback without impacting the CPU performance of the live broadcast machine.
**The Solution:** A Python/FFmpeg wrapper that enforces VP9/WebM standards. Designed as an **operator-triggered** tool (rather than a background daemon) to ensure encoding never competes with live signal transmission.

---

*Note: Some logic has been simplified or anonymized for public viewing. Proprietary configuration details have been redacted.*
