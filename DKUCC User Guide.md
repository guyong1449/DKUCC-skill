---
tags:
  - area/dkucc
  - type/reference
  - status/published
status: published
---

# DKUCC User Guide

[中文](DKUCC%20使用指南.md) · [Repository overview](README.en.md)

The following mainly covers DKUCC account applications, SSH login, directory planning, Slurm jobs, and data retention. The explanations of accounts and GPU permissions have been updated based on the DKU IT notice of October 23, 2026, and its attachment, DKU Research GPU management. **The new account conversion and research usage billing will take effect on December 1, 2026**. See Section 1 and Section 5 for details.

## Reporting Problems

For network, DKU VPN, or other DKU account problems, contact [service-desk@dukekunshan.edu.cn](mailto:service-desk@dukekunshan.edu.cn) to submit a ticket;
if you use DKUCC regularly, you can add me on WeChat: 18851063808, and I will add you to the DKUCC problem-reporting group. This can get a quicker response to problems than email.

## Contents

- [Section 0 Quick Reference](#section-0)
- [Section 1 DKUCC and Official Entry Points](#section-1)
- [Section 2 SSH Setup on Windows / macOS](#section-2)
- [Section 3 First Login and Directory Planning](#section-3)
- [Section 4 SSH Public Keys, LDAP, and Passwordless Login](#section-4)
- [Section 5 Roles of Login Nodes, Compute Nodes, and Slurm](#section-5)
- [Section 6 GPU Jobs: Submission, Queuing, Preemption, and Recovery](#section-6)
- [Section 7 `/work` Data Retention and Regular Checks](#section-7)
- [Official Resources](#official-resources)
- [Useful Tools](#tools)

<a id="section-0"></a>

## 「Section 0」Quick Reference

1. Your initial SSH connection takes you to a login node. **Do not run GPU tasks or computational tasks that need large amounts of CPU directly on `dkucc-login-01`**.
2. Large computational tasks should be submitted to compute nodes through Slurm. See Section 5 and Section 6 for instructions.
3. Your directory after login is usually `/dkucc/home/<NETID>`. Use it for small files such as code and configuration. Large amounts of data, checkpoints, and outputs should go in `/work/<NETID>`.
4. The `/work` cleanup rule is: files with timestamps older than 75 days are cleaned up every half month.
5. Do not bypass Slurm by using SSH to connect directly to compute nodes to occupy GPUs or run training.
6. Release the resources you requested when a job ends. Do not rely on leftover processes to keep running; after a task is `COMPLETED`, the GPU node allocated to it will soon be reclaimed, even if you have screen or tmux running.
7. Learning and research accounts have different GPU counts, monthly runtime limits, and available partitions. **Both account types can run at most 1 job at a time**. See Section 5 for details.

---

<a id="section-1"></a>

## 「Section 1」DKUCC and Official Entry Points

Official website: https://dkucc.dukekunshan.edu.cn/

| Page | Contents |
| --------------------------------- | ------------------------------- |
| Logging into the DKUCC | SSH, VPN, limits on login-node use, and Open OnDemand |
| Working with files | Uses, capacity, file retention, and transfers for Home and `/work` |
| Scheduling Jobs | Slurm job submission and management |
| How to Use GPU Resource | GPU partitions and interactive sessions |
| Modules / User installed software | Environment modules and personal software installation |
| OnDemand / OnDemand Overview | OnDemand is recommended for requesting nodes and using interactive applications |
| Training | DCC introductory training and self-study materials |

DKUCC is a computing cluster made up of login nodes, compute nodes, and shared storage. Users connect to a login node through an encrypted SSH (Secure Shell Protocol) connection, then request computing resources according to their tasks.

### Account Applications and Policy Transition

**Learning accounts** are available to DKU students, faculty, and staff, and can also be used for basic research. Sending an email to [it-infrastructure@dukekunshan.edu.cn](mailto:it-infrastructure@dukekunshan.edu.cn) creates a service ticket; activation does not involve a transfer of funds or additional approval, and is usually handled within 1 working day. You do not need to obtain funding from an advisor before applying for a learning account.

**Research accounts** are for GPU usage beyond the limits of learning accounts and require available funding and the corresponding approvals:

- For projects billed monthly, the project owner or lead with available funding completes the [research account application form](https://dukekunshan.formstack.com/forms/dku_gpu_research_account_request_form), specifying the account end date; multiple accounts for the same project can be requested on one form.
- SW or independent research supporting SW that needs additional computing power can apply for Signature Work Research Grants (SWRG); the faculty or staff member responsible for overseeing the funding completes the form, specifying the amount approved for each student and the funding expiration date.
- Additional computing power for SRS projects is funded by faculty under existing SRS agreements. IT configures research permissions after all required approvals are in place.

The October 23, 2026 notice specifies the following arrangements:

1. On **December 1, 2026**, existing accounts will be converted to learning accounts, except those already configured as research accounts.
2. On or after that date, jobs exceeding learning account limits will be canceled, except for jobs already associated with established research accounts and within research account limits.
3. Research usage tracking and billing begin on that date; the first bills will be sent to project leads **after the 2027 New Year holiday**.

Users who need research permissions should complete their applications in advance. Do not treat membership in the old `faculty` group or an old job configuration as confirmation that you have the new research permissions. See Section 5 for the resource limits, partitions, and fees for the two account types.

**Account type is not the same as file permissions.** Learning / Research are classifications for GPU resource usage and billing; SSH login authentication, Slurm `Account` / QoS, and directory POSIX / NFSv4 ACLs are separate layers. Having a research account does not mean you can read other people's directories or bypass Slurm.

### Login Entry Point

The current DKUCC hostname is `dkucc-login-01.rc.duke.edu`, and our username is our netid.
If you have not configured a public key for passwordless login as described in Section 4, you need to enter a password every time you SSH into DKUCC. The password is the same as your DKU email password.

```
ssh <NETID>@dkucc-login-01.rc.duke.edu
#      ↑              ↑
#   username       hostname
```

The image below shows an example of SSH into DKUCC. Reaching the state shown in the image means you have logged in successfully.
The SSH command in the image does not use the full hostname because `.ssh/config` has been configured, which will be covered in Section 2 next.

![](DKUCC%20使用指南.assets/image-706.png)

### DCC Training PDF

The official website's [Training](https://dkucc.dukekunshan.edu.cn/) page states: DKUCC is a replica of DCC (Duke Compute Cluster), so DKU users can also refer to DCC's introductory training materials. The PDF version of the slides is on Duke Box:

[Current pdf version of the slides (DCC introductory training)](https://duke.app.box.com/s/zosu3w34iap7o4yuvlq44zep8ic27byq)

The Slurm concepts, job submission process, and common commands in the PDF still apply and are well worth learning!
Some details may be outdated, though. For example, the hostname in the materials is `dkucc-login-01.rc.duke.edu`, while the current DKUCC entry point is `dkucc-login-01.rc.duke.edu`.

---

<a id="section-2"></a>

## 「Section 2」SSH Setup on Windows / macOS

Section 2 only covers installing an SSH client and setting up a connection alias, using the default password login. If you want passwordless login later, read Section 4 to configure a public key.

### Windows: Check OpenSSH Client

Open PowerShell and run:

```powershell
ssh -V
```

If you see a version number, it is already installed and you can proceed to the next step.
If you are told that `ssh` cannot be found, install OpenSSH Client:
on Windows 11, search for `OpenSSH Client` under “Settings → System → Optional features → View features”;
on Windows 10, search under “Settings → Apps → Optional features → Add a feature”.

You can also open PowerShell as administrator:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

After installation, close and reopen PowerShell, then run `ssh -V` again. If a version number appears, the download was successful.
The built-in Windows component is sufficient. There is no need to download an unfamiliar SSH client just to connect to the cluster.

Next, create an SSH configuration file:

```powershell
New-Item -ItemType Directory -Force "$HOME\.ssh"
notepad "$HOME\.ssh\config"
```

Paste in the following, replacing `<NETID>` with your own Duke NetID. This lets you simply type `ssh dkucc` in the future without typing the full hostname each time. Here, dkucc is a custom hostname alias. You can replace it with any name you find easy to remember and convenient to type.

```sshconfig
Host dkucc
    HostName dkucc-login-01.rc.duke.edu
    User <NETID>
```

### macOS: The Built-in Tools Should Already Include OpenSSH

macOS comes with OpenSSH. Simply open Terminal:

```bash
ssh -V
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/config
```

Enter the configuration, replacing `<NETID>` with your own Duke NetID. This lets you simply type `ssh dkucc` in the future without typing the full hostname each time. Here, dkucc is a custom hostname alias. You can replace it with any name you find easy to remember and convenient to type:
```sshconfig
Host dkucc
    HostName dkucc-login-01.rc.duke.edu
    User <NETID>
```

In `nano`, press `Ctrl+O` to save, Enter to confirm, and then `Ctrl+X` to exit. Then restrict the configuration file's permissions:

```bash
chmod 600 ~/.ssh/config
```

---

<a id="section-3"></a>

## 「Section 3」First Login and Directory Planning

### Connect to DKU VPN First

Before using SSH to connect to the DKUCC login node from an off-campus network, **you must connect to DKU VPN first**. Without the VPN, your local computer generally cannot reach port 22 on `dkucc-login-01.rc.duke.edu`, resulting in `ssh: Could not resolve hostname dkucc-login-01.rc.duke.edu:` or ` Connection timed out `.

For VPN installation, refer to **[墙记 | Enrollment Guide: Online Preparation 2!](https://mp.weixin.qq.com/s?__biz=MzkxMjczOTA4NQ==&mid=2247486055&idx=1&sn=3b54062b550949ad329987943817fea0&chksm=c1091405f67e9d136dde1c38d40b50e0791a13518b3dbc1d9bf741e0aabf2b02a1fb8bd2ec4a&cur_album_id=3994731232327254017&scene=190#rd)**. You can also refer to the [cisco-vpn-autoconnect](https://github.com/guyong1449/cisco-vpn-autoconnect) helper script at the end of this guide.
After confirming that the VPN is connected, run `ssh dkucc`, as configured in Section 2, or `ssh <NETID>@dkucc-login-01.rc.duke.edu` on your local computer to log in to the node.

### Connection and Initial Checks

The DKUCC login hostname is `dkucc-login-01.rc.duke.edu`. Once you can log in for the first time, run:

```bash
whoami
pwd
df -h /dkucc/home/$USER /work/$USER # Check memory usage
sinfo -s # Check cluster partitions
```

Example output:

![](DKUCC%20使用指南.assets/image-708.png)

After login, you will be in `/dkucc/home/<NETID>`, which is the Linux `~` directory. This area has a total capacity of 1 T, but even so, writing checkpoints or storing large amounts of data here is **strongly discouraged**. It may affect login and shared storage for all users, cause the network drive to stall, and so on. To keep usage in line with the rules, put large files on the `/work` shared drive to avoid slowing down the whole system.

| Location | What to Store | Notes |
| --------------------- | --------------------------------------- | --------------------------------------------------------------------------------- |
| `/dkucc/home/<NETID>` | Environments, `.bashrc`, small scripts, personal configuration, code, and outputs such as small logs | Network drive, 1 TB limit; other users' home directories cannot be accessed |
| `/work/<NETID>` | Datasets, checkpoints, outputs | Network drive mounted using NFSv4, a shared high-speed volume totaling 70 TB;<br>files older than 75 days are automatically cleaned up, with the script running every half month (see Section 7 for how to refresh data timestamps) |

DKUCC is intended as a single-user server. By default, you can use `ls` to view other people's directories (755), but cannot read, write, or execute file contents (600).
(In addition, although the permissions look like the typical Linux POSIX permission model, such as rwxr-xr-x shown by ls -l, what actually takes effect is probably the NFS4 ACL system, with POSIX presumably synthesized from the ACL.
In short, to be rigorous, directly using `chmod` to grant permissions is not recommended, as it is easy to break them.
For specific ways to edit NFS4 ACL permissions, or if you really need a shared directory for collaboration and need to assign directory permissions properly, see `nfs4-acl` / `dkucc-permission-audit` in [DKUCC-skill](https://github.com/guyong1449/DKUCC-skill) at the end of this guide.)

DKUCC has no formal backups, but if files disappear, data files are deleted by the scheduled 75-day script, or you delete them by mistake, there is still a way they might be recovered in the short term. You can check `bash_history` to see whether any relevant `rm` commands were run. The NFS storage for both `/work` and `/home` has short-cycle snapshots. You can check `ls -la /work/$(whoami)/.snapshot/`, compare snapshots from before and after the data was deleted, and use `rsync` to copy it back from a snapshot; see [dkucc-data-recovery](https://github.com/guyong1449/DKUCC-skill) for the specific steps.

Also, the current node is named `dkucc-login-01`, which is the login node. It only has CPUs. If you need to run code requiring heavy calculation, request a compute node. **Running tasks that need GPUs on the CPU node is prohibited; likewise, for tasks that need large amounts of CPU, please also request one.**

---

<a id="section-4"></a>

## 「Section 4」SSH Public Keys, LDAP, and Passwordless Login

If you use a server regularly, entering a password every time is really inconvenient. Normally, you can simply copy your public key into `~/authorized_keys` to log in without a password, but this does not work on DKUCC.

### Generate a Key Pair on Your Local Computer

This works on both Windows (PowerShell) and macOS (Terminal):

```bash
ssh-keygen -t ed25519 -C "<NETID>@duke.edu"
```

Press Enter several times to generate `~/.ssh/id_ed25519` by default; setting a passphrase for the private key is optional. After generation, confirm the public key fingerprint:

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
cat ~/.ssh/id_ed25519.pub
```

Add the following under `Host dkucc` in `~/.ssh/config`:

```sshconfig
    IdentityFile ~/.ssh/id_ed25519
```

Why does simply using `~/.ssh/authorized_keys` not enable passwordless login on DKUCC?
This is because, on the login node `dkucc-login-01.rc.duke.edu`, the reading order for public key authentication is as follows:

Enter this command:
```bash
grep -E 'AuthorizedKeys|PubkeyAuthentication' /etc/ssh/sshd_config
```
The output is as follows:
```text
PubkeyAuthentication yes
AuthorizedKeysCommand /usr/local/bin/ldap-authkey.sh
AuthorizedKeysCommandUser nobody
AuthorizedKeysFile /var/lib/ssh/%u/authorized_keys /%u/.ssh/authorized_keys
```

See the diagram below. The sshd system needs to read the public key content we copied over to enable passwordless login, but the locations it reads are `/var/lib/ssh/<NETID>/authorized_keys` and `/<NETID>/.ssh/authorized_keys` (see the right-hand path in the diagram below).

The problem is that the user's actual home directory is `/dkucc/home/<NETID>/`, so **`/dkucc/home/<NETID>/.ssh/authorized_keys` is not in sshd's AuthorizedKeysFile list**.

Public keys written to that file will not be read by the right-hand path (2). To enable public key login on DKUCC, the public key must be recognized by the left-hand path (1), meaning it must be registered in LDAP's `sshPublicKey`.

```text
                     Local SSH client
                    (Offering public key)
                             │
                             ▼
                dkucc-login-01.rc.duke.edu
                           sshd
                             │
                             ▼
                   PubkeyAuthentication
                             │
             ┌───────────────┴───────────────────┐
             │                                   │
             ▼                                   ▼
   ① AuthorizedKeysCommand            ② AuthorizedKeysFile
   /usr/local/bin/ldap-authkey.sh        (In listed path order)
   Run as user nobody                            │
             │                                   │
             ▼                                   ▼
   getent passwd <NETID>               /var/lib/ssh/<NETID>/authorized_keys
   Confirm local account exists                  │
             │                                   ▼
             ▼                        /<NETID>/.ssh/authorized_keys
   ldapsearch (uid=<NETID>)                       │
   idms-authdir.oit.duke.edu                      │
   Read LDAP attribute sshPublicKey              │
             │                                   │
             ▼                                   │
   Validate each public key with ssh-keygen -l    │
             │                                   │
             └───────────────┬───────────────────┘
                             ▼
              Merge public key sets from ① and ②
                             │
                             ▼
       Compare with the public key offered by this client
                             │
                ┌────────────┴─────────────┐
                ▼                          ▼
              Match                     No match
                │                          │
                ▼                          ▼
       Passwordless login      Continue with password or
                               other authentication methods
```

How do you set up the left-hand path (1)?
Go to [DKU Self-Service](https://idms-web-selfservice.oit.duke.edu/advanced), click manage your ssh keys, and paste in the public key content you copied.

![](DKUCC%20使用指南.assets/image-709.png)

Test `ssh dkucc`. If you can log in without a password, it has worked.

---

<a id="section-5"></a>

## 「Section 5」Roles of Login Nodes, Compute Nodes, and Slurm

Login nodes are suitable for editing code, Git operations, lightweight tests, and job management commands such as `sbatch`, `squeue`, `scancel`, and `sacct`. Please do not run epoch-level training, large-scale `torchrun`, lengthy compression, or model services here.

Compute nodes are allocated through `sbatch` or `srun` and are used to run training, use GPUs, read large amounts of data, and write checkpoints. ACLs are an exception: as long as `/work` is mounted and the current user is the directory owner or has `C` permission, ACLs can be modified on a compute node; however, new Slurm jobs cannot be submitted or managed on compute nodes.

For long interactive work, you can use `tmux` to keep the shell from disappearing after a network interruption:

```bash
tmux new -s main
# After logging in again
tmux attach -t main
```

![](DKUCC%20使用指南.assets/image-707.png)

The official website discourages using local VS Code Remote-SSH to run code directly on DKUCC.
Use the proper entry point: **code-server** on Open OnDemand (the cluster has module `code-server/4.96.4` installed at `/opt/apps/almalinux9/code-server-4.96.4/`). Starting your own `vscode-server` on a node for SSH connections is not recommended.

### GPU Permissions for Learning and Research Accounts

The table below is based on the October 16, 2026 attachment. See Section 1 for the December 1, 2026 conversion and billing arrangements; check your actual configuration both before and after the transition.

| Item | Learning Account | Research Account |
| --- | --- | --- |
| Eligible Users | DKU students, faculty, and staff; can be used for learning and basic research | Project users who need additional computing power and have funding and the required approvals, including eligible SRS / SWRG students |
| Concurrent Running Jobs | At most 1 | At most 1 |
| GPUs per Job | At most 1 | At most 4 |
| GPU Runtime per Calendar Month | At most 1 week | At most 4 weeks |
| Available GPU Partitions | Common-GPU, L20-GPU, temporary 4090 (if available) | L20-GPU, H20-GPU, temporary 4090 (if available) |
| Scheduling Requirements | Must be submitted through Slurm; may queue | Must be submitted through Slurm; may queue |
| Funding and Billing | Covered by DKU within the limits; no direct charges to individuals | Billed according to actual project usage, or given a total resource allowance based on the SWRG grant amount |

**Queuing and priority:** Common-GPU is exclusively for learning accounts, H20-GPU is exclusively for research accounts, and L20-GPU and temporary 4090 are available to both account types. All partitions require queuing; the attachment explicitly states that **no scheduling priority will be given based on account type or project**. Temporary 4090 resources may be released from the teaching environment during the summer and are not guaranteed to be available year-round.

### Fees and Account Validity

| Research Resource | Rate (RMB / GPU-hour) |
| --- | ---: |
| L20-GPU | ¥0.94 |
| H20-GPU | ¥3.95 |
| Temporary 4090 | ¥0.47 |

Learning accounts do not charge individuals directly within the specified limits. The research rates above are charged by GPU-hour.

- **Monthly project billing:** IT summarizes the previous month's usage and sends a bill to the project lead; the lead should raise any questions within 5 working days of receiving it. If funding is insufficient, another billing account should be provided within 5 working days; otherwise, IT may terminate the associated running or queued jobs and suspend the research account until the outstanding balance is paid.
- **SWRG total allowance control:** IT sets a total usage allowance based on the grant amount; the account remains valid until the grant expires or the allowance is exhausted, whichever comes first. No bills are sent to students.
- Project leads should specify the research account end date in the application. Any updates to account permissions, resource allocation, and rates are subject to subsequent IT notices.

### Check Your Slurm Associations

```bash
sacctmgr show assoc user=$USER format=Account,Partition,QOS,MaxJobs,MaxTRES -P
```

“At most 1 job running at a time” does not mean “at most 1 job submitted”, nor does it mean unlimited CPU resource usage. Submission limits, time limits, and available resources for CPU partitions should be checked separately.

### Partitions and GPU Models

Use `sinfo` to check specific partitions and nodes:

```bash
sinfo
```

Example output:

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


| Slurm Partition Example | `--gres` Example | Memory per GPU Listed in the Attachment | Allowed Account Types | Common Uses |
| --- | --- | --- | --- | --- |
| `common-gpu` | `gpu:a40:N` | A40 48 GB | Learning only | Interactive 3D / engine development such as [Unity](https://unity.com/), remote visualization, general inference, and small- to medium-scale training |
| `l20-gpu` | `gpu:l20:N` | L20 48 GB | Learning / Research | Computer vision (detection, segmentation, video understanding) training and inference |
| `h20-gpu` | `gpu:h20:N` | H20 141 GB | Research only | Large language models and long-context inference, fine-tuning, and multi-GPU parallel training |

`N` is the number of GPUs requested: at most 1 for learning accounts and at most 4 for research accounts. Regardless of account type, only one requested task can run on GPU nodes.

---

<a id="section-6"></a>

## 「Section 6」GPU Jobs: Submission, Queuing, Preemption, and Recovery

### Common Slurm Commands

#### 1. View an Overview of All Partitions and Nodes: `sinfo`

```bash
sinfo
```

**Meaning:** Lists the availability, job time limit, node count, node states (`idle` free / `mix` partly occupied / `alloc` fully occupied / `down*` faulty), and node names for each Slurm partition. A partition marked with `*` (such as `common*`) is the default partition.

**Example output:**

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

For a more compact summary, use `sinfo -s` (the `NODES(A/I/O/T)` column means allocated / idle / other / total).

#### 2. View Hardware and Usage for a Single Node: `scontrol show node`

```bash
scontrol show node dkucc-core-gpu-dkurc-d-it09-3
```

**Meaning:** Shows the detailed configuration and current resource usage of the specified compute node.
**Common fields:** `Gres` (GPU model and count), `RealMemory` (physical memory in MB), `State` (`IDLE` / `MIXED` / `ALLOCATED`, etc.), `AllocTRES` (allocated CPU / memory / GPU).

**Example output (`common-gpu` node, excerpt):**

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

The fields have the same meanings for nodes in other partitions such as L20; `State=MIXED` with `AllocTRES gres/gpu=N` means some GPUs on the node are occupied.

#### 3. Request an Interactive Shell on a Specific Node: `srun` + `-w`

**The four-GPU example below is only for accounts that have already been granted the corresponding research permissions**; learning accounts must not copy the four-GPU request. Check `sinfo` before submitting, and note that the job may queue.

```bash
srun -p l20-gpu -w dkucc-core-gpu-dkurc-a-it09-26 \
  --gres=gpu:l20:4 --cpus-per-task=48 --mem=256G --pty bash -i
```

| Parameter | Meaning |
|---|---|
| `-p l20-gpu` | Use the `l20-gpu` partition |
| `-w <node name>` | Specify which compute node to run on (`-w` / `--nodelist`) |
| `--gres=gpu:l20:4` | Request 4 L20 GPUs; only for research accounts within the limits |
| `--cpus-per-task=48` | Request 48 CPU cores |
| `--mem=256G` | Request 256 GB of memory |
| `--pty bash -i` | Enter an interactive bash shell after successful allocation |

Usually, you do not need to specify `-w`; let the scheduler choose a node automatically. Only consider `-w` when you know that a particular machine's memory, GPU model, or driver meets your requirements and that the node still has free resources.

#### 4. View Your Job Queue: `squeue -u`

```bash
squeue -u $USER
# Or
squeue -u <NETID>
```

**Meaning:** Lists the specified user's jobs that are currently queued or running. The `ST` column: `R` running, `PD` pending, `CG` completing, etc. `TIME` is the elapsed runtime; `NODELIST` is the node where the job is actually running.

**Example output:**

```text
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             30136   l20-gpu     bash   user_a  R 2-16:14:52      1 dkucc-core-gpu-dkurc-a-it08-22
             30429   l20-gpu pomo_cvrp20   user_b  R    7:31:11      1 dkucc-core-gpu-dkurc-a-it09-26
```

#### 5. Reconnect to the Shell of an Existing Interactive Job: `srun --jobid`

```bash
srun --jobid=<JOBID> --pty bash
```

**Meaning:** After a network interruption or closing the terminal, attach again to an interactive Slurm job that is **still running**. Each job **allows only one** attached shell; if there is already a connection, first exit the old shell or open a new terminal before reconnecting.

#### 6. Cancel a Job: `scancel`

```bash
scancel <JOBID>
```

**Meaning:** Cancels the specified job (pending or running). Canceling your own job does not require additional permissions; accidentally deleting someone else's job requires administrator intervention.

#### 7. View Details of a Single Job: `squeue -j` + Custom Format

```bash
squeue -j <JOBID> -o "%.18i %.9P %.20j %.8u %.2t %.10M %.9l %.6D %.20R"
```

**Meaning:** Outputs the partition, job name, user, state, elapsed runtime, time limit, node count, and scheduling reason (`REASON` / `NODELIST`) for a single `JOBID` in fixed-width columns.

**Example output:**

```text
             JOBID PARTITION                 NAME     USER ST       TIME TIME_LIMI  NODES     NODELIST(REASON)
             30136   l20-gpu                 bash   user_a  R 2-16:14:52 4-10:00:00      1 dkucc-core-gpu-dkurc-a-it08-22
```

After a job ends, query its history with `sacct -j <JOBID>`.

### Batch Script Example

Below is an example Slurm batch script. You need to create the log directory before submission. Slurm opens the files specified by `--output` and `--error` before the script starts, so you cannot rely on `mkdir` inside the script:

```bash
mkdir -p /work/<NETID>/slurm/logs
```

Then create `/work/<NETID>/slurm/train.sh`:

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

Submit and monitor:

```bash
sbatch /work/<NETID>/slurm/train.sh
squeue -u $USER
tail -f /work/<NETID>/slurm/logs/my-train-<JOBID>.out
sacct -j <JOBID> --format=JobID,State,ExitCode,Elapsed,Timelimit,MaxRSS,DerivedExitCode
```

### Short Debugging Sessions and Long Training Runs

For short debugging sessions, you can request an interactive shell:

```bash
srun --partition=l20-gpu --gres=gpu:l20:1 --cpus-per-task=4 --mem=32G --time=01:00:00 --pty bash -l
```

Interactive shells are only for debugging. Use `sbatch` for long training runs. If a single script handles both job submission and training, you can branch on `SLURM_JOB_ID`:

```bash
if [[ -z "${SLURM_JOB_ID:-}" ]]; then
  sbatch --export=ALL,MY_VAR=value "$0" "$@"
else
  python main.py
fi
```


<a id="section-7"></a>

## 「Section 7」`/work` Data Retention and Regular Checks

Files under `/work` with timestamps older than 75 days are cleaned up every half month. When you need to update file timestamps, limit the scope to directories you actually need to retain. Scanning all of `/work/<NETID>` generates a large amount of I/O and should not be run directly on a login node. The following command requests CPU resources through Slurm and processes only `data` and `outputs`:

```bash
srun --partition=common --time=04:00:00 --mem=2G --cpus-per-task=1 \
  bash -lc 'find /work/<NETID>/data /work/<NETID>/outputs -type f -print0 | xargs -0 -r touch'
```

To run this automatically, you can have cron submit a CPU job on the 1st of each month.
First create the log directory and `/work/<NETID>/slurm/touch-work.sh`:

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

After confirming that the script runs correctly through `sbatch`, run `crontab -e` and add:

```cron
0 2 1 * * /usr/bin/sbatch /work/<NETID>/slurm/touch-work.sh >> /work/<NETID>/slurm/touch-submit.log 2>&1
```

This expression means rescan “at 02:00 on the 1st of each month”. Cron only submits the job on the login node; the actual scan is performed by a CPU node allocated by Slurm.

Check the age of the oldest file:

```bash
tail -n 50 /work/<NETID>/slurm/touch-submit.log
srun --partition=common --time=01:00:00 --mem=2G --cpus-per-task=1 bash -lc '
  OLDEST=$(find /work/<NETID>/data /work/<NETID>/outputs -type f -printf "%T@\n" 2>/dev/null | sort -n | head -1)
  if [[ -n "$OLDEST" ]]; then
    DAYS=$(( ($(date +%s) - ${OLDEST%.*}) / 86400 ))
    echo "The oldest file was modified or touched about ${DAYS} days ago"
  fi
'
```

For the complete file timestamp maintenance skill (`~/bin/touch_work_timestamps.sh`, cron runs on the 1st/15th of each month, and `monthly_check.sh` for space in specific folders), refer to [dkucc-data-ingest](https://github.com/guyong1449/DKUCC-skill); if files have already been cleaned up or accidentally deleted, see [dkucc-data-recovery](https://github.com/guyong1449/DKUCC-skill) for snapshot recovery steps.

---

<a id="official-resources"></a>

## Official Resources


- DKU IT email dated October 23, 2026: confirms the December 1, 2026 account conversion, research usage tracking and billing, and first-bill arrangements.
- Email attachment “DKU Research GPU management”, dated October 16, 2026: account types, resource limits, partitions, application process, rates, and validity periods.
- [Research account application form](https://dukekunshan.formstack.com/forms/dku_gpu_research_account_request_form).

Other references:

- DKUCC Official Website:
  [Home](https://dkucc.dukekunshan.edu.cn/)
- [Duke Research Computing Support Documentation](https://oit-rc.pages.oit.duke.edu/rcsupportdocs/)
- [VS Code Remote-SSH](https://code.visualstudio.com/docs/remote/ssh)
- [NVIDIA A40 Datasheet](https://images.nvidia.com/content/Solutions/data-center/a40/nvidia-a40-datasheet.pdf)
- [NVIDIA L20 Product Overview](https://www.nvidia.com/en-us/data-center/l20/)
- [NVIDIA H20 Product Overview](https://www.nvidia.com/en-us/data-center/h20/)
- [NVIDIA AI Enterprise GPU Support Table](https://docs.nvidia.com/ai-enterprise/release-7/latest/infra-software/vgpu/features.html)

<a id="tools"></a>

## Useful Tools

- [guyong1449/cisco-vpn-autoconnect](https://github.com/guyong1449/cisco-vpn-autoconnect): automatically invokes Cisco Secure Client on Windows to connect to DKU VPN, with support for Duo Push
- [guyong1449/DKUCC-skill](https://github.com/guyong1449/DKUCC-skill): a collection of skills that break down and organize workflows in this guide that are relatively complex, inconvenient to describe in full in the main text, or suitable for AI-assisted troubleshooting, along with cached DKUCC website pages, archived original IT-related emails, and other related content
