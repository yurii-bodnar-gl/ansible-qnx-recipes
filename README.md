# QNX 7.1 Bare-Metal Software Provisioning (PoC)

This repository contains a Proof of Concept (PoC) Ansible playbook for the automated deployment of software on bare-metal QNX 7.1 systems. 

Currently, the playbook demonstrates how to securely deliver and execute a real POSIX-compliant application (using `acme.sh` as a test payload) on a closed embedded system.

## Architecture: The "Payload Embedding" Approach

The main challenge with bare-metal QNX systems is the absence of a Python environment, package managers, and often basic network utilities like `curl` or `wget`. This makes standard Ansible pull-based modules useless.

This playbook solves the problem using a **Payload Embedding (Push)** architecture:
1. **Local Retrieval:** The Ansible control node (Ubuntu) downloads the target artifact from the internet/artifact registry.
2. **Code Injection:** Ansible dynamically generates a shell script and injects the raw source code of the downloaded application directly into it using the `lookup` plugin.
3. **Delivery & Self-Extraction:** The "fat" installer script is transferred via a standard SSH connection to the QNX target. When executed, the script self-extracts the embedded payload into the user's home directory (`$HOME/real_app`).
4. **Execution & Cleanup:** The extracted application is executed to verify integrity, and all local temporary files are safely removed.

## Setup Instructions

1. **Clone the repository and navigate to the directory:**
```bash
git clone <YOUR_REPOSITORY_URL>
cd ansible-qnx-poc

```

2. **Create and activate a virtual environment:**

```bash
python3 -m venv .venv
source .venv/bin/activate

```

3. **Install dependencies (Ansible core & lint):**

```bash
pip install -r requirements.txt
# Or manually: pip install ansible ansible-lint

```

4. **Configure the inventory:**
The project uses a secure approach without hardcoding passwords. Copy the template file:

```bash
cp inventory.example.ini inventory.ini

```

Open `inventory.ini` and specify the real IP address of your QNX server and the username. **Do not add the password to this file.**

## ▶Running the Playbook

Execute the following command to start the deployment. The `-k` (or `--ask-pass`) flag will force Ansible to securely prompt for the QNX server password in the terminal:

```bash
ansible-playbook playbook.yml -i inventory.ini -k

```

**Expected Result:**
Ansible will download the payload locally, inject it into an installer script, and push it to the QNX target. On the target, it will create the `$HOME/real_app` directory, self-extract the application, run it, and return the application's version output (e.g., `v3.1.3`) directly to your Ansible terminal. Finally, it will clean up local temporary files.