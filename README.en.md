# DKUCC User Guide and Skills

[简体中文](README.md) | **English**

DKUCC (Duke Kunshan University Compute Cluster) is the university's computing cluster. You can connect from your own computer, request computing resources through Slurm, and run your jobs on the cluster.

This repository provides a practical user guide, plus optional Skills for storage maintenance, data recovery, and permission management.

## Start with the user guide

**New to DKUCC? Read the [DKUCC User Guide](DKUCC%20User%20Guide.md) first.**

We suggest this reading order:

1. **Start with the guide**: learn about accounts and official access points, SSH setup on Windows / macOS, directory layout, and the roles of login and compute nodes.
2. **Before submitting jobs**: read the Slurm and GPU sections to check your account type, eligible partitions, resource limits, and billing arrangements.
3. **For a specific maintenance task**: choose a Skill below, then read its `SKILL.md` and relevant references before taking action.

You do not need to install a Skill to read or use the guide. Skills organize specific tasks into workflows that agents can follow, with scripts that people can inspect and use. Installing or loading a Skill does not authorize it to run commands, nor does it mean every operation is appropriate for your account or environment.

## Choose a Skill by task

| What you want to do | Start here | What it changes |
| --- | --- | --- |
| Check `/work` storage use and file timestamps | [`dkucc-data-ingest`](dkucc-data-ingest/SKILL.md) | Storage checks do not modify your working files; the timestamp script changes file timestamps |
| Investigate missing files, find snapshots, or restore data | [`dkucc-data-recovery`](dkucc-data-recovery/SKILL.md) | Diagnose and compare first; restoration writes to the destination, with a separate option to refresh timestamps afterward |
| Check default and inherited permissions on home / work | [`dkucc-permission-audit`](dkucc-permission-audit/SKILL.md) | Creates and deletes temporary probes, so it is not strictly read-only; it does not manage changes to existing ACLs |
| Grant collaborator access or repair NFSv4 ACLs | [`nfs4-acl`](nfs4-acl/SKILL.md) | Changes permissions; a Chinese reference is available in [`reference-zh.md`](nfs4-acl/reference-zh.md) |

## Repository layout

```text
.
├── README.md                     # Chinese entry point
├── README.en.md                  # English entry point
├── DKUCC 使用指南.md             # Chinese user guide
├── DKUCC User Guide.md           # English user guide — start here
├── DKUCC 使用指南.assets/        # Guide images
├── dkucc-data-ingest/             # /work storage checks and timestamp maintenance
├── dkucc-data-recovery/           # NFS snapshot diagnosis, recovery, and evidence
├── dkucc-permission-audit/        # home / work permission checks
├── nfs4-acl/                     # NFSv4 ACL access management and repair
└── dkucc.dukekunshan.edu.cn/      # Historical offline copies of official web pages
```

Each Skill's `SKILL.md` describes its scope and workflow. Where present, `scripts/` contains shell tools, `references/` contains supplementary notes, SOPs, or support-ticket templates, and `agents/` contains agent display information and default prompts.

## Scripts and reference material

### `/work` checks and timestamp maintenance

[`dkucc-data-ingest/`](dkucc-data-ingest/) contains two scripts:

- `scripts/monthly_check.sh`: shows `df` and `du` results and checks recent touch / cron logs.
- `scripts/touch_work_timestamps.sh`: walks the current user's files under `/work`, refreshes their timestamps, and writes a log.

Before using the timestamp script, confirm the current retention rules and permitted maintenance practices. Refreshing timestamps is not a backup and does not guarantee that files will be retained. Review the target paths and scope before adding the script to a scheduled job.

### Missing files and snapshot recovery

[`dkucc-data-recovery/`](dkucc-data-recovery/) covers `/work/<NETID>` and `/dkucc/home/<NETID>`. Confirm that files are missing, inspect available snapshots, and compare their contents before selecting a recovery source and destination. The `references/` directory includes a manual SOP and IT support-ticket templates.

| Script | Purpose |
| --- | --- |
| `scripts/common.sh` | Handles `/work`, `/dkucc/home`, and `.snapshot` paths consistently |
| `scripts/check-live-path.sh` | Checks whether the current directory is missing, unusually small, or empty |
| `scripts/list-snapshots.sh` | Lists NFS snapshots visible on the current storage volume |
| `scripts/compare-snapshots.sh` | Compares the target directory in snapshots from two points in time |
| `scripts/restore-from-snapshot.sh` | Restores to the live directory with `rsync`, with dry-run support |
| `scripts/scan-bash-history.sh` | Looks for deletion commands involving the target path as supporting evidence |
| `scripts/print-it-evidence.sh` | Collects identity, mount, directory, snapshot, and history information for an IT ticket |
| `scripts/touch-after-restore.sh` | Counts restored files and refreshes their timestamps |

Snapshot availability and retention depend on the actual storage system; snapshots are not long-term backups. Before restoring, inspect the dry-run output and confirm the source snapshot, destination, and any files that may be overwritten. Review collected evidence for sensitive paths, commands, or other private information before sharing it.

### Permission checks and changes

- In [`dkucc-permission-audit/`](dkucc-permission-audit/), `scripts/audit-natural-perms.sh` collects identity, umask, mount, POSIX mode, and NFSv4 ACL information, then tests inheritance with temporary files when auditing the current user. See [`references/interpretation.md`](dkucc-permission-audit/references/interpretation.md) for help interpreting the results.
- [`nfs4-acl/`](nfs4-acl/) covers collaborator access, permission inheritance, and anomalies such as `070`. Check the existing ACL and intended access scope before making and verifying changes. Do not judge NFSv4 permissions solely by the output of `ls -l`.

## Before you run anything

- Use login nodes for logging in, editing, transferring files, and submitting jobs. Run substantial CPU / GPU workloads on compute nodes through Slurm.
- Check your account, paths, and resource limits before running commands. Restoring data, refreshing timestamps, and editing ACLs can all change existing state.
- When working with an agent, specify the target paths, allowed actions, and steps that require confirmation. Inspect and preview before writing data or changing permissions.
- Keep separate backups of important data. Do not rely on `/work` retention periods, refreshed timestamps, or short-lived snapshots.

## Policy sources and currency

The guide's GPU account, resource-limit, and billing sections are based on DKU IT's **DKU Research GPU management (2026-10-16)** and its **2026-10-23 announcement email**. The announcement sets **2026-12-01** as the start date for research-account billing and the account transition. Read the guide for the rules, transition arrangements, and application process rather than submitting jobs based only on old screenshots or examples.

This repository is a reference, not a replacement for university policy. The [DKUCC official website](https://dkucc.dukekunshan.edu.cn/), current DKU IT notices, and your actual account configuration take precedence over outdated repository content. The `dkucc.dukekunshan.edu.cn/` directory contains historical offline copies of official web pages. They are useful for background but may not reflect current partitions, allowances, prices, or storage rules.
