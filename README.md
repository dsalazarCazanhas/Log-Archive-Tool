# Log-Archive-Tool

A CLI tool that compresses a log directory into a timestamped `.tar.gz` archive. Optionally uploads it to [**Backblaze B2**](https://www.backblaze.com/) via [rclone](https://rclone.org/) and removes the local copy once the upload is confirmed.

---

## Requirements

- `bash` ≥ 4
- `tar`
- `rclone` — only needed for `--upload` ([install guide](https://rclone.org/install/))

---

## Installation

```bash
git clone https://github.com/dsalazarCazanhas/Log-Archive-Tool.git
cd Log-Archive-Tool
chmod +x log-archive
sudo cp log-archive /usr/local/bin/log-archive   # optional: make available system-wide
```

---

## Usage

```bash
log-archive <log-directory> [--upload]
```

The tool will show an execution plan and ask for confirmation before doing anything.

### Archive only (local)

```bash
log-archive /var/log
```

Creates `~/.log-archives/logs_archive_YYYYMMDD_HHMMSS.tar.gz` and keeps it locally.

### Archive + upload to Backblaze B2

```bash
export B2_REMOTE="b2:my-bucket/logs"          # rclone remote:bucket/path
export BACKUP_RETENTION_DAYS=30               # optional, default 30

log-archive /var/log --upload
```

What happens:
1. Compresses `<log-directory>` into a `.tar.gz` archive.
2. Uploads it to the configured B2 bucket using `rclone`.
3. Deletes remote archives older than `BACKUP_RETENTION_DAYS` days.
4. Removes the local `.tar.gz` to free disk space.

---

## Backblaze B2 setup with rclone

1. Create a [Backblaze B2](https://www.backblaze.com/b2/cloud-storage.html) account and a bucket.
2. Generate an **Application Key** with read/write access to that bucket.
3. Configure rclone:

```bash
rclone config
# Choose: n (new remote) → name it "b2" → type: b2
# Enter your Account ID and Application Key when prompted
```

4. Verify the connection:

```bash
rclone lsd b2:my-bucket
```

5. Run the tool:

```bash
B2_REMOTE="b2:my-bucket/logs" log-archive /var/log --upload
```

---

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `B2_REMOTE` | — | rclone remote destination. Required for `--upload`. |
| `BACKUP_RETENTION_DAYS` | `30` | Days to keep archives in B2 before pruning. |

---

## Logs

| File | Content |
|---|---|
| `~/.log-archives/archive_activity.log` | One line per run with timestamp, source, archive name, size, and upload status. |
| `~/.log-archives/rclone_<timestamp>.log` | Detailed rclone DEBUG log for each upload run. |

---
> A journey to grow up from [Roadmap.sh](https://roadmap.sh/projects/log-archive-tool)