# DKUCC 使用指南与 Skills

**简体中文** | [English](README.en.md)

DKUCC（Duke Kunshan University Compute Cluster）是昆山杜克大学的计算集群。你可以从自己的电脑SSH连接集群，通过 Slurm 申请计算资源并运行任务。

本仓库提供一份面向使用者的入门指南，以及按需使用的存储、数据恢复和权限管理 Skills。

## 开始

**建议先阅读 [《DKUCC 使用指南》](DKUCC%20使用指南.md)。**

阅读顺序：

1. **先读指南**：了解账号与官方入口、Windows / macOS 的 SSH 配置、目录规划，以及登录节点和计算节点的分工。
2. **准备提交任务时**：阅读指南中的 Slurm 与 GPU 章节，确认自己的账号类型、可用分区、资源限制和计费安排。
3. **遇到具体维护需求时**：再从下面选择对应的 Skill，阅读其 `SKILL.md` 和相关参考资料后操作。

注意事项：

1. SSH 初次进入的是登录节点，**不要在 `dkucc-login-01` 直接运行 GPU 任务或需要大量 CPU 的计算任务**。
2. 大型计算任务应通过 Slurm 提交到计算节点，具体操作见 Section 5 和 Section 6。
3. 登录后的目录通常是 `/dkucc/home/<NETID>`，建议存放代码、配置等小文件。大量数据、checkpoint 和输出产物应放到 `/work/<NETID>`。
4. `/work` 清理规则为：每半个月清理时间戳旧于 75 天的文件。
5. 不要绕过 Slurm，直接 SSH 到计算节点占用 GPU 或运行训练。
6. 作业结束后应释放所申请的资源，不要依赖残余进程继续运行；任务 `COMPLETED` 后，就算开 screen 或者 tmux 对应申请的 GPU 节点也很快会被回收。
7. 学习账号与科研账号的 GPU 数量、月运行时间和可用分区不同。**两类账号均最多同时运行 1 个作业**，详见 Section 5。


## 具体 Skill

| 功能 | 入口 | 实现内容 |
| --- | --- | --- |
| 检查 `/work` 空间与文件时间戳 | [`dkucc-data-ingest`](dkucc-data-ingest/SKILL.md) | 空间检查不修改业务文件；时间戳维护脚本会修改文件时间戳 |
| 排查文件丢失、查找快照或恢复数据 | [`dkucc-data-recovery`](dkucc-data-recovery/SKILL.md) | 先诊断、对比；执行恢复会向目标目录写入数据，恢复后可另行刷新时间戳 |
| 检查 home / work 的默认权限与继承效果 | [`dkucc-permission-audit`](dkucc-permission-audit/SKILL.md) | 会创建并删除临时探针，不是纯只读；不负责修改现有 ACL |
| 给协作者授权或修复 NFSv4 ACL | [`nfs4-acl`](nfs4-acl/SKILL.md) | 会修改权限；中文操作说明见 [`reference-zh.md`](nfs4-acl/reference-zh.md) |

## 仓库结构

```text
.
├── README.md                     # 中文入口
├── README.en.md                  # English entry point
├── DKUCC 使用指南.md             # 中文使用指南，建议先读
├── DKUCC User Guide.md           # English user guide
├── DKUCC 使用指南.assets/        # 指南配图
├── dkucc-data-ingest/             # /work 空间检查与时间戳维护
├── dkucc-data-recovery/           # NFS 快照诊断、恢复与证据收集
├── dkucc-permission-audit/        # home / work 权限检查
├── nfs4-acl/                     # NFSv4 ACL 授权与修复
└── dkucc.dukekunshan.edu.cn/      # 官方网站页面的历史离线副本
```

## 脚本与参考资料

### `/work` 检查与时间戳维护

[`dkucc-data-ingest/`](dkucc-data-ingest/) 中的两个脚本分别负责：

- `scripts/monthly_check.sh`：查看 `df`、`du` 结果，以及最近的 touch / cron 日志。
- `scripts/touch_work_timestamps.sh`：遍历当前用户 `/work` 下的文件，刷新时间戳并写入日志。


### 数据丢失与快照恢复

[`dkucc-data-recovery/`](dkucc-data-recovery/) 面向 `/work/<NETID>` 和 `/dkucc/home/<NETID>`。先确认文件是否丢失，再检查可见快照、比较内容，确定恢复目标后再写入数据。`references/` 中提供人工操作 SOP 和 IT 工单模板。

| 脚本 | 用途 |
| --- | --- |
| `scripts/common.sh` | 统一处理 `/work`、`/dkucc/home` 与 `.snapshot` 路径 |
| `scripts/check-live-path.sh` | 检查当前目录是否缺失、异常变小或被清空 |
| `scripts/list-snapshots.sh` | 列出当前存储卷上可见的 NFS 快照 |
| `scripts/compare-snapshots.sh` | 比较两个时间点的快照目录 |
| `scripts/restore-from-snapshot.sh` | 用 `rsync` 恢复到当前目录，支持 dry-run 预览 |
| `scripts/scan-bash-history.sh` | 查找与目标路径有关的删除命令，作为辅助排查线索 |
| `scripts/print-it-evidence.sh` | 汇总身份、挂载、目录、快照和历史记录等工单证据 |
| `scripts/touch-after-restore.sh` | 统计恢复后的文件并刷新时间戳 |

快照是否存在、能保留多久，以实际存储状态为准；快照不是长期备份。恢复前应检查 dry-run 输出，确认源快照、目标目录和可能覆盖的内容。分享工单证据前，先检查是否包含敏感路径、命令或其他不宜公开的信息。

### 权限检查与变更

- [`dkucc-permission-audit/`](dkucc-permission-audit/) 的 `scripts/audit-natural-perms.sh` 采集身份、umask、挂载、POSIX mode 和 NFSv4 ACL，并在审计当前用户时，用临时文件验证权限继承。结果解释见 [`references/interpretation.md`](dkucc-permission-audit/references/interpretation.md)。
- [`nfs4-acl/`](nfs4-acl/) 处理协作者授权、继承权限和 `070` 等异常。先确认现有 ACL 和目标访问范围，再按流程修改并验证；不要只根据 `ls -l` 的显示判断 NFSv4 权限

## 政策来源与更新范围

指南中的 GPU 账号、资源限制和计费说明依据 DKU IT 的 **《DKU Research GPU management》（2026-10-16）**，并结合 **2026-10-23 的通知邮件**整理。研究账号计费及账号切换的开始时间为 **2026-12-01**。具体规则、过渡安排和申请方式请阅读指南。

本仓库是使用参考，不替代学校的正式政策。[DKUCC 官方网站](https://dkucc.dukekunshan.edu.cn/)、DKU IT 的最新通知和实际账号配置应优先于仓库中的旧说明。`dkucc.dukekunshan.edu.cn/` 保存的是官网页面的历史离线副本，可用于查阅背景。
