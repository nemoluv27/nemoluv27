# Phase 1 – Initial Service Setup & Operational Baseline

## 🎯 Goal
Establish a **minimal but realistic service baseline** that allows me to focus on  
**DevOps and operational concerns**, rather than application feature development.

The goal of this phase is **not** to build complex business logic, but to:
- Set up a runnable service
- Integrate external managed services
- Prepare the system for CI/CD, monitoring, and future expansion

---

## 🧩 Service Overview

This project starts from a simple **personal finance / expense tracking service**.

To intentionally reduce application-level complexity:
- Authentication is delegated to an external service
- Database is managed externally
- The application code is treated as a given input

This allows me to focus on **how the service is built, deployed, observed, and evolved**.

---

## 🏗 Initial Architecture

**Application**
- Expense tracking (ledger-style) service
- Minimal CRUD functionality

**Authentication**
- Managed authentication via Clerk
- Login and user identity handled externally

**Database**
- Neon (managed PostgreSQL)
- No local database management in early phases

**Infrastructure**
- Local Kubernetes (Docker Desktop / kind)
- Containerized application

---

## 🔧 What I Implemented in Phase 1

### Application Baseline
- Imported an existing expense tracking service codebase
- Containerized the application
- Verified local execution in Kubernetes

### External Services Integration
- Configured Clerk for authentication and login
- Connected the application to Neon using environment variables
- Avoided embedding secrets directly into the codebase

### Operational Focus (DevOps Perspective)
Instead of adding application features, I focused on:
- How the service is deployed
- How configuration is managed
- How failures will be detected later

This phase intentionally sets the foundation for:
- CI/CD pipelines
- Monitoring and logging integration
- Platform-level changes without touching application logic

---

## 🔄 Planned Evolution from This Baseline

This initial setup is designed to evolve without rewriting the service.

Planned changes include:
- **CI/CD**
  - GitHub Actions → Jenkins migration
- **Platform**
  - Local Kubernetes → Amazon EKS
- **Database**
  - Neon → self-managed MySQL or PostgreSQL
- **Observability**
  - Prometheus & Grafana for metrics
  - Loki for logs
  - Jaeger for tracing
- **Service Mesh**
  - Istio with Kiali for traffic visualization

---

## 🧠 Key Takeaways (So Far)

- Using managed external services early reduces cognitive load
- Treating application code as immutable enables platform-level experimentation
- A stable baseline is critical before introducing CI/CD and observability
- Designing for change early prevents tight coupling later

---

## ➡️ Next Phase
Phase 2 will focus on introducing **CI pipelines using GitHub Actions**,  
automating builds and deployments while keeping the platform unchanged.
