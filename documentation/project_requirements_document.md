# Project Requirements Document (PRD)

## 1. Project Overview

The Educational Management System (EMS) is a web-based application designed to centralize and simplify school operations for three user groups: Kurikulum (administrators/curriculum planners), Guru (teachers), and Siswa (students). By providing secure, role-specific portals, EMS streamlines tasks such as grade entry, attendance tracking, schedule management, and reporting. This reduces manual paperwork, minimizes data errors, and accelerates communication between staff and learners.

EMS is being built to replace scattered spreadsheets and disconnected tools with a unified, scalable platform. Key objectives for the first version include:

- Secure session-based authentication and strict role-based access control (RBAC).
- Three distinct dashboards (Kurikulum, Guru, Siswa) each tailored to the user’s responsibility.
- CRUD (Create, Read, Update, Delete) operations for core entities: users, classes, subjects, grades, and schedules.
- Data visualization components for quick insights (e.g., grade charts, attendance summaries).
- A responsive, modern UI supporting light/dark themes for improved usability.

Success will be measured by: on-time user onboarding, accurate data entry workflows, sub-500ms page load times under typical load, and positive feedback from pilot users.

## 2. In-Scope vs. Out-of-Scope

### In-Scope (Version 1)

- Session-based authentication using Better Auth (sign-up, sign-in, sign-out).
- Role-based dashboards under `/app/kurikulum`, `/app/guru`, and `/app/siswa`.
- User management (create, edit, deactivate) limited to Kurikulum role.
- Class and subject management (CRUD) for Kurikulum.
- Grade entry and review workflows for Guru.
- Schedule viewing for all roles; schedule creation/editing for Kurikulum.
- Basic data tables and charts (using Shadcn/ui components).
- Database schema definitions for users, classes, subjects, grades, and schedules (PostgreSQL + Drizzle ORM).
- Light/dark mode toggle via centralized theming.
- Containerized development environment with Docker and Docker Compose.
- Basic linting and formatting (ESLint, Prettier).

### Out-of-Scope (Phase 2+)

- Real-time chat or messaging between users (Socket.IO).
- File upload and digital resource management (e.g., lesson plans, assignments).
- Mobile or native apps (iOS/Android).
- Advanced analytics or AI-driven insights.
- API rate limiting or throttling.
- Internationalization (multi-language support).
- Automated testing and CI/CD pipelines (to be introduced later).

## 3. User Flow

A new user (e.g., a teacher) receives an account invitation or self-registers on `/sign-up`. They set a password and verify their email. After registration, they are redirected to `/sign-in`, enter their credentials, and initiate a session. Once authenticated, middleware checks their `role` in the database and routes them to the correct dashboard (`/guru/dashboard`). The left sidebar presents navigation links like “My Classes,” “Enter Grades,” and “Profile.” The main panel fetches data on the server (Server Component) and displays a list of classes or recent notifications.

From the dashboard, a teacher clicks “Enter Grades,” lands on a data table showing students and assessments for a selected class. They input scores, hit “Save,” and the system updates the PostgreSQL database through a Drizzle ORM query. If a curriculum planner logs in, they see “Manage Users,” “Manage Classes,” and “View Reports” instead. A student logging in sees “My Grades,” “My Schedule,” and a chart of their semester performance. At any point, the user can toggle light/dark mode, open the profile menu to sign out, or navigate back to their dashboard.

## 4. Core Features

- **Authentication & RBAC**: Secure sign-up/sign-in flows using Better Auth; middleware enforces route protection based on user roles.
- **Kurikulum Portal**: User management (create/edit/deactivate), class and subject management, schedule builder, and report overview.
- **Guru Portal**: Class roster view, grade entry interface, attendance tracker (future), and personal profile.
- **Siswa Portal**: View grades, view schedule, and performance charts.
- **Data Visualization**: Charts (e.g., line, bar) for grades and attendance using Shadcn/ui.
- **Theming**: Auto-detect system theme; manual toggle between light and dark modes.
- **Database & ORM**: Type-safe schemas and queries using Drizzle ORM on PostgreSQL.
- **Modular UI Components**: Reusable components (DataTable, Chart, Sidebar) with Tailwind CSS styling.
- **Containerization**: Local development via Docker Compose, including Node.js, PostgreSQL, and Redis placeholder.

## 5. Tech Stack & Tools

- **Frontend**: Next.js (App Router) + React, TypeScript, Tailwind CSS v4, Shadcn/ui.
- **Backend**: Next.js API routes (serverless functions) on Node.js, Better Auth for sessions.
- **Database**: PostgreSQL with Drizzle ORM (type-safe query builder).
- **Containerization**: Docker & Docker Compose.
- **Development Tools**: VS Code (recommended), ESLint, Prettier.
- **Potential Integrations Later**: Redis for caching, Socket.IO for real-time, AWS SDK for file uploads.

## 6. Non-Functional Requirements

- **Performance**: Page loads under 500ms for common views; database queries optimized via indexes.
- **Security**: HTTPS-only, secure cookies, session timeouts, RBAC checks on every API route.
- **Reliability**: Automatic retries on transient database errors; health-check endpoints in Docker.
- **Usability**: Responsive design for desktop and tablet; clear error messages and form validations.
- **Scalability**: Stateless API routes, containerized services; later add auto-scaling.
- **Maintainability**: Consistent TypeScript types, modular folder structure, JSDoc/TSDoc comments on complex logic.

## 7. Constraints & Assumptions

- **Better Auth Availability**: Assumes the Better Auth service is reliably available; fallback to JWT is not yet implemented.
- **Monolithic Architecture**: All API routes run in Next.js; no separate microservices for Version 1.
- **Browser Compatibility**: Modern browsers only (Chrome, Firefox, Safari, Edge).
- **Data Volume**: Initial pilot with up to 1,000 users and 10,000 grade records; performance tuned for this scale.
- **Infrastructure**: Development and production use Docker; production orchestration (Kubernetes) is assumed but not yet configured.

## 8. Known Issues & Potential Pitfalls

- **API Rate Limits**: Next.js serverless API routes may throttle under heavy traffic. Mitigation: plan for Redis caching and possibly move to a dedicated Node.js server.
- **Session Management Edge Cases**: Session expiry may log out active users unexpectedly. Mitigation: implement silent token refresh or warn users before timeout.
- **Database Schema Migrations**: Adding new columns or tables can cause downtime if not handled carefully. Mitigation: use Drizzle’s migration tooling and zero-downtime patterns.
- **Role Escalation Risk**: Improper RBAC configuration could let lower-privileged roles access higher-privileged data. Mitigation: write unit tests for middleware and API route guards.
- **UI Performance with Large Data**: Rendering large tables on the client can be slow. Mitigation: implement server-side pagination and virtualized tables in future iterations.

---

This PRD captures the core requirements for EMS Version 1 and provides a clear roadmap for development. Subsequent technical documents (Tech Stack Document, Frontend Guidelines, Backend Structure) can be derived directly from these specifications without ambiguity.