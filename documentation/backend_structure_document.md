# Backend Structure Document

## 1. Backend Architecture

We have a monolithic backend powered by Next.js (App Router) running on Node.js. This setup gives us built-in routing, server-side rendering, and API routes without needing a separate Express server. Here’s how it’s organized and why it works:

• Monolithic Next.js App Router  
  • Uses Server Components for secure, server-only data fetching  
  • Leverages file-based routing under `/app/api` for REST endpoints

• Design Patterns & Frameworks  
  • Authentication via Better Auth  
  • Type safety and ORM logic via Drizzle ORM (built on PostgreSQL)  
  • Utility-first styling (Tailwind CSS) & component library (shadcn/ui)

• Scalability  
  • Dockerized services allow horizontal scaling in a container platform (ECS, Kubernetes)  
  • Stateless API routes can scale behind a load balancer  

• Maintainability  
  • Clear separation: `/app` (routes & pages), `/db` (schema & data access), `/components` (UI), `/lib` (shared utilities)  
  • TypeScript throughout for self-documenting code and fewer runtime errors

• Performance  
  • Server-side rendering (SSR) for initial page loads  
  • Redis caching for frequent read operations  
  • CDN for static assets (CSS/JS/images)

---

## 2. Database Management

Our persistent layer uses PostgreSQL, a reliable, relational SQL database. We manage data with Drizzle ORM, which provides type-safe queries that align with our TypeScript code.

• Database Type: SQL (PostgreSQL)  
• ORM: Drizzle ORM (type-safe, migration support)  
• Data Organization: normalized tables for users, classes, subjects, grades, schedules, question banks  
• Access Patterns:  
  • CRUD via API routes in Next.js  
  • Server Components fetch only necessary fields  
  • Caching layer (Redis) for hot data like class rosters

• Data Management Practices:  
  • Versioned migrations (Drizzle CLI)  
  • Automated daily backups (RDS snapshotting)  
  • Indexing on foreign keys, frequently queried columns

---

## 3. Database Schema

Below is a human-readable overview, followed by example SQL (PostgreSQL) DDL for core tables.

### Human-Readable Entity Descriptions

• **users**: stores all accounts (Kurikulum, Guru, Siswa) with roles  
• **classes**: school classes, linked to one or more teachers  
• **subjects**: topics taught in classes, owned by classes  
• **grades**: numeric results for students per subject and semester  
• **schedules**: weekly class timetable entries  
• **bank_soal**: question bank for quizzes and exams

### Example SQL Schema (PostgreSQL)

```sql
-- 1. users
type role_enum as enum('kurikulum', 'guru', 'siswa');

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  role role_enum NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- 2. classes
CREATE TABLE classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  teacher_id UUID REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- 3. subjects
CREATE TABLE subjects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  class_id UUID REFERENCES classes(id),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- 4. grades
CREATE TABLE grades (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID REFERENCES users(id),
  subject_id UUID REFERENCES subjects(id),
  grade_value NUMERIC CHECK (grade_value >= 0 AND grade_value <= 100),
  semester TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- 5. schedules
CREATE TABLE schedules (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  class_id UUID REFERENCES classes(id),
  subject_id UUID REFERENCES subjects(id),
  day_of_week SMALLINT CHECK (day_of_week BETWEEN 1 AND 7),
  start_time TIME NOT NULL,
  end_time TIME NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- 6. bank_soal
CREATE TABLE bank_soal (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID REFERENCES subjects(id),
  question_text TEXT NOT NULL,
  options JSONB NOT NULL,
  correct_answer TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);
```  

---

## 4. API Design and Endpoints

We follow a RESTful convention using Next.js API routes. Each route lives under `/app/api` and maps HTTP methods to CRUD operations.

• **Authentication**  
  • POST `/api/auth/sign-up` – create account  
  • POST `/api/auth/sign-in` – log in and set session  
  • POST `/api/auth/sign-out` – clear session  

• **User Management**  
  • GET `/api/users` – list users  
  • POST `/api/users` – add new user  
  • PUT `/api/users/{id}` – update user  
  • DELETE `/api/users/{id}` – remove user

• **Classes & Subjects**  
  • GET `/api/classes`  
  • POST `/api/classes`  
  • ...similar pattern for `/api/subjects`

• **Grades**  
  • GET `/api/grades?studentId={}`  
  • POST `/api/grades` – assign or update a grade

• **Schedules**  
  • GET `/api/schedules?classId={}`  
  • POST `/api/schedules`

• **Question Bank**  
  • GET `/api/bank-soal?subjectId={}`  
  • POST `/api/bank-soal`

• **File Upload**  
  • GET `/api/upload-url?filename={}` – get S3 presigned URL  

• **Real-Time Chat** (optional)  
  • Socke­t.IO endpoint on custom Express server at `/socket.io`

All `/api/*` routes enforce RBAC via middleware based on session and the user’s `role`.

---

## 5. Hosting Solutions

We host on AWS for a cost-effective, reliable, and scalable environment:

• **Compute**  
  • Containerized Next.js app on ECS Fargate behind an Application Load Balancer  
  • Auto-scale tasks based on CPU/memory

• **Database**  
  • Amazon RDS for PostgreSQL with Multi-AZ for high availability  

• **Caching**  
  • Amazon ElastiCache (Redis) for query results and session caching

• **Storage**  
  • Amazon S3 for file uploads and static assets  
  • CloudFront CDN for global delivery of assets

Benefits:  
  • Managed services reduce operational overhead  
  • Elastic scaling matches traffic demands  
  • Pay-as-you-go pricing

---

## 6. Infrastructure Components

These components work together to deliver a fast, fault-tolerant service:

• **Load Balancer (ALB)**  
  • Distributes incoming requests across ECS tasks  
  • Performs health checks

• **Container Orchestration (ECS/Kubernetes)**  
  • Manages service scaling and failover

• **Cache Layer (Redis)**  
  • Speeds up reads for non-volatile data (e.g., class rosters)

• **CDN (CloudFront)**  
  • Caches and serves static assets closer to users

• **Object Storage (S3)**  
  • Stores uploaded files (lesson materials, quizzes)

• **Real-Time Server (Express + Socket.IO)**  
  • Runs alongside Next.js for chat features

---

## 7. Security Measures

We apply multiple layers of security to protect data and comply with regulations:

• **Authentication**  
  • Session-based via Better Auth, HTTP-only Secure cookies  
  • Option to switch to JWT if microservices are introduced

• **Authorization (RBAC)**  
  • Next.js middleware inspects sessions and `role` to guard routes  

• **Encryption**  
  • TLS (HTTPS) for all in-transit data  
  • Encryption at rest in RDS and S3 buckets

• **Data Validation & Sanitization**  
  • Input schemas and server-side checks prevent injection attacks

• **Network Security**  
  • Private subnets for databases  
  • Security groups restrict access to only necessary ports

• **OWASP Best Practices**  
  • CSRF protection  
  • Rate limiting on authentication endpoints  
  • Helmet.js or built-in headers for XSS protection

---

## 8. Monitoring and Maintenance

We combine managed tools and best practices to keep the backend healthy:

• **Logging**  
  • Application logs via Winston / Next.js logging to CloudWatch  
  • Structured JSON logs for easy querying

• **Metrics & Alerts**  
  • AWS CloudWatch metrics for CPU, memory, request latency  
  • Custom application metrics (error rates, cache hit ratio)

• **Error Tracking**  
  • Sentry for real-time exception reporting

• **Dashboards**  
  • Grafana / CloudWatch Dashboards for visualizing system health

• **Maintenance**  
  • Automated RDS snapshots and retention policy  
  • Blue/green or rolling deployments via ECS  
  • Regular dependency and security patch updates  

---

## 9. Conclusion and Overall Backend Summary

Our backend is a modern, containerized Next.js monolith that meets the Educational Management System’s needs for security, performance, and scalability. Key takeaways:

• **Unified Architecture:** Next.js App Router + API routes simplify development and deployment.  
• **Robust Data Layer:** PostgreSQL + Drizzle ORM enforces type safety and relational integrity.  
• **Flexible Scaling:** AWS managed services (ECS, RDS, ElastiCache, S3, CloudFront) ensure reliability and cost-effectiveness.  
• **Security by Design:** Layered authentication, authorization, encryption, and network controls protect sensitive student and staff data.  

With this foundation, the EMS can grow—adding features like file uploads, real-time chat, and complex business logic—while maintaining high uptime, fast response times, and strict security standards.