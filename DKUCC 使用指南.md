---
tags:
  - area/dkucc
  - type/reference
  - status/published
status: published
---

# DKUCC 使用指南

[English](DKUCC%20User%20Guide.md) · [仓库说明](README.md)

以下内容主要包括 DKUCC 的账号申请、SSH 登录、目录规划、Slurm 作业及数据保留。账号与 GPU 权限依据 2026 年 10 月 23 日的 DKU IT 通知及其附件 DKU Research GPU management 进行了相关说明的更新。**新账号转换及科研使用计费将于 2026 年 12 月 1 日实施**，详见 Section 1 和 Section 5。

## 问题反馈

网络、DKU VPN 或其他 DKU 账号问题，请联系 [service-desk@dukekunshan.edu.cn](mailto:service-desk@dukekunshan.edu.cn) 提交工单，可以加本人微信：18851063808，把你拉进 DKUCC 问题反馈群，这样遇到问题比邮件反馈时效性高一些。

## 目录

- [Section 0 使用要点速查](#section-0)
- [Section 1 DKUCC 与官方入口](#section-1)
- [Section 2 Windows / macOS 的 SSH 初始化](#section-2)
- [Section 3 第一次登录与目录规划](#section-3)
- [Section 4 SSH 公钥、LDAP 与免密登录](#section-4)
- [Section 5 登录节点、计算节点和 Slurm 的分工](#section-5)
- [Section 6 GPU 作业：提交、排队、抢占和恢复](#section-6)
- [Section 7 `/work` 数据保留与定期检查](#section-7)
- [官方资料](#official-resources)
- [可参考的工具](#tools)

<a id="section-0"></a>

## 「Section 0」要点速查

1. SSH 初次进入的是登录节点，**不要在 `dkucc-login-01` 直接运行 GPU 任务或需要大量 CPU 的计算任务**。
2. 大型计算任务应通过 Slurm 提交到计算节点，具体操作见 Section 5 和 Section 6。
3. 登录后的目录通常是 `/dkucc/home/<NETID>`，建议存放代码、配置等小文件。大量数据、checkpoint 和输出产物应放到 `/work/<NETID>`。
4. `/work` 清理规则为：每半个月清理时间戳旧于 75 天的文件。
5. 不要绕过 Slurm，直接 SSH 到计算节点占用 GPU 或运行训练。
6. 初次使用可先提交普通作业。array job 的作业 ID 与子任务 ID 排查困难；使用数组作业时，应结合 `sacct` 和 `SLURM_JOB_ID` 等信息定位，这不是账号政策对数组作业的禁令。
7. 作业结束后应释放所申请的资源，不要依赖残余进程继续运行；任务 `COMPLETED` 后，就算开 screen 或者 tmux 对应申请的 GPU 节点也很快会被回收。
8. 学习账号与科研账号的 GPU 数量、月运行时间和可用分区不同。**两类账号均最多同时运行 1 个作业**，详见 Section 5。

---

<a id="section-1"></a>

## 「Section 1」DKUCC 与官方入口

官网网址：https://dkucc.dukekunshan.edu.cn/

| 页面                                | 内容                              |
| --------------------------------- | ------------------------------- |
| Logging into the DKUCC            | SSH、VPN、登录节点使用边界和 Open OnDemand |
| Working with files                | Home 与 `/work` 的用途、容量、文件保留和传输   |
| Scheduling Jobs                   | Slurm 作业提交与管理                   |
| How to Use GPU Resource           | GPU 分区和交互式会话                    |
| Modules / User installed software | 环境模块和个人软件安装                     |
| OnDemand / OnDemand Overview      | 推荐通过 OnDemand 进行节点申请和交互式应用       |
| Training                          | DCC 入门培训和自学资料                   |

DKUCC 是由登录节点、计算节点和共享存储组成的计算集群。用户通过 SSH（Secure Shell Protocol）加密连接登录节点，再按任务需要申请计算资源。

### 账号申请与政策切换

**学习账号（Learning account）**可向 DKU 的学生、教师和员工开通，也可用于基础研究。发送邮件至 [it-infrastructure@dukekunshan.edu.cn](mailto:it-infrastructure@dukekunshan.edu.cn) 即会生成服务工单；开通不涉及经费划转或额外审批，通常在 1 个工作日内处理。无需先取得导师经费才能申请学习账号。

**科研账号（Research account）**适用于超出学习账号限制的 GPU 使用，需要可用经费及相应审批：

- 按月计费的项目，由有可用经费的项目所有者或负责人填写[科研账号申请表](https://dukekunshan.formstack.com/forms/dku_gpu_research_account_request_form)，注明账号结束日期；同一项目可在一张表中申请多个账号。
- SW 或支持 SW 的独立研究如需额外算力，可申请 Signature Work Research Grants（SWRG）；由负责监督资助的教师或员工填写表单，注明每名学生获批金额及资助到期日。
- SRS 项目的额外算力由教师按既有 SRS 协议提供经费。IT 在所需审批齐全后配置科研权限。

2026 年 10 月 23 日通知明确了以下安排：

1. **2026 年 12 月 1 日**，除已配置为科研账号者外，现有账号将转换为学习账号。
2. 自该日或之后，超出学习账号限制的作业将被取消；已归属于设立好的科研账号且符合科研账号限制的作业除外。
3. 科研使用统计和计费从该日开始；首轮账单将在 **2027 年元旦假期后**发送给项目负责人。

需要科研权限的用户应提前完成申请，不要把旧的 `faculty` 分组或旧作业配置当作已获新科研权限。两类账号的资源限制、分区和费用见 Section 5。

**账号类型不等于文件权限。** Learning / Research 是 GPU 资源使用与计费分类；SSH 登录认证、Slurm 的 `Account` / QoS，以及目录的 POSIX / NFSv4 ACL 是不同层面。拥有科研账号不代表可以读取他人的目录，也不代表可以绕过 Slurm。

### 登录入口

目前 DKUCC 的 hostname 是 `dkucc-login-01.rc.duke.edu`, 我们的用户名也就是 netid .
如果没有像 Section 4 那样配置公钥实现免密登录，每次 ssh DKUCC 都需要输入密码，密码和 DKU 邮箱的密码一致。

```
ssh <NETID>@dkucc-login-01.rc.duke.edu
#      ↑              ↑
#    用户名         hostname
```

下图是 ssh DKUCC 的样例，出现如图的状态即代表登陆成功
图片上的 ssh 指令并没有使用完整的 hostname，这是因为配置了 `.ssh/config`，接下来的 Section 2 就会介绍。

![](DKUCC%20使用指南.assets/image-706.png)

### DCC 培训 PDF

官网 [Training](https://dkucc.dukekunshan.edu.cn/) 页说明：DKUCC is a replica of DCC（Duke Compute Cluster），因此 DKU 用户也可以参考 DCC 的入门培训材料。Slides 的 PDF 版本见 Duke Box：

[Current pdf version of the slides（DCC 入门培训）](https://duke.app.box.com/s/zosu3w34iap7o4yuvlq44zep8ic27byq)

PDF 中的 Slurm 概念、作业提交流程和常用命令仍然适用，很值得学习！
只不过部分细节可能过时，例如材料里的 hostname 是 `dkucc-login-01.rc.duke.edu`，当前 DKUCC 入口则是 `dkucc-login-01.rc.duke.edu`。

---

<a id="section-2"></a>

## 「Section 2」Windows / macOS 的 SSH 初始化

Section 2 这部分仅包含安装 SSH 客户端并写好连接别名，采用默认密码登录；之后需要免密登录，可以阅读 Section 4 配置公钥。

### Windows：检查 OpenSSH Client

打开 PowerShell运行：

```powershell
ssh -V
```

看到版本号，说明已经安装过，可以进行下一步。
如果提示找不到 `ssh` ，安装OpenSSH Client：
Windows 11 在“设置 → 系统 → 可选功能 → 查看功能”中搜索 `OpenSSH Client`；
Windows 10 在“设置 → 应用 → 可选功能 → 添加功能”中搜索。

也可以打开管理员 PowerShell：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

安装完成后关闭并重新打开 PowerShell，再运行一次 `ssh -V`，如果出现版本号，那么意味着下载成功。
Windows 自带组件已经足够，不需要为了连集群额外下载不明 SSH 客户端。

接着创建 SSH 配置文件：

```powershell
New-Item -ItemType Directory -Force "$HOME\.ssh"
notepad "$HOME\.ssh\config"
```

把下面内容粘贴进去，将 `<NETID>` 换成自己的 Duke NetID。这样以后只需输入 `ssh dkucc`，不必每次手写完整主机名。这里的 dkucc 是自定义的 hostname 别称，可以换成任何你觉得好记的，输入方便的名称。

```sshconfig
Host dkucc
    HostName dkucc-login-01.rc.duke.edu
    User <NETID>
```

### macOS：系统自带工具应该已经包含 OpenSSH

macOS 自带 OpenSSH，打开“终端”即可：

```bash
ssh -V
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/config
```

写入配置，将 `<NETID>` 换成自己的 Duke NetID。这样以后只需输入 `ssh dkucc`，不必每次手写完整主机名。这里的 dkucc 是自定义的 hostname 别称，可以换成任何你觉得好记的，输入方便的名称：
```sshconfig
Host dkucc
    HostName dkucc-login-01.rc.duke.edu
    User <NETID>
```

在 `nano` 中按 `Ctrl+O` 保存、回车确认，再按 `Ctrl+X` 退出。然后限制配置文件权限：

```bash
chmod 600 ~/.ssh/config
```

---

<a id="section-3"></a>

## 「Section 3」第一次登录与目录规划

### 先连接 DKU VPN

从校外网络 SSH 到 DKUCC 登录节点前，**必须先连接 DKU VPN**，未连 VPN 时，本机一般无法到达 `dkucc-login-01.rc.duke.edu` 的 22 端口，表现为 `ssh: Could not resolve hostname dkucc-login-01.rc.duke.edu:` 或 ` Connection timed out `。

VPN 安装可参考 **[墙记｜入学指南之线上准备2！](https://mp.weixin.qq.com/s?__biz=MzkxMjczOTA4NQ==&mid=2247486055&idx=1&sn=3b54062b550949ad329987943817fea0&chksm=c1091405f67e9d136dde1c38d40b50e0791a13518b3dbc1d9bf741e0aabf2b02a1fb8bd2ec4a&cur_album_id=3994731232327254017&scene=190#rd)**，也可参考文末的 [cisco-vpn-autoconnect](https://github.com/guyong1449/cisco-vpn-autoconnect) 辅助脚本。
确认 VPN 已连接后，再在本机执行 Section 2 中配置好的 `ssh dkucc` 或 `ssh <NETID>@dkucc-login-01.rc.duke.edu`，即可登录节点。

### 连接与首次自检

DKUCC 的登录 hostname 是 `dkucc-login-01.rc.duke.edu`，首次能登录后执行：

```bash
whoami
pwd
df -h /dkucc/home/$USER /work/$USER # 查看内存占用
sinfo -s # 查看集群分区
```

示例输出：

![](DKUCC%20使用指南.assets/image-708.png)

登录后，所在位置会在 `/dkucc/home/<NETID>`，也就是 linux 的 `~` 目录，这部分总体容量为 1 T，但是就算这样，也**极度不建议**写入 checkpoint 或者存放大量 data，可能会影响所有用户的登录与共享存储，导致网络盘卡顿等等，为了保证使用规范，请把大文件应放到 `/work` 共享盘，避免拖累整个系统。

| 位置                    | 放什么                                     | 注意事项                                                                              |
| --------------------- | --------------------------------------- | --------------------------------------------------------------------------------- |
| `/dkucc/home/<NETID>` | 环境、`.bashrc`、小脚本、个人配置，代码，占用不大的 log 等等产物 | 网络盘，上限 1 TB，无法访问其他用户的家目录                                                          |
| `/work/<NETID>`       | 数据集、checkpoint、输出产物                     | 网络盘，采用 NFSv4 挂载，总共 70 TB 共享高速卷；<br>超过 75 天的文件自动清理，脚本半个月执行一次（刷新数据时间戳方案见 Section 7） |

DKUCC 定位是单人服务器，默认情况下可以 `ls` 查看别人的目录（755），但是无法读写或者执行文件内容（600）
（此外，虽然看上去权限是 linux 的典型权限模型 POSIX，比如ls -l 看到的 rwxr-xr-x，但实际上生效的大概是NFS4 ACL 这一套系统，POSIX 应该是根据 ACL 合成出来的. 
总之从严谨角度来说，不建议直接 `chmod` 给权限，容易改坏。
具体 NFS4 ACL 编辑权限的方法，或者如果实在需要多人协作共享目录，需要合理地给目录权限，请参考文末 [DKUCC-skill](https://github.com/guyong1449/DKUCC-skill) 中的 `nfs4-acl` / `dkucc-permission-audit`）

DKUCC 无正式备份，但是如果文件消失了，数据文件被 75 天定时脚本删除或者自己误删了，仍有短期可能找回的方法，可以查 `bash_history` 查看有没有执行过相关 `rm` 指令，`/work` 和 `/home` 的 NFS 都存在短周期快照，可查看 `ls -la /work/$(whoami)/.snapshot/`，对比数据被删前后的快照，可以从快照 `rsync` 回来；具体步骤见 [dkucc-data-recovery](https://github.com/guyong1449/DKUCC-skill)。

此外，当前的节点名为 `dkucc-login-01`，也就是登陆节点，这里只有 CPU，如果需要执行需要 heavy calculation 的代码，请申请计算节点，**禁止在 CPU 节点执行需要 GPU 的任务，同样的，需要大量 CPU 的任务，也请申请**

---

<a id="section-4"></a>

## 「Section 4」SSH 公钥、LDAP 与免密登录

长期使用服务器的话每次都输出密码实在很麻烦，一般情况下直接把公钥复制到 `~/authorized_keys` 内就能免密登录，但是在 DKUCC 却无法实现这一点。

### 在本机生成密钥对

Windows（PowerShell）或 macOS（终端）均可：

```bash
ssh-keygen -t ed25519 -C "<NETID>@duke.edu"
```

直接多次回车，默认生成 `~/.ssh/id_ed25519`；是否为私钥设口令可选。生成后确认公钥指纹：

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
cat ~/.ssh/id_ed25519.pub
```

在 `~/.ssh/config` 的 `Host dkucc` 中加上：

```sshconfig
    IdentityFile ~/.ssh/id_ed25519
```

DKUCC 为什么无法实现 `~/.ssh/authorized_keys` 就能免密登录呢？
这是因为登录节点 `dkucc-login-01.rc.duke.edu` 上，公钥认证的读取顺序如下：

输入以下指令：
```bash
grep -E 'AuthorizedKeys|PubkeyAuthentication' /etc/ssh/sshd_config
```
可以看到输出如下：
```text
PubkeyAuthentication yes
AuthorizedKeysCommand /usr/local/bin/ldap-authkey.sh
AuthorizedKeysCommandUser nobody
AuthorizedKeysFile /var/lib/ssh/%u/authorized_keys /%u/.ssh/authorized_keys
```

参见下图，sshd 系统需要读取我们复制过去的公钥内容，实现免密，但是读取的文件夹位置是 `/var/lib/ssh/<NETID>/authorized_keys` 和 `/<NETID>/.ssh/authorized_keys` (参见下图的右侧链路)

但问题在于，实际上用户实际 home 目录在 `/dkucc/home/<NETID>/`，因此 **`/dkucc/home/<NETID>/.ssh/authorized_keys` 不在 sshd 的 AuthorizedKeysFile 列表中**。

写入该文件的公钥不会被右侧的（2）链路读取，要在 DKUCC 启用公钥登录，公钥需被左侧（1）链路识别，也就是登记到 LDAP 的 `sshPublicKey`

```text
                    本机 ssh 客户端
                  （Offering public key）
                           │
                           ▼
              dkucc-login-01.rc.duke.edu
                        sshd
                           │
                           ▼
                 PubkeyAuthentication
                           │
           ┌───────────────┴───────────────┐
           │                               │
           ▼                               ▼
   ① AuthorizedKeysCommand          ② AuthorizedKeysFile
   /usr/local/bin/ldap-authkey.sh      （按列出的路径顺序）
   以用户 nobody 运行                        │
           │                               │
           ▼                               ▼
   getent passwd <NETID>            /var/lib/ssh/<NETID>/authorized_keys
   确认本地账号存在                          │
           │                               ▼
           ▼                      /<NETID>/.ssh/authorized_keys
   ldapsearch (uid=<NETID>)               │
   idms-authdir.oit.duke.edu              │
   读取 LDAP 属性 sshPublicKey            │
           │                               │
           ▼                               │
   每条公钥经 ssh-keygen -l 校验            │
           │                               │
           └───────────────┬───────────────┘
                           ▼
              合并 ① 与 ② 得到的公钥集合
                           │
                           ▼
           与客户端本次提供的公钥比对
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
           匹配成功                   不匹配
              │                         │
              ▼                         ▼
         免密登录              继续密码等其他认证方式
```

如何实现左侧（1）链路？
 [DKU Self-Service](https://idms-web-selfservice.oit.duke.edu/advanced)，点击 manage your ssh keys，粘贴你复制的公钥内容即可。

![](DKUCC%20使用指南.assets/image-709.png)

测试 `ssh dkucc`免密登录即为成功.

---

<a id="section-5"></a>

## 「Section 5」登录节点、计算节点和 Slurm 的分工

登录节点适合编辑代码、Git 操作、轻量测试，以及 `sbatch`、`squeue`、`scancel`、`sacct` 等作业管理命令，请不要在这里跑 epoch 级训练、大规模 `torchrun`、长期压缩或模型服务。

计算节点由 `sbatch` 或 `srun` 分配，用于运行训练、使用 GPU、读取大数据和写 checkpoint。ACL 是一个例外：只要 `/work` 已挂载，并且当前用户是目录 owner 或拥有 `C` 权限，就可以在计算节点上修改 ACL；但不能在计算节点提交或管理新的 Slurm 作业。

长时间的交互工作可以用 `tmux`，防止网络中断后 shell 消失：

```bash
tmux new -s main
# 重新登录后
tmux attach -t main
```

![](DKUCC%20使用指南.assets/image-707.png)

官网不鼓励用本机 VS Code Remote-SSH 在 DKUCC 上直接跑代码，
需要走正规入口：Open OnDemand 上的 **code-server**（集群已装 module `code-server/4.96.4`，路径在 `/opt/apps/almalinux9/code-server-4.96.4/`），不建议自己在节点上私自拉起 `vscode-server` 进行 ssh 连接。

### 学习账号与科研账号的 GPU 权限

下表依据 2026 年 10 月 16 日附件。2026 年 12 月 1 日的转换与计费安排见 Section 1；切换前后都应核对本人实际配置。

| 项目 | 学习账号 Learning | 科研账号 Research |
| --- | --- | --- |
| 适用对象 | DKU 学生、教师、员工；可用于学习和基础研究 | 需要额外算力、且具备经费与所需审批的项目用户，包括符合条件的 SRS / SWRG 学生 |
| 同时运行的作业数 | 最多 1 个 | 最多 1 个 |
| 每个作业的 GPU 数量 | 最多 1 张 | 最多 4 张 |
| 每自然月的 GPU 运行时间 | 最多 1 周 | 最多 4 周 |
| 可用 GPU 分区 | Common-GPU、L20-GPU、临时 4090（若开放） | L20-GPU、H20-GPU、临时 4090（若开放） |
| 调度要求 | 必须经 Slurm 提交，可能排队 | 必须经 Slurm 提交，可能排队 |
| 经费与计费 | 限额内由 DKU 承担，不向个人直接收费 | 按项目实际使用量计费，或按 SWRG 资助额设置总资源额度 |

**运行时间口径：**附件原文为 “Total GPU runtime”，并按自然月规定 1 周／4 周；未进一步说明多 GPU 作业如何汇总这一限额。因此，这里不把它擅自换算成 GPU-hours，也不把月累计额度理解成单个作业的最长运行时间。需要精确预算或长任务安排时，请向 IT 确认计量及限额规则。资源限制可按使用情况和资源可用性调整。

**排队与优先级：**Common-GPU 专供学习账号，H20-GPU 专供科研账号，L20-GPU 与临时 4090 可供两类账号使用。所有分区均需排队；附件明确说明，**不会因账号类型或项目而给予调度优先权**。临时 4090 可能在暑期从教学环境释放，不保证全年可用。

### 费用与账号有效期

| 科研使用资源 | 费率（人民币／GPU-hour） |
| --- | ---: |
| L20-GPU | ¥0.94 |
| H20-GPU | ¥3.95 |
| 临时 4090 | ¥0.47 |

学习账号在规定限额内不向个人直接收费。上述科研费率按 GPU-hour 计费。

- **项目按月计费：**IT 汇总上月使用情况并向项目负责人发送账单；负责人应在收到后 5 个工作日内提出疑问。经费不足时，应在 5 个工作日内提供其他计费账号，否则 IT 可终止相关运行中或排队作业，并暂停科研账号直至欠款支付。
- **SWRG 总额控制：**IT 按资助金额设置总使用额度；账号有效期到资助到期或额度用尽为止，以先发生者为准，不向学生发送账单。
- 项目负责人应在申请中填写科研账号结束日期。账号权限、资源分配及费率如有更新，以后续 IT 通知为准。

### 查看本人 Slurm 关联

```bash
sacctmgr show assoc user=$USER format=Account,Partition,QOS,MaxJobs,MaxTRES -P
```

“最多同时运行 1 个作业”不是“最多提交 1 个作业”，也不代表 CPU 资源可无限使用。CPU 分区的提交限制、时间上限和可用资源应另行核对。

### 分区与 GPU 型号

`sinfo` 可以查看具体分区和节点情况：

```bash
sinfo
```

以下为输出示例：

```text
PARTITION           AVAIL  TIMELIMIT  NODES  STATE NODELIST
common*                up 90-00:00:0      1  down* dkucc-core-dkurc-u-it06-2-2
common*                up 90-00:00:0      4    mix dkucc-core-dkurc-u-it06-1-1,dkucc-core-dkurc-u-it06-2-[3-4],dkucc-core-dkurc-u-it07-3-1
common*                up 90-00:00:0      4  alloc dkucc-core-dkurc-u-it06-1-[2-4],dkucc-core-dkurc-u-it06-2-1
common*                up 90-00:00:0      9   idle dkucc-core-dkurc-u-it07-3-[2-5],dkucc-core-dkurc-u-it07-4-[1-5]
common-gpu             up 7-00:00:00      1    mix dkucc-core-gpu-dkurc-d-it09-8
common-gpu             up 7-00:00:00      1   idle dkucc-core-gpu-dkurc-d-it09-3
l20-gpu                up 7-00:00:00      3    mix dkucc-core-gpu-dkurc-a-it08-[22,28],dkucc-core-gpu-dkurc-a-it09-26
l20-gpu                up 7-00:00:00      1  alloc dkucc-core-gpu-dkurc-a-it09-14
h20-gpu                up 7-00:00:00      1   mix- dkucc-core-gpu-dkurc-z-it13-05
```


| Slurm 分区示例 | `--gres` 示例 | 附件所列单卡显存 | 允许的账号类型 | 常见用途 |
| --- | --- | --- | --- | --- |
| `common-gpu` | `gpu:a40:N` | A40 48 GB | 仅学习 | 交互式 3D / [Unity](https://unity.com/) 等引擎开发、远程可视化、通用推理与中小规模训练 |
| `l20-gpu` | `gpu:l20:N` | L20 48 GB | 学习／科研 | 计算机视觉（检测、分割、视频理解）训练与推理 |
| `h20-gpu` | `gpu:h20:N` | H20 141 GB | 仅科研 | 大语言模型与长上下文推理、微调及多卡并行训练 |

`N` 是申请的 GPU 数量：学习账号最多为 1，科研账号最多为 4，无论是什么账号，都只有一个申请的任务能在 GPU 节点运行。

---

<a id="section-6"></a>

## 「Section 6」GPU 作业：提交、排队、抢占和恢复

### Slurm 常用工具指令

#### 1. 查看所有分区与节点概览：`sinfo`

```bash
sinfo
```

**含义：** 列出各 Slurm 分区（partition）的可用性、作业时间上限、节点数量、节点状态（`idle` 空闲 / `mix` 部分占用 / `alloc` 已满 / `down*` 故障）及节点名列表。带 `*` 的分区（如 `common*`）为默认分区。

**输出示例：**

```text
PARTITION           AVAIL  TIMELIMIT  NODES  STATE NODELIST
common*                up 90-00:00:0      1  down* dkucc-core-dkurc-u-it06-2-2
common*                up 90-00:00:0      4    mix dkucc-core-dkurc-u-it06-1-1,dkucc-core-dkurc-u-it06-2-[3-4],dkucc-core-dkurc-u-it07-3-1
common*                up 90-00:00:0      4  alloc dkucc-core-dkurc-u-it06-1-[2-4],dkucc-core-dkurc-u-it06-2-1
common*                up 90-00:00:0      9   idle dkucc-core-dkurc-u-it07-3-[2-5],dkucc-core-dkurc-u-it07-4-[1-5]
common-gpu             up 7-00:00:00      1    mix dkucc-core-gpu-dkurc-d-it09-8
common-gpu             up 7-00:00:00      1   idle dkucc-core-gpu-dkurc-d-it09-3
l20-gpu                up 7-00:00:00      3    mix dkucc-core-gpu-dkurc-a-it08-[22,28],dkucc-core-gpu-dkurc-a-it09-26
l20-gpu                up 7-00:00:00      1  alloc dkucc-core-gpu-dkurc-a-it09-14
h20-gpu                up 7-00:00:00      1   mix- dkucc-core-gpu-dkurc-z-it13-05
...
```

更紧凑的汇总可用 `sinfo -s`（列 `NODES(A/I/O/T)` 表示已分配 / 空闲 / 其他 / 总计）。

#### 2. 查看单台节点的硬件与占用：`scontrol show node`

```bash
scontrol show node dkucc-core-gpu-dkurc-d-it09-3
```

**含义：** 查看指定计算节点的详细配置与当前资源占用。
**常用字段：**`Gres`（GPU 型号与数量）、`RealMemory`（物理内存，单位 MB）、`State`（`IDLE` / `MIXED` / `ALLOCATED` 等）、`AllocTRES`（已分配的 CPU / 内存 / GPU）

**输出示例（`common-gpu` 节点，节选）：**

```text
NodeName=dkucc-core-gpu-dkurc-d-it09-3
   Gres=gpu:a40:8
   CPUTot=96  CPUAlloc=0  CPULoad=1.51
   RealMemory=1030745  AllocMem=0  FreeMem=22643
   State=IDLE
   Partitions=common-gpu
   CfgTRES=cpu=96,mem=1030745M,billing=96,gres/gpu=8
   AllocTRES=
```

对 L20 等其它分区节点，字段含义相同；`State=MIXED` 且 `AllocTRES gres/gpu=N` 表示该节点部分 GPU 已被占用。

#### 3. 指定节点申请交互 shell：`srun` + `-w`

**以下四卡示例仅适用于已获相应科研权限的账号**；学习账号不得照抄四卡请求。提交前请核对 `sinfo`，且可能排队。

```bash
srun -p l20-gpu -w dkucc-core-gpu-dkurc-a-it09-26 \
  --gres=gpu:l20:4 --cpus-per-task=48 --mem=256G --pty bash -i
```

| 参数 | 含义 |
|---|---|
| `-p l20-gpu` | 使用 `l20-gpu` 分区 |
| `-w <节点名>` | 指定运行在哪台计算节点（`-w` / `--nodelist`） |
| `--gres=gpu:l20:4` | 申请 4 张 L20 GPU，仅适用于符合限制的科研账号 |
| `--cpus-per-task=48` | 申请 48 个 CPU 核 |
| `--mem=256G` | 申请 256 GB 内存 |
| `--pty bash -i` | 分配成功后进入交互式 bash |

一般不必指定 `-w`，让调度器自动选节点即可；只有在确知某台机器内存、GPU 型号或驱动满足需求、且该节点仍有空闲资源时，才考虑 `-w`。

#### 4. 查看自己的作业队列：`squeue -u`

```bash
squeue -u $USER
# 或
squeue -u <NETID>
```

**含义：** 列出指定用户当前在队列中或运行中的作业。`ST` 列：`R` 运行中，`PD` 排队中，`CG` 正在结束等。`TIME` 为已运行时长；`NODELIST` 为实际运行节点。

**输出示例：**

```text
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             30136   l20-gpu     bash   user_a  R 2-16:14:52      1 dkucc-core-gpu-dkurc-a-it08-22
             30429   l20-gpu pomo_cvrp20   user_b  R    7:31:11      1 dkucc-core-gpu-dkurc-a-it09-26
```

#### 5. 重新连接已有交互作业的 shell：`srun --jobid`

```bash
srun --jobid=<JOBID> --pty bash
```

**含义：** 网络中断或终端关闭后，重新 attach 到**仍在运行**的交互式 Slurm 作业。每个 job **只允许一个** attach 的 shell；若已有连接，需先退出旧 shell 或新开终端再连。

#### 6. 取消作业：`scancel`

```bash
scancel <JOBID>
```

**含义：** 取消指定作业（排队中的 pending 或正在运行的 running）。取消自己的 job 不需要额外权限；误删他人作业需管理员介入。

#### 7. 查看单个作业的详细信息：`squeue -j` + 自定义格式

```bash
squeue -j <JOBID> -o "%.18i %.9P %.20j %.8u %.2t %.10M %.9l %.6D %.20R"
```

**含义：** 对单个 `JOBID` 按固定列宽输出分区、作业名、用户、状态、已运行时间、时间上限、节点数和调度原因（`REASON` / `NODELIST`）。

**输出示例：**

```text
             JOBID PARTITION                 NAME     USER ST       TIME TIME_LIMI  NODES     NODELIST(REASON)
             30136   l20-gpu                 bash   user_a  R 2-16:14:52 4-10:00:00      1 dkucc-core-gpu-dkurc-a-it08-22
```

作业结束后历史信息用 `sacct -j <JOBID>` 查询

### 批处理脚本示例

下面是 Slurm 批处理脚本示例，需要先在提交前创建日志目录，Slurm 会在脚本开始前打开 `--output` 和 `--error` 指定的文件，因此不能依赖脚本内部的 `mkdir`：

```bash
mkdir -p /work/<NETID>/slurm/logs
```

然后新建 `/work/<NETID>/slurm/train.sh`：

```bash
#!/bin/bash
#SBATCH --job-name=my-train
#SBATCH --partition=l20-gpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --gres=gpu:l20:1
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --time=24:00:00
#SBATCH --chdir=/work/<NETID>/repos/myproject
#SBATCH --output=/work/<NETID>/slurm/logs/%x-%j.out
#SBATCH --error=/work/<NETID>/slurm/logs/%x-%j.err

set -euo pipefail
mkdir -p /work/<NETID>/slurm/logs
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /work/<NETID>/envs/myenv
export PYTHONNOUSERSITE=1

echo "Job $SLURM_JOB_ID on $(hostname) at $(date)"
nvidia-smi -L
python train.py --output-dir /work/<NETID>/outputs/run_${SLURM_JOB_ID}
```

提交和观察：

```bash
sbatch /work/<NETID>/slurm/train.sh
squeue -u $USER
tail -f /work/<NETID>/slurm/logs/my-train-<JOBID>.out
sacct -j <JOBID> --format=JobID,State,ExitCode,Elapsed,Timelimit,MaxRSS,DerivedExitCode
```

### 短时间调试与长训练

短时间调试可以申请交互 shell：

```bash
srun --partition=l20-gpu --gres=gpu:l20:1 --cpus-per-task=4 --mem=32G --time=01:00:00 --pty bash -l
```

交互 shell 只用于调试。长训练应使用 `sbatch`。如果一个脚本同时负责提交作业和执行训练，可以按 `SLURM_JOB_ID` 分支：

```bash
if [[ -z "${SLURM_JOB_ID:-}" ]]; then
  sbatch --export=ALL,MY_VAR=value "$0" "$@"
else
  python main.py
fi
```


<a id="section-7"></a>

## 「Section 7」`/work` 数据保留与定期检查

`/work` 下时间戳旧于 75 天的文件，会被每半个月清理一次，需要更新文件时间戳时，应把范围限制在确实需要保留的目录。扫描整个 `/work/<NETID>` 会产生大量 I/O，不应直接在登录节点运行。下面的命令通过 Slurm 申请 CPU 资源，只处理 `data` 和 `outputs`：

```bash
srun --partition=common --time=04:00:00 --mem=2G --cpus-per-task=1 \
  bash -lc 'find /work/<NETID>/data /work/<NETID>/outputs -type f -print0 | xargs -0 -r touch'
```

如需自动运行，可以让 cron 在每月 1 日提交一次 CPU 作业。
先创建日志目录和 `/work/<NETID>/slurm/touch-work.sh`：

```bash
mkdir -p /work/<NETID>/slurm/logs
```

```bash
#!/bin/bash
#SBATCH --job-name=touch-work
#SBATCH --partition=common
#SBATCH --time=04:00:00
#SBATCH --mem=2G
#SBATCH --cpus-per-task=1
#SBATCH --output=/work/<NETID>/slurm/logs/touch-work-%j.out
#SBATCH --error=/work/<NETID>/slurm/logs/touch-work-%j.err

set -euo pipefail
find /work/<NETID>/data /work/<NETID>/outputs -type f -print0 2>/dev/null | xargs -0 -r touch
```

确认脚本可以通过 `sbatch` 正常运行后，再执行 `crontab -e`，加入：

```cron
0 2 1 * * /usr/bin/sbatch /work/<NETID>/slurm/touch-work.sh >> /work/<NETID>/slurm/touch-submit.log 2>&1
```

该表达式表示“每月 1 日 02:00” 重新扫描，cron 只在登录节点提交作业，实际扫描由 Slurm 分配的 CPU 节点执行。

检查最旧文件年龄：

```bash
tail -n 50 /work/<NETID>/slurm/touch-submit.log
srun --partition=common --time=01:00:00 --mem=2G --cpus-per-task=1 bash -lc '
  OLDEST=$(find /work/<NETID>/data /work/<NETID>/outputs -type f -printf "%T@\n" 2>/dev/null | sort -n | head -1)
  if [[ -n "$OLDEST" ]]; then
    DAYS=$(( ($(date +%s) - ${OLDEST%.*}) / 86400 ))
    echo "最旧文件约 ${DAYS} 天前被修改或 touch"
  fi
'
```

完整的文件时间戳维护 skill 位于（`~/bin/touch_work_timestamps.sh`、每月 1/15 日 cron 一遍。针对具体文件夹空间的 `monthly_check.sh`）可参考 [dkucc-data-ingest](https://github.com/guyong1449/DKUCC-skill)；若文件已被清理或误删，快照恢复步骤见 [dkucc-data-recovery](https://github.com/guyong1449/DKUCC-skill)。

---

<a id="official-resources"></a>

## 官方资料


- DKU IT 2026 年 10 月 23 日邮件：确定 2026 年 12 月 1 日账号转换、科研使用统计与计费，以及首轮账单安排。
- 邮件附件《DKU Research GPU management》，日期 2026 年 10 月 16 日：账号类型、资源限制、分区、申请流程、费率与有效期。
- [科研账号申请表](https://dukekunshan.formstack.com/forms/dku_gpu_research_account_request_form)。

其他参考资料：

- DKUCC Official Website：
  [首页](https://dkucc.dukekunshan.edu.cn/)
- [Duke Research Computing Support Documentation](https://oit-rc.pages.oit.duke.edu/rcsupportdocs/)
- [VS Code Remote-SSH](https://code.visualstudio.com/docs/remote/ssh)
- [NVIDIA A40 数据表](https://images.nvidia.com/content/Solutions/data-center/a40/nvidia-a40-datasheet.pdf)
- [NVIDIA L20 产品说明](https://www.nvidia.com/en-us/data-center/l20/)
- [NVIDIA H20 产品说明](https://www.nvidia.com/en-us/data-center/h20/)
- [NVIDIA AI Enterprise GPU 支持表](https://docs.nvidia.com/ai-enterprise/release-7/latest/infra-software/vgpu/features.html)

<a id="tools"></a>

## 可参考的工具

- [guyong1449/cisco-vpn-autoconnect](https://github.com/guyong1449/cisco-vpn-autoconnect)：在 Windows 下自动调用 Cisco Secure Client 连接 DKU VPN，并支持 Duo Push
- [guyong1449/DKUCC-skill](https://github.com/guyong1449/DKUCC-skill)：将本指南内较为复杂，不方便在正文内完整描述，或者适合用 AI 协助排查的工作流拆分整理成的 skill 合集，还有 DKUCC 官网网页缓存，IT 相关邮件原文留档等相关内容
