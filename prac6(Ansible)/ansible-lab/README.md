# Practical 6: Ansible with a Control Node and Two Managed Nodes

> **Exam / Lab Weightage:** 30 Marks  
> **Aim:** From one control node, use an Ansible playbook to install and configure Apache and utilities on two managed (remote) servers over SSH, and demonstrate that the playbook is idempotent.  
> **Architecture:** 3 Ubuntu 22.04 Docker containers connected over a shared bridge network (`control`, `node1`, `node2`).  
> **Target Ports:** `http://localhost:8091` (node1) and `http://localhost:8092` (node2).

---

## Table of Contents
1. [Overview & Architecture](#overview--architecture)
2. [Container Environment Summary](#container-environment-summary)
3. [Part A: Start the Three Servers (Docker Compose in PowerShell)](#part-a-start-the-three-servers-docker-compose-in-powershell)
4. [Part B: Install Ansible on the Control Node (Q6.1, 4 Marks)](#part-b-install-ansible-on-the-control-node-q61-4-marks)
5. [Part C: Inventory & Connectivity Ping Test (Q6.2, 5 Marks)](#part-c-inventory--connectivity-ping-test-q62-5-marks)
6. [Part D: Write `install-packages.yml` Playbook (Q6.3, 12 Marks)](#part-d-write-install-packagesyml-playbook-q63-12-marks)
7. [Part E: Execute Playbook (Q6.4, 5 Marks)](#part-e-execute-playbook-q64-5-marks)
8. [Part F: Verify Web App & Demonstrate Idempotency (Q6.5, 4 Marks)](#part-f-verify-web-app--demonstrate-idempotency-q65-4-marks)
9. [Safe Customizations & Configuration Reference](#safe-customizations--configuration-reference)
10. [Troubleshooting Guide](#troubleshooting-guide)
11. [Viva Voce Questions & Answers](#viva-voce-questions--answers)
12. [Screenshots Checklist](#screenshots-checklist)

---

## Overview & Architecture

Ansible is an **agentless** configuration management and automation tool:
- **Control Node (`control`):** The central machine where Ansible is installed. It reads the inventory and executes playbooks.
- **Managed Nodes (`node1`, `node2`):** Target servers configured by Ansible over standard SSH. No client agent needs to be installed on them—they only require an SSH daemon and Python 3.
- **Idempotency:** An operation is idempotent if executing it multiple times produces the exact same result with no unwanted side effects. If a package is already installed or a file already matches, Ansible makes no changes (`changed=0`).

```
+-------------------------------------------------------------------------+
|                        Docker Network (ansible-lab)                     |
|                                                                         |
|  +--------------------+                                                 |
|  |    control Node    | (Runs Ansible, inventory.ini, install-packages) |
|  +--------------------+                                                 |
|           |                                                             |
|           | SSH (Port 22, root / ansible123)                            |
|     +-----+------------------------------+                              |
|     |                                    |                              |
|     v                                    v                              |
|  +----------------------+             +----------------------+          |
|  |        node1         |             |        node2         |          |
|  |  (Apache Web Server) |             |  (Apache Web Server) |          |
|  |  Host Port: 8091     |             |  Host Port: 8092     |          |
|  +----------------------+             +----------------------+          |
+-------------------------------------------------------------------------+
           |                                       |
           v                                       v
    http://localhost:8091                   http://localhost:8092
```

---

## Container Environment Summary

| Container Name | Role in Lab | Packages / Responsibilities | Exposed Host Port |
| :--- | :--- | :--- | :--- |
| **`control`** | Ansible Control Node | Ubuntu 22.04 + Ansible + `sshpass` + `curl` | *None (internal only)* |
| **`node1`** | Managed Node 1 | Ubuntu 22.04 + OpenSSH Server &rarr; Configured with Apache | `8091:80` |
| **`node2`** | Managed Node 2 | Ubuntu 22.04 + OpenSSH Server &rarr; Configured with Apache | `8092:80` |

---

## Part A: Start the Three Servers (Docker Compose in PowerShell)

Run these commands in a standard PowerShell window on Windows:

```powershell
mkdir ansible-lab
cd ansible-lab
```

### 1. Create `Dockerfile.node` (SSH Server Image for Managed Nodes)
```powershell
@'
FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y openssh-server python3 sudo \
    && mkdir -p /var/run/sshd \
    && echo "root:ansible123" | chpasswd \
    && sed -i "s/^#\?PermitRootLogin.*/PermitRootLogin yes/" /etc/ssh/sshd_config
EXPOSE 22 80
CMD ["/usr/sbin/sshd", "-D"]
'@ | Set-Content -Encoding ascii Dockerfile.node
```

---

### 2. Create `compose.yaml` (Defines control, node1, node2)
```powershell
@'
services:
  control:
    image: ubuntu:22.04
    container_name: control
    hostname: control
    command: sleep infinity
  node1:
    build:
      context: .
      dockerfile: Dockerfile.node
    container_name: node1
    hostname: node1
    ports:
      - "8091:80"
  node2:
    build:
      context: .
      dockerfile: Dockerfile.node
    container_name: node2
    hostname: node2
    ports:
      - "8092:80"
'@ | Set-Content -Encoding ascii compose.yaml
```

---

### 3. Build & Launch Containers
```powershell
dir
docker compose config
docker compose up -d --build
docker compose ps
```

Verify that all three containers (`control`, `node1`, `node2`) report status **Up**.

---

## Part B: Install Ansible on the Control Node (Q6.1, 4 Marks)

Enter the `control` container bash shell:

```powershell
docker exec -it control bash
```
*(Your prompt will change to `root@control:/#`). All subsequent commands in Parts B to F run inside this Linux shell.*

Install Ansible, SSH password provider (`sshpass`), and curl:

```bash
export DEBIAN_FRONTEND=noninteractive
apt update
apt install -y ansible sshpass curl
ansible --version
```

> **Screenshot 1:** Terminal output of `ansible --version` showing Ansible core version and Python interpreter.

---

## Part C: Inventory & Connectivity Ping Test (Q6.2, 5 Marks)

Create the lab workspace directory:

```bash
mkdir -p /root/lab && cd /root/lab
```

### 1. Create Inventory File (`inventory.ini`)
```bash
cat > inventory.ini << 'EOF'
[webservers]
node1 ansible_host=node1
node2 ansible_host=node2

[webservers:vars]
ansible_user=root
ansible_password=ansible123
ansible_python_interpreter=/usr/bin/python3
EOF
```

---

### 2. Create Ansible Configuration (`ansible.cfg`)
Disables strict SSH host key checking so Ansible connects automatically without interactive prompts:

```bash
cat > ansible.cfg << 'EOF'
[defaults]
inventory = inventory.ini
host_key_checking = False
EOF
```

---

### 3. Verify Inventory & Test Connectivity
```bash
cat inventory.ini
ansible all -m ping
```

### Expected Output:
```json
node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
node2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

> **Screenshot 2:** Terminal output of `cat inventory.ini` and `ansible all -m ping` showing both nodes returning green `"ping": "pong"` SUCCESS.

---

## Part D: Write `install-packages.yml` Playbook (Q6.3, 12 Marks)

### 1. Create the Custom Web Page (`index.html`)
Inside `/root/lab`:

```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<body style="font-family:Arial;text-align:center;margin-top:100px;">
  <h1>Deployed by Ansible</h1>
  <p>Lalmohammad Patel - Roll Number</p>
</body>
</html>
EOF
```

---

### 2. Create the Playbook (`install-packages.yml`)
```bash
cat > install-packages.yml << 'EOF'
- name: Install and configure web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install packages
      apt:
        name:
          - apache2
          - curl
          - git
          - vim
        state: present

    - name: Ensure apache2 is started and enabled
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Copy custom index.html
      copy:
        src: index.html
        dest: /var/www/html/index.html
EOF
```

Verify the playbook:
```bash
cat install-packages.yml
```

### Playbook Tasks & Modules Breakdown

| Task | Module | Purpose & Explanation |
| :--- | :--- | :--- |
| **Update apt cache** | `apt` | Refreshes package lists (`apt update`). `cache_valid_time: 3600` avoids re-downloading if updated within the last hour. |
| **Install packages** | `apt` | Installs `apache2`, `curl`, `git`, and `vim` with `state: present`. |
| **Ensure apache2 is started & enabled** | `service` | Starts the `apache2` service now (`state: started`) and enables it on system boot (`enabled: yes`). |
| **Copy custom index.html** | `copy` | Copies local `index.html` from `control` to `/var/www/html/index.html` on both nodes. |

> **Screenshot 3:** Terminal output of `cat install-packages.yml`.

---

## Part E: Execute Playbook (Q6.4, 5 Marks)

Run the playbook from `/root/lab`:

```bash
ansible-playbook install-packages.yml
```

### Expected PLAY RECAP (First Execution):
```text
PLAY RECAP *********************************************************************
node1                      : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node2                      : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
- `ok=5`: All 5 steps completed successfully.
- `changed=4`: 4 tasks modified the system (apt cache updated, packages installed, service started, index.html deployed).
- `failed=0`: No failures encountered.

> **Screenshot 4:** Full output of `ansible-playbook install-packages.yml` showing `failed=0` on both nodes in PLAY RECAP.

---

## Part F: Verify Web App & Demonstrate Idempotency (Q6.5, 4 Marks)

### 1. Browser Verification
Open both URLs on your host Windows browser:
- `http://localhost:8091` &rarr; Displays: **Deployed by Ansible** (from `node1`)
- `http://localhost:8092` &rarr; Displays: **Deployed by Ansible** (from `node2`)

---

### 2. Verify Internal IP Addresses & Network Responses (from Control Node)
```bash
ansible webservers -m command -a "hostname -I"
curl http://node1
curl http://node2
```

---

### 3. Demonstrate Idempotency (Second Execution)
Run the exact same playbook again:

```bash
ansible-playbook install-packages.yml
```

### Expected PLAY RECAP (Second Execution):
```text
PLAY RECAP *********************************************************************
node1                      : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node2                      : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

> **Why `changed=0`?**  
> Since Apache is already installed, the service is already running, and `index.html` already matches, Ansible changes nothing. This strictly proves **Idempotency**.

> **Screenshot 5:** Browser pages at `http://localhost:8091` and `http://localhost:8092`.  
> **Screenshot 6:** Second playbook run output showing `changed=0` in PLAY RECAP.

To exit the container:
```bash
exit
```

---

## Safe Customizations & Configuration Reference

| Target File | Permitted Customizations | Important Cautions |
| :--- | :--- | :--- |
| `index.html` | Heading text, Name, Roll Number, CSS colors | Re-run `ansible-playbook install-packages.yml` to push updates |
| `install-packages.yml` | Add more packages (e.g. `htop`, `tree`, `nano`) | When added, the next run shows `changed=1`, then `changed=0` thereafter |
| `inventory.ini` / `Dockerfile.node` | Change password from `ansible123` | Password must match in both files; requires `docker compose up -d --build` |
| `compose.yaml` | Change host ports `8091` / `8092` | Update the browser URL accordingly |

---

## Troubleshooting Guide

| Issue / Error | Root Cause | Solution |
| :--- | :--- | :--- |
| `UNREACHABLE / Connection refused` | Managed containers are stopped | Run `docker compose up -d` in `ansible-lab` from PowerShell |
| `to use 'ssh' connection type with passwords, install sshpass` | Missing `sshpass` package | Run `apt install -y sshpass` inside the `control` container |
| `Host key verification failed` | Strict host key checking enabled | Ensure `ansible.cfg` contains `host_key_checking = False` and you are inside `/root/lab` |
| `Could not find or access 'index.html'` | `index.html` missing in working directory | Create `index.html` in `/root/lab` before running the playbook |
| `YAML error / found a tab character` | Tabs used instead of spaces in YAML | YAML forbids tab characters (`\t`). Indent using spaces only |
| `No such container: control` | Containers not started from `compose.yaml` | Run `docker compose up -d` in the folder containing `compose.yaml` |

---

## Viva Voce Questions & Answers

1. **What is Ansible?**  
   An open-source, agentless IT automation and configuration management tool that automates software provisioning, configuration, and application deployment using SSH and YAML playbooks.

2. **What is the difference between a Control Node and a Managed Node?**  
   - **Control Node:** The machine where Ansible is installed and executed from.  
   - **Managed Node:** Target systems/servers configured by the control node. They do not have Ansible installed.

3. **Why is Ansible referred to as "Agentless"?**  
   Unlike tools like Puppet or Chef, Ansible requires no custom background daemon or agent installed on target machines; it communicates purely over standard SSH and uses the remote system's existing Python interpreter.

4. **What is an Inventory file?**  
   A configuration file (INI or YAML) that specifies the hostnames/IP addresses of managed nodes, grouped logically (such as `[webservers]`), along with connection credentials and variables.

5. **What is a Playbook and a Task?**  
   A **Playbook** is a YAML document containing ordered "plays" and tasks. A **Task** is an individual execution action invoking an Ansible module (e.g. `apt`, `service`, `copy`).

6. **What does `become: yes` do?**  
   Enables privilege escalation, running the specified tasks as a superuser (`sudo`) on the managed host.

7. **What is Idempotency?**  
   The mathematical and operational property where running an operation multiple times leaves the system in the exact same state without unintended duplicate changes. If the target state is already achieved, Ansible reports `changed=0`.

---

## Screenshots Checklist

- [ ] **1.** `ansible --version` output inside control node
- [ ] **2.** `cat inventory.ini` and `ansible all -m ping` (green pong)
- [ ] **3.** `cat install-packages.yml`
- [ ] **4.** First run: `ansible-playbook install-packages.yml` with `changed=4`
- [ ] **5.** Browser at `http://localhost:8091` and `http://localhost:8092`
- [ ] **6.** Second run: `ansible-playbook install-packages.yml` with `changed=0` (Idempotency proof)
