# Python Media Ingest Tooling (FFmpeg Wrapper)

**Theme:** Toil Reduction, Compute Isolation, Standardization.

## 1. The Operational Challenge
My team was manually converting video files for web playback using inconsistent settings.
*   **Toil:** Engineers spent hours on repetitive CLI commands.
*   **Risk:** Running heavy encoding jobs on the same workstations used for live broadcasts caused CPU spikes, risking frame drops in the outgoing signal.

## 2. The Solution
I built a Python wrapper for `FFmpeg` to standardize the ingest workflow.

### Key Operational Controls:
1.  **Compute Isolation (Manual Trigger):** Unlike standard "watch folder" automation, this script is designed to be **manually triggered**. This ensures an operator never accidentally starts a CPU-intensive encode during a live segment.
2.  **Idempotency:** The script checks `if not os.path.exists(output_path)` before running. If the script is interrupted and re-run, it skips files that were already successfully processed.
3.  **Standardization:** Enforces **VP9/WebM** codecs at specific bitrates, removing human error from the parameter selection.

## 3. The Automation Code (`mp4towebm.py`)

```python
import os
import subprocess
import shutil

source_dir = "./ingest"
output_dir = "./ready"
archive_dir = "./archive"

# Loop through files
for filename in os.listdir(source_dir):
    if filename.endswith(".mp4"):
        
        # Define Paths
        input_path = os.path.join(source_dir, filename)
        output_path = os.path.join(output_dir, filename.replace(".mp4", ".webm"))
        
        # Idempotency Check (Don't double work)
        if not os.path.exists(output_path):
            print(f"Starting encode for: {filename}")
            
            # Execute FFmpeg via Subprocess (Secure Argument Handling)
            try:
                subprocess.run([
                    "ffmpeg", 
                    "-i", input_path, 
                    "-c:v", "libvpx-vp9", 
                    "-b:v", "0", 
                    "-crf", "30", 
                    output_path
                ], check=True)
                
                # Move source to archive on success
                shutil.move(input_path, archive_dir)
                print(f"Success: {filename} moved to archive.")

            except subprocess.CalledProcessError:
                print(f"Error: Failed to encode {filename}. Skipping.")
```

## 4. Operational Trade-off: Why not a Daemon?
I was asked why I didn't make this a background service (daemon) that watches for new files.
*   **Decision:** In a Live Operations environment, **predictability > speed**.
*   **Reasoning:** A background daemon might trigger an encode at an unpredictable time (e.g., during a high-load broadcast moment). By keeping it manual, we ensure the Operator retains control over *when* CPU resources are consumed.
