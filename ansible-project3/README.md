# Ansible Automation 🤖

Ansible playbooks that automatically configure a fresh Ubuntu EC2 server — from zero to a hardened, production-ready web server with a single command.

---

## Architecture

```
Control Node EC2                    Managed Node EC2
(Runs Ansible)                      (Gets Configured)
      │                                     │
      │─────────── SSH (port 22) ──────────▶│
      │         ansible-playbook            │
      │                                     │
      │                             ✅ System packages updated
      │                             ✅ Deploy user created
      │                             ✅ Nginx installed & running
      │                             ✅ SSH hardened
      │                             ✅ UFW firewall configured
      │                             ✅ Custom HTML page deployed
```

---

## What Gets Configured

| Role | What it does |
|------|-------------|
| `common` | Updates apt cache and upgrades all system packages |
| `user` | Creates `deployuser` with passwordless sudo access |
| `nginx` | Installs Nginx, starts it, deploys a custom HTML page |
| `ssh_hardening` | Disables root login and password authentication |
| `firewall` | Configures UFW — allows ports 22, 80, 443 only |

---

## Project Structure

```
ansible-project3/
├── ansible.cfg                          # Ansible config (SSH key, remote user)
├── inventory.ini                        # Managed node IP
├── site.yml                             # Master playbook
├── templates/
│   └── index.html.j2                    # Custom Nginx webpage (Jinja2)
└── roles/
    ├── common/
    │   └── tasks/main.yml               # Package updates
    ├── user/
    │   └── tasks/main.yml               # User creation + sudo
    ├── nginx/
    │   ├── tasks/main.yml               # Nginx install + deploy
    │   └── handlers/main.yml            # Reload Nginx on change
    ├── ssh_hardening/
    │   └── tasks/main.yml               # SSH config hardening
    └── firewall/
        └── tasks/main.yml               # UFW rules
```

---

## Prerequisites

- Two AWS EC2 instances (Ubuntu) in the same VPC
- Control node has Ansible installed
- Same `.pem` key pair used for both instances
- Managed node Security Group allows port 22 from control node, port 80 from anywhere

---

## Setup

### 1. Install Ansible on the control node

```bash
sudo apt update
sudo apt install -y ansible
ansible --version
```

### 2. Clone this repo

```bash
git clone git@github.com:<your-username>/ansible-project3.git
cd ansible-project3
```

### 3. Configure `ansible.cfg`

```ini
[defaults]
inventory = inventory.ini
remote_user = ubuntu
private_key_file = ~/.ssh/your-key.pem
host_key_checking = False
```

### 4. Set your managed node IP in `inventory.ini`

```ini
[webservers]
managed-node ansible_host=<MANAGED_NODE_PRIVATE_IP>
```

### 5. Test connectivity

```bash
ansible all -m ping
```

Expected output:
```
managed-node | SUCCESS => { "ping": "pong" }
```

---

## Usage

### Dry run (check mode — no changes made)

```bash
ansible-playbook site.yml --check
```

### Full run

```bash
ansible-playbook site.yml
```

### Run a specific role only

```bash
ansible-playbook site.yml --tags nginx
ansible-playbook site.yml --tags firewall
```

---

## Verification

After the playbook runs, verify everything is working:

```bash
# Nginx is running
ansible all -m command -a "systemctl status nginx"

# Custom page is served
curl http://<MANAGED_NODE_PRIVATE_IP>

# Deploy user exists
ansible all -m command -a "id deployuser"

# SSH hardening applied
ansible all -m command -a "grep -E 'PermitRootLogin|PasswordAuthentication' /etc/ssh/sshd_config"

# Firewall rules active
ansible all -m command -a "ufw status"
```

---

## Custom HTML Page

The deployed page at `http://<PUBLIC_IP>` is generated from `templates/index.html.j2` and automatically shows the server's hostname and IP address using Ansible facts:

```html
<p>Server: {{ ansible_hostname }}</p>
<p>IP: {{ ansible_default_ipv4.address }}</p>
```

---

## Security Notes

- Root login is disabled via `PermitRootLogin no`
- Password authentication is disabled — SSH key only
- UFW default policy is `deny` — only ports 22, 80, 443 are open
- `deployuser` uses passwordless sudo (suitable for automation; restrict further in production)

---

## Tech Stack

- **Ansible** — automation engine
- **Ubuntu** — managed node OS
- **Nginx** — web server
- **UFW** — firewall
- **AWS EC2** — cloud infrastructure
- **Jinja2** — HTML templating

---

