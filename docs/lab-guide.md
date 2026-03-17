# 🔐 Zero Trust & Identity Lab Guide

> **Difficulty:** Beginner friendly  
> **Time Required:** ~2 hours  
> **Author:** Dhruti Avadhani  

---

## Table of Contents
- [Introduction](#introduction)
- [Lab Environment](#lab-environment)
- [Milestone 1 — Identity-Centric Connectivity](#milestone-1----identity-centric-connectivity)
- [Milestone 2 — Micro-segmentation](#milestone-2----micro-segmentation)
- [Milestone 3 — Least Privilege](#milestone-3----least-privilege)
- [Milestone 4 — GenAI as Security Co-pilot](#milestone-4----genai-as-security-co-pilot)
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

## Milestone 1 — Identity-Centric Connectivity

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

![apt update output](../assets/step1-apt-update.png)

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

![Tailscale install complete](../assets/step2-tailscale-install.png)

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

![Tailscale login URL in terminal](../assets/step2-tailscale-login-url.png)

1. Copy that URL and open it in your browser
2. Click **"Sign in with GitHub"** or **"Sign in with Google"**
3. Use your **personal** account (not university)
4. Click **"Connect"** when prompted

![Browser showing Tailscale connect page](../assets/step2-tailscale-browser.png)

![Tailscale connected success](../assets/step2-tailscale-success.png)

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

![tailscale status output](../assets/step2-tailscale-status.png)

> 💡 **What is the 100.x.x.x address?** This is your Tailscale IP — a private address tied to your identity, not your physical network location. Anyone who wants to reach your machine must be authenticated on your tailnet.

---

### Step 6 — Verify the admin dashboard

Open your browser and go to:
```
https://login.tailscale.com/admin/machines
```

You should see your Kali machine listed with its Tailscale IP and your account identity next to it.

![Tailscale admin dashboard](../assets/step2-tailscale-dashboard.png)

---

### What You Just Proved ✅

| Before | After |
|---|---|
| Machine accessible by IP address | Machine accessible by verified identity only |
| Anyone who knows the IP can try to connect | Only authenticated users on your tailnet can connect |
| No record of who accessed what | Every connection tied to a named identity |

> 🎉 **Milestone 1 Complete!** Your machine is now on an identity-based network. Access is controlled by **who you are**, not **where you are**.
