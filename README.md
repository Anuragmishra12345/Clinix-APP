# Clinix — Healthcare Triage & Clinical Case Management Platform

![Java](https://img.shields.io/badge/Java-Spring%20Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![React](https://img.shields.io/badge/React-Vite-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![TailwindCSS](https://img.shields.io/badge/Tailwind-CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-Database-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![AWS EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)

**Clinix** is a full-stack, multi-role healthcare triage and clinical case management platform engineered to streamline patient consultations, department-level doctor workflows, and administrative auditing. Built with **Spring Boot**, **React (Vite)**, and **MySQL**, the entire three-tier stack is containerized with **Docker** and hosted on **AWS EC2** with **DuckDNS** routing.

---

## Live Deployment

* **Live Web Application:** [http://clinix-app.duckdns.org:5173](http://clinix-app.duckdns.org:5173)
* **Backend REST API:** `http://clinix-app.duckdns.org:8080`
* **Docker Hub Images:**
  * Frontend: `mishraanurag/clinix-frontend:latest`
  * Backend: `mishraanurag/clinix-backend:latest`

---

## Key Features by Role

### 1. Patient Portal
* **Dual Authentication:** Sign in seamlessly using standard credentials (JWT-backed) or **Google OAuth 2.0** (Google Identity Services).
* **Clinical Case Filing:** Submit new consultation requests with primary symptom details and government ID verification.
* **Real-Time Consultation Pipeline:** Track active cases as they move from triage through doctor assignment, active examination, and resolution, alongside a complete historical archive.
* **Downloadable PDF Medical Reports:** Generate and download formatted clinical summaries and prescriptions (`PrimeLife_Report_<CaseID>.pdf`) directly from the dashboard.
* **Automated Email Alerts:** Receive real-time SMTP email notifications on case status updates with direct deep-links back to the patient dashboard (`/patient`).

### 2. Doctor Workspace
* **Department & Personal Queues:** View incoming unassigned cases filtered by medical department alongside a dedicated personal workspace queue.
* **Concurrency-Safe Case Assignment:** Claim open triage cases with race-condition and double-click protection so two clinicians never overwrite the same case.
* **End-to-End Clinical Lifecycle Controls:** Start patient examinations, defer cases when needed, and finalize consultations with structured clinical notes and prescriptions.

### 3. Admin Console
* **Global Case Oversight:** Monitor and inspect all active, deferred, and completed cases across every hospital department.
* **Centralized Audit Logs:** Track system-wide user events and security actions through real-time audit logs.
* **Role-Gated Onboarding:** Protect privileged clinician and administrator registration routes using dedicated security keys.

---

## System Architecture

```text
                        ┌──────────────────────────────────────┐
                        │         Client Web Browser           │
                        │  [http://clinix-app.duckdns.org:5173](http://clinix-app.duckdns.org:5173)  │
                        └──────────────────┬───────────────────┘
                                           │ HTTP / REST + JWT
                        ┌──────────────────▼───────────────────┐
                        │        AWS EC2 Ubuntu Server         │
                        │        (clinix-app.duckdns.org)      │
                        │                                      │
                        │  ┌────────────────────────────────┐  │
                        │  │  clinix_frontend (Nginx :80)   │  │
                        │  │  Host Port: 5173 -> 80         │  │
                        │  └───────────────┬────────────────┘  │
                        │                  │                   │
                        │  ┌───────────────▼────────────────┐  │
                        │  │  clinix_app (Spring Boot)      │  │
                        │  │  Host Port: 8080 -> 8080       │  │
                        │  └───┬────────────────────────┬───┘  │
                        │      │ JDBC (Docker DNS)      │      │
                        │  ┌───▼────────────────┐       │      │
                        │  │ clinix_mysql (:3306)│       │      │
                        │  │ Volume: mysql_data │       │      │
                        │  └────────────────────┘       │      │
                        └───────────────────────────────┼──────┘
                                                        │
                                     ┌──────────────────┴──────────────────┐
                                     ▼                                     ▼
                       ┌──────────────────────────┐          ┌──────────────────────────┐
                       │   Google Identity (OAuth) │          │    Gmail SMTP Server     │
                       └──────────────────────────┘          └──────────────────────────┘
```

---

## Tech Stack

| Layer | Technologies |
| :--- | :--- |
| **Frontend** | React, Vite, Tailwind CSS, Nginx (Multi-stage Docker build) |
| **Backend** | Java, Spring Boot, Spring Security, Spring Data JPA, Hibernate, Log4j2, Spring Mail |
| **Authentication** | Stateless JWT Bearer Tokens, Google OAuth 2.0 (GSI) |
| **Database** | MySQL 8+ (`clinix_db`) with persistent Docker volume storage |
| **DevOps & Cloud** | Docker, Docker Compose, Docker Hub, AWS EC2 (Ubuntu), DuckDNS |

---

## Project Structure

```text
Clinix-APP/
├── docker-compose.yml                  # Multi-container orchestration (MySQL, Backend, Frontend)
├── clinixBackend/
│   ├── Dockerfile                      # Java runtime container build for Spring Boot JAR
│   └── clinix/
│       ├── pom.xml                     # Maven dependencies & build configuration
│       └── src/main/
│           ├── java/...                # Controllers, Services, Security, Repositories, Entities
│           └── resources/
│               ├── application.properties
│               └── log4j2.xml
└── ClinixFrontend/
    └── clinix-ui/
        ├── Dockerfile                  # Node build stage + Nginx production serving stage
        ├── .env.production             # Production VITE_API_URL configuration
        └── src/
            ├── services/api.js         # Centralized API configuration
            └── views/Dashboard/
                ├── PatientDashboard.jsx
                ├── DoctorDashboard.jsx
                └── AdminDashboard.jsx
```

---

## Core REST API Endpoints

| Access Role | Method | Endpoint | Description |
| :--- | :--- | :--- | :--- |
| **Public** | `POST` | `/api/auth/login` | Authenticate user (standard or Google OAuth) and issue JWT |
| **Patient** | `POST` | `/api/cases/create` | Lodge a new clinical triage consultation request |
| **Patient** | `GET` | `/api/cases/patient/active-pipeline` | Fetch active consultations in the patient's pipeline |
| **Patient** | `GET` | `/api/cases/patient/history` | Retrieve completed consultation history |
| **Patient** | `GET` | `/api/cases/{id}/download-report` | Download generated PDF clinical report |
| **Doctor** | `GET` | `/api/cases/queue/{department}` | List unassigned triage cases for a department |
| **Doctor** | `GET` | `/api/cases/my-queue` | List cases currently assigned to the logged-in doctor |
| **Doctor** | `POST` | `/api/cases/{id}/assign` | Claim an open case into the doctor's workspace |
| **Doctor** | `POST` | `/api/cases/{id}/start-exam` | Transition an assigned case to active examination |
| **Doctor** | `POST` | `/api/cases/{id}/defer` | Defer a case for follow-up |
| **Doctor** | `POST` | `/api/cases/{id}/complete` | Finalize examination with clinical notes and prescription |
| **Admin** | `GET` |
