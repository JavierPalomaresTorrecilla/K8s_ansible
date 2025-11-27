# K8s Ansible Playbooks

This repository provides quick automation to bootstrap a Kubernetes control plane either in a single-host (all-in-one) setup or in a multi-node topology with dedicated workers.

## Prepare Every Node

Run the following on each host (master and workers) before using the deploy scripts:

```bash
ssh-keygen -t rsa -b 4096
sudo apt update
sudo apt install net-tools
```

Copy the master's public key (`~/.ssh/id_rsa.pub`) to every worker's `~/.ssh/authorized_keys` so Ansible can log in without prompts.

## Prepare the Master (control machine)

On the machine where you run the deploy scripts:

```bash
sudo apt install python3-pip
sudo apt install python3-venv
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install "ansible-core>=2.16,<2.18"
ansible-galaxy collection install ansible.posix -vvv
```

Stay inside the virtualenv (`source .venv/bin/activate`) whenever you run the scripts.

## Configure Node Details

All configuration lives under `config/`:

- `config/single-node.conf` — set `HOST_IP` to the IP address that other machines will use to reach the node (check with `hostname -I` or `ip -4 addr`). Set `HOST_NAME` to the hostname you want registered in `/etc/hosts` (usually `hostname` output). Example:
  ```
  HOST_IP=192.168.123.164
  HOST_NAME=vm1
  ```
  The deploy script reads this file automatically; CLI flags can override it temporarily.

- `config/multi-node.conf` — set `MASTER_HOST_NAME`/`MASTER_HOST_IP` exactly like above but for the control-plane host. Populate `WORKERS` with a semicolon-separated list of worker definitions in the form `name,IP,ssh_key`. Use absolute or `$HOME`-based SSH key paths. Examples:
  ```
  WORKERS="worker1,192.168.10.11,$HOME/.ssh/id_rsa"
  WORKERS="worker1,192.168.10.11,$HOME/.ssh/id_rsa;worker2,192.168.10.12,$HOME/.ssh/id_rsa"
  ```
  - `name`: inventory hostname (must be unique)
  - `IP`: the node’s SSH-accessible address
  - `ssh_key`: private key on the control host that matches the worker’s `authorized_keys`

Keep every entry on a single line even if you add multiple workers; separate them with `;`.

## Single-Node Deployment

From the repo root:

```bash
./single-node_deploy
```


## Multi-Node Deployment

Configure `config/multi-node.conf`, then run:

```bash
./multi-node_deploy
```


After either workflow, the `kubectl get nodes -o wide` command at the end verifies cluster health. Use `./*_deploy --uninstall` to roll back.
