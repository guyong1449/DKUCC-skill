# DKUCC Skill Collection

**简体中文** | [English](#english)

面向 Duke Kunshan University Compute Cluster（DKUCC）的运维型 Skill 集合。将常见的集群存储维护、数据恢复和 NFSv4 权限操作拆成独立 Skill，并附带可直接执行的 Shell 工具、参考文档以及 DKUCC 官网的离线页面资料。


## Repository Structure

```text
.
├── dkucc-data-ingest/          # /work 文件保活与容量检查
├── dkucc-data-recovery/        # NFS 快照数据恢复与故障取证
├── dkucc-permission-audit/     # home/work 默认权限只读审计
├── nfs4-acl/                   # /work NFSv4 ACL 授权与修复
└── dkucc.dukekunshan.edu.cn/   # DKUCC 官网离线资料镜像
```

## Skill Directories

### `dkucc-data-ingest/`

用于预防 `/work/<NETID>` 中长期未更新的数据被清理，并检查存储使用情况。

* `SKILL.md`：定义 `/work` 保活和定期检查流程。
* `scripts/touch_work_timestamps.sh`：遍历当前用户 `/work` 下的文件并刷新时间戳，同时写入日志。
* `scripts/monthly_check.sh`：输出 `df`、`du` 结果，并检查最近的 touch/cron 日志。

### `dkucc-data-recovery/`

用于 `/work/<NETID>` 或 `/dkucc/home/<NETID>` 数据丢失后的诊断、NFS `.snapshot` 对比、恢复和证据收集。

* `SKILL.md`：定义"确认丢失 → 查找快照 → 对比快照 → `rsync` 恢复 → 校验 → touch → 必要时提交 IT 证据"的完整流程。
* `agents/`：面向 OpenAI agent 的 Skill 展示信息和默认任务提示。
* `references/`：人工阅读用 SOP 与 IT 工单模板。
* `scripts/`：恢复流程的可执行工具：

  * `common.sh`：统一解析 `/work`、`/dkucc/home` 与 `.snapshot` 路径。
  * `check-live-path.sh`：检查当前路径是否缺失、异常变小或被清空。
  * `list-snapshots.sh`：列出当前存储卷可见的 NFS 快照。
  * `compare-snapshots.sh`：比较两个时间点快照中的目标目录。
  * `restore-from-snapshot.sh`：使用 `rsync` 将指定快照恢复到 live 路径，支持 dry-run。
  * `scan-bash-history.sh`：检查 shell 历史中是否存在针对目标路径的删除命令，作为辅助取证。
  * `print-it-evidence.sh`：汇总身份、挂载、目录状态、快照和历史记录等 IT 工单证据。
  * `touch-after-restore.sh`：恢复后统计文件并刷新文件时间戳。

### `dkucc-permission-audit/`

用于只读审计 DKUCC 用户在 `/dkucc/home` 和 `/work` 上的自然权限，重点区分 POSIX mode 与 NFSv4 ACL 的实际效果。

* `SKILL.md`：定义身份、umask、挂载、ACL 和新建文件权限探针的审计流程。
* `agents/`：OpenAI agent 的展示信息和默认审计提示。
* `references/interpretation.md`：解释 NFSv4 ACE、常见权限模式、异常信号及审计结论。
* `scripts/audit-natural-perms.sh`：采集 home/work 的 POSIX 与 NFSv4 ACL，并通过临时文件验证继承后的实际权限。

该模块原则上只负责检查，不负责直接修改 ACL。

### `nfs4-acl/`

用于 DKUCC `/work` 的 NFSv4 ACL 修改，包括协作者授权、继承权限配置以及 `070` 权限异常修复。

* `SKILL.md`：英文主流程，包含 ACL 模型、ACE 权限位、授权步骤、验证方式和故障修复方案。
* `reference-zh.md`：对应的中文操作参考。

该目录负责权限变更；自然权限检查由 `dkucc-permission-audit/` 负责。


## File Organization Convention

各 Skill 基本遵循以下组织方式：

```text
<skill>/
├── SKILL.md        # Agent 使用的任务定义、边界和操作流程
├── scripts/        # 可直接在 DKUCC 上执行的 Shell 工具（如存在）
├── references/     # 补充说明、SOP 或输出解释（如存在）
└── agents/         # Agent 平台展示/默认提示配置（如存在）
```

四个 Skill 的职责边界如下：

| Skill                    | 主要职责              | 是否修改数据/权限       |
| ------------------------ | -------------------- | --------------------- |
| `dkucc-data-ingest`      | `/work` 保活、空间检查  | 会刷新文件时间戳         |
| `dkucc-data-recovery`    | 快照诊断与数据恢复       | 会在明确恢复时写入数据     |
| `dkucc-permission-audit` | 默认权限审计           | 默认只读，仅创建并删除探针文件 |
| `nfs4-acl`               | NFSv4 ACL 授权与修复   | 会修改 ACL            |

## DKUCC Offline Documentation

### `dkucc.dukekunshan.edu.cn/`

DKUCC 官方网站页面的本地保存副本，用作集群使用说明和背景资料。各子目录按网站主题划分：

---

# English

[简体中文](#dkucc-skill-collection) | **English**

An operations-oriented collection of Skills for the Duke Kunshan University Compute Cluster (DKUCC). Common cluster storage maintenance, data recovery, and NFSv4 permission operations are decomposed into standalone Skills, each accompanied by directly executable Shell utilities, reference documentation, and offline copies of DKUCC official website pages.


## Repository Structure

```text
.
├── dkucc-data-ingest/          # /work file keep-alive and capacity check
├── dkucc-data-recovery/        # NFS snapshot data recovery and failure forensics
├── dkucc-permission-audit/     # Read-only audit of default home/work permissions
├── nfs4-acl/                   # /work NFSv4 ACL authorization and repair
└── dkucc.dukekunshan.edu.cn/   # Offline mirror of the DKUCC official website
```

## Skill Directories

### `dkucc-data-ingest/`

Prevents data under `/work/<NETID>` that has not been updated for a long time from being cleaned up, and checks storage usage.

* `SKILL.md`: Defines the `/work` keep-alive and periodic check workflow.
* `scripts/touch_work_timestamps.sh`: Traverses the current user's files under `/work`, refreshes their timestamps, and writes a log.
* `scripts/monthly_check.sh`: Outputs the results of `df` and `du`, and checks the most recent touch/cron logs.

### `dkucc-data-recovery/`

Covers diagnosis, NFS `.snapshot` comparison, recovery, and evidence collection after data loss under `/work/<NETID>` or `/dkucc/home/<NETID>`.

* `SKILL.md`: Defines the complete workflow "confirm loss → locate snapshots → compare snapshots → recover with `rsync` → verify → touch → submit IT evidence if necessary".
* `agents/`: Skill showcase information and default task prompts for OpenAI agents.
* `references/`: Human-readable SOPs and IT ticket templates.
* `scripts/`: Executable utilities for the recovery workflow:

  * `common.sh`: Uniformly resolves `/work`, `/dkucc/home`, and `.snapshot` paths.
  * `check-live-path.sh`: Checks whether the current path is missing, abnormally shrunk, or emptied.
  * `list-snapshots.sh`: Lists the NFS snapshots visible on the current storage volume.
  * `compare-snapshots.sh`: Compares the target directory across snapshots at two points in time.
  * `restore-from-snapshot.sh`: Restores a specified snapshot to the live path with `rsync`, supporting dry-run.
  * `scan-bash-history.sh`: Checks shell history for deletion commands targeting the path in question, as supplementary forensics.
  * `print-it-evidence.sh`: Aggregates identity, mount, directory status, snapshot, and history evidence for IT tickets.
  * `touch-after-restore.sh`: Counts files after restoration and refreshes file timestamps.

### `dkucc-permission-audit/`

Performs read-only audits of DKUCC users' natural permissions on `/dkucc/home` and `/work`, with emphasis on distinguishing the actual effects of POSIX modes from those of NFSv4 ACLs.

* `SKILL.md`: Defines the audit workflow covering identity, umask, mount, ACLs, and new-file permission probes.
* `agents/`: Showcase information and default audit prompts for OpenAI agents.
* `references/interpretation.md`: Explains NFSv4 ACEs, common permission patterns, anomalous signals, and audit conclusions.
* `scripts/audit-natural-perms.sh`: Collects the POSIX and NFSv4 ACLs of home/work, and verifies the inherited effective permissions with temporary files.

In principle, this module is responsible only for inspection, not for directly modifying ACLs.

### `nfs4-acl/`

Modifies NFSv4 ACLs on DKUCC `/work`, including collaborator authorization, inherited-permission configuration, and repair of the `070` permission anomaly.

* `SKILL.md`: Main workflow in English, covering the ACL model, ACE permission bits, authorization steps, verification methods, and troubleshooting plans.
* `reference-zh.md`: The corresponding Chinese operations reference.

This directory handles permission changes; natural-permission inspection is handled by `dkucc-permission-audit/`.


## File Organization Convention

Each Skill generally follows the layout below:

```text
<skill>/
├── SKILL.md        # Task definition, boundaries, and workflow for agent use
├── scripts/        # Shell utilities directly executable on DKUCC (if any)
├── references/     # Supplementary notes, SOPs, or output interpretation (if any)
└── agents/         # Agent platform showcase/default prompt configuration (if any)
```

The responsibility boundaries of the four Skills are as follows:

| Skill                    | Primary responsibility              | Modifies data/permissions?        |
| ------------------------ | ----------------------------------- | --------------------------------- |
| `dkucc-data-ingest`      | `/work` keep-alive, space check     | Refreshes file timestamps         |
| `dkucc-data-recovery`    | Snapshot diagnosis and data recovery | Writes data during explicit recovery |
| `dkucc-permission-audit` | Default permission audit            | Read-only by default; creates and deletes probe files only |
| `nfs4-acl`               | NFSv4 ACL authorization and repair  | Modifies ACLs                     |

## DKUCC Offline Documentation

### `dkucc.dukekunshan.edu.cn/`

Local saved copies of pages from the DKUCC official website, used as cluster usage instructions and background material. Subdirectories are organized by website topic:
