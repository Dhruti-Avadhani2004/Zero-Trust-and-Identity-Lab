# 🔐 Zero Trust & Identity Lab Guide

> **Difficulty:** Beginner friendly  
> **Time Required:** ~2 hours  
> **Author:** Dhruti Avadhani  

---

## Table of Contents

- [Introduction](#introduction)
- [Lab Environment](#lab-environment)
- [Milestone 1: Identity-Centric Connectivity](#milestone-1-identity-centric-connectivity)
- [Milestone 2: Micro-segmentation](#milestone-2-micro-segmentation)
- [Milestone 3: Least Privilege](#milestone-3-least-privilege)
- [Milestone 4: GenAI as Security Co-pilot](#milestone-4-genai-as-security-co-pilot)
- [Trust Boundary Diagram](#trust-boundary-diagram)
- [Check Your Understanding](#check-your-understanding)

---

## Introduction

### What is Zero Trust?

Traditional network security works like a building with one front door:

> "If you're inside the building, we trust you. If you're outside, we don't."

Once someone gets past the front door (firewall), they can roam freely inside. This is called **perimeter security** — and it's dangerously outdated.

**Zero Trust Architecture (ZTA)** replaces this with a simple principle:

> "Never trust, always verify — regardless of where the request comes from."

Instead of one front door, every single resource has its own lock. Every access request must prove:
- **Who** is making the request (identity)
- **What** they are allowed to access (least privilege)
- **Whether** the request is legitimate (continuous verification)

This lab is based on **NIST SP 800-207** — the US government's official Zero Trust standard.

---

### What You Will Build

By the end of this lab you will have a real Zero Trust setup running on your Linux machine:

| Component | Tool | Purpose |
|---|---|---|
| Identity network | Tailscale | Replace IP-based access with identity-based access |
| Micro-segmentation | Tailscale ACLs | Restrict access to port 8080 only |
| Least privilege | Linux sudoers | Junior admin can only restart one service |
| Security monitoring | journalctl + AI | Audit logs using an LLM |

---

### The Traditional vs Zero Trust Comparison

| | Traditional Perimeter | Zero Trust |
|---|---|---|
| Trust model | Trust everyone inside | Trust no one by default |
| Access control | IP address based | Identity based |
| Lateral movement | Freely possible | Blocked by micro-segmentation |
| Privilege | Broad permissions | Least privilege enforced |
| Monitoring | Perimeter logs only | Every action logged |

---

## Lab Environment

### What You Need

| Requirement | Details |
|---|---|
| OS | Kali Linux (Ubuntu/Debian also works) |
| Network tool | Free [Tailscale account](https://tailscale.com) |
| Identity | Personal GitHub or Google account |
| Skills | Basic Linux terminal usage |
| Time | ~2 hours |

> ⚠️ **Important:** Use a **personal** Google or GitHub account for Tailscale — not a university or corporate account. University accounts may not have admin access to the Tailscale dashboard.

> ⚠️ **Network note:** If you are on a university or corporate WiFi, Tailscale may be blocked by the firewall. Switch to a mobile hotspot if you experience connection issues.

### Lab Architecture

Your entire lab runs on a single Linux machine. Here is what you will set up:
```
Your Linux Machine (Kali VM)
│
├── Tailscale installed
│     └── Connected to your personal tailnet
│           └── Identity: your GitHub/Google account
│
├── Python web server
│     └── Running on port 8080
│           └── Protected by Tailscale ACL rules
│
├── junior-admin user
│     └── sudoers rule: can ONLY restart ssh service
│           └── blocked from everything else
│
└── auth logs (journalctl)
      └── Records every sudo action
            └── Analysed by Claude/ChatGPT
```

---

## Milestone 1: Identity-Centric Connectivity

### What This Milestone Covers

In traditional networks, access is granted based on IP addresses:
> "192.168.1.5 is allowed in"

The problem — IP addresses can be spoofed, shared, or reassigned. They tell you **where** a request came from, not **who** made it.

In this milestone we replace IP-based access with **identity-based access** using Tailscale. After this milestone, your machine is accessible only to authenticated users — not to anyone who just knows an IP address.

---

### Step 1 — Update your system

Before installing anything, make sure your system is up to date:
```bash
sudo apt update
```
```bash
sudo apt upgrade -y
```

> ⏱️ This may take 5-10 minutes depending on how many updates are available.

Expected output after `apt update`:
```
Reading package lists... Done
Building dependency tree... Done
All packages are up to date.
```

![apt update output](/Zero-Trust-and-Identity-Lab/assets/sudo_apt_update.png)

---

### Step 2 — Enable IP Forwarding

Tailscale requires IP forwarding to be enabled on your machine:
```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Expected output:
```
net.ipv4.ip_forward = 1
```

> 💡 **What is IP forwarding?** It allows your machine to forward network packets between interfaces — required for Tailscale to route traffic correctly.

---

### Step 3 — Install Tailscale

Run this single command to install Tailscale from their official repository:
```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Expected output at the end:
```
Installation complete! Log in to start using Tailscale by running:
sudo tailscale up
```

![Tailscale install complete](/Zero-Trust-and-Identity-Lab/assets/tailscale_installed.png)

---

### Step 4 — Log in with your identity

This is the key step — connecting your machine to your personal identity:
```bash
sudo tailscale up --operator=$USER
```

Tailscale will output a URL like this:
```
To authenticate, visit:
https://login.tailscale.com/a/xxxxxxxxxxxxxxx
```

![Tailscale login URL in terminal](/Zero-Trust-and-Identity-Lab/assets/login_to_tailscale.png)

1. Copy that URL and open it in your browser
2. Click **"Sign in with GitHub"** or **"Sign in with Google"**
3. Use your **personal** account (not university)
4. Click **"Connect"** when prompted

![Browser showing Tailscale connect page](/Zero-Trust-and-Identity-Lab/assets/connecting_tailscale_to_github.png)

![Tailscale connected success](/Zero-Trust-and-Identity-Lab/assets/connected_to_tailscale.png)

> ⚠️ **Use a personal account!** If you use a university or corporate Google account, you will not have admin access to the Tailscale dashboard.

---

### Step 5 — Verify the connection
```bash
tailscale status
```

Expected output:
```
100.x.x.x    kali    yourname@gmail.com    linux    -
```
```bash
tailscale ip
```

Expected output:
```
100.x.x.x
```

![tailscale status output](/Zero-Trust-and-Identity-Lab/assets/tailscale_status_and_ip.png)

> 💡 **What is the 100.x.x.x address?** This is your Tailscale IP — a private address tied to your identity, not your physical network location. Anyone who wants to reach your machine must be authenticated on your tailnet.

---

### Step 6 — Verify the admin dashboard

Open your browser and go to:
```
https://login.tailscale.com/admin/machines
```

You should see your Kali machine listed with its Tailscale IP and your account identity next to it.

![Tailscale admin dashboard](/Zero-Trust-and-Identity-Lab/assets/tailscale_admin_panel(1).png)

---

### What You Just Proved 

| Before | After |
|---|---|
| Machine accessible by IP address | Machine accessible by verified identity only |
| Anyone who knows the IP can try to connect | Only authenticated users on your tailnet can connect |
| No record of who accessed what | Every connection tied to a named identity |

> 🎉 **Milestone 1 Complete!** Your machine is now on an identity-based network. Access is controlled by **who you are**, not **where you are**.


---

## Milestone 2: Micro-segmentation

### What This Milestone Covers

Being on a private network doesn't mean you should have access to everything on it. In traditional networks, once you're inside, you can reach any service on any port — this is called **lateral movement** and it's how attackers spread through a network after getting in.

**Micro-segmentation** solves this by creating strict rules about which services are accessible and on which ports — even for authenticated users.

In this milestone we will:
- Spin up a real web service on port 8080
- Write Tailscale ACL rules that allow ONLY port 8080
- Prove that all other ports are blocked

---

### Step 1 — Create and start a web server on port 8080

First create a simple webpage to serve:
```bash
mkdir -p ~/lab-server && echo "<h1>Zero Trust Lab - Service Running on Port 8080</h1>" > ~/lab-server/index.html
```

Navigate into the folder:
```bash
cd ~/lab-server
```

Start the web server:
```bash
python3 -m http.server 8080
```

Expected output:
```
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

![Web server running on port 8080](/Zero-Trust-and-Identity-Lab/assets/starting_http_server_8080.png)

---

### Step 2 — Verify the web server works

Open your browser and visit your Tailscale IP on port 8080:
```
http://100.x.x.x:8080
```

> 💡 Replace `100.x.x.x` with your actual Tailscale IP from `tailscale ip`

You should see:
```
Zero Trust Lab - Service Running on Port 8080
```

![Browser showing web server via Tailscale IP](/Zero-Trust-and-Identity-Lab/assets/service_running_on_port_8080.png)

---

### Step 3 — View the default Tailscale ACL

Open your browser and go to:
```
https://login.tailscale.com/admin/acls
```

You will see the default ACL which looks like this:
```json
{
  "grants": [
    {
      "src": ["*"],
      "dst": ["*"],
      "ip": ["*"]
    }
  ]
}
```

![Default Tailscale ACL](/Zero-Trust-and-Identity-Lab/assets/grants_before.png)

> ⚠️ **What this means:** Everyone can reach everything on every port. This is the equivalent of having no security at all — classic perimeter thinking.

---

### Step 4 — Write the micro-segmentation ACL

Update the ACL to allow access on port 8080 only:
```json
{
  "grants": [
    {
      "src": ["*"],
      "dst": ["*"],
      "ip": ["8080"]
    }
  ],
  "ssh": [
    {
      "action": "check",
      "src": ["autogroup:member"],
      "dst": ["autogroup:self"],
      "users": ["autogroup:nonroot", "root"]
    }
  ]
}
```

![New ACL JSON in editor](/Zero-Trust-and-Identity-Lab/assets/grants_after.png)

Click **"Save"**

---

### What each part of the ACL means

| Part | Meaning |
|---|---|
| `"src": ["*"]` | From any authenticated user on the tailnet |
| `"dst": ["*"]` | To any device on the tailnet |
| `"ip": ["8080"]` | But ONLY on port 8080 |
| Everything else | Implicitly blocked |

> 💡 **Key concept:** In Zero Trust, the default is DENY. You explicitly allow only what is needed. Everything not mentioned is automatically blocked.

---

### Step 5 — Test port 8080 is allowed

Visit your web server again via Tailscale IP:
```
http://100.x.x.x:8080
```

It should still load successfully.

![Port 8080 still accessible](/Zero-Trust-and-Identity-Lab/assets/after_updating_acl_rules_getting_response_on_8080.png)

---

### Step 6 — Test other ports are blocked

Open a new terminal and run:
```bash
curl -v http://100.x.x.x:9090 --max-time 5
```

Expected output:
```
* Trying 100.x.x.x:9090...
* Connection timed out after 5001 milliseconds
* Closing connection 0
curl: (28) Connection timed out after 5000 milliseconds
```

![Port 9090 blocked](/Zero-Trust-and-Identity-Lab/assets/connection_closed_on_port_9090_acl_works.png)

> 💡 **The timeout is a success!** It means the connection never went through. The ACL rule is working exactly as intended.

---

### What You Just Proved

| Port | Result | Reason |
|---|---|---|
| 8080 | Accessible | Explicitly allowed in ACL |
| 9090 | Blocked | Not in ACL, default deny |
| All others | Blocked | Not in ACL, default deny |

> 🎉 **Milestone 2 Complete!** Even authenticated users on your private network can only reach the specific service you explicitly allowed. Lateral movement is now impossible.


---

## Milestone 3: Least Privilege

### What This Milestone Covers

Even authenticated users with network access should only be able to do exactly what their job requires — nothing more. This is the **Principle of Least Privilege.**

In this milestone we will:
- Create a `junior-admin` user representing a low-privilege employee
- Configure a `sudoers` rule giving them exactly ONE permission
- Prove they can restart the ssh service but cannot read sensitive files

---

### Step 1 — Create the junior-admin user
```bash
sudo useradd -m -s /bin/bash junior-admin
```

Set a password:
```bash
sudo passwd junior-admin
```

> 💡 Use a simple password like `Junior@123` for this lab environment.

Verify the user was created:
```bash
cat /etc/passwd | grep junior-admin
```

Expected output:
```
junior-admin:x:1001:1001::/home/junior-admin:/bin/bash
```

![junior-admin user created](/Zero-Trust-and-Identity-Lab/assets/create_junior_admin_profile.png)

---

### Step 2 — Understand the sudoers file

The `/etc/sudoers` file controls who can run which commands as root. The syntax is:
```
username    ALL=(ALL)    NOPASSWD: /path/to/command
```

| Part | Meaning |
|---|---|
| `username` | Which user this rule applies to |
| `ALL=` | On any machine |
| `(ALL)` | Can run as any user |
| `NOPASSWD:` | Without entering a password |
| `/path/to/command` | ONLY this specific command |

> ⚠️ **Always use `visudo` to edit the sudoers file** — it checks for syntax errors before saving. A syntax error in sudoers can lock you out of sudo completely.

---

### Step 3 — Add the least privilege rule

Open the sudoers file safely:
```bash
sudo visudo
```

Scroll to the very bottom and add this line:
```bash
junior-admin ALL=(ALL) NOPASSWD: /usr/sbin/service ssh restart
```

![sudoers file with junior-admin rule](/Zero-Trust-and-Identity-Lab/assets/new_sudoer_file.png)

Save and exit:
- **Nano:** Press `Ctrl + X` → `Y` → `Enter`
- **Vim:** Press `Esc` → type `:wq` → `Enter`

---

### Step 4 — Switch to junior-admin and test

Switch to the junior-admin user:
```bash
su - junior-admin
```

Your prompt should change to:
```
junior-admin@kali:~$
```

---

### Test 1 — junior-admin CAN restart ssh
```bash
sudo service ssh restart && service ssh status
```

Expected output:
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service)
     Active: active (running) since ...
```

![SSH restart success](/Zero-Trust-and-Identity-Lab/assets/junior_admin_can_only_restart_service.png)

> 💡 **No output = success in Linux.** If the command runs silently and returns to the prompt with no error, it worked. We add `service ssh status` to get visible confirmation.

---

### Test 2 — junior-admin CANNOT read sensitive files
```bash
sudo cat /etc/shadow
```

Expected output:
```
Sorry, user junior-admin is not allowed to execute
'/usr/bin/cat /etc/shadow' as root on kali.
```

![Shadow file blocked](/Zero-Trust-and-Identity-Lab/assets/junior_admin_has_no_permission_for_anything_else.png)

---

### Test 3 — junior-admin CANNOT install software
```bash
sudo apt update
```

Expected output:
```
Sorry, user junior-admin is not allowed to execute
'/usr/bin/apt update' as root on kali.
```

![apt update blocked](/Zero-Trust-and-Identity-Lab/assets/junior_admin_has_no_permission_for_anything_else.png)

---

### Step 5 — Switch back to your main user
```bash
exit
```

Your prompt should return to:
```
kali@kali:~$
```

![Exited junior-admin session](/Zero-Trust-and-Identity-Lab/assets/logout_as_junior_admin.png)

---

### What You Just Proved

| Action | User | Result | Reason |
|---|---|---|---|
| `sudo service ssh restart` | junior-admin | Allowed | Explicitly in sudoers rule |
| `sudo cat /etc/shadow` | junior-admin | Blocked | Not in sudoers rule |
| `sudo apt update` | junior-admin | Blocked | Not in sudoers rule |

> 🎉 **Milestone 3 Complete!** A junior employee can do exactly their job — restart a service when needed — but cannot access sensitive data or make system-wide changes.


---

## Milestone 4: GenAI as Security Co-pilot

### What This Milestone Covers

Security analysts deal with thousands of log lines every day. Manually reading every line is slow, error-prone, and exhausting. **Generative AI** can act as a co-pilot — reading logs instantly, explaining what happened, and flagging suspicious activity in plain English.

In this milestone we will:
- Read the authentication logs from our lab session
- Paste them into Claude or ChatGPT
- Ask the AI to analyse and explain what security events occurred

> 💡 **This is a real technique** used by security teams today. AI doesn't replace the analyst — it speeds up the investigation so the analyst can focus on decisions, not manual log reading.

---

### Step 1 — View your authentication logs

On Kali Linux, authentication events are stored in the systemd journal. Run this to see all sudo-related activity from the last hour:
```bash
sudo journalctl _COMM=sudo --since "1 hour ago"
```

You will see entries recording every sudo command that was run during your lab — including the junior-admin permission grants and denials.

![journalctl output](/Zero-Trust-and-Identity-Lab/assets/logs_from_1_hr_ago.png)

> 💡 **Why no auth.log?** Kali Linux uses `systemd-journald` instead of the traditional `auth.log` file. The `journalctl` command is the modern equivalent and contains the same information.

---

### Step 2 — Copy the log output

Run the command again and select all the output text with your mouse. Copy it.
```bash
sudo journalctl _COMM=sudo --since "1 hour ago"
```

---

### Step 3 — Open Claude or ChatGPT

Open a new browser tab and go to:
- `https://claude.ai`
- `https://chatgpt.com`

---

### Step 4 — Paste this prompt

Copy this entire prompt, paste it into the AI chat, then paste your log lines after it:
```
You are a security analyst. I am a Junior Security Analyst 
learning about Zero Trust Architecture.

Please analyse the following Linux authentication logs from 
my lab session and explain:
1. What security events occurred
2. Which events show successful privilege use
3. Which events show blocked/denied access attempts
4. What this tells us about the Principle of Least Privilege
5. Are there any suspicious or concerning entries?

Here are the logs:

[PASTE YOUR LOG LINES HERE]
```

![Prompt and logs pasted into AI](/Zero-Trust-and-Identity-Lab/assets/using_ai_to_analyze_logs.png)

---

### Step 5 — Review the AI's response

The AI will analyse your logs and explain:
- Which commands were run with sudo
- Which were allowed vs denied
- What the pattern tells us about access control
- Any anomalies or concerns

![AI analysis response](/Zero-Trust-and-Identity-Lab/assets/1.png)

---

### What good AI log analysis looks like

A good AI response will identify entries like these and explain them:

| Log entry type | What it means |
|---|---|
| `COMMAND=/usr/sbin/service ssh restart` | junior-admin restarted ssh — allowed by sudoers |
| `command not allowed` | junior-admin tried something outside their permissions |
| `sudo: pam_unix(sudo:auth)` | A sudo authentication event occurred |
| `session opened for user root` | A sudo command successfully ran as root |
| `session closed for user root` | The sudo session ended |

---

### Using AI as a Security Co-pilot — Best Practices

> 💡 **Tip 1 — Be specific in your prompts**
> The more context you give the AI, the better its analysis. Tell it what you were doing, what users were involved, and what you're looking for.

> 💡 **Tip 2 — Always verify AI findings**
> AI can make mistakes. Use it to speed up your analysis but always verify important findings yourself.

> 💡 **Tip 3 — Never paste real production logs into public AI tools**
> In a real company, logs may contain sensitive information. Use private/enterprise AI tools or anonymise logs before pasting.

> 💡 **Tip 4 — Iterate on your prompts**
> If the first response isn't detailed enough, ask follow-up questions like "Can you explain the denied entries in more detail?" or "What would an attacker learn from these logs?"

---

### What You Just Proved

| Traditional approach | AI-assisted approach |
|---|---|
| Manually read hundreds of log lines | AI reads and summarises instantly |
| Easy to miss patterns | AI identifies patterns across all entries |
| Time consuming | Results in seconds |
| Requires deep log expertise | AI explains in plain English |

> 🎉 **Milestone 4 Complete!** You have used AI as a real security tool — not just a chatbot. This is how modern security analysts work.


---

## Trust Boundary Diagram

This diagram shows how your Zero Trust setup controls access at every layer — from network identity all the way down to individual user permissions.

![Trust Boundary Diagram](/Zero-Trust-and-Identity-Lab/assets/trust_boundary.png)

### The Three Trust Boundaries

**Boundary 1 — Network layer (Tailscale)**
No one can even reach your machine without first authenticating with a verified identity. IP addresses mean nothing here.

**Boundary 2 — Service layer (ACL rules)**
Even authenticated users can only reach port 8080. All other ports are silently blocked regardless of who is asking.

**Boundary 3 — Privilege layer (sudoers)**
Even users with network access can only perform the specific actions their role requires. Everything else is denied.

> 💡 **This is defence in depth** — multiple independent layers of security. An attacker would need to break through all three layers, not just one.

---

## Check Your Understanding

Test yourself with these questions. Try to answer them before looking at the answers!

---

**Question 1**
What is the main difference between Traditional Perimeter Security and Zero Trust Architecture?

<details>
<summary>Click to reveal answer</summary>

Traditional Perimeter Security trusts everyone inside the network — once you're past the firewall, you can reach anything. Zero Trust trusts no one by default — every access request must be verified regardless of where it comes from.

</details>

---

**Question 2**
In your Tailscale ACL, what happens to traffic on port 443 (HTTPS)?

<details>
<summary>Click to reveal answer</summary>

It is blocked. In Zero Trust, the default is DENY. Only port 8080 was explicitly allowed in the ACL. Any port not mentioned is automatically blocked — including port 443.

</details>

---

**Question 3**
Why did we use `visudo` instead of directly editing `/etc/sudoers` with a text editor?

<details>
<summary>Click to reveal answer</summary>

`visudo` checks the sudoers file for syntax errors before saving. A syntax error in `/etc/sudoers` can completely lock you out of sudo — meaning you cannot run any admin commands. `visudo` prevents this by validating the file first.

</details>

---

**Question 4**
What would happen if `junior-admin` tried to run `sudo reboot`?

<details>
<summary>Click to reveal answer</summary>

It would be blocked. The sudoers rule only allows `junior-admin` to run `/usr/sbin/service ssh restart`. Any other command — including `sudo reboot` — would return "Sorry, user junior-admin is not allowed to execute this command."

</details>

---

**Question 5**
Why should you never paste real production logs into a public AI tool like ChatGPT?

<details>
<summary>Click to reveal answer</summary>

Production logs may contain sensitive information such as usernames, IP addresses, hostnames, service names, and access patterns. Pasting these into a public AI tool means that data leaves your organisation and could potentially be used to train AI models or be exposed in a data breach. Always use enterprise AI tools or anonymise logs before analysis.

</details>

---

**Question 6**
What does the `--operator=$USER` flag do in `sudo tailscale up --operator=$USER`?

<details>
<summary>Click to reveal answer</summary>

It grants your normal (non-root) user the ability to control Tailscale without needing `sudo` every time. Without this flag, only root can run Tailscale commands. `$USER` automatically fills in your current username.

</details>

---

## Congratulations! 🎉

You have successfully completed the Zero Trust & Identity Lab. Here is what you built:

| Milestone | What you proved |
|---|---|
| 1 — Identity access | Network access controlled by identity, not IP |
| 2 — Micro-segmentation | Only port 8080 accessible, all else blocked |
| 3 — Least privilege | junior-admin limited to one exact command |
| 4 — GenAI co-pilot | AI can analyse logs and explain security events |

---

*Lab guide authored by Dhruti Avadhani — Zero Trust & Identity Lab — 2026*
