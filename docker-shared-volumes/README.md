# Docker Shared Data Volumes & Race Condition Mitigation

**Theme:** Infrastructure Optimization, Debugging Distributed Systems, Storage Efficiency.

## 1. The Operational Challenge
I was managing an infrastructure of multiple stateful game server containers that required access to a shared, massive asset library (50GB+). 

*   **Inefficiency:** Storing copies of this data in each container was redundant and slow.
*   **Failure Mode:** When multiple containers launched simultaneously, I observed intermittent "Directory Not Found" crashes.

## 2. The Diagnosis (Root Cause Analysis)
Investigation revealed a **Race Condition** in the application logic. 
*   Two asynchronous processes (Asset Ingest and Image Tokenizer) attempted to create the same nested directory structure (e.g., `/tokens/goblins/archers/`) at the exact same millisecond.
*   The OS locked the parent directory for one process, causing the second process to fail with a "path does not exist" error.
*   This was a classic non-deterministic failure that only appeared under high-concurrency startup (e.g., restarting the whole stack at once).

## 3. The Solution
I implemented a **Read-Once, Write-Many** architecture using Docker Bind Volumes.

### Architecture Changes:
1.  **Host-Level Storage:** Moved the asset library to a centralized Host directory.
2.  **Bind Mounting:** Mounted the host directory to all containers at runtime.
    *   *Result:* Zero duplication. Setup time reduced from 2 hours (downloading per container) to 2 minutes (mapping existing data).
3.  **Path Flattening:** I refactored the JSON configuration to flatten the directory structure, removing the need for deep recursive creation during startup. This eliminated the race condition entirely.
4.  **Permission Fixing:** Aligned Host (Root/0) and Container (User/1000) permissions using `chown` to ensure the application could read the shared volume without privilege escalation.

## 4. Operational Outcome
*   **Reliability:** Eliminated startup crashes caused by directory contention.
*   **Efficiency:** Reduced disk usage by ~90% across the environment.
*   **Recovery Speed:** Enabled near-instant container replacement since assets persist outside the container lifecycle.

---

### Key Configuration Snippet (JSON Injection)
*Refactoring the path structure to prevent collision:*

```json
{
  "id": "shared-assets-library",
  "packs": [
    { 
      "name": "monsters", 
      "path": "./packs/monsters.db", 
      "system": "dnd5e",
      "locking": "false"
    }
  ]
}
```
