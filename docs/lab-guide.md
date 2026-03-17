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
