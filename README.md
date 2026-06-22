# proton-pulse-data-backup

Sanitized automated backups of Proton Pulse Supabase tables and the deployed static site.
Archives are stored as GitHub Release assets so the repo itself stays small regardless of
how many backups accumulate.

## Browsing backups

Go to the [Releases page](https://github.com/mdeguzis/proton-pulse-data-backup/releases).
Each release is tagged by date and trigger type:

| Tag format | When it runs |
|---|---|
| `backup-YYYY-MM-DD-scheduled` | Every Sunday at 04:00 UTC |
| `backup-YYYY-MM-DDTHHMMSSz-post-deploy` | After every successful data pipeline CI run |
| `backup-YYYY-MM-DDTHHMMSSz-manual` | Triggered manually via workflow_dispatch |

Scheduled tags use date only (one per week, safe to overwrite same-day). Post-deploy and
manual tags include a timestamp so multiple same-day runs never collide.

## What is in each release

| Asset | Contents | PII handling |
|---|---|---|
| `backup-*-schema.tar.gz` | SQL DDL for all public tables, RLS policies, functions | No PII |
| `backup-*-user_configs.tar.gz` | All visible game compatibility reports | client_id + proton_pulse_user_id HMAC-pseudonymized; notes sanitized |
| `backup-*-author_avatars.tar.gz` | Steam avatar display names and URLs | steam_id excluded; proton_pulse_user_id HMAC-pseudonymized |
| `backup-*-site.tar.gz` | Snapshot of the deployed gh-pages HTML/JS/CSS | No PII |

Auth data (auth.users, admins, banned_users, claimed_client_ids) is never exported.

## Downloading a specific backup

Click any release and download the asset you need directly from the GitHub UI, or via CLI:

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

## Cloning this repo

The repo only contains `README.md`, `backups.jsonl`, and git history for those two files.
It stays small permanently. A normal clone is fine:

```bash
git clone https://github.com/mdeguzis/proton-pulse-data-backup.git
```

If it ever grows large (unlikely), shallow clone with:

```bash
git clone --depth 1 https://github.com/mdeguzis/proton-pulse-data-backup.git
```

## Backup log (backups.jsonl)

Every run appends one line to `backups.jsonl`. Each entry:

```json
{
  "ts":          "2026-06-22T16:00:00.000Z",
  "date":        "2026-06-22",
  "tag":         "backup-2026-06-22-scheduled",
  "trigger":     "scheduled",
  "release_url": "https://github.com/mdeguzis/proton-pulse-data-backup/releases/tag/backup-2026-06-22-scheduled",
  "run_id":      "12345678",
  "run_url":     "https://github.com/mdeguzis/proton-pulse-web/actions/runs/12345678",
  "source_sha":  "abc1234",
  "source_url":  "https://github.com/mdeguzis/proton-pulse-web/commit/abc1234",
  "assets": [
    { "name": "backup-2026-06-22-schema.tar.gz",       "size_bytes": 8192,   "row_count": null, "download_url": "..." },
    { "name": "backup-2026-06-22-user_configs.tar.gz", "size_bytes": 204800, "row_count": 1234, "download_url": "..." }
  ]
}
```

## How to restore

See [RESTORE.md in claude-configs](https://github.com/mdeguzis/claude-configs/blob/main/RESTORE.md)
for the full team handoff and step-by-step restore guide.

## Triggering a manual backup

In the [proton-pulse-web Actions tab](https://github.com/mdeguzis/proton-pulse-web/actions/workflows/backup.yml),
click "Run workflow". Choose the backup type (default: all).
