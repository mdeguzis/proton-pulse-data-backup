# proton-pulse-data-backup

Sanitized automated backups of Proton Pulse Supabase tables and the deployed static site.

## Quick access: latest backup

The `latest/` folder always contains the most recent full backup as stable filenames:

```
latest/
  latest-schema.tar.gz
  latest-user_configs.tar.gz
  latest-author_avatars.tar.gz
  latest-site.tar.gz
```

Clone the repo and grab them directly:

```bash
git clone --depth 1 https://github.com/mdeguzis/proton-pulse-data-backup.git
ls proton-pulse-data-backup/latest/
```

Or download a single file without cloning:

```bash
gh api repos/mdeguzis/proton-pulse-data-backup/contents/latest/latest-user_configs.tar.gz \
  --jq '.download_url' | xargs curl -L -o user_configs.tar.gz
```

## Archive: all historical backups

Every backup is also published as a [GitHub Release](https://github.com/mdeguzis/proton-pulse-data-backup/releases)
with dated, tagged assets. Releases are the permanent archive. The `latest/` folder is
just a convenience pointer to the most recent run.

### Release tag format

| Tag | When |
|---|---|
| `backup-YYYY-MM-DD-scheduled` | Weekly, Sunday 04:00 UTC. Overwrites same-day tag if re-run. |
| `backup-YYYYMMDDTHHMMSSz-post-deploy` | After every successful data pipeline CI run. Timestamped so multiple same-day runs never collide. |
| `backup-YYYYMMDDTHHMMSSz-manual` | On-demand via workflow_dispatch. Timestamped. |

### Downloading a specific historical backup

```bash
# Download all assets from a specific release
gh release download backup-2026-06-22-scheduled \
  --repo mdeguzis/proton-pulse-data-backup \
  --dir ./restore-2026-06-22

# Download just the user_configs archive
gh release download backup-2026-06-22-scheduled \
  --repo mdeguzis/proton-pulse-data-backup \
  --pattern "*user_configs*"
```

## What is in each backup

| Asset | Contents | PII handling |
|---|---|---|
| `*-schema.tar.gz` | SQL DDL for all public tables, RLS policies, functions | No PII |
| `*-user_configs.tar.gz` | All visible game compatibility reports | client_id + proton_pulse_user_id HMAC-pseudonymized; notes sanitized |
| `*-author_avatars.tar.gz` | Steam avatar display names and URLs | steam_id excluded; proton_pulse_user_id HMAC-pseudonymized |
| `*-site.tar.gz` | Snapshot of the deployed gh-pages HTML/JS/CSS | No PII |

Auth data (auth.users, admins, banned_users, claimed_client_ids) is never exported.

## Backup log

Every run appends one line to `backups.jsonl` at the root. Each entry has `ts`, `date`,
`tag`, `trigger`, `release_url`, `run_url`, `source_sha`, `source_url`, and per-asset
`download_url`. Use it to audit what ran and when, or to script downloading a specific
point-in-time restore.

## How to restore

See [RESTORE.md in claude-configs](https://github.com/mdeguzis/claude-configs/blob/main/RESTORE.md)
for the full team handoff and step-by-step restore guide.

## Triggering a manual backup

Go to [Actions > Backup](https://github.com/mdeguzis/proton-pulse-web/actions/workflows/backup.yml)
in the proton-pulse-web repo and click "Run workflow".
