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

The project is being developed from the ground up, beginning as a local application and gradually evolving into a production service running on AWS.

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
```

Database schema changes are managed using **Alembic**, while PostgreSQL runs locally using **Docker Compose**.

---

## Technology Stack

### Frontend

- Next.js
- React
- TypeScript
- Tailwind CSS

### Backend

- Python
- FastAPI
- Uvicorn
- SQLAlchemy

### Database

- PostgreSQL
- Alembic

### Local Development

- Docker
- Docker Compose
- Python virtual environment

### Version Control

- Git
- GitHub

---

## Repository Structure

The project uses a monorepo structure containing both the frontend and backend.

```text
AllAboutKorea/
├── backend/
│   ├── alembic/
│   ├── app/
│   │   ├── database.py
│   │   ├── main.py
│   │   └── models.py
│   └── alembic.ini
│
├── frontend/
│   ├── app/
│   ├── components/
│   └── public/
│
├── docker-compose.yml
└── .gitignore
```

This structure allows the frontend, backend, and local infrastructure configuration to be managed within a single repository.

---

## Backend Foundation

A minimal FastAPI application was created with an initial health check endpoint:

```http
GET /health
```

This provides a simple way to verify that the backend service is running correctly.

The application is served locally using Uvicorn.

---

## PostgreSQL Environment

PostgreSQL was configured as the application's initial relational database and runs locally in a Docker container.

Database credentials are provided through environment variables rather than being stored directly in the application source code.

SQLAlchemy is used by the backend to communicate with PostgreSQL.

The database connection flow is:

```text
FastAPI
   |
   v
SQLAlchemy Session
   |
   v
SQLAlchemy Engine
   |
   v
PostgreSQL
```

A reusable SQLAlchemy session dependency was implemented so that FastAPI endpoints can obtain a database session for each request and close it after the request is completed.

---

## Database Migration

Alembic was configured to manage database schema changes.

The first migration created the `categories` table.

```text
categories
├── id
├── name
└── slug
```

The corresponding SQLAlchemy model was also created.

This establishes a migration-based workflow for future database changes rather than manually modifying the database schema.

---

## Initial Application Data

The first three application categories were added to PostgreSQL:

| ID | Name | Slug |
|---:|---|---|
| 1 | History | history |
| 2 | Culture | culture |
| 3 | Language | language |

During development, both direct SQL operations and SQLAlchemy sessions were used to understand and verify the database workflow.

---

## First Database-Backed API

The first database-backed API endpoint was implemented:

```http
GET /categories
```

The endpoint retrieves category records from PostgreSQL using SQLAlchemy.

The complete request flow is:

```text
Client
   |
   | GET /categories
   v
Uvicorn
   |
   v
FastAPI
   |
   v
SQLAlchemy Session
   |
   | SELECT categories
   v
PostgreSQL
   |
   v
FastAPI JSON Response
```

Example response:

```json
[
  {
    "name": "History",
    "id": 1,
    "slug": "history"
  },
  {
    "name": "Culture",
    "id": 2,
    "slug": "culture"
  },
  {
    "name": "Language",
    "id": 3,
    "slug": "language"
  }
]
```

This verified the first complete data flow between the FastAPI backend and PostgreSQL database.

---

## Phase 01 Completed

Phase 01 established the initial development foundation for **All About Korea**.

Completed work includes:

- Defined the initial product concept and development direction
- Designed the initial application and content structure
- Selected the frontend, backend, and database technology stack
- Created the monorepo project structure
- Initialized the Next.js frontend
- Initialized the FastAPI backend
- Created a local PostgreSQL environment using Docker
- Integrated SQLAlchemy with PostgreSQL
- Implemented reusable database session management
- Configured Alembic database migrations
- Created the initial `categories` database model and migration
- Added the initial application categories
- Implemented the first database-backed API endpoint
- Verified the FastAPI → SQLAlchemy → PostgreSQL data flow
- Added the application source code to Git and GitHub

---

## Next Phase

### Phase 02 — Core Full-Stack Application Development

The next phase will focus on expanding the application beyond the initial foundation.

Planned work includes:

- Define API request and response schemas
- Expand backend API functionality
- Build the core application data flow
- Connect the Next.js frontend to the FastAPI backend
- Retrieve backend data from the frontend
- Begin developing the application's core user-facing functionality

The goal of Phase 02 is to move from independently working frontend and backend foundations toward a connected full-stack application.
