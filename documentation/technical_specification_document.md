# Educational Management System (EMS) - Technical Specification Document

## Document Information

- **Document Version:** 1.0
- **Date:** October 19, 2025
- **Author:** EMS Development Team
- **Status:** Draft
- **Reviewers:** Technical Lead, Project Manager, System Architect

---

## 1. Executive Summary

### 1.1 Project Overview

The Educational Management System (EMS) is a comprehensive, web-based platform designed to digitalize and integrate core school administrative, educational, and communication functions. The system serves three primary user groups through specialized portals:

- **Kurikulum (Curriculum/Administrative)**: System configuration, data integrity, scheduling
- **Guru (Teacher)**: Classroom execution, assessment, content creation, student interaction
- **Siswa (Student)**: Learning, progress tracking, collaboration

### 1.2 Technical Vision

The EMS system is built on a modern, scalable 3-tier architecture leveraging:
- **Frontend**: Next.js with React, TypeScript, and Tailwind CSS
- **Backend**: Node.js with Next.js API routes and Better Auth
- **Database**: PostgreSQL with Drizzle ORM
- **Infrastructure**: Docker containerization with CI/CD pipeline

### 1.3 Success Metrics

- **Performance**: 95% of API requests respond in <500ms
- **Security**: Zero critical vulnerabilities, 100% RBAC compliance
- **Usability**: >90% user satisfaction score
- **Reliability**: 99.9% system uptime
- **Scalability**: Support for 1,000+ concurrent users

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend Layer                            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │ Kurikulum   │ │    Guru     │ │       Siswa         │   │
│  │  Dashboard  │ │  Dashboard  │ │     Dashboard       │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           Next.js API Routes                        │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │   │
│  │  │    Auth     │ │   CRUD API  │ │   Business  │   │   │
│  │  │ Middleware  │ │  Endpoints  │ │   Logic     │   │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Data Layer                              │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │ PostgreSQL  │ │    Redis    │ │    File Storage     │   │
│  │   Database  │ │    Cache    │ │    (AWS S3)         │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Component Architecture

#### 2.2.1 Frontend Architecture
- **Framework**: Next.js 14+ with App Router
- **UI Library**: shadcn/ui components with Tailwind CSS
- **State Management**: React Query + Zustand
- **Authentication**: Better Auth client integration
- **Routing**: File-based routing with middleware protection

#### 2.2.2 Backend Architecture
- **Runtime**: Node.js 18+
- **Framework**: Next.js API Routes
- **Authentication**: Better Auth session management
- **Database**: PostgreSQL 15+ with Drizzle ORM
- **Caching**: Redis for session and data caching

#### 2.2.3 Infrastructure Architecture
- **Containerization**: Docker + Docker Compose
- **Orchestration**: Kubernetes (production)
- **CI/CD**: GitHub Actions
- **Monitoring**: Application logs + error tracking

---

## 3. Functional Requirements

### 3.1 Kurikulum (Administrative) Requirements

#### 3.1.1 User Management (FR-A-01)
**Description**: Full CRUD operations for all system users

**Requirements**:
- Create, read, update, delete user accounts
- Assign and modify user roles (Guru, Siswa, Kurikulum)
- Bulk user import/export functionality
- User activity monitoring and audit logs
- Account deactivation and reactivation

**API Endpoints**:
- `POST /api/admin/users` - Create user
- `GET /api/admin/users` - List users (with filters)
- `PUT /api/admin/users/:id` - Update user
- `DELETE /api/admin/users/:id` - Delete/deactivate user
- `POST /api/admin/users/bulk` - Bulk operations

#### 3.1.2 Scheduling (FR-A-02)
**Description**: Academic schedule creation and management

**Requirements**:
- Create class schedules by subject, teacher, and time slot
- Schedule conflict detection and resolution
- Bulk schedule updates for semester changes
- Schedule publishing and notifications
- Substitute teacher assignment

**API Endpoints**:
- `POST /api/admin/schedules` - Create schedule
- `GET /api/admin/schedules` - List schedules
- `PUT /api/admin/schedules/:id` - Update schedule
- `POST /api/admin/schedules/conflicts` - Check conflicts
- `POST /api/admin/schedules/publish` - Publish schedule

#### 3.1.3 Grade Oversight (FR-A-03)
**Description**: Comprehensive grade management and oversight

**Requirements**:
- View all student grades across all subjects
- Grade adjustments with audit trail
- Grade report generation
- Grade analytics and statistics
- Bulk grade validation

**API Endpoints**:
- `GET /api/admin/grades` - View all grades
- `PUT /api/admin/grades/:id/adjust` - Adjust grade
- `GET /api/admin/grades/analytics` - Grade statistics
- `POST /api/admin/grades/validate` - Validate grades
- `GET /api/admin/grades/export` - Export grades

#### 3.1.4 Curriculum Setup (FR-A-04)
**Description**: Academic curriculum structure management

**Requirements**:
- Create and manage subjects (Mata Pelajaran)
- Define class structures and grade levels
- Manage student transfers (Mutasi)
- Curriculum mapping and prerequisites
- Academic calendar management

**API Endpoints**:
- `POST /api/admin/curriculum/subjects` - Create subject
- `GET /api/admin/curriculum/subjects` - List subjects
- `POST /api/admin/curriculum/classes` - Create class
- `PUT /api/admin/curriculum/classes/:id` - Update class
- `POST /api/admin/curriculum/transfers` - Process transfer

#### 3.1.5 Attendance (FR-A-05)
**Description**: Centralized attendance monitoring

**Requirements**:
- Monitor attendance patterns across all stakeholders
- Generate attendance reports
- Set attendance policies and thresholds
- Automated absence notifications
- Attendance trend analysis

**API Endpoints**:
- `GET /api/admin/attendance/dashboard` - Attendance overview
- `GET /api/admin/attendance/reports` - Attendance reports
- `POST /api/admin/attendance/policies` - Set policies
- `GET /api/admin/attendance/analytics` - Trend analysis

### 3.2 Guru (Teacher) Requirements

#### 3.2.1 Grade Input (FR-G-01)
**Description**: Secure grade entry and management for assigned classes

**Requirements**:
- Input grades for assigned subjects/students only
- Grade validation and error checking
- Bulk grade import functionality
- Grade history tracking
- Grade calculation formulas

**API Endpoints**:
- `POST /api/teacher/grades` - Submit grades
- `GET /api/teacher/grades/:classId` - View class grades
- `PUT /api/teacher/grades/:id` - Update grade
- `POST /api/teacher/grades/bulk` - Bulk grade entry
- `GET /api/teacher/grades/history` - Grade history

#### 3.2.2 Content Upload (FR-G-02)
**Description**: Educational content management and distribution

**Requirements**:
- Upload RPP/SILABUS (lesson plans)
- Upload and organize Modul (modules)
- File version control
- Content categorization by subject/class
- Secure file storage with access control

**API Endpoints**:
- `POST /api/teacher/content/upload` - Upload content
- `GET /api/teacher/content` - List content
- `PUT /api/teacher/content/:id` - Update content metadata
- `DELETE /api/teacher/content/:id` - Delete content
- `GET /api/teacher/content/:id/download` - Download content

#### 3.2.3 Bank Soal Management (FR-G-03)
**Description**: Question bank creation and management

**Requirements**:
- Create, edit, and categorize questions
- Tag questions by subject, topic, difficulty
- Question organization into test banks
- Question reuse and versioning
- Question import/export functionality

**API Endpoints**:
- `POST /api/teacher/bank-soal` - Create question
- `GET /api/teacher/bank-soal` - List questions
- `PUT /api/teacher/bank-soal/:id` - Update question
- `POST /api/teacher/bank-soal/import` - Import questions
- `GET /api/teacher/bank-soal/export` - Export questions

#### 3.2.4 Consultation Chat (FR-G-04)
**Description**: Real-time communication with students

**Requirements**:
- Private chat with assigned students
- Message history and search
- File sharing in chat
- Online status indicators
- Chat archiving and reporting

**API Endpoints**:
- `WebSocket /api/chat/consultation` - Real-time chat
- `GET /api/chat/consultation/rooms` - Chat rooms
- `POST /api/chat/consultation/messages` - Send message
- `GET /api/chat/consultation/history` - Message history

### 3.3 Siswa (Student) Requirements

#### 3.3.1 Schedule View (FR-S-01)
**Description**: Personalized academic schedule access

**Requirements**:
- View daily and weekly class schedules
- Personalized schedule based on enrollment
- Schedule change notifications
- Calendar integration
- Schedule export functionality

**API Endpoints**:
- `GET /api/student/schedule/daily` - Daily schedule
- `GET /api/student/schedule/weekly` - Weekly schedule
- `GET /api/student/schedule/changes` - Schedule changes
- `POST /api/student/schedule/export` - Export schedule

#### 3.3.2 Progress Report (FR-S-02)
**Description**: Academic performance tracking and reporting

**Requirements**:
- View current grades and scores
- Attendance record viewing
- Semester performance graphs
- Progress trends analysis
- Personalized feedback access

**API Endpoints**:
- `GET /api/student/report/grades` - Grade report
- `GET /api/student/report/attendance` - Attendance record
- `GET /api/student/report/analytics` - Performance analytics
- `GET /api/student/report/feedback` - Personal feedback

#### 3.3.3 Interactive Practice (FR-S-03)
**Description**: Self-assessment and practice tools

**Requirements**:
- Access to practice quizzes from question bank
- Self-paced learning modules
- Instant feedback on answers
- Progress tracking for practice sessions
- Difficulty level selection

**API Endpoints**:
- `GET /api/student/quizzes` - Available quizzes
- `POST /api/student/quizzes/:id/start` - Start quiz
- `POST /api/student/quizzes/:id/submit` - Submit answers
- `GET /api/student/quizzes/results` - Quiz results

#### 3.3.4 Collaboration (FR-S-04)
**Description**: Peer and teacher collaboration tools

**Requirements**:
- Access to class discussion forums
- Real-time teacher communication
- Group project collaboration
- Document sharing capabilities
- Discussion thread management

**API Endpoints**:
- `GET /api/discussion/forums` - Discussion forums
- `POST /api/discussion/forums/:id/messages` - Post message
- `WebSocket /api/discussion/realtime` - Real-time updates
- `GET /api/discussion/groups` - Collaboration groups

#### 3.3.5 Tabungan (FR-S-05)
**Description**: Financial account management for students

**Requirements**:
- View current savings balance
- Transaction history viewing
- Account statements generation
- Deposit/withdrawal request submission
- Financial activity reporting

**API Endpoints**:
- `GET /api/student/savings/balance` - Current balance
- `GET /api/student/savings/transactions` - Transaction history
- `POST /api/student/savings/statement` - Generate statement
- `POST /api/student/savings/requests` - Submit requests

---

## 4. Non-Functional Requirements

### 4.1 Security Requirements

#### 4.1.1 Authentication
- **Implementation**: Better Auth for session-based authentication
- **Session Management**: Secure HTTP-only cookies
- **Session Timeout**: 1 hour inactivity timeout
- **Password Policy**: Minimum 8 characters, complexity requirements
- **Multi-Factor**: Optional MFA for administrative roles

#### 4.1.2 Authorization
- **Model**: Role-Based Access Control (RBAC)
- **Roles**: Kurikulum, Guru, Siswa
- **Middleware**: Route protection at application level
- **API Security**: Endpoint-level authorization checks
- **Audit Logging**: All access attempts logged

#### 4.1.3 Data Protection
- **Encryption**: AES-256 for sensitive data at rest
- **Transit**: TLS 1.3 for all communications
- **Input Validation**: Zod schema validation
- **SQL Injection Prevention**: Drizzle ORM parameterized queries
- **XSS Protection**: Content Security Policy headers

#### 4.1.4 Security Monitoring
- **Logging**: Comprehensive audit trails
- **Monitoring**: Real-time security event monitoring
- **Incident Response**: Security breach procedures
- **Penetration Testing**: Regular security assessments
- **Compliance**: Educational data privacy standards

### 4.2 Performance Requirements

#### 4.2.1 Response Time Targets
- **API Responses**: 95% < 500ms (non-reporting endpoints)
- **Page Load**: 90% < 2 seconds (first paint)
- **Database Queries**: Optimized for < 100ms average
- **File Upload**: < 30 seconds for 10MB files
- **Real-time Updates**: < 100ms latency

#### 4.2.2 Scalability Requirements
- **Concurrent Users**: 1,000+ simultaneous users
- **Database**: Support for 100,000+ records
- **File Storage**: Scalable to 1TB+ with AWS S3
- **Load Balancing**: Horizontal scaling capability
- **Caching**: Redis for frequently accessed data

#### 4.2.3 Caching Strategy
- **Application Cache**: In-memory for session data
- **Database Cache**: Redis for query results
- **CDN**: Static assets cached at edge
- **Browser Cache**: Appropriate cache headers
- **Cache Invalidation**: Smart cache busting

### 4.3 Reliability Requirements

#### 4.3.1 Availability
- **Uptime Target**: 99.9% availability
- **Maintenance Windows**: Planned downtime < 4 hours/month
- **Disaster Recovery**: RPO < 1 hour, RTO < 4 hours
- **Backup Strategy**: Daily automated backups
- **Failover**: Automatic failover mechanisms

#### 4.3.2 Error Handling
- **Graceful Degradation**: Core functions remain available
- **Error Recovery**: Automatic retry mechanisms
- **User Feedback**: Clear error messages
- **Logging**: Comprehensive error logging
- **Monitoring**: Real-time error tracking

### 4.4 Usability Requirements

#### 4.4.1 User Interface
- **Responsive Design**: Mobile-first approach
- **Accessibility**: WCAG 2.1 AA compliance
- **Theme Support**: Light/dark mode toggle
- **Browser Support**: Modern browsers (last 2 versions)
- **Internationalization**: UTF-8 support for Bahasa Indonesia

#### 4.4.2 User Experience
- **Navigation**: Intuitive menu structure
- **Onboarding**: User guidance and help
- **Feedback**: Action confirmations and status updates
- **Performance**: Perceived performance optimization
- **Error Prevention**: Validation and confirmation dialogs

---

## 5. Data Model

### 5.1 Core Entities

#### 5.1.1 User Management
```typescript
// Users Table
interface User {
  id: string
  email: string
  passwordHash: string
  role: 'kurikulum' | 'guru' | 'siswa'
  profile: UserProfile
  isActive: boolean
  createdAt: Date
  updatedAt: Date
}

interface UserProfile {
  firstName: string
  lastName: string
  phone?: string
  avatar?: string
  preferences: UserPreferences
}
```

#### 5.1.2 Academic Structure
```typescript
// Classes Table
interface Class {
  id: string
  name: string
  gradeLevel: number
  academicYear: string
  homeroomTeacherId: string
  maxStudents: number
  isActive: boolean
}

// Subjects Table
interface Subject {
  id: string
  code: string
  name: string
  description: string
  credits: number
  isActive: boolean
}

// Teachers-Subjects Assignment
interface TeacherSubject {
  teacherId: string
  subjectId: string
  classId: string
  academicYear: string
  semester: number
}
```

#### 5.1.3 Student Management
```typescript
// Students Table
interface Student {
  id: string
  userId: string
  nis: string // Nomor Induk Siswa
  classId: string
  enrollmentDate: Date
  parentInfo: ParentInfo
  isActive: boolean
}

interface ParentInfo {
  fatherName?: string
  motherName?: string
  guardianName?: string
  phone?: string
  email?: string
}
```

#### 5.1.4 Grading System
```typescript
// Grades Table
interface Grade {
  id: string
  studentId: string
  teacherId: string
  subjectId: string
  assessmentType: 'daily' | 'midterm' | 'final' | 'assignment'
  score: number
  maxScore: number
  assessmentDate: Date
  semester: number
  academicYear: string
  remarks?: string
}

// Grade Categories
interface GradeCategory {
  id: string
  name: string
  weight: number
  isActive: boolean
}
```

#### 5.1.5 Scheduling
```typescript
// Schedules Table
interface Schedule {
  id: string
  classId: string
  teacherId: string
  subjectId: string
  dayOfWeek: number // 1-7 (Monday-Sunday)
  startTime: string
  endTime: string
  room: string
  semester: number
  academicYear: string
  isActive: boolean
}
```

#### 5.1.6 Question Bank
```typescript
// Bank Soal Table
interface QuestionBank {
  id: string
  teacherId: string
  subjectId: string
  question: string
  questionType: 'multiple_choice' | 'essay' | 'true_false'
  options?: string[] // For multiple choice
  correctAnswer: string
  difficulty: 'easy' | 'medium' | 'hard'
  topic: string
  tags: string[]
  isActive: boolean
  createdAt: Date
}
```

#### 5.1.7 Attendance
```typescript
// Attendance Records
interface Attendance {
  id: string
  studentId: string
  classId: string
  subjectId: string
  teacherId: string
  date: Date
  status: 'present' | 'absent' | 'late' | 'excused'
  remarks?: string
  recordedAt: Date
}
```

### 5.2 Database Schema

#### 5.2.1 Primary Tables
```sql
-- Users and Profiles
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) CHECK (role IN ('kurikulum', 'guru', 'siswa')) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Academic Structure
CREATE TABLE classes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    grade_level INTEGER NOT NULL,
    academic_year VARCHAR(9) NOT NULL,
    homeroom_teacher_id UUID REFERENCES users(id),
    max_students INTEGER DEFAULT 40,
    is_active BOOLEAN DEFAULT true
);

CREATE TABLE subjects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    credits INTEGER DEFAULT 1,
    is_active BOOLEAN DEFAULT true
);

-- Students
CREATE TABLE students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID UNIQUE REFERENCES users(id),
    nis VARCHAR(20) UNIQUE NOT NULL,
    class_id UUID REFERENCES classes(id),
    enrollment_date DATE NOT NULL,
    is_active BOOLEAN DEFAULT true
);
```

#### 5.2.2 Relationships and Indexes
```sql
-- Teacher-Subject Assignments
CREATE TABLE teacher_subjects (
    teacher_id UUID REFERENCES users(id),
    subject_id UUID REFERENCES subjects(id),
    class_id UUID REFERENCES classes(id),
    academic_year VARCHAR(9),
    semester INTEGER,
    PRIMARY KEY (teacher_id, subject_id, class_id, academic_year, semester)
);

-- Grades
CREATE TABLE grades (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID REFERENCES students(id),
    teacher_id UUID REFERENCES users(id),
    subject_id UUID REFERENCES subjects(id),
    assessment_type VARCHAR(20) NOT NULL,
    score DECIMAL(5,2) NOT NULL,
    max_score DECIMAL(5,2) NOT NULL,
    assessment_date DATE NOT NULL,
    semester INTEGER NOT NULL,
    academic_year VARCHAR(9) NOT NULL,
    remarks TEXT
);

-- Performance Indexes
CREATE INDEX idx_students_class_id ON students(class_id);
CREATE INDEX idx_grades_student_id ON grades(student_id);
CREATE INDEX idx_grades_subject_id ON grades(subject_id);
CREATE INDEX idx_schedules_class_id ON schedules(class_id);
CREATE INDEX idx_attendance_student_date ON attendance(student_id, date);
```

### 5.3 Data Storage Strategy

#### 5.3.1 Relational Data (PostgreSQL)
- User accounts and authentication data
- Academic records and grades
- Scheduling information
- Attendance records
- System configuration

#### 5.3.2 File Storage (AWS S3)
- Profile pictures and avatars
- Lesson plans (RPP/SILABUS)
- Learning modules and materials
- Assignment submissions
- System-generated reports

#### 5.3.3 Cache Storage (Redis)
- User session data
- Frequently accessed schedules
- Grade calculation results
- API response caching
- Real-time data for WebSocket connections

---

## 6. API Design

### 6.1 API Architecture

#### 6.1.1 RESTful API Design
- **Base URL**: `/api/v1`
- **Content-Type**: `application/json`
- **Authentication**: Bearer token (JWT) or session cookies
- **Rate Limiting**: 100 requests per minute per user
- **Pagination**: Cursor-based for large datasets
- **Versioning**: URL versioning strategy

#### 6.1.2 Response Format
```typescript
interface APIResponse<T> {
  success: boolean
  data?: T
  error?: {
    code: string
    message: string
    details?: any
  }
  meta?: {
    pagination?: PaginationMeta
    timestamp: string
    requestId: string
  }
}
```

### 6.2 Authentication Endpoints

#### 6.2.1 Session Management
```typescript
// Sign In
POST /api/auth/signin
{
  email: string
  password: string
}

// Sign Out
POST /api/auth/signout

// Get Current User
GET /api/auth/me

// Refresh Session
POST /api/auth/refresh
```

#### 6.2.2 Role-Based Access
```typescript
// Middleware for route protection
interface AuthMiddleware {
  requires: ('kurikulum' | 'guru' | 'siswa')[]
  optional?: boolean
}
```

### 6.3 Administrative API (Kurikulum)

#### 6.3.1 User Management
```typescript
// List Users
GET /api/admin/users?role=guru&page=1&limit=20

// Create User
POST /api/admin/users
{
  email: string
  password: string
  role: 'guru' | 'siswa'
  profile: UserProfile
}

// Update User
PUT /api/admin/users/:id
{
  email?: string
  profile?: Partial<UserProfile>
  isActive?: boolean
}

// Delete User
DELETE /api/admin/users/:id
```

#### 6.3.2 Schedule Management
```typescript
// Create Schedule
POST /api/admin/schedules
{
  classId: string
  teacherId: string
  subjectId: string
  dayOfWeek: number
  startTime: string
  endTime: string
  room: string
}

// Get Schedule Conflicts
POST /api/admin/schedules/conflicts
{
  teacherId: string
  dayOfWeek: number
  startTime: string
  endTime: string
  excludeScheduleId?: string
}
```

### 6.4 Teacher API (Guru)

#### 6.4.1 Grade Management
```typescript
// Submit Grades
POST /api/teacher/grades
{
  studentIds: string[]
  subjectId: string
  assessmentType: string
  scores: { studentId: string, score: number }[]
  assessmentDate: string
  maxScore: number
}

// Get Class Grades
GET /api/teacher/grades/:classId?subjectId=&semester=
```

#### 6.4.2 Content Management
```typescript
// Upload Content
POST /api/teacher/content/upload
Content-Type: multipart/form-data
{
  file: File
  metadata: {
    title: string
    description: string
    subjectId: string
    type: 'lesson_plan' | 'module' | 'material'
  }
}
```

### 6.5 Student API (Siswa)

#### 6.5.1 Schedule and Grades
```typescript
// Get Student Schedule
GET /api/student/schedule?week=2024-01-15

// Get Student Grades
GET /api/student/grades?semester=1&academicYear=2024/2025

// Get Progress Report
GET /api/student/report?format=pdf
```

### 6.6 Real-Time Communication (WebSocket)

#### 6.6.1 Chat Implementation
```typescript
// WebSocket Endpoints
ws://localhost:3000/api/chat/consultation
ws://localhost:3000/api/discussion/realtime

// Message Format
interface ChatMessage {
  id: string
  senderId: string
  receiverId?: string
  roomId?: string
  content: string
  timestamp: Date
  type: 'text' | 'file' | 'system'
}
```

### 6.7 Error Handling

#### 6.7.1 Error Codes
```typescript
enum ErrorCodes {
  UNAUTHORIZED = 'UNAUTHORIZED',
  FORBIDDEN = 'FORBIDDEN',
  NOT_FOUND = 'NOT_FOUND',
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  CONFLICT = 'CONFLICT',
  INTERNAL_ERROR = 'INTERNAL_ERROR',
  RATE_LIMITED = 'RATE_LIMITED'
}
```

#### 6.7.2 Validation
```typescript
// Request Validation Schemas (Zod)
const CreateUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  role: z.enum(['kurikulum', 'guru', 'siswa']),
  profile: z.object({
    firstName: z.string().min(1),
    lastName: z.string().min(1),
    phone: z.string().optional()
  })
})
```

---

## 7. Security Architecture

### 7.1 Authentication Implementation

#### 7.1.1 Better Auth Configuration
```typescript
// auth.config.ts
import { betterAuth } from "better-auth"
import { drizzleAdapter } from "better-auth/adapters/drizzle"
import { db } from "./db"

export const auth = betterAuth({
  database: drizzleAdapter(db, {
    provider: "pg",
    schema: {
      users: users,
      sessions: sessions,
      accounts: accounts
    }
  }),
  emailAndPassword: {
    enabled: true,
    requireEmailVerification: true
  },
  session: {
    expiresIn: 60 * 60, // 1 hour
    updateAge: 60 * 5 // 5 minutes
  },
  socialProviders: {
    // Future: Google, Microsoft SSO
  }
})
```

#### 7.1.2 Session Management
```typescript
// middleware.ts
import { authMiddleware } from "better-auth/middleware"

export default authMiddleware((req) => {
  // Session validation and role checking
  if (req.nextUrl.pathname.startsWith('/admin')) {
    return auth.redirectIfAuthenticated(req, '/kurikulum/dashboard')
  }
  
  if (req.nextUrl.pathname.startsWith('/guru')) {
    return auth.redirectIfNotRole(req, 'guru', '/sign-in')
  }
  
  if (req.nextUrl.pathname.startsWith('/siswa')) {
    return auth.redirectIfNotRole(req, 'siswa', '/sign-in')
  }
})
```

### 7.2 Role-Based Access Control

#### 7.2.1 Authorization Middleware
```typescript
// authorization.ts
export function authorize(roles: string[]) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const session = await auth.api.getSession({
      headers: req.headers
    })
    
    if (!session || !roles.includes(session.user.role)) {
      return res.status(403).json({
        success: false,
        error: {
          code: 'FORBIDDEN',
          message: 'Insufficient permissions'
        }
      })
    }
    
    req.user = session.user
    next()
  }
}
```

#### 7.2.2 API Route Protection
```typescript
// api/admin/users/route.ts
export async function GET(req: Request) {
  const session = await auth.api.getSession({ headers: req.headers })
  
  if (!session || session.user.role !== 'kurikulum') {
    return NextResponse.json(
      { error: { code: 'FORBIDDEN', message: 'Admin access required' } },
      { status: 403 }
    )
  }
  
  // Proceed with admin logic
}
```

### 7.3 Data Protection

#### 7.3.1 Encryption Implementation
```typescript
// encryption.ts
import crypto from 'crypto'

class DataEncryption {
  private algorithm = 'aes-256-gcm'
  private key = Buffer.from(process.env.ENCRYPTION_KEY!, 'hex')
  
  encrypt(text: string): { encrypted: string; iv: string; tag: string } {
    const iv = crypto.randomBytes(16)
    const cipher = crypto.createCipher(this.algorithm, this.key, iv)
    
    let encrypted = cipher.update(text, 'utf8', 'hex')
    encrypted += cipher.final('hex')
    
    const tag = cipher.getAuthTag()
    
    return {
      encrypted,
      iv: iv.toString('hex'),
      tag: tag.toString('hex')
    }
  }
  
  decrypt(encryptedData: string, iv: string, tag: string): string {
    const decipher = crypto.createDecipher(this.algorithm, this.key, Buffer.from(iv, 'hex'))
    decipher.setAuthTag(Buffer.from(tag, 'hex'))
    
    let decrypted = decipher.update(encryptedData, 'hex', 'utf8')
    decrypted += decipher.final('utf8')
    
    return decrypted
  }
}
```

#### 7.3.2 Input Validation
```typescript
// validation.ts
import { z } from 'zod'

export const schemas = {
  createUser: z.object({
    email: z.string().email().max(255),
    password: z.string().min(8).regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
    role: z.enum(['kurikulum', 'guru', 'siswa']),
    profile: z.object({
      firstName: z.string().min(1).max(100),
      lastName: z.string().min(1).max(100),
      phone: z.string().regex(/^\+?[\d\s-()]+$/).optional()
    })
  }),
  
  gradeInput: z.object({
    studentIds: z.array(z.string().uuid()),
    subjectId: z.string().uuid(),
    assessmentType: z.enum(['daily', 'midterm', 'final', 'assignment']),
    scores: z.array(z.object({
      studentId: z.string().uuid(),
      score: z.number().min(0).max(100)
    })),
    assessmentDate: z.string().datetime(),
    maxScore: z.number().min(1).max(100)
  })
}
```

### 7.4 Security Monitoring

#### 7.4.1 Audit Logging
```typescript
// audit.ts
interface AuditLog {
  id: string
  userId: string
  action: string
  resource: string
  resourceId?: string
  ipAddress: string
  userAgent: string
  timestamp: Date
  details?: Record<string, any>
}

export async function logAuditEvent(event: Omit<AuditLog, 'id' | 'timestamp'>) {
  await db.insert(auditLogs).values({
    ...event,
    timestamp: new Date(),
    id: crypto.randomUUID()
  })
}

// Usage example
await logAuditEvent({
  userId: session.user.id,
  action: 'CREATE',
  resource: 'USER',
  resourceId: newUserId,
  ipAddress: req.ip,
  userAgent: req.headers.get('user-agent') || 'Unknown'
})
```

#### 7.4.2 Security Headers
```typescript
// security.ts
export const securityHeaders = {
  'Content-Security-Policy': [
    "default-src 'self'",
    "script-src 'self' 'unsafe-inline' 'unsafe-eval'",
    "style-src 'self' 'unsafe-inline'",
    "img-src 'self' data: https:",
    "font-src 'self'",
    "connect-src 'self' wss:"
  ].join('; '),
  'X-Frame-Options': 'DENY',
  'X-Content-Type-Options': 'nosniff',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Permissions-Policy': 'camera=(), microphone=(), geolocation=()'
}
```

---

## 8. Deployment Architecture

### 8.1 Container Strategy

#### 8.1.1 Docker Configuration
```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Dependencies
FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Build
FROM base AS builder
WORKDIR /app
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN npm run build

# Runtime
FROM base AS runner
WORKDIR /app
ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

#### 8.1.2 Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/ems
      - REDIS_URL=redis://redis:6379
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
    depends_on:
      - db
      - redis
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=ems
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

### 8.2 Kubernetes Deployment

#### 8.2.1 Application Deployment
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ems-app
  labels:
    app: ems-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ems-app
  template:
    metadata:
      labels:
        app: ems-app
    spec:
      containers:
      - name: ems-app
        image: ems-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: ems-secrets
              key: database-url
        - name: REDIS_URL
          value: "redis://redis-service:6379"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

#### 8.2.2 Service Configuration
```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ems-service
spec:
  selector:
    app: ems-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer

---
apiVersion: v1
kind: Service
metadata:
  name: ems-service-internal
spec:
  selector:
    app: ems-app
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  type: ClusterIP
```

### 8.3 CI/CD Pipeline

#### 8.3.1 GitHub Actions Workflow
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linting
      run: npm run lint
    
    - name: Run type checking
      run: npm run type-check
    
    - name: Run tests
      run: npm run test
      env:
        DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}
    
    - name: Run E2E tests
      run: npm run test:e2e

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Login to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ secrets.REGISTRY_URL }}
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ secrets.REGISTRY_URL }}/ems-app:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to Kubernetes
      run: |
        # Update image tag in deployment
        kubectl set image deployment/ems-app ems-app=${{ secrets.REGISTRY_URL }}/ems-app:${{ github.sha }}
        kubectl rollout status deployment/ems-app
```

### 8.4 Infrastructure as Code

#### 8.4.1 Terraform Configuration
```hcl
# infrastructure/main.tf
resource "aws_eks_cluster" "ems_cluster" {
  name     = "ems-cluster"
  role_arn = aws_iam_role.eks_cluster.arn
  version  = "1.28"

  vpc_config {
    subnet_ids = [
      aws_subnet.private_1.id,
      aws_subnet.private_2.id,
      aws_subnet.public_1.id,
      aws_subnet.public_2.id
    ]
  }
}

resource "aws_rds_instance" "ems_database" {
  identifier     = "ems-db"
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.t3.micro"
  
  allocated_storage     = 20
  max_allocated_storage = 100
  storage_encrypted     = true
  
  db_name  = "ems"
  username = var.db_username
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.ems.name
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
}

resource "aws_elasticache_subnet_group" "redis" {
  name       = "ems-cache-subnet"
  subnet_ids = [aws_subnet.private_1.id, aws_subnet.private_2.id]
}

resource "aws_elasticache_cluster" "redis" {
  cluster_id           = "ems-redis"
  engine               = "redis"
  node_type            = "cache.t3.micro"
  num_cache_nodes      = 1
  parameter_group_name = "default.redis7"
  subnet_group_name    = aws_elasticache_subnet_group.redis.name
  security_group_ids   = [aws_security_group.redis.id]
}
```

### 8.5 Monitoring and Logging

#### 8.5.1 Application Monitoring
```typescript
// monitoring.ts
import { createPrometheusMetrics } from 'next-prometheus'

export const prometheusMetrics = createPrometheusMetrics({
  prefix: 'ems_',
  labels: ['method', 'route', 'status_code'],
  
  metrics: {
    http_requests_total: {
      type: 'counter',
      help: 'Total number of HTTP requests'
    },
    http_request_duration_seconds: {
      type: 'histogram',
      help: 'HTTP request duration in seconds'
    },
    active_sessions_total: {
      type: 'gauge',
      help: 'Number of active user sessions'
    },
    database_connections_active: {
      type: 'gauge',
      help: 'Number of active database connections'
    }
  }
})

// Health check endpoint
export async function GET() {
  return NextResponse.json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.npm_package_version,
    uptime: process.uptime()
  })
}
```

#### 8.5.2 Error Tracking
```typescript
// error-tracking.ts
import * as Sentry from '@sentry/nextjs'

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,
  beforeSend(event) {
    // Filter out sensitive information
    if (event.request?.cookies) {
      delete event.request.cookies
    }
    return event
  }
})

export class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public code: string,
    public isOperational: boolean = true
  ) {
    super(message)
    Error.captureStackTrace(this, this.constructor)
  }
}
```

---

## 9. Testing Strategy

### 9.1 Test Pyramid

#### 9.1.1 Unit Tests
```typescript
// __tests__/utils/validation.test.ts
import { z } from 'zod'
import { schemas } from '@/lib/validation'

describe('Validation Schemas', () => {
  test('should validate valid user creation data', () => {
    const validData = {
      email: 'teacher@school.edu',
      password: 'SecurePass123',
      role: 'guru',
      profile: {
        firstName: 'John',
        lastName: 'Doe'
      }
    }
    
    expect(() => schemas.createUser.parse(validData)).not.toThrow()
  })
  
  test('should reject invalid email', () => {
    const invalidData = {
      email: 'invalid-email',
      password: 'SecurePass123',
      role: 'guru',
      profile: { firstName: 'John', lastName: 'Doe' }
    }
    
    expect(() => schemas.createUser.parse(invalidData)).toThrow()
  })
})
```

#### 9.1.2 Integration Tests
```typescript
// __tests__/api/auth.test.ts
import { createMocks } from 'node-mocks-http'
import handler from '@/pages/api/auth/signin'

describe('/api/auth/signin', () => {
  test('should authenticate valid credentials', async () => {
    const { req, res } = createMocks({
      method: 'POST',
      body: {
        email: 'test@school.edu',
        password: 'testpassword'
      }
    })
    
    await handler(req, res)
    
    expect(res._getStatusCode()).toBe(200)
    expect(res._getJSONData()).toHaveProperty('success', true)
  })
  
  test('should reject invalid credentials', async () => {
    const { req, res } = createMocks({
      method: 'POST',
      body: {
        email: 'test@school.edu',
        password: 'wrongpassword'
      }
    })
    
    await handler(req, res)
    
    expect(res._getStatusCode()).toBe(401)
    expect(res._getJSONData()).toHaveProperty('success', false)
  })
})
```

#### 9.1.3 End-to-End Tests
```typescript
// e2e/student-workflow.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Student Workflow', () => {
  test('student can view grades and schedule', async ({ page }) => {
    // Login as student
    await page.goto('/sign-in')
    await page.fill('[data-testid=email]', 'student@school.edu')
    await page.fill('[data-testid=password]', 'studentpass')
    await page.click('[data-testid=signin-button]')
    
    // Should redirect to student dashboard
    await expect(page).toHaveURL('/siswa/dashboard')
    
    // Navigate to grades
    await page.click('[data-testid=nav-grades]')
    await expect(page.locator('[data-testid=grades-table]')).toBeVisible()
    
    // Navigate to schedule
    await page.click('[data-testid=nav-schedule]')
    await expect(page.locator('[data-testid=schedule-table]')).toBeVisible()
  })
})
```

### 9.2 Performance Testing

#### 9.2.1 Load Testing
```typescript
// load-tests/api-load-test.js
import http from 'k6/http'
import { check, sleep } from 'k6'

export let options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up to 100 users
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 200 }, // Ramp up to 200 users
    { duration: '5m', target: 200 }, // Stay at 200 users
    { duration: '2m', target: 0 },   // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    http_req_failed: ['rate<0.01'],   // Less than 1% failed requests
  },
}

export default function () {
  let response = http.get('http://localhost:3000/api/student/grades', {
    headers: {
      'Authorization': 'Bearer ' + __ENV.AUTH_TOKEN
    }
  })
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  })
  
  sleep(1)
}
```

### 9.3 Security Testing

#### 9.3.1 OWASP Security Testing
```typescript
// security/sql-injection.test.ts
import { createMocks } from 'node-mocks-http'
import handler from '@/pages/api/admin/users'

describe('SQL Injection Protection', () => {
  test('should prevent SQL injection in user search', async () => {
    const maliciousInput = "'; DROP TABLE users; --"
    
    const { req, res } = createMocks({
      method: 'GET',
      query: { search: maliciousInput }
    })
    
    await handler(req, res)
    
    // Should return 400 Bad Request for malicious input
    expect(res._getStatusCode()).toBe(400)
    
    // Database should still be intact
    const users = await db.select().from(usersTable)
    expect(users.length).toBeGreaterThan(0)
  })
})
```

---

## 10. Maintenance and Support

### 10.1 Monitoring Strategy

#### 10.1.1 Application Performance Monitoring
- **Metrics Collection**: Prometheus + Grafana
- **Error Tracking**: Sentry
- **Log Aggregation**: ELK Stack
- **Health Checks**: Custom health endpoints
- **Performance Budgets**: Core Web Vitals monitoring

#### 10.1.2 Infrastructure Monitoring
```yaml
# monitoring/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'ems-app'
    static_configs:
      - targets: ['ems-service:3000']
    metrics_path: '/api/metrics'
    scrape_interval: 30s

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
```

### 10.2 Backup and Recovery

#### 10.2.1 Database Backup Strategy
```bash
#!/bin/bash
# scripts/backup-database.sh

BACKUP_DIR="/backups/postgres"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="ems"

# Create backup directory
mkdir -p $BACKUP_DIR

# Perform database backup
pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME | gzip > "$BACKUP_DIR/ems_backup_$DATE.sql.gz"

# Remove backups older than 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

# Upload to S3
aws s3 cp "$BACKUP_DIR/ems_backup_$DATE.sql.gz" "s3://ems-backups/database/"

echo "Backup completed: ems_backup_$DATE.sql.gz"
```

#### 10.2.2 Disaster Recovery Plan
1. **RTO (Recovery Time Objective)**: 4 hours
2. **RPO (Recovery Point Objective)**: 1 hour
3. **Backup Verification**: Weekly restore tests
4. **Failover Testing**: Monthly disaster recovery drills
5. **Documentation**: Recovery runbooks updated quarterly

### 10.3 Scaling Strategy

#### 10.3.1 Horizontal Scaling
```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ems-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ems-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

#### 10.3.2 Database Scaling
- **Read Replicas**: For read-heavy operations
- **Connection Pooling**: PgBouncer for connection management
- **Query Optimization**: Regular performance tuning
- **Index Strategy**: Optimized for common query patterns

### 10.4 Compliance and Governance

#### 10.4.1 Data Privacy Compliance
- **GDPR Principles**: Data minimization, purpose limitation
- **Educational Data Privacy**: Compliance with local regulations
- **Data Retention**: Automated cleanup of expired data
- **Access Logs**: Complete audit trail
- **Data Encryption**: End-to-end encryption for sensitive data

#### 10.4.2 Security Compliance
- **OWASP Top 10**: Regular security assessments
- **Penetration Testing**: Quarterly external security tests
- **Vulnerability Scanning**: Weekly automated scans
- **Security Training**: Annual security awareness training
- **Incident Response**: 24/7 security incident response

---

## 11. Future Enhancements

### 11.1 Phase 2 Features

#### 11.1.1 Advanced Communication
- **Real-time Notifications**: Push notifications for important updates
- **Video Conferencing**: Integrated video calls for remote learning
- **Mobile Applications**: Native iOS and Android apps
- **Parent Portal**: Dedicated interface for parents/guardians

#### 11.1.2 Enhanced Analytics
- **Learning Analytics**: AI-powered learning pattern analysis
- **Predictive Analytics**: Early warning system for at-risk students
- **Performance Dashboards**: Advanced visualization tools
- **Custom Reports**: Flexible report generation system

#### 11.1.3 Integration Capabilities
- **LMS Integration**: Connect with external learning platforms
- **SIS Integration**: Student Information System synchronization
- **Payment Gateway**: Online fee payment processing
- **Library System**: Digital library management

### 11.2 Technical Improvements

#### 11.2.1 Architecture Evolution
- **Microservices**: Decompose into specialized services
- **Event-Driven Architecture**: Implement message queues
- **GraphQL API**: Optional GraphQL endpoint for flexible queries
- **Edge Computing**: CDN and edge processing

#### 11.2.2 Performance Optimizations
- **Database Sharding**: Horizontal database scaling
- **Advanced Caching**: Multi-layer caching strategy
- **CDN Integration**: Global content delivery
- **Lazy Loading**: Progressive content loading

---

## 12. Appendix

### 12.1 Technical Glossary

| Term | Definition |
|------|------------|
| **RBAC** | Role-Based Access Control - security approach for restricting system access |
| **JWT** | JSON Web Token - compact URL-safe means of representing claims |
| **ORM** | Object-Relational Mapping - technique for converting data between incompatible systems |
| **API** | Application Programming Interface - set of protocols for building software |
| **CI/CD** | Continuous Integration/Continuous Deployment - automated software delivery pipeline |
| **Docker** | Container platform for developing, shipping, and running applications |
| **Kubernetes** | Container orchestration platform for automating deployment and scaling |

### 12.2 Environment Variables

```bash
# .env.example
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/ems"
REDIS_URL="redis://localhost:6379"

# Authentication
NEXTAUTH_SECRET="your-secret-key"
NEXTAUTH_URL="http://localhost:3000"

# File Storage
AWS_ACCESS_KEY_ID="your-access-key"
AWS_SECRET_ACCESS_KEY="your-secret-key"
AWS_S3_BUCKET="ems-files"

# External Services
SENTRY_DSN="your-sentry-dsn"
STRIPE_SECRET_KEY="your-stripe-key"

# Application
NODE_ENV="production"
LOG_LEVEL="info"
```

### 12.3 API Reference Summary

| Endpoint | Method | Description | Authentication |
|----------|--------|-------------|----------------|
| `/api/auth/signin` | POST | User authentication | Public |
| `/api/admin/users` | GET/POST | User management | Kurikulum |
| `/api/teacher/grades` | POST | Grade submission | Guru |
| `/api/student/schedule` | GET | View schedule | Siswa |
| `/api/health` | GET | Health check | Public |

### 12.4 Database Schema Summary

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `users` | Authentication | email, password_hash, role |
| `students` | Student data | user_id, nis, class_id |
| `teachers` | Teacher data | user_id, employee_id |
| `classes` | Class management | name, grade_level, academic_year |
| `subjects` | Subject catalog | code, name, credits |
| `grades` | Grade records | student_id, subject_id, score |
| `schedules` | Time scheduling | class_id, teacher_id, subject_id |
| `attendance` | Attendance tracking | student_id, date, status |

---

## 13. Conclusion

This Technical Specification Document provides a comprehensive blueprint for the development, deployment, and maintenance of the Educational Management System (EMS). The system is designed with security, scalability, and usability as primary considerations, ensuring it can effectively serve the needs of all stakeholders in the educational environment.

The modular architecture and well-defined interfaces allow for future enhancements and integrations, while the comprehensive security measures ensure the protection of sensitive educational data. The performance and reliability requirements establish clear benchmarks for system quality and user experience.

Successful implementation of this specification will result in a robust, modern educational management platform that streamlines administrative processes, enhances teaching effectiveness, and improves student learning outcomes.

---

**Document Approval**

- **Technical Lead**: _______________________ Date: _________
- **Project Manager**: _______________________ Date: _________
- **System Architect**: _______________________ Date: _________
- **Security Officer**: _______________________ Date: _________

*This document is version-controlled and should be updated as requirements evolve during the development lifecycle.*