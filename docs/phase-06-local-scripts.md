# 🧩 Phase 6 – Local Platform Bootstrap & Automation Baseline

## 📌 Overview

This phase introduces a standardized local Kubernetes bootstrap workflow using `kind`.
It provides a reproducible, cloud-independent environment to continue platform evolution
despite temporary AWS usage constraints.

> This phase was conducted locally to develop and validate automation scripts before
> re-deploying to AWS, reducing cloud cost during the scripting iteration cycle.

---

## 🎯 Goals

- 🧱 Provide a **single entry point** for spinning up a local Kubernetes environment
- 🔁 Ensure repeatable and idempotent local setup
- 🔌 Enable further platform work (GitOps, observability, CI) without cloud dependency

---

## 🛠️ What Was Added

### 🚀 Local Bootstrap Script

A lightweight Bash script was introduced to automate:

- Prerequisite checks (Docker, kubectl, kind)
- Idempotent kind cluster creation
- Local Docker image loading into the cluster
- Kubernetes manifest application and rollout verification
```bash
./bootstrap.sh
```
