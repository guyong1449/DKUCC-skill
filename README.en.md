# DKUCC User Guide and Skills

[简体中文](README.md) | **English**

DKUCC (Duke Kunshan University Compute Cluster) is Duke Kunshan University's compute cluster. You can connect to the cluster from your own computer via SSH, request computing resources through Slurm, and run jobs.

This repository provides an introductory guide for users, along with Skills for storage, data recovery, and permission management to use as needed.

## Getting Started

**We recommend reading the [DKUCC User Guide](DKUCC%20User%20Guide.md) first.**

Reading order:

1. **Read the guide first**: Learn about accounts and official entry points, SSH configuration on Windows / macOS, directory planning, and the division of responsibilities between login nodes and compute nodes.
2. **When preparing to submit a job**: Read the Slurm and GPU sections of the guide to confirm your account type, available partitions, resource limits, and billing arrangements.
3. **When a specific maintenance need arises**: Select the corresponding Skill below, then read its `SKILL.md` and related references before proceeding.

Notes:

1. Your initial SSH connection takes you to a login node. **Do not run GPU jobs or CPU-intensive computing jobs directly on `dkucc-login-01`.**
2. Large computing jobs should be submitted to compute nodes through Slurm. See Section 5 and Section 6 for specific steps.
3. After logging in, your directory is usually `/dkucc/home/<NETID>`, which is recommended for small files such as code and configuration. Large amounts of data, checkpoints, and outputs should be placed in `/work/<NETID>`.
4. The `/work` cleanup rule is: every half month, files with timestamps older than 75 days are removed.
5. Do not bypass Slurm by connecting directly to compute nodes via SSH to occupy GPUs or run training.
6. Release the resources you requested after a job ends; do not rely on leftover processes to keep running. After a job is `COMPLETED`, the GPU node allocated to it will soon be reclaimed even if you use screen or tmux.
7. Learning accounts and research accounts differ in GPU count, monthly runtime, and available partitions. **Both account types can run at most 1 job at a time.** See Section 5 for details.


## Specific Skills

| Function | Entry Point | What It Does |
| --- | --- | --- |
| Check `/work` space and file timestamps | [`dkucc-data-ingest`](dkucc-data-ingest/SKILL.md) | Space checks do not modify workload files; the timestamp maintenance script modifies file timestamps |
| Investigate missing files, find snapshots, or recover data | [`dkucc-data-recovery`](dkucc-data-recovery/SKILL.md) | Diagnose and compare first; performing a restore writes data to the target directory, and timestamps can be refreshed separately afterward |
| Check default permissions and inheritance for home / work | [`dkucc-permission-audit`](dkucc-permission-audit/SKILL.md) | Creates and deletes temporary probes, so it is not purely read-only; it does not modify existing ACLs |
| Grant access to collaborators or repair NFSv4 ACLs | [`nfs4-acl`](nfs4-acl/SKILL.md) | Modifies permissions; see [`reference-zh.md`](nfs4-acl/reference-zh.md) for operating instructions in Chinese |

## Repository Structure

```text
.
├── README.md                     # Chinese entry point
├── README.en.md                  # English entry point
├── DKUCC 使用指南.md             # Chinese user guide, recommended first reading
├── DKUCC User Guide.md           # English user guide
├── DKUCC 使用指南.assets/        # Guide illustrations
├── dkucc-data-ingest/             # /work space checks and timestamp maintenance
├── dkucc-data-recovery/           # NFS snapshot diagnosis, recovery, and evidence collection
├── dkucc-permission-audit/        # home / work permission checks
├── nfs4-acl/                     # NFSv4 ACL access grants and repairs
└── dkucc.dukekunshan.edu.cn/      # Historical offline copies of official website pages
```

## Scripts and References

### `/work` Checks and Timestamp Maintenance

The two scripts in [`dkucc-data-ingest/`](dkucc-data-ingest/) are responsible for:

- `scripts/monthly_check.sh`: View `df` and `du` results, along with recent touch / cron logs.
- `scripts/touch_work_timestamps.sh`: Traverse files under the current user's `/work`, refresh their timestamps, and write logs.


### Data Loss and Snapshot Recovery

[`dkucc-data-recovery/`](dkucc-data-recovery/) targets `/work/<NETID>` and `/dkucc/home/<NETID>`. First confirm whether files are missing, then check visible snapshots and compare contents, and only write data after determining the restore target. `references/` provides manual-operation SOPs and IT ticket templates.

| Script | Purpose |
| --- | --- |
| `scripts/common.sh` | Handle `/work`, `/dkucc/home`, and `.snapshot` paths consistently |
| `scripts/check-live-path.sh` | Check whether the current directory is missing, has shrunk abnormally, or has been emptied |
| `scripts/list-snapshots.sh` | List visible NFS snapshots on the current storage volume |
| `scripts/compare-snapshots.sh` | Compare snapshot directories from two points in time |
| `scripts/restore-from-snapshot.sh` | Restore to the current directory using `rsync`, with dry-run preview support |
| `scripts/scan-bash-history.sh` | Find deletion commands related to the target path as supporting clues for troubleshooting |
| `scripts/print-it-evidence.sh` | Summarize ticket evidence such as identity, mounts, directories, snapshots, and command history |
| `scripts/touch-after-restore.sh` | Count restored files and refresh their timestamps |

Whether snapshots exist and how long they can be retained depends on the actual storage state; snapshots are not long-term backups. Before restoring, check the dry-run output to confirm the source snapshot, target directory, and content that may be overwritten. Before sharing ticket evidence, check whether it contains sensitive paths, commands, or other information that should not be made public.

### Permission Checks and Changes

- `scripts/audit-natural-perms.sh` in [`dkucc-permission-audit/`](dkucc-permission-audit/) collects identity, umask, mounts, POSIX mode, and NFSv4 ACLs, and uses temporary files to verify permission inheritance when auditing the current user. See [`references/interpretation.md`](dkucc-permission-audit/references/interpretation.md) for an explanation of the results.
- [`nfs4-acl/`](nfs4-acl/) handles collaborator access grants, inherited permissions, and anomalies such as `070`. First confirm the existing ACLs and intended scope of access, then follow the procedure to modify and verify them; do not judge NFSv4 permissions solely from the output of `ls -l`

## Policy Sources and Update Scope

The guide's descriptions of GPU accounts, resource limits, and billing are based on DKU IT's **DKU Research GPU management (2026-10-16)** and the **notification email of 2026-10-23**. Research account billing and account switching begin on **2026-12-01**. Read the guide for specific rules, transition arrangements, and application procedures.

This repository is a usage reference and does not replace the university's official policies. The [DKUCC official website](https://dkucc.dukekunshan.edu.cn/), the latest notices from DKU IT, and actual account configurations take precedence over older descriptions in the repository. `dkucc.dukekunshan.edu.cn/` contains historical offline copies of official website pages for background reference.

