# Mitto Pay — Rollback Runbook

**Target Host:** Oracle VM `130.210.28.51`  
**Service:** `jury-staging.service`  
**Rollback Triggers:**
- Any unexpected 500 error or crash on service startup.
- Any failure of canary merchant allowlist enforcement (e.g. unauthorized merchant accepted).
- Any single-channel premature payment success emission.
- Cloudflare ingress failure (`pay.vvisa.in` / `pay-api.vvisa.in`).
- Any interference with V-Visa's primary Zoho Payments gateway.

---

## 1. Rollback Targets

1. **Immediate Pre-Canary Snapshot:** `/opt/jury-backup-pre-mitto-canary` (Created during deployment step 3)
2. **Prior Known-Good Gate 4 Backup:** `/opt/jury-backup-pre-gate4` (`560037147b5dcdc3620ee4efa40c5fff2496a418+phase9d-hardened`)
3. **Database Recovery Target:** `/opt/backups/`

---

## 2. Emergency Rollback Execution Procedure

Execute sequentially:

```bash
# 1. Stop the application service immediately to prevent unintended processing
sudo systemctl stop jury-staging.service

# 2. Swap back to the pre-deployment snapshot
sudo rm -rf /opt/jury
sudo cp -r /opt/jury-backup-pre-mitto-canary /opt/jury

# 3. If pre-canary snapshot was corrupted, fall back to Gate 4 known-good backup:
# sudo cp -r /opt/jury-backup-pre-gate4 /opt/jury

# 4. Revert /etc/jury/staging.env if canary flags need restoration
sudo sed -i '/CANARY_/d' /etc/jury/staging.env

# 5. Fix permissions
sudo chown -R ubuntu:ubuntu /opt/jury

# 6. Restart the known-good service
sudo systemctl start jury-staging.service

# 7. Check service status
systemctl status jury-staging.service --no-pager
```

---

## 3. Post-Rollback Verification

1. **Verify Running Commit:**
   ```bash
   cat /opt/jury/DEPLOYED_COMMIT
   ```
   *Expected:* `560037147b5dcdc3620ee4efa40c5fff2496a418+gate4-gmail-observer`
2. **Verify Health Endpoint:**
   ```bash
   curl -sSf http://127.0.0.1:6100/api/v1/health
   ```
3. **Check Journalctl Logs:**
   ```bash
   sudo journalctl -u jury-staging.service -n 20 --no-pager
   ```
4. **Confirm Normal Operations Restored.**
