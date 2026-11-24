---
title: "INFN Trieste Computing Farm"
description: "Practical reference for storage, quotas, AFS, XRootD and HTCondor on the INFN Trieste CMS farm."
tags:
  - computing
  - cms
  - hpc
date: 2026-11-24
draft: false
---

# INFN Trieste Computing Farm

Reference for working on the CMS farm at INFN Trieste, based on the initial account setup and filesystem inspection in November 2026. Commands below are examples; replace placeholders with your own account details and site hostnames, obtained privately. Access permissions and storage policies must be checked locally.

## Access and software

Connect from the local workstation:

```bash
LOGIN_HOST="your-login-host"
LOGIN_USER="your-infn-username"
ssh "$LOGIN_USER@$LOGIN_HOST"
```

The CMS CVMFS repository is mounted on the login node. Check the environment before setting up [[Computing/CMSSW|CMSSW]]:

```bash
cat /etc/os-release
ls /cvmfs/cms.cern.ch/
command -v apptainer
command -v singularity
command -v root-docker
```

`root-docker` is a local command encountered during setup; its exact behavior has not been verified. Changes to the shell environment can be made in `~/.bashrc` and reloaded with `source ~/.bashrc`.

## Storage and quotas

| Path | Purpose / considerations |
| --- | --- |
| `$HOME` | NFS home directory; scripts, configuration and small files. |
| `/gfsvol01/cms/users/$USER/work` | CMS shared storage; candidate for software work areas and analysis outputs. |
| `/gfsvol01/cms/users/$USER/personal` | Separate user-managed directory; not necessarily private or backed up. |
| `/gpfs` | Large shared GPFS filesystem; allocation and permissions must be established before use. |
| `/localdisk` | Node-local XFS storage; do not assume persistence or availability on worker nodes. |
| `/afs/infn.it/ts/user/$USER` | INFN Trieste AFS user space, including an optional web directory. |
| `/afs/infn.it/ts/project/cms` | CMS project AFS space; check access before using it. |

The CMS work and personal directories can be initialized with:

```bash
mkdir -p "/gfsvol01/cms/users/$USER"/{work,personal}
cd "/gfsvol01/cms/users/$USER/work"
```

These directories were created during the initial setup. Check actual write access, quota and worker-node visibility before installing a release or launching production.

```bash
cmsquota
df -h "$HOME" "/gfsvol01/cms/users/$USER"
df -hT
quota -vs
du -sh "/gfsvol01/cms/users/$USER/work"
```

- `cmsquota`: site-specific quota command encountered during setup; inspect its output for the applicable CMS allocation.
- `df`: filesystem-wide capacity, not the individual quota.
- `quota -vs`: conventional user quota query. An empty result does not establish unlimited storage.
- `du`: actual space occupied by the selected directory.

## AFS and web publishing

Check AFS volume quota and directory access-control lists (ACLs):

```bash
fs lq "/afs/infn.it/ts/user/$USER"
fs la "/afs/infn.it/ts/user/$USER/html"
fs lq /afs/infn.it/ts/project/cms
```

An optional `html` directory was prepared for web publishing. If intentionally publishing public material, the setup uses:

```bash
mkdir -p "/afs/infn.it/ts/user/$USER/html"
fs sa -dir "/afs/infn.it/ts/user/$USER/html" -acl ts:www rl
fs la "/afs/infn.it/ts/user/$USER/html"
```

The AFS permissions `r` and `l` allow reading files and looking up directory entries. Granting access to a web-server identity may expose the contents through a website. Check existing ACLs and the site publishing policy first; never place credentials or collaboration-private material there. AFS volume quotas may be shared by multiple directories.

## EOS through XRootD

Inspect the CMS storage namespace with the XRootD client:

```bash
EOS_HOST="your-eos-host"
xrdfs "$EOS_HOST" ls /eos/infnts/cms/store
xrdfs "$EOS_HOST" ls /eos/infnts/cms/store/user/
```

These commands list remote directories. A successful listing does not establish write permission, personal allocation or authentication requirements.

## HTCondor and monitoring

```bash
condor_userprio
condor_status -total
condor_q
lscpu
top
htop
```

- `condor_userprio`: inspect scheduling priorities.
- `condor_status -total`: inspect advertised pool resources.
- `condor_q`: inspect queued jobs.
- `lscpu`, `top`, `htop`: inspect the current node, not the full batch pool.

The initial setup also explored a Ganglia monitoring page; the supported monitoring URL and job submission conventions remain to be confirmed.

## SSH key authentication

If desired and permitted by the site, generate the key on the **local workstation**, not on the remote login node. Reuse an existing suitable key rather than overwriting it.

```bash
ssh-keygen -t ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub "$LOGIN_USER@$LOGIN_HOST"
```

Only the public key belongs in the remote `~/.ssh/authorized_keys` file. Keep the private key on the workstation and use a passphrase. Do not commit keys or authorization files to a repository. Key-based login was attempted during setup but has not been confirmed.

## Before the first CMSSW / HiForest production

- Check the CMS quota with `cmsquota` and confirm the storage policy.
- Verify that the work directory is accessible from batch worker nodes.
- Check the operating system, supported container runtime and CMSSW architecture.
- Validate a short interactive job before submitting a batch production.
- Confirm the site authentication and output-transfer requirements for HTCondor and XRootD.

The source for this note is the initial farm setup command history and filesystem inspection (November 2026); command success and site policies are not inferred from history alone.
