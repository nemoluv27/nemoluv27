
# Phase 01 — Product Planning & Project Foundation

## Overview

The first phase of **All About Korea** focused on defining the initial product direction and establishing the local development foundation for the application.

The goal of this phase was to create a working full-stack project structure with a frontend, backend, database, and database migration system before expanding the application with additional features.

At the end of this phase, the backend can successfully connect to PostgreSQL, retrieve persisted data through SQLAlchemy, and expose the data through a FastAPI endpoint.

---

## Product Concept

**All About Korea** is an application designed to help users explore and learn about Korea through structured content.

Initial content categories include:

- Korean History
- Korean Culture
- Korean Language

The application will eventually provide structured lessons, interactive learning content, and quizzes within each category.

The project is being developed entirely from the ground up, beginning as a local application and gradually evolving into a production service running on AWS.

---

## Initial Architecture

The initial local architecture consists of:

```text
Frontend
Next.js + React + TypeScript
        |
        | API Requests
        v
Backend
FastAPI + Python
        |
        | SQLAlchemy
        v
Database
PostgreSQL
