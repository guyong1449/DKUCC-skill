---
tags:
  - area/dkucc
  - type/reference
  - status/published
status: published
---

# DKUCC User Guide

[中文](DKUCC%20使用指南.md) · English · [Repository overview](README.en.md)

This guide covers DKUCC account applications, SSH access, file organization, Slurm jobs, and data retention. The account and GPU access information has been updated using the DKU IT notice dated October 23, 2026, and its attachment, *DKU Research GPU management* (October 16, 2026 edition). **The account transition and billing for research usage will take effect on December 1, 2026.** See Sections 1 and 5 for details.

**Last checked: October 31, 2026.** Command output, node status, and software versions in this guide include historical examples, not a live view of the cluster. Follow the latest official notices and the configuration actually assigned to your account.

## Getting help

For networking, the DKU VPN, or other DKU account issues, email [service-desk@dukekunshan.edu.cn](mailto:service-desk@dukekunshan.edu.cn) to open a support ticket. See Section 1 for Learning and Research account applications.

## Contents

- [Section 0 Quick reference](#section-0)
- [Section 1 About DKUCC and official resources](#section-1)
- [Section 2 Setting up SSH on Windows and macOS](#section-2)
- [Section 3 Your first login and file organization](#section-3)
- [Section 4 SSH keys, LDAP, and passwordless login](#section-4)
- [Section 5 Login nodes, compute nodes, and Slurm](#section-5)
- [Section 6 GPU jobs: submission, queues, preemption, and recovery](#section-6)
- [Section 7 Data retention and regular checks in `/work`](#section-7)
- [Official resources](#official-resources)
- [Related tools](#tools)

<a id="section-0"></a>

## Section 0 Quick reference

1. SSH first takes you to a login node. Do not run GPU jobs or CPU-intensive computation directly on `dkucc-login-01`.
2. Submit substantial computation to compute nodes through Slurm. See Sections 5 and 6 for instructions.
3. Your starting directory is usually `/dkucc/home/<NETID>`. Use it for small files such as code and configuration. Store large datasets, checkpoints, and outputs in `/work/<NETID>`.
4. The original guide recorded a `/work` cleanup policy of deleting files with timestamps older than 75 days, twice a month. Confirm the current policy with IT and back up important data elsewhere. Use the timestamp scripts in Section 7 only with administrator approval.
5. Do not bypass Slurm by SSHing directly to compute nodes to occupy GPUs or run training. GPU processes must belong to an approved job.
6. Start with a regular job if you are new to Slurm. The original guide noted difficulties matching array job IDs to individual tasks; use `sacct`, `SLURM_JOB_ID`, and related information when troubleshooting job arrays. This is not an account-policy ban on array jobs.
7. Release the allocated resources when a job ends. Do not rely on leftover processes to keep running; the original guide observed that remaining processes were cleaned up after a job reached `COMPLETED`.
8. Learning and Research accounts have different GPU counts, monthly runtime limits, and eligible partitions. **Both account types allow at most one running job at a time.** This does not mean that you can submit only one job; see Section 5.

---

<a id="section-1"></a>

## Section 1 About DKUCC and official resources

Official website: https://dkucc.dukekunshan.edu.cn/

| Page | What it covers |
| --- | --- |
| Logging into the DKUCC | SSH, VPN, appropriate use of login nodes, and Open OnDemand |
| Working with files | Home and `/work`, capacity, file retention, and transfers |
| Scheduling Jobs | Submitting and managing Slurm jobs |
| How to Use GPU Resource | GPU partitions and interactive sessions |
| Modules / User installed software | Environment modules and installing your own software |
| OnDemand / OnDemand Overview | Requesting nodes and using interactive apps through OnDemand, the recommended interface |
| Training | Introductory DCC training and self-study resources |

DKUCC is a computing cluster made up of login nodes, compute nodes, and shared storage. To access it from your own computer, use SSH (Secure Shell Protocol), which establishes an encrypted connection to a remote machine. You will need an account to access the login node.

### Applying for an account and the policy transition

**Learning accounts** are available to DKU students, faculty, and staff, and may also be used for basic research. Email [it-infrastructure@dukekunshan.edu.cn](mailto:it-infrastructure@dukekunshan.edu.cn) to open a service ticket. No funding transfer or additional approval is required, and requests are usually processed within one business day. You do not need funding from an advisor before applying for a Learning account.

**Research accounts** are for GPU use beyond the Learning account limits and require available funding and the relevant approvals:

- For projects billed monthly, a project owner or lead with available funding should complete the [Research account request form](https://dukekunshan.formstack.com/forms/dku_gpu_research_account_request_form), including an account end date. Multiple accounts for the same project can be requested on one form.
- Students who need additional computing resources for Signature Work (SW) or independent study supporting their SW may apply for Signature Work Research Grants (SWRG). The faculty or staff member overseeing the grant completes the form and specifies each student's approved amount and grant expiration date.
- Faculty fund additional computing resources for SRS projects under the existing SRS agreement. IT provisions Research access once the required approvals are in place.

The October 23, 2026 notice sets out the following transition:

1. On **December 1, 2026**, existing accounts will become Learning accounts unless they have already been configured as Research accounts.
2. On or after that date, jobs exceeding the Learning account limits will be canceled, except for jobs already associated with established Research accounts and compliant with the Research account limits.
3. Research usage tracking and billing begin on that date. The first invoices will be sent to project leads **after the 2027 New Year holiday**.

Apply in advance if you need Research access. Membership in an old `faculty` group or a previous job configuration does not establish that you have the new Research permissions. Section 5 covers resource limits, partitions, and charges for both account types.

**Account type is separate from file permissions.** Learning and Research describe GPU resource access and billing. SSH authentication, Slurm `Account` / QoS settings, and directory POSIX permissions / NFSv4 ACLs are separate layers. A Research account does not give you access to other users' directories or permission to bypass Slurm.

### Logging in

Once your account is ready, log in with your own NetID. This guide uses the hostname `dkucc-login-01.rc.duke.edu`. Unless you have set up key-based login as described in Section 4, you will need password authentication. The original guide recorded that this was the same password as the DKU email account; follow IT's instructions if authentication has changed.

```bash
ssh <NETID>@dkucc-login-01.rc.duke.edu
#      ↑              ↑
#   username        hostname
```

The screenshot below shows a successful SSH login to DKUCC. Its command uses a short alias rather than the full hostname because `.ssh/config` has been configured. Section 2 explains how to do this.

![Historical example of a successful SSH login](DKUCC%20使用指南.assets/image-706.png)

### DCC training PDF

The official [Training](https://dkucc.dukekunshan.edu.cn/) page describes DKUCC as a replica of DCC (Duke Compute Cluster), so DKU users can also learn from the introductory DCC training materials. A PDF of the slides is available on Duke Box:

[Current PDF version of the slides (DCC introductory training)](https://duke.app.box.com/s/zosu3w34iap7o4yuvlq44zep8ic27byq)

The PDF is a useful introduction to Slurm concepts, job submission, and common commands. Its hostnames, resource configurations, and account rules may differ from DKUCC or have changed since publication. Use DKUCC's current instructions for connection details and access rules.

---

<a id="section-2"></a>

## Section 2 Setting up SSH on Windows and macOS

This section covers installing an SSH client and creating a connection alias, using password authentication to start. To set up key-based login later, see Section 4.

### Windows: check for OpenSSH Client

Open PowerShell and run:

```powershell
ssh -V
```

If you see a version number, SSH is installed and you can move on. If `ssh` is not found, install OpenSSH Client:

- Windows 11: open **Settings → System → Optional features → View features** and search for `OpenSSH Client`.
- Windows 10: open **Settings → Apps → Optional features → Add a feature** and search for it.

Alternatively, open PowerShell as an administrator and run:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

After installation, close and reopen PowerShell, then run `ssh -V` again. A version number confirms that installation succeeded. The Windows component is sufficient; you do not need an unfamiliar third-party SSH client just to connect to the cluster.

Next, create your SSH configuration file:

```powershell
New-Item -ItemType Directory -Force "$HOME\.ssh"
notepad "$HOME\.ssh\config"
```

Paste the following, replacing `<NETID>` with your Duke NetID. You will then be able to type `ssh dkucc` instead of the full hostname. `dkucc` is just a local alias; choose another name if you find it easier to remember or type.

```sshconfig
Host dkucc
    HostName dkucc-login-01.rc.duke.edu
    User <NETID>
```

### macOS: use the built-in OpenSSH client

macOS includes OpenSSH. Open Terminal and run:

```bash
ssh -V
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/config
```

Add the following configuration, replacing `<NETID>` with your Duke NetID. You will then be able to type `ssh dkucc` instead of the full hostname. As on Windows, `dkucc` is a local alias that you can change to a name you prefer:

```sshconfig
Host dkucc
    HostName dkucc-login-01.rc.duke.edu
    User <NETID>
```

In `nano`, press `Ctrl+O` to save, Enter to confirm, and `Ctrl+X` to exit. Then restrict access to the configuration file:

```bash
chmod 600 ~/.ssh/config
```

---

<a id="section-3"></a>

## Section 3 Your first login and file organization

### Connect to the DKU VPN first

Before connecting to the DKUCC login node from off campus, **connect to the DKU VPN**. Without the VPN, your computer generally cannot reach port 22 on `dkucc-login-01.rc.duke.edu`. You may see errors such as `ssh: Could not resolve hostname dkucc-login-01.rc.duke.edu:` or `Connection timed out`.

For VPN setup, see **[墙记｜入学指南之线上准备2！ (Chinese-language onboarding guide)](https://mp.weixin.qq.com/s?__biz=MzkxMjczOTA4NQ==&mid=2247486055&idx=1&sn=3b54062b550949ad329987943817fea0&chksm=c1091405f67e9d136dde1c38d40b50e0791a13518b3dbc1d9bf741e0aabf2b02a1fb8bd2ec4a&cur_album_id=3994731232327254017&scene=190#rd)**. The [cisco-vpn-autoconnect](https://github.com/guyong1449/cisco-vpn-autoconnect) helper script linked at the end of this guide may also be useful.

Once the VPN is connected, run `ssh dkucc` using the alias from Section 2, or `ssh <NETID>@dkucc-login-01.rc.duke.edu`, on your own computer.

### Connect and check your environment

The DKUCC login hostname is `dkucc-login-01.rc.duke.edu`. After your first successful login, run:

```bash
whoami
pwd
df -h /dkucc/home/$USER /work/$USER # Check filesystem space usage
sinfo -s # List cluster partitions
```

Example output:

![Historical output from the first-login checks](DKUCC%20使用指南.assets/image-708.png)

You will usually start in `/dkucc/home/<NETID>`, your Linux home directory (`~`). Avoid writing checkpoints or keeping large datasets here, as this can affect login and shared storage. Put large files on the `/work` shared filesystem instead.

The original guide recorded observations of roughly 1 TB for home storage and 70 TB for the shared `/work` volume. These figures are not current individual quotas. The **400 TB Dell EMC Isilon filesystem** listed in the October 16, 2026 attachment is the overall infrastructure capacity, not a personal `/work` allocation. `df -h` reports filesystem space; ask IT about your individual quota and usable capacity.

| Location | What to store | Notes |
| --- | --- | --- |
| `/dkucc/home/<NETID>` | Environments, `.bashrc`, small scripts, personal configuration, code, and small logs | Network storage; actual quotas depend on IT configuration. Do not assume you can access other users' home directories. |
| `/work/<NETID>` | Datasets, checkpoints, and outputs | Recorded in the original guide as NFSv4 shared storage. See Section 7 for retention rules and timestamp maintenance permitted by administrators. |

Use your own account on DKUCC. The original guide observed `755` permissions on some directories and `600` on some files, but these were specific historical examples, not a universal default for users' directories. Whether you can list, read, or modify files depends on the actual permissions and ACLs of the target path.

The original guide also recorded the use of NFSv4 ACLs: the POSIX permissions shown by `ls -l` may not fully represent an ACL. Avoid changing shared-directory permissions with `chmod` without understanding the existing ACL. For collaboration, first confirm who is authorized to access the data, then consult `nfs4-acl` / `dkucc-permission-audit` in [DKUCC-skill](https://github.com/guyong1449/DKUCC-skill). These file permissions are unrelated to the Learning / Research account classification.

Do not treat cluster storage as a guaranteed backup. The original guide recorded short-retention snapshots for `/work` and home storage, but ask IT whether snapshots are currently available, how long they are retained, and whether recovery is possible. If a file disappears, you can check `bash_history` for relevant `rm` commands and try `ls -la /work/$(whoami)/.snapshot/`. If an accessible snapshot from before the deletion exists, restore from it with `rsync` within your permissions; see [dkucc-data-recovery](https://github.com/guyong1449/DKUCC-skill) for the steps. Snapshots do not replace off-cluster backups.

Remember that `dkucc-login-01` is a login node, not a place to run compute jobs. Request compute resources through Slurm for GPU jobs and CPU-intensive work rather than running them directly on the login node.

---

<a id="section-4"></a>

## Section 4 SSH keys, LDAP, and passwordless login

For regular use, public-key authentication saves you from repeatedly entering your account password. Many servers use `~/.ssh/authorized_keys`, but the original DKUCC guide found that adding a public key to that file in the user's home directory did not work. It therefore used the Self-Service registration method below. The configuration and explanation here come from those historical tests; check with IT if current login behavior differs.

### Generate a key pair on your computer

Run this in PowerShell on Windows or Terminal on macOS:

```bash
ssh-keygen -t ed25519 -C "<NETID>@duke.edu"
```

Follow the prompts to choose a save location; the default is `~/.ssh/id_ed25519`. You can optionally protect the private key with a passphrase. If a key already exists at that path, do not overwrite it: use the existing key or choose a different filename. Then check the public-key fingerprint and display the public key:

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
cat ~/.ssh/id_ed25519.pub
```

Add this line under `Host dkucc` in `~/.ssh/config`:

```sshconfig
    IdentityFile ~/.ssh/id_ed25519
```

### Public-key authentication recorded in the original guide

The original guide observed the following configuration on `dkucc-login-01.rc.duke.edu`. It explains why `~/.ssh/authorized_keys` in the user's home directory was not read at the time.

Command:

```bash
grep -E 'AuthorizedKeys|PubkeyAuthentication' /etc/ssh/sshd_config
```

Historical output:

```text
PubkeyAuthentication yes
AuthorizedKeysCommand /usr/local/bin/ldap-authkey.sh
AuthorizedKeysCommandUser nobody
AuthorizedKeysFile /var/lib/ssh/%u/authorized_keys /%u/.ssh/authorized_keys
```

To authenticate a key, `sshd` needs to find the matching public key. In the file-based branch on the right of the diagram below, it checks `/var/lib/ssh/<NETID>/authorized_keys` and `/<NETID>/.ssh/authorized_keys`.

The user's actual home directory, however, is `/dkucc/home/<NETID>/`. **`/dkucc/home/<NETID>/.ssh/authorized_keys` is not in the `sshd` `AuthorizedKeysFile` list.**

Keys saved there are not read by branch ②. For the DKUCC setup recorded here, register your key as an LDAP `sshPublicKey` so branch ① can find it.

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
           ┌───────────────┴────────────────────┐
           │                                    │
           ▼                                    ▼
   ① AuthorizedKeysCommand             ② AuthorizedKeysFile
   /usr/local/bin/ldap-authkey.sh        (paths in listed order)
   Runs as user nobody                          │
           │                                    │
           ▼                                    ▼
   getent passwd <NETID>                /var/lib/ssh/<NETID>/authorized_keys
   Check that the local account exists          │
           │                                    ▼
           ▼                            /<NETID>/.ssh/authorized_keys
   ldapsearch (uid=<NETID>)                      │
   idms-authdir.oit.duke.edu                     │
   Read the LDAP sshPublicKey attribute         │
           │                                    │
           ▼                                    │
   Validate each key with ssh-keygen -l          │
           │                                    │
           └───────────────┬────────────────────┘
                           ▼
               Combine keys from ① and ②
                           │
                           ▼
          Compare with the key offered by the client
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
            Match                    No match
              │                         │
              ▼                         ▼
        Key-based login       Continue with password or
                              other authentication methods
```

To register a key for branch ①, open [Duke Self-Service](https://idms-web-selfservice.oit.duke.edu/advanced), select **manage your ssh keys**, and paste the public key you copied.

![Example of SSH key management in Duke Self-Service](DKUCC%20使用指南.assets/image-709.png)

Test with `ssh dkucc`. If you can authenticate with your key, the setup is working.

---

<a id="section-5"></a>

## Section 5 Login nodes, compute nodes, and Slurm

Use the login node for editing code, Git operations, lightweight tests, and job-management commands such as `sbatch`, `squeue`, `scancel`, and `sacct`. Do not use it for full training epochs, large `torchrun` workloads, lengthy compression jobs, or model-serving processes.

Compute nodes are allocated through `sbatch` or `srun` for training, GPU use, large-data reads, and checkpoint writes. Changes to directory permissions still require authorization. The original guide recorded that ACLs could be changed on a compute node when `/work` was mounted and the user owned the directory or had the relevant `C` permission. Submit and manage jobs from the login node rather than relying on compute nodes to provide those functions.

For long interactive sessions, use `tmux` to keep your shell available if the network connection drops:

```bash
tmux new -s main
# After logging in again
tmux attach -t main
```

![Example of login-node and interactive-session commands](DKUCC%20使用指南.assets/image-707.png)

The original guide recommends **code-server** through Open OnDemand rather than starting your own `vscode-server` on a node for a local VS Code Remote-SSH connection. It recorded the module `code-server/4.96.4` at `/opt/apps/almalinux9/code-server-4.96.4/`; check the official pages and current environment for the available entry point and version. Whatever editor you use, do not run heavy computation directly on the login node.

### GPU access for Learning and Research accounts

The table below follows the October 16, 2026 attachment. See Section 1 for the December 1, 2026 transition and billing schedule. Check your actual account configuration both before and after the transition.

| Item | Learning account | Research account |
| --- | --- | --- |
| Intended users | DKU students, faculty, and staff; for learning and basic research | Project users who need additional computing resources and have funding and the required approvals, including eligible SRS / SWRG students |
| Concurrent running jobs | At most 1 | At most 1 |
| GPUs per job | At most 1 | At most 4 |
| Total GPU runtime per calendar month | At most 1 week | At most 4 weeks |
| Available GPU partitions | Common-GPU, L20-GPU, and temporary 4090 resources when available | L20-GPU, H20-GPU, and temporary 4090 resources when available |
| Scheduling | Must submit through Slurm; jobs may queue | Must submit through Slurm; jobs may queue |
| Funding and billing | DKU covers usage within the limits; users are not charged directly | Billed to the project based on actual usage, or capped by a total resource allowance based on the SWRG grant amount |

**How runtime is measured:** The attachment uses the term “Total GPU runtime” and sets limits of one week / four weeks per calendar month. It does not explain how runtime is aggregated for multi-GPU jobs. This guide therefore does not convert those limits into GPU-hours or interpret the monthly allowance as a maximum wall time for an individual job. Ask IT about the measurement and enforcement rules when planning a budget or a long-running workload. Resource limits may be adjusted based on usage and availability.

**Queues and priority:** Common-GPU is exclusively for Learning accounts, H20-GPU is exclusively for Research accounts, and L20-GPU and temporary 4090 resources can be used by both. Jobs may queue in every partition. The attachment explicitly states that **account type or project does not confer scheduling priority**. Temporary 4090 resources may be released from teaching environments during the summer; they are not guaranteed year-round, and the attachment does not specify a fixed Slurm partition name for them.

### Charges and account validity

| Resource used for research | Rate (RMB per GPU-hour) |
| --- | ---: |
| L20-GPU | ¥0.94 |
| H20-GPU | ¥3.95 |
| Temporary 4090 | ¥0.47 |

Learning account users are not charged directly for usage within their limits. The research rates above are billed per GPU-hour. This is separate from the monthly runtime allowance in the preceding table, whose detailed measurement rules still need clarification from IT.

- **Monthly project billing:** IT compiles the previous month's usage and sends an invoice to the project lead. Questions should be raised within five business days of receipt. If funding is insufficient, an alternative billing account must be provided within five business days; otherwise IT may terminate the affected running or queued jobs and suspend the Research account until the outstanding charges are paid.
- **SWRG spending cap:** IT sets a total usage allowance based on the grant amount. The account remains valid until the grant expires or the allowance is exhausted, whichever comes first. Students are not sent invoices.
- Project leads should include a Research account end date in the application. Later IT notices take precedence if permissions, resource allocations, or rates change.

### Check your Slurm associations

```bash
sacctmgr show assoc user=$USER format=Account,Partition,QOS,MaxJobs,MaxTRES -P
```

Read your own current output rather than copying an old `faculty` example. A Slurm `Account` name is not the same as the Learning / Research policy classification. A name, a single `MaxTRES` value, or historical output cannot establish all your permissions, monthly allowance, or billing status. If you have multiple associations or are unsure about the transition, ask IT which Slurm account and QoS to use. Set job options such as `--account` only after confirming them.

“At most one running job at a time” does not mean “only one submitted job,” nor does it imply unlimited CPU use. Check CPU-partition submission limits, time limits, and available resources separately.

### Partitions and GPU models

Use `sinfo` to inspect partitions and nodes:

```bash
sinfo
```

The following is historical sample output, not current node status:

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

The attachment lists the following three permanent GPU resources; eligibility depends on account type. The lowercase partition names below come from the original guide's Slurm examples. Confirm the actual names with `sinfo`.

| Example Slurm partition | Example `--gres` value | Memory per GPU listed in the attachment | Eligible accounts | Common uses |
| --- | --- | --- | --- | --- |
| `common-gpu` | `gpu:a40:N` | A40 48 GB | Learning only | Interactive 3D / engine development such as [Unity](https://unity.com/), remote visualization, general inference, and small-to-medium training workloads |
| `l20-gpu` | `gpu:l20:N` | L20 48 GB | Learning / Research | Computer-vision training and inference, including detection, segmentation, and video understanding |
| `h20-gpu` | `gpu:h20:N` | H20 141 GB | Research only | Large language models, long-context inference, fine-tuning, and multi-GPU parallel training |

`N` is the number of GPUs requested: at most 1 for Learning accounts and 4 for Research accounts, subject to partition restrictions. The 141 GB H20 configuration is explicitly listed in this attachment and replaces the old draft's 96 GB figure. The original guide also noted FP8 inference as a use for L20; suitability depends on support in your software stack and model.

---

<a id="section-6"></a>

## Section 6 GPU jobs: submission, queues, preemption, and recovery

### Common Slurm commands

#### 1. View partitions and nodes: `sinfo`

```bash
sinfo
```

**What it does:** Lists each Slurm partition's availability, job time limit, node count, node state (`idle` for idle, `mix` for partially allocated, `alloc` for fully allocated, and `down*` for down), and node names. A partition marked with `*`, such as `common*`, is the default partition.

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

For a more compact summary, use `sinfo -s`. Its `NODES(A/I/O/T)` column means allocated / idle / other / total.

#### 2. Inspect one node's hardware and allocations: `scontrol show node`

```bash
scontrol show node dkucc-core-gpu-dkurc-d-it09-3
```

**What it does:** Shows the detailed configuration and current resource allocations of a compute node.

**Useful fields:** `Gres` (GPU model and count), `RealMemory` (physical memory in MB), `State` (such as `IDLE`, `MIXED`, or `ALLOCATED`), and `AllocTRES` (allocated CPU, memory, and GPU resources).

**Example output (excerpt from a `common-gpu` node):**

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

The fields have the same meaning for nodes in L20 and other partitions. `State=MIXED` with `AllocTRES gres/gpu=N` indicates that some GPUs on the node are already allocated.

#### 3. Request an interactive shell on a specific node: `srun` + `-w`

**This four-GPU example is only for accounts with the required Research access.** Do not copy the four-GPU request with a Learning account. The node name comes from a historical record; check `sinfo` before submitting, and expect that the job may queue.

```bash
srun -p l20-gpu -w dkucc-core-gpu-dkurc-a-it09-26 \
  --gres=gpu:l20:4 --cpus-per-task=48 --mem=256G --pty bash -i
```

| Option | Meaning |
| --- | --- |
| `-p l20-gpu` | Use the `l20-gpu` partition |
| `-w <node-name>` | Choose the compute node (`-w` / `--nodelist`) |
| `--gres=gpu:l20:4` | Request 4 L20 GPUs; only for Research accounts within their limits |
| `--cpus-per-task=48` | Request 48 CPU cores |
| `--mem=256G` | Request 256 GB of memory |
| `--pty bash -i` | Open an interactive bash shell once resources are allocated |

Usually, leave out `-w` and let the scheduler choose a node. Consider pinning a node only when you know its memory, GPU model, or driver meets a particular requirement and it still has free resources. Learning account users can start with the single-GPU debugging or batch examples later in this section. CPU, memory, and time requests must also fit your actual limits.

#### 4. View your job queue: `squeue -u`

```bash
squeue -u $USER
# Or
squeue -u <NETID>
```

**What it does:** Lists the specified user's queued and running jobs. In the `ST` column, `R` means running, `PD` means pending, and `CG` means completing. `TIME` is elapsed runtime; `NODELIST` gives the assigned node.

**Anonymized historical output:**

```text
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             30136   l20-gpu     bash   user_a  R 2-16:14:52      1 dkucc-core-gpu-dkurc-a-it08-22
             30429   l20-gpu pomo_cvrp20   user_b  R    7:31:11      1 dkucc-core-gpu-dkurc-a-it09-26
```

#### 5. Open a shell in an existing interactive job: `srun --jobid`

```bash
srun --jobid=<JOBID> --pty bash
```

**What it does:** Attempts to start a new shell inside an interactive Slurm job that is **still running**. It does not guarantee restoration of the original shell's state. The original guide encountered restrictions on opening a new connection when a shell already existed, but this does not establish a universal one-shell limit. Check that the job is still running and has resources available for a new step. Use `tmux` in advance if you need to preserve your original terminal session.

#### 6. Cancel a job: `scancel`

```bash
scancel <JOBID>
```

**What it does:** Cancels a specified pending or running job. Only cancel jobs you are authorized to manage; contact an administrator about problems involving someone else's jobs.

#### 7. Inspect a single job: `squeue -j` with a custom format

```bash
squeue -j <JOBID> -o "%.18i %.9P %.20j %.8u %.2t %.10M %.9l %.6D %.20R"
```

**What it does:** Shows the partition, job name, user, state, elapsed time, time limit, node count, and scheduling reason or node list for one `JOBID`, using fixed-width columns.

**Anonymized historical output:**

```text
             JOBID PARTITION                 NAME     USER ST       TIME TIME_LIMI  NODES     NODELIST(REASON)
             30136   l20-gpu                 bash   user_a  R 2-16:14:52 4-10:00:00      1 dkucc-core-gpu-dkurc-a-it08-22
```

After a job ends, use `sacct -j <JOBID>` to view its history.

### Example batch script

This Slurm batch example requests one L20 GPU and can be a starting point for either a Learning or Research account. Whether it can run also depends on your remaining monthly allowance, resource limits, account configuration, and the queue. Replace every `<NETID>`, project path, and environment name before using it.

Create the log directory **before submitting**. Slurm opens the files specified by `--output` and `--error` before the script starts, so a `mkdir` inside the script is not enough:

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

Submit and monitor it:

```bash
sbatch /work/<NETID>/slurm/train.sh
squeue -u $USER
tail -f /work/<NETID>/slurm/logs/my-train-<JOBID>.out
sacct -j <JOBID> --format=JobID,State,ExitCode,Elapsed,Timelimit,MaxRSS,DerivedExitCode
```

### Short debugging sessions and long training runs

For short debugging sessions, request an interactive shell:

```bash
srun --partition=l20-gpu --gres=gpu:l20:1 --cpus-per-task=4 --mem=32G --time=01:00:00 --pty bash -l
```

Use interactive shells for debugging and `sbatch` for long training runs. If one script handles both job submission and training, branch on `SLURM_JOB_ID`:

```bash
if [[ -z "${SLURM_JOB_ID:-}" ]]; then
  sbatch --export=ALL,MY_VAR=value "$0" "$@"
else
  python main.py
fi
```

### Troubleshooting queues, termination, and recovery

Queues vary by partition and node. If a job pinned to a node stays `PENDING`, check `NODELIST(REASON)`, your resource request, and your account limits. Cancel and adjust the request if necessary rather than repeatedly switching nodes without diagnosing the cause. Both account types allow at most one running job at a time, and **Research accounts receive no scheduling priority based on account type or project**.

The original guide observed an earlier job receiving `Signal 9 (SIGKILL)` roughly one to two hours after another job began waiting in the queue. This is not a fixed preemption mechanism established by the attachment, and the signal alone does not show that a waiting job eventually preempts your own earlier job. Check `sacct -j <JOBID>`, job logs, and Slurm's actual state and reason codes. Ask IT to inspect the scheduling records if needed.

Save checkpoints regularly during long training runs, and check your monthly allowance, job time limit, and the December 1, 2026 transition requirements. A canceled or terminated job must be resubmitted using your training program's resume procedure. Do not rely on processes left over from a finished job.

<a id="section-7"></a>

## Section 7 Data retention and regular checks in `/work`

The original guide recorded a policy of cleaning up files in `/work` with timestamps older than 75 days, twice a month. The new GPU account attachment does not reconfirm this retention rule. Follow current storage notices or IT's guidance.

**Back up first; maintain timestamps only if permitted.** The examples below are retained from the original guide. Use them only when administrators explicitly allow it and you still need to retain the data. Updating timestamps is not a backup and does not guarantee that files will survive cleanup. If this kind of retention maintenance is not allowed, disable the scripts and cron jobs.

Limit timestamp updates to directories containing data you actually need to retain. Scanning all of `/work/<NETID>` generates substantial I/O and should not be done directly on the login node. The following command requests CPU resources through Slurm and touches only `data` and `outputs`:

```bash
srun --partition=common --time=04:00:00 --mem=2G --cpus-per-task=1 \
  bash -lc 'find /work/<NETID>/data /work/<NETID>/outputs -type f -print0 | xargs -0 -r touch'
```

For an automated run, cron can submit a CPU job on the first day of each month. `*/30` in cron's day-of-month field does not mean a fixed 30-day interval. First create the log directory and `/work/<NETID>/slurm/touch-work.sh`:

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

After confirming that the script runs successfully with `sbatch`, run `crontab -e` and add:

```cron
0 2 1 * * /usr/bin/sbatch /work/<NETID>/slurm/touch-work.sh >> /work/<NETID>/slurm/touch-submit.log 2>&1
```

This means “02:00 on the first day of each month,” not exactly every 30 days. Cron only submits the job from the login node; a CPU node allocated by Slurm performs the scan. If administrators do not permit this kind of retention maintenance, disable the cron entry and rely on off-cluster backups.

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

For another timestamp-maintenance workflow using `~/bin/touch_work_timestamps.sh`, cron on the 1st and 15th of each month, and `monthly_check.sh` for storage checks, see [dkucc-data-ingest](https://github.com/guyong1449/DKUCC-skill). If files have already been cleaned up or accidentally deleted, see [dkucc-data-recovery](https://github.com/guyong1449/DKUCC-skill) for snapshot recovery steps.

---

<a id="official-resources"></a>

## Official resources

Sources for this account and access update:

- DKU IT email dated October 23, 2026: the December 1, 2026 account transition, research usage tracking and billing, and the first invoice schedule.
- The email attachment *DKU Research GPU management*, dated October 16, 2026: account types, resource limits, partitions, application procedures, rates, and validity periods. It does not specify how monthly runtime is aggregated for multi-GPU jobs.
- [Research account request form](https://dukekunshan.formstack.com/forms/dku_gpu_research_account_request_form).

Other references:

- [DKUCC official website](https://dkucc.dukekunshan.edu.cn/)
- [Duke Research Computing Support Documentation](https://oit-rc.pages.oit.duke.edu/rcsupportdocs/)
- [VS Code Remote-SSH](https://code.visualstudio.com/docs/remote/ssh)
- [NVIDIA A40 datasheet](https://images.nvidia.com/content/Solutions/data-center/a40/nvidia-a40-datasheet.pdf)
- [NVIDIA L20 product information](https://www.nvidia.com/en-us/data-center/l20/)
- [NVIDIA H20 product information](https://www.nvidia.com/en-us/data-center/h20/)
- [NVIDIA AI Enterprise GPU support matrix](https://docs.nvidia.com/ai-enterprise/release-7/latest/infra-software/vgpu/features.html)

<a id="tools"></a>

## Related tools

- [guyong1449/cisco-vpn-autoconnect](https://github.com/guyong1449/cisco-vpn-autoconnect): connects to the DKU VPN through Cisco Secure Client on Windows and supports Duo Push.
- [guyong1449/DKUCC-skill](https://github.com/guyong1449/DKUCC-skill): a collection of skills for workflows that are too detailed for this guide or benefit from AI-assisted troubleshooting. It also includes cached DKUCC website pages and archived IT emails.
