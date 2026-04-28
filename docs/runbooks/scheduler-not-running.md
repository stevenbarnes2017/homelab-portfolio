# Runbook: Scheduler Not Running / No Logs

## Overview

This runbook documents how to troubleshoot issues where the `football-scheduler` service is deployed but not producing logs or executing scheduled jobs.

The scheduler is responsible for background tasks such as:
- Sending reminder emails
- Fetching spreads
- Processing weekly updates

---

## 🚨 Symptoms

- No logs from scheduler:
  ```bash
  kubectl logs -l app=football-scheduler

logs only show the startup message:
[SCHEDULER] STARTED scheduler_runner.py
-No scheduled jobs executing
-No emails or updates triggered

## 🔍 Troubleshooting Steps
# 1. Verify Pod is Running
  kubectl get pods -l app=football-scheduler

  Expected:
    - Pod is in Running state
# 2. Check Logs
  kubectl logs -l app=football-scheduler
  If only startup message appears:
  - Scheduler may be running but idle
  - Jobs may not be triggering yet
# 3. Confirm Correct Image is Deployed
  kubectl describe pod -l app=football-scheduler | grep -E "Image:|Image ID:"

⚠️ Common Issue:
Deployment updated but still using old image
latest tag confusion

# 4. Force Deployment Restart
kubectl rollout restart deployment football-scheduler
kubectl rollout status deployment football-scheduler
# 5. Verify Environment Variables
kubectl describe pod -l app=football-scheduler

Check:

DISABLE_APSCHEDULER=0
Database connection string present
Email credentials injected
# 6. Check Job Timing

⚠️ Important behavior:

Scheduler jobs may run every 5 minutes or longer
No logs will appear until next execution window

Wait:

5–10 minutes minimum
7. Validate Application Logic

If no logs after expected interval:

Confirm scheduler code includes logging
Ensure jobs are registered correctly
Verify no silent exceptions
8. Check Image Build / Push Pipeline
Confirm GitHub Actions built latest image
Verify image exists in Harbor
Ensure correct tag used in deployment
🧯 Resolution

In this case, the issue was:

Scheduler was running correctly
Logs were delayed due to job interval timing
No actual failure occurred
