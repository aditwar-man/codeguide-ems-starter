# Educational Management System (EMS) - Database Schema Design

## Document Information

- **Document Version**: 1.0
- **Date:** October 19, 2025
- **Author:** EMS Development Team
- **Status:** Draft
- **Database:** PostgreSQL 15+
- **ORM:** Drizzle ORM
- **Schema Version:** v1.0

---

## Table of Contents

1. [Database Overview](#1-database-overview)
2. [Schema Design Principles](#2-schema-design-principles)
3. [Core Tables](#3-core-tables)
4. [Academic Tables](#4-academic-tables)
5. [Assessment Tables](#5-assessment-tables)
6. [Communication Tables](#6-communication-tables)
7. [System Tables](#7-system-tables)
8. [Relationships and Constraints](#8-relationships-and-constraints)
9. [Indexes and Performance](#9-indexes-and-performance)
10. [Migration Strategy](#10-migration-strategy)
11. [Seed Data](#11-seed-data)
12. [Data Storage Strategy](#12-data-storage-strategy)

---

## 1. Database Overview

### 1.1 Database Architecture

The EMS database uses PostgreSQL as the primary relational database with the following characteristics:

- **Database Name:** `ems_db`
- **Character Set:** UTF-8
- **Collation:** `en_US.UTF-8`
- **Timezone:** UTC
- **Connection Pooling:** PgBouncer
- **Backup Strategy:** Daily full backups + Point-in-time recovery

### 1.2 Schema Organization

```
ems_db
├── public/                 # Main application schema
│   ├── auth/              # Authentication tables
│   ├── academic/          # Academic structure tables
│   ├── assessment/        # Grades and assessment tables
│   ├── communication/     # Chat and discussion tables
│   ├── content/           # File and content management
│   └── system/            # System and audit tables
├── audit/                 # Read-only audit logs
└── analytics/             # Data warehouse for reporting
```

### 1.3 Naming Conventions

- **Tables:** `snake_case` (e.g., `user_profiles`, `grade_records`)
- **Columns:** `snake_case` (e.g., `created_at`, `user_id`)
- **Primary Keys:** `id` (UUID)
- **Foreign Keys:** `{table}_id` (e.g., `user_id`, `class_id`)
- **Indexes:** `idx_{table}_{column(s)}` (e.g., `idx_users_email`)
- **Constraints:** `ck_{table}_{condition}` (e.g., `ck_users_role`)

---

## 2. Schema Design Principles

### 2.1 Design Considerations

1. **Data Integrity:** Foreign key constraints and validation rules
2. **Performance:** Proper indexing and query optimization
3. **Scalability:** Horizontal scaling through sharding capabilities
4. **Security:** Row-level security for multi-tenant access
5. **Auditability:** Comprehensive audit trail for all data changes
6. **Flexibility:** Schema evolution support with minimal downtime

### 2.2 Data Types Strategy

| Data Type | Usage | Rationale |
|-----------|-------|-----------|
| `UUID` | Primary keys | Globally unique, non-sequential, secure |
| `VARCHAR` | Names, descriptions | Variable-length text with reasonable limits |
| `TEXT` | Long content | Large text blocks for descriptions, content |
| `TIMESTAMP WITH TIME ZONE` | Dates/times | Timezone-aware timestamps |
| `DECIMAL(10,2)` | Financial/grade values | Precise decimal calculations |
| `INTEGER` | Counts, identifiers | Whole numbers for counts and references |
| `BOOLEAN` | Flags | True/false values |
| `JSONB` | Flexible metadata | Structured key-value data |
| `ENUM` | Fixed categories | Defined value sets for consistency |

---

## 3. Core Tables

### 3.1 Authentication and User Management

#### 3.1.1 Users Table

```typescript
// db/schema/auth/users.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  timestamp, 
  boolean, 
  text 
} from 'drizzle-orm/pg-core'
import { relations } from 'drizzle-orm'

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  emailVerified: boolean('email_verified').default(false),
  passwordHash: varchar('password_hash', { length: 255 }).notNull(),
  role: varchar('role', { 
    length: 20, 
    enum: ['kurikulum', 'guru', 'siswa'] 
  }).notNull(),
  isActive: boolean('is_active').default(true),
  lastLoginAt: timestamp('last_login_at', { withTimezone: true }),
  passwordChangedAt: timestamp('password_changed_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const usersRelations = relations(users, ({ one, many }) => ({
  profile: one(userProfiles, {
    fields: [users.id],
    references: [userProfiles.userId],
  }),
  sessions: many(sessions),
  auditLogs: many(auditLogs),
  student: one(students, {
    fields: [users.id],
    references: [students.userId],
  }),
  teacher: one(teachers, {
    fields: [users.id],
    references: [teachers.userId],
  }),
  kurikulum: one(kurikulumStaff, {
    fields: [users.id],
    references: [kurikulumStaff.userId],
  }),
}))
```

#### 3.1.2 User Profiles Table

```typescript
// db/schema/auth/user-profiles.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  text, 
  timestamp,
  jsonb 
} from 'drizzle-orm/pg-core'

export const userProfiles = pgTable('user_profiles', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().unique().references(() => users.id),
  firstName: varchar('first_name', { length: 100 }).notNull(),
  lastName: varchar('last_name', { length: 100 }).notNull(),
  displayName: varchar('display_name', { length: 201 }), // firstName + lastName
  phone: varchar('phone', { length: 20 }),
  avatarUrl: varchar('avatar_url', { length: 500 }),
  bio: text('bio'),
  preferences: jsonb('preferences').$type<{
    theme?: 'light' | 'dark' | 'system'
    language?: string
    notifications?: {
      email?: boolean
      push?: boolean
      sms?: boolean
    }
  }>(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const userProfilesRelations = relations(userProfiles, ({ one }) => ({
  user: one(users, {
    fields: [userProfiles.userId],
    references: [users.id],
  }),
}))
```

#### 3.1.3 Sessions Table

```typescript
// db/schema/auth/sessions.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  timestamp, 
  text,
  boolean 
} from 'drizzle-orm/pg-core'

export const sessions = pgTable('sessions', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().references(() => users.id),
  sessionToken: varchar('session_token', { length: 255 }).notNull().unique(),
  accessToken: varchar('access_token', { length: 500 }),
  refreshToken: varchar('refresh_token', { length: 500 }),
  expiresAt: timestamp('expires_at', { withTimezone: true }).notNull(),
  ipAddress: varchar('ip_address', { length: 45 }),
  userAgent: text('user_agent'),
  isActive: boolean('is_active').default(true),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const sessionsRelations = relations(sessions, ({ one }) => ({
  user: one(users, {
    fields: [sessions.userId],
    references: [users.id],
  }),
}))
```

### 3.2 User Role-Specific Tables

#### 3.2.1 Students Table

```typescript
// db/schema/academic/students.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  timestamp, 
  boolean,
  jsonb,
  integer
} from 'drizzle-orm/pg-core'

export const students = pgTable('students', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().unique().references(() => users.id),
  nis: varchar('nis', { length: 20 }).notNull().unique(), // Nomor Induk Siswa
  nisn: varchar('nisn', { length: 20 }).unique(), // Nomor Induk Siswa Nasional
  classId: uuid('class_id').references(() => classes.id),
  enrollmentDate: timestamp('enrollment_date', { withTimezone: true }).notNull(),
  graduationDate: timestamp('graduation_date', { withTimezone: true }),
  isActive: boolean('is_active').default(true),
  parentInfo: jsonb('parent_info').$type<{
    fatherName?: string
    motherName?: string
    guardianName?: string
    parentPhone?: string
    parentEmail?: string
    address?: string
  }>(),
  medicalInfo: jsonb('medical_info').$type<{
    allergies?: string[]
    medications?: string[]
    emergencyContact?: string
    bloodType?: string
  }>(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const studentsRelations = relations(students, ({ one, many }) => ({
  user: one(users, {
    fields: [students.userId],
    references: [users.id],
  }),
  class: one(classes, {
    fields: [students.classId],
    references: [classes.id],
  }),
  grades: many(grades),
  attendance: many(attendance),
  savings: many(studentSavings),
}))
```

#### 3.2.2 Teachers Table

```typescript
// db/schema/academic/teachers.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  timestamp, 
  boolean,
  jsonb,
  text
} from 'drizzle-orm/pg-core'

export const teachers = pgTable('teachers', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().unique().references(() => users.id),
  employeeId: varchar('employee_id', { length: 20 }).notNull().unique(),
  nik: varchar('nik', { length: 20 }).unique(), // Nomor Induk Kependudukan
  specialization: varchar('specialization', { length: 100 }),
  qualification: varchar('qualification', { length: 100 }),
  hireDate: timestamp('hire_date', { withTimezone: true }).notNull(),
  isActive: boolean('is_active').default(true),
  isHomeroom: boolean('is_homeroom').default(false),
  bankInfo: jsonb('bank_info').$type<{
    bankName?: string
    accountNumber?: string
    accountName?: string
  }>(),
  emergencyContact: jsonb('emergency_contact').$type<{
    name?: string
    phone?: string
    relationship?: string
  }>(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const teachersRelations = relations(teachers, ({ one, many }) => ({
  user: one(users, {
    fields: [teachers.userId],
    references: [users.id],
  }),
  homeroomClass: one(classes, {
    fields: [teachers.id],
    references: [classes.homeroomTeacherId],
  }),
  subjects: many(teacherSubjects),
  grades: many(grades),
  content: many(contentFiles),
  questions: many(questionBank),
}))
```

#### 3.2.3 Kurikulum Staff Table

```typescript
// db/schema/academic/kurikulum-staff.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  timestamp, 
  boolean,
  text
} from 'drizzle-orm/pg-core'

export const kurikulumStaff = pgTable('kurikulum_staff', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().unique().references(() => users.id),
  employeeId: varchar('employee_id', { length: 20 }).notNull().unique(),
  position: varchar('position', { length: 100 }).notNull(),
  department: varchar('department', { length: 100 }),
  hireDate: timestamp('hire_date', { withTimezone: true }).notNull(),
  isActive: boolean('is_active').default(true),
  permissions: jsonb('permissions').$type<string[]>(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const kurikulumStaffRelations = relations(kurikulumStaff, ({ one }) => ({
  user: one(users, {
    fields: [kurikulumStaff.userId],
    references: [users.id],
  }),
}))
```

---

## 4. Academic Tables

### 4.1 Academic Structure

#### 4.1.1 Classes Table

```typescript
// db/schema/academic/classes.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  integer, 
  timestamp, 
  boolean,
  jsonb
} from 'drizzle-orm/pg-core'

export const classes = pgTable('classes', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 100 }).notNull(),
  gradeLevel: integer('grade_level').notNull(), // 1-12
  academicYear: varchar('academic_year', { length: 9 }).notNull(), // 2024/2025
  homeroomTeacherId: uuid('homeroom_teacher_id').references(() => teachers.id),
  room: varchar('room', { length: 20 }),
  capacity: integer('capacity').default(40),
  currentEnrollment: integer('current_enrollment').default(0),
  isActive: boolean('is_active').default(true),
  metadata: jsonb('metadata').$type<{
    department?: string
    program?: string
    shift?: string
  }>(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const classesRelations = relations(classes, ({ one, many }) => ({
  homeroomTeacher: one(teachers, {
    fields: [classes.homeroomTeacherId],
    references: [teachers.id],
  }),
  students: many(students),
  schedules: many(schedules),
  teacherSubjects: many(teacherSubjects),
}))
```

#### 4.1.2 Subjects Table

```typescript
// db/schema/academic/subjects.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  text, 
  integer, 
  timestamp, 
  boolean
} from 'drizzle-orm/pg-core'

export const subjects = pgTable('subjects', {
  id: uuid('id').primaryKey().defaultRandom(),
  code: varchar('code', { length: 20 }).notNull().unique(),
  name: varchar('name', { length: 100 }).notNull(),
  description: text('description'),
  credits: integer('credits').default(1),
  department: varchar('department', { length: 100 }),
  isActive: boolean('is_active').default(true),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const subjectsRelations = relations(subjects, ({ many }) => ({
  teacherSubjects: many(teacherSubjects),
  grades: many(grades),
  schedules: many(schedules),
  questions: many(questionBank),
  content: many(contentFiles),
}))
```

#### 4.1.3 Teacher-Subject Assignment Table

```typescript
// db/schema/academic/teacher-subjects.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  integer, 
  timestamp, 
  boolean
} from 'drizzle-orm/pg-core'

export const teacherSubjects = pgTable('teacher_subjects', {
  id: uuid('id').primaryKey().defaultRandom(),
  teacherId: uuid('teacher_id').notNull().references(() => teachers.id),
  subjectId: uuid('subject_id').notNull().references(() => subjects.id),
  classId: uuid('class_id').notNull().references(() => classes.id),
  academicYear: varchar('academic_year', { length: 9 }).notNull(),
  semester: integer('semester').notNull(), // 1 or 2
  isActive: boolean('is_active').default(true),
  assignedAt: timestamp('assigned_at', { withTimezone: true }).defaultNow(),
  assignedBy: uuid('assigned_by').references(() => kurikulumStaff.id),
}, (table) => ({
  // Composite unique constraint
  uniqueAssignment: {
    columns: [table.teacherId, table.subjectId, table.classId, table.academicYear, table.semester],
  },
}))

export const teacherSubjectsRelations = relations(teacherSubjects, ({ one }) => ({
  teacher: one(teachers, {
    fields: [teacherSubjects.teacherId],
    references: [teachers.id],
  }),
  subject: one(subjects, {
    fields: [teacherSubjects.subjectId],
    references: [subjects.id],
  }),
  class: one(classes, {
    fields: [teacherSubjects.classId],
    references: [classes.id],
  }),
  assignedByUser: one(kurikulumStaff, {
    fields: [teacherSubjects.assignedBy],
    references: [kurikulumStaff.id],
  }),
}))
```

### 4.2 Scheduling

#### 4.2.1 Schedules Table

```typescript
// db/schema/academic/schedules.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  integer, 
  timestamp, 
  boolean,
  text
} from 'drizzle-orm/pg-core'

export const schedules = pgTable('schedules', {
  id: uuid('id').primaryKey().defaultRandom(),
  classId: uuid('class_id').notNull().references(() => classes.id),
  teacherId: uuid('teacher_id').notNull().references(() => teachers.id),
  subjectId: uuid('subject_id').notNull().references(() => subjects.id),
  dayOfWeek: integer('day_of_week').notNull(), // 1-7 (Monday-Sunday)
  startTime: varchar('start_time', { length: 5 }).notNull(), // HH:MM format
  endTime: varchar('end_time', { length: 5 }).notNull(), // HH:MM format
  room: varchar('room', { length: 20 }),
  semester: integer('semester').notNull(),
  academicYear: varchar('academic_year', { length: 9 }).notNull(),
  isActive: boolean('is_active').default(true),
  notes: text('notes'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
}, (table) => ({
  // Prevent schedule conflicts
  uniqueSchedule: {
    columns: [table.classId, table.dayOfWeek, table.startTime, table.endTime, table.semester, table.academicYear],
  },
}))

export const schedulesRelations = relations(schedules, ({ one }) => ({
  class: one(classes, {
    fields: [schedules.classId],
    references: [classes.id],
  }),
  teacher: one(teachers, {
    fields: [schedules.teacherId],
    references: [teachers.id],
  }),
  subject: one(subjects, {
    fields: [schedules.subjectId],
    references: [subjects.id],
  }),
}))
```

#### 4.2.2 Schedule Changes Table

```typescript
// db/schema/academic/schedule-changes.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  timestamp, 
  boolean,
  text,
  jsonb
} from 'drizzle-orm/pg-core'

export const scheduleChanges = pgTable('schedule_changes', {
  id: uuid('id').primaryKey().defaultRandom(),
  originalScheduleId: uuid('original_schedule_id').references(() => schedules.id),
  newScheduleId: uuid('new_schedule_id').references(() => schedules.id),
  changeType: varchar('change_type', { 
    length: 20, 
    enum: ['substitution', 'cancellation', 'room_change', 'time_change'] 
  }).notNull(),
  reason: text('reason').notNull(),
  effectiveDate: timestamp('effective_date', { withTimezone: true }).notNull(),
  endDate: timestamp('end_date', { withTimezone: true }),
  isNotified: boolean('is_notified').default(false),
  approvedBy: uuid('approved_by').references(() => kurikulumStaff.id),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  metadata: jsonb('metadata').$type<{
    originalTeacherId?: string
    substituteTeacherId?: string
    originalRoom?: string
    newRoom?: string
  }>(),
})

export const scheduleChangesRelations = relations(scheduleChanges, ({ one }) => ({
  originalSchedule: one(schedules, {
    fields: [scheduleChanges.originalScheduleId],
    references: [schedules.id],
  }),
  newSchedule: one(schedules, {
    fields: [scheduleChanges.newScheduleId],
    references: [schedules.id],
  }),
  approvedByUser: one(kurikulumStaff, {
    fields: [scheduleChanges.approvedBy],
    references: [kurikulumStaff.id],
  }),
}))
```

---

## 5. Assessment Tables

### 5.1 Grading System

#### 5.1.1 Grade Categories Table

```typescript
// db/schema/assessment/grade-categories.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  integer, 
  timestamp, 
  boolean
} from 'drizzle-orm/pg-core'

export const gradeCategories = pgTable('grade_categories', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 50 }).notNull(),
  code: varchar('code', { length: 10 }).notNull().unique(),
  description: varchar('description', { length: 200 }),
  weight: integer('weight').notNull(), // Percentage weight (0-100)
  isActive: boolean('is_active').default(true),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

// Common grade categories
export const GRADE_CATEGORIES = {
  DAILY: 'daily',
  QUIZ: 'quiz',
  ASSIGNMENT: 'assignment',
  MIDTERM: 'midterm',
  FINAL: 'final',
  PROJECT: 'project',
  PRACTICAL: 'practical'
} as const
```

#### 5.1.2 Grades Table

```typescript
// db/schema/assessment/grades.ts
import { 
  pgTable, 
  uuid, 
  decimal, 
  timestamp, 
  boolean,
  text,
  integer
} from 'drizzle-orm/pg-core'

export const grades = pgTable('grades', {
  id: uuid('id').primaryKey().defaultRandom(),
  studentId: uuid('student_id').notNull().references(() => students.id),
  teacherId: uuid('teacher_id').notNull().references(() => teachers.id),
  subjectId: uuid('subject_id').notNull().references(() => subjects.id),
  categoryId: uuid('category_id').references(() => gradeCategories.id),
  classId: uuid('class_id').notNull().references(() => classes.id),
  assessmentType: varchar('assessment_type', { length: 20 }).notNull(),
  title: varchar('title', { length: 200 }).notNull(),
  description: text('description'),
  score: decimal('score', { precision: 5, scale: 2 }).notNull(),
  maxScore: decimal('max_score', { precision: 5, scale: 2 }).notNull(),
  percentage: decimal('percentage', { precision: 5, scale: 2 }), // Calculated field
  assessmentDate: timestamp('assessment_date', { withTimezone: true }).notNull(),
  semester: integer('semester').notNull(),
  academicYear: varchar('academic_year', { length: 9 }).notNull(),
  remarks: text('remarks'),
  isPublished: boolean('is_published').default(false),
  publishedAt: timestamp('published_at', { withTimezone: true }),
  submittedAt: timestamp('submitted_at', { withTimezone: true }).defaultNow(),
  submittedBy: uuid('submitted_by').references(() => teachers.id),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
}, (table) => ({
  // Prevent duplicate grade entries
  uniqueGrade: {
    columns: [table.studentId, table.subjectId, table.assessmentType, table.title, table.assessmentDate],
  },
}))

export const gradesRelations = relations(grades, ({ one }) => ({
  student: one(students, {
    fields: [grades.studentId],
    references: [students.id],
  }),
  teacher: one(teachers, {
    fields: [grades.teacherId],
    references: [teachers.id],
  }),
  subject: one(subjects, {
    fields: [grades.subjectId],
    references: [subjects.id],
  }),
  category: one(gradeCategories, {
    fields: [grades.categoryId],
    references: [gradeCategories.id],
  }),
  class: one(classes, {
    fields: [grades.classId],
    references: [classes.id],
  }),
  submittedByUser: one(teachers, {
    fields: [grades.submittedBy],
    references: [teachers.id],
  }),
  adjustments: many(gradeAdjustments),
}))
```

#### 5.1.3 Grade Adjustments Table

```typescript
// db/schema/assessment/grade-adjustments.ts
import { 
  pgTable, 
  uuid, 
  decimal, 
  timestamp, 
  text
} from 'drizzle-orm/pg-core'

export const gradeAdjustments = pgTable('grade_adjustments', {
  id: uuid('id').primaryKey().defaultRandom(),
  gradeId: uuid('grade_id').notNull().references(() => grades.id),
  originalScore: decimal('original_score', { precision: 5, scale: 2 }).notNull(),
  newScore: decimal('new_score', { precision: 5, scale: 2 }).notNull(),
  reason: text('reason').notNull(),
  adjustedBy: uuid('adjusted_by').notNull().references(() => kurikulumStaff.id),
  adjustedAt: timestamp('adjusted_at', { withTimezone: true }).defaultNow(),
  approvedBy: uuid('approved_by').references(() => kurikulumStaff.id),
  approvedAt: timestamp('approved_at', { withTimezone: true }),
  status: varchar('status', { 
    length: 20, 
    enum: ['pending', 'approved', 'rejected'] 
  }).default('pending'),
  notes: text('notes'),
})

export const gradeAdjustmentsRelations = relations(gradeAdjustments, ({ one }) => ({
  grade: one(grades, {
    fields: [gradeAdjustments.gradeId],
    references: [grades.id],
  }),
  adjustedByUser: one(kurikulumStaff, {
    fields: [gradeAdjustments.adjustedBy],
    references: [kurikulumStaff.id],
  }),
  approvedByUser: one(kurikulumStaff, {
    fields: [gradeAdjustments.approvedBy],
    references: [kurikulumStaff.id],
  }),
}))
```

### 5.2 Attendance System

#### 5.2.1 Attendance Records Table

```typescript
// db/schema/assessment/attendance.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  timestamp, 
  boolean,
  text
} from 'drizzle-orm/pg-core'

export const attendance = pgTable('attendance', {
  id: uuid('id').primaryKey().defaultRandom(),
  studentId: uuid('student_id').notNull().references(() => students.id),
  classId: uuid('class_id').notNull().references(() => classes.id),
  subjectId: uuid('subject_id').references(() => subjects.id),
  teacherId: uuid('teacher_id').references(() => teachers.id),
  date: timestamp('date', { withTimezone: true }).notNull(),
  status: varchar('status', { 
    length: 20, 
    enum: ['present', 'absent', 'late', 'excused', 'sick'] 
  }).notNull(),
  checkInTime: timestamp('check_in_time', { withTimezone: true }),
  checkOutTime: timestamp('check_out_time', { withTimezone: true }),
  remarks: text('remarks'),
  recordedBy: uuid('recorded_by').references(() => teachers.id),
  recordedAt: timestamp('recorded_at', { withTimezone: true }).defaultNow(),
  isVerified: boolean('is_verified').default(false),
  verifiedBy: uuid('verified_by').references(() => kurikulumStaff.id),
  verifiedAt: timestamp('verified_at', { withTimezone: true }),
}, (table) => ({
  // Prevent duplicate attendance records
  uniqueAttendance: {
    columns: [table.studentId, table.date, table.classId, table.subjectId],
  },
}))

export const attendanceRelations = relations(attendance, ({ one }) => ({
  student: one(students, {
    fields: [attendance.studentId],
    references: [students.id],
  }),
  class: one(classes, {
    fields: [attendance.classId],
    references: [classes.id],
  }),
  subject: one(subjects, {
    fields: [attendance.subjectId],
    references: [subjects.id],
  }),
  teacher: one(teachers, {
    fields: [attendance.teacherId],
    references: [teachers.id],
  }),
  recordedByUser: one(teachers, {
    fields: [attendance.recordedBy],
    references: [teachers.id],
  }),
  verifiedByUser: one(kurikulumStaff, {
    fields: [attendance.verifiedBy],
    references: [kurikulumStaff.id],
  }),
}))
```

---

## 6. Communication Tables

### 6.1 Content Management

#### 6.1.1 Content Files Table

```typescript
// db/schema/content/content-files.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  integer, 
  timestamp, 
  boolean,
  text,
  jsonb
} from 'drizzle-orm/pg-core'

export const contentFiles = pgTable('content_files', {
  id: uuid('id').primaryKey().defaultRandom(),
  title: varchar('title', { length: 200 }).notNull(),
  description: text('description'),
  fileName: varchar('file_name', { length: 255 }).notNull(),
  originalFileName: varchar('original_file_name', { length: 255 }).notNull(),
  fileSize: integer('file_size').notNull(), // in bytes
  mimeType: varchar('mime_type', { length: 100 }).notNull(),
  fileUrl: varchar('file_url', { length: 500 }).notNull(),
  storageKey: varchar('storage_key', { length: 500 }).notNull(), // S3 key
  contentType: varchar('content_type', { 
    length: 20, 
    enum: ['lesson_plan', 'module', 'assignment', 'reference', 'other'] 
  }).notNull(),
  subjectId: uuid('subject_id').references(() => subjects.id),
  classId: uuid('class_id').references(() => classes.id),
  uploadedBy: uuid('uploaded_by').notNull().references(() => teachers.id),
  tags: jsonb('tags').$type<string[]>(),
  isPublic: boolean('is_public').default(false),
  isActive: boolean('is_active').default(true),
  downloadCount: integer('download_count').default(0),
  lastDownloadedAt: timestamp('last_downloaded_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const contentFilesRelations = relations(contentFiles, ({ one }) => ({
  subject: one(subjects, {
    fields: [contentFiles.subjectId],
    references: [subjects.id],
  }),
  class: one(classes, {
    fields: [contentFiles.classId],
    references: [classes.id],
  }),
  uploadedByUser: one(teachers, {
    fields: [contentFiles.uploadedBy],
    references: [teachers.id],
  }),
}))
```

### 6.2 Question Bank

#### 6.2.1 Question Bank Table

```typescript
// db/schema/content/question-bank.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  text, 
  integer, 
  timestamp, 
  boolean,
  jsonb,
  decimal
} from 'drizzle-orm/pg-core'

export const questionBank = pgTable('question_bank', {
  id: uuid('id').primaryKey().defaultRandom(),
  subjectId: uuid('subject_id').notNull().references(() => subjects.id),
  createdBy: uuid('created_by').notNull().references(() => teachers.id),
  question: text('question').notNull(),
  questionType: varchar('question_type', { 
    length: 20, 
    enum: ['multiple_choice', 'true_false', 'essay', 'fill_blank', 'matching'] 
  }).notNull(),
  options: jsonb('options').$type<string[]>(), // For multiple choice
  correctAnswer: text('correct_answer').notNull(),
  explanation: text('explanation'),
  difficulty: varchar('difficulty', { 
    length: 10, 
    enum: ['easy', 'medium', 'hard'] 
  }).notNull(),
  topic: varchar('topic', { length: 100 }).notNull(),
  subtopics: jsonb('subtopics').$type<string[]>(),
  tags: jsonb('tags').$type<string[]>(),
  points: decimal('points', { precision: 5, scale: 2 }).default(1),
  timeLimit: integer('time_limit'), // in seconds
  isActive: boolean('is_active').default(true),
  isPublic: boolean('is_public').default(false),
  usageCount: integer('usage_count').default(0),
  averageScore: decimal('average_score', { precision: 5, scale: 2 }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const questionBankRelations = relations(questionBank, ({ one, many }) => ({
  subject: one(subjects, {
    fields: [questionBank.subjectId],
    references: [subjects.id],
  }),
  createdByUser: one(teachers, {
    fields: [questionBank.createdBy],
    references: [teachers.id],
  }),
  quizQuestions: many(quizQuestions),
}))
```

### 6.3 Quiz System

#### 6.3.1 Quizzes Table

```typescript
// db/schema/content/quizzes.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  text, 
  integer, 
  timestamp, 
  boolean,
  jsonb
} from 'drizzle-orm/pg-core'

export const quizzes = pgTable('quizzes', {
  id: uuid('id').primaryKey().defaultRandom(),
  title: varchar('title', { length: 200 }).notNull(),
  description: text('description'),
  subjectId: uuid('subject_id').notNull().references(() => subjects.id),
  createdBy: uuid('created_by').notNull().references(() => teachers.id),
  classId: uuid('class_id').references(() => classes.id),
  quizType: varchar('quiz_type', { 
    length: 20, 
    enum: ['practice', 'assignment', 'exam'] 
  }).notNull(),
  difficulty: varchar('difficulty', { 
    length: 10, 
    enum: ['easy', 'medium', 'hard'] 
  }).notNull(),
  timeLimit: integer('time_limit'), // in minutes
  attemptsAllowed: integer('attempts_allowed').default(1),
  passingScore: integer('passing_score'),
  shuffleQuestions: boolean('shuffle_questions').default(false),
  shuffleOptions: boolean('shuffle_options').default(false),
  showResults: boolean('show_results').default(true),
  showCorrectAnswers: boolean('show_correct_answers').default(false),
  isActive: boolean('is_active').default(true),
  startsAt: timestamp('starts_at', { withTimezone: true }),
  endsAt: timestamp('ends_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const quizzesRelations = relations(quizzes, ({ one, many }) => ({
  subject: one(subjects, {
    fields: [quizzes.subjectId],
    references: [subjects.id],
  }),
  createdByUser: one(teachers, {
    fields: [quizzes.createdBy],
    references: [teachers.id],
  }),
  class: one(classes, {
    fields: [quizzes.classId],
    references: [classes.id],
  }),
  questions: many(quizQuestions),
  attempts: many(quizAttempts),
}))
```

#### 6.3.2 Quiz Questions Table

```typescript
// db/schema/content/quiz-questions.ts
import { 
  pgTable, 
  uuid, 
  integer, 
  boolean,
  jsonb
} from 'drizzle-orm/pg-core'

export const quizQuestions = pgTable('quiz_questions', {
  id: uuid('id').primaryKey().defaultRandom(),
  quizId: uuid('quiz_id').notNull().references(() => quizzes.id),
  questionId: uuid('question_id').notNull().references(() => questionBank.id),
  orderNumber: integer('order_number').notNull(),
  points: integer('points').notNull().default(1),
  isRequired: boolean('is_required').default(true),
}, (table) => ({
  // Prevent duplicate questions in quiz
  uniqueQuizQuestion: {
    columns: [table.quizId, table.questionId],
  },
}))

export const quizQuestionsRelations = relations(quizQuestions, ({ one }) => ({
  quiz: one(quizzes, {
    fields: [quizQuestions.quizId],
    references: [quizzes.id],
  }),
  question: one(questionBank, {
    fields: [quizQuestions.questionId],
    references: [questionBank.id],
  }),
}))
```

#### 6.3.3 Quiz Attempts Table

```typescript
// db/schema/content/quiz-attempts.ts
import { 
  pgTable, 
  uuid, 
  integer, 
  timestamp, 
  boolean,
  jsonb
} from 'drizzle-orm/pg-core'

export const quizAttempts = pgTable('quiz_attempts', {
  id: uuid('id').primaryKey().defaultRandom(),
  quizId: uuid('quiz_id').notNull().references(() => quizzes.id),
  studentId: uuid('student_id').notNull().references(() => students.id),
  attemptNumber: integer('attempt_number').notNull(),
  score: integer('score'),
  maxScore: integer('max_score'),
  percentage: integer('percentage'),
  startedAt: timestamp('started_at', { withTimezone: true }).defaultNow(),
  completedAt: timestamp('completed_at', { withTimezone: true }),
  timeSpent: integer('time_spent'), // in seconds
  answers: jsonb('answers').$type<Array<{
    questionId: string
    answer: string
    timeSpent: number
    isCorrect: boolean
  }>>(),
  status: varchar('status', { 
    length: 20, 
    enum: ['in_progress', 'completed', 'abandoned', 'expired'] 
  }).default('in_progress'),
  isPassed: boolean('is_passed'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
}, (table) => ({
  // Prevent duplicate attempts
  uniqueAttempt: {
    columns: [table.quizId, table.studentId, table.attemptNumber],
  },
}))

export const quizAttemptsRelations = relations(quizAttempts, ({ one }) => ({
  quiz: one(quizzes, {
    fields: [quizAttempts.quizId],
    references: [quizzes.id],
  }),
  student: one(students, {
    fields: [quizAttempts.studentId],
    references: [students.id],
  }),
}))
```

### 6.4 Chat and Discussion

#### 6.4.1 Chat Rooms Table

```typescript
// db/schema/communication/chat-rooms.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  timestamp, 
  boolean,
  jsonb
} from 'drizzle-orm/pg-core'

export const chatRooms = pgTable('chat_rooms', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 100 }),
  type: varchar('type', { 
    length: 20, 
    enum: ['consultation', 'group', 'announcement'] 
  }).notNull(),
  subjectId: uuid('subject_id').references(() => subjects.id),
  classId: uuid('class_id').references(() => classes.id),
  createdBy: uuid('created_by').notNull().references(() => users.id),
  isActive: boolean('is_active').default(true),
  metadata: jsonb('metadata').$type<{
    participantIds?: string[]
    maxParticipants?: number
    isPublic?: boolean
  }>(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const chatRoomsRelations = relations(chatRooms, ({ one, many }) => ({
  subject: one(subjects, {
    fields: [chatRooms.subjectId],
    references: [subjects.id],
  }),
  class: one(classes, {
    fields: [chatRooms.classId],
    references: [classes.id],
  }),
  createdByUser: one(users, {
    fields: [chatRooms.createdBy],
    references: [users.id],
  }),
  messages: many(chatMessages),
  participants: many(chatParticipants),
}))
```

#### 6.4.2 Chat Messages Table

```typescript
// db/schema/communication/chat-messages.ts
import { 
  pgTable, 
  uuid, 
  text, 
  timestamp, 
  boolean,
  jsonb
} from 'drizzle-orm/pg-core'

export const chatMessages = pgTable('chat_messages', {
  id: uuid('id').primaryKey().defaultRandom(),
  roomId: uuid('room_id').notNull().references(() => chatRooms.id),
  senderId: uuid('sender_id').notNull().references(() => users.id),
  content: text('content').notNull(),
  messageType: varchar('message_type', { 
    length: 20, 
    enum: ['text', 'file', 'image', 'system'] 
  }).notNull(),
  replyToId: uuid('reply_to_id').references(() => chatMessages.id),
  attachments: jsonb('attachments').$type<Array<{
    type: string
    url: string
    name: string
    size: number
  }>>(),
  isEdited: boolean('is_edited').default(false),
  editedAt: timestamp('edited_at', { withTimezone: true }),
  isDeleted: boolean('is_deleted').default(false),
  deletedAt: timestamp('deleted_at', { withTimezone: true }),
  metadata: jsonb('metadata').$type<{
    reactions?: Array<{
      emoji: string
      userId: string
      createdAt: string
    }>
  }>(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
})

export const chatMessagesRelations = relations(chatMessages, ({ one }) => ({
  room: one(chatRooms, {
    fields: [chatMessages.roomId],
    references: [chatRooms.id],
  }),
  sender: one(users, {
    fields: [chatMessages.senderId],
    references: [users.id],
  }),
  replyTo: one(chatMessages, {
    fields: [chatMessages.replyToId],
    references: [chatMessages.id],
  }),
}))
```

---

## 7. System Tables

### 7.1 Audit and Logging

#### 7.1.1 Audit Logs Table

```typescript
// db/schema/system/audit-logs.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  text, 
  timestamp, 
  jsonb
} from 'drizzle-orm/pg-core'

export const auditLogs = pgTable('audit_logs', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id),
  action: varchar('action', { length: 50 }).notNull(),
  resource: varchar('resource', { length: 50 }).notNull(),
  resourceId: varchar('resource_id', { length: 255 }),
  oldValues: jsonb('old_values'),
  newValues: jsonb('new_values'),
  ipAddress: varchar('ip_address', { length: 45 }),
  userAgent: text('user_agent'),
  sessionId: varchar('session_id', { length: 255 }),
  timestamp: timestamp('timestamp', { withTimezone: true }).defaultNow(),
  metadata: jsonb('metadata'),
})

export const auditLogsRelations = relations(auditLogs, ({ one }) => ({
  user: one(users, {
    fields: [auditLogs.userId],
    references: [users.id],
  }),
}))
```

### 7.2 System Configuration

#### 7.2.1 System Settings Table

```typescript
// db/schema/system/system-settings.ts
import { 
  pgTable, 
  varchar, 
  text, 
  timestamp, 
  jsonb
} from 'drizzle-orm/pg-core'

export const systemSettings = pgTable('system_settings', {
  key: varchar('key', { length: 100 }).primaryKey(),
  value: jsonb('value').notNull(),
  description: text('description'),
  category: varchar('category', { length: 50 }).notNull(),
  isPublic: boolean('is_public').default(false),
  updatedBy: uuid('updated_by').references(() => kurikulumStaff.id),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})
```

### 7.3 Financial Management

#### 7.3.1 Student Savings Table

```typescript
// db/schema/system/student-savings.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  integer, 
  timestamp, 
  text,
  jsonb
} from 'drizzle-orm/pg-core'

export const studentSavings = pgTable('student_savings', {
  id: uuid('id').primaryKey().defaultRandom(),
  studentId: uuid('student_id').notNull().unique().references(() => students.id),
  accountNumber: varchar('account_number', { length: 20 }).notNull().unique(),
  balance: integer('balance').notNull().default(0), // in cents
  currency: varchar('currency', { length: 3 }).default('IDR'),
  isActive: boolean('is_active').default(true),
  openedAt: timestamp('opened_at', { withTimezone: true }).defaultNow(),
  lastTransactionAt: timestamp('last_transaction_at', { withTimezone: true }),
  metadata: jsonb('metadata').$type<{
    dailyLimit?: number
    monthlyLimit?: number
    parentApproval?: boolean
  }>(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
})

export const studentSavingsRelations = relations(studentSavings, ({ one, many }) => ({
  student: one(students, {
    fields: [studentSavings.studentId],
    references: [students.id],
  }),
  transactions: many(savingsTransactions),
}))
```

#### 7.3.2 Savings Transactions Table

```typescript
// db/schema/system/savings-transactions.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  integer, 
  timestamp, 
  text
} from 'drizzle-orm/pg-core'

export const savingsTransactions = pgTable('savings_transactions', {
  id: uuid('id').primaryKey().defaultRandom(),
  accountId: uuid('account_id').notNull().references(() => studentSavings.id),
  type: varchar('type', { 
    length: 20, 
    enum: ['deposit', 'withdrawal', 'fee', 'refund', 'transfer_in', 'transfer_out'] 
  }).notNull(),
  amount: integer('amount').notNull(), // in cents
  balanceAfter: integer('balance_after').notNull(),
  description: text('description').notNull(),
  referenceNumber: varchar('reference_number', { length: 50 }).unique(),
  category: varchar('category', { length: 50 }),
  processedBy: uuid('processed_by').references(() => kurikulumStaff.id),
  transactionDate: timestamp('transaction_date', { withTimezone: true }).notNull(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  metadata: jsonb('metadata').$type<{
    method?: string
    receiptUrl?: string
    approvedBy?: string
  }>(),
})

export const savingsTransactionsRelations = relations(savingsTransactions, ({ one }) => ({
  account: one(studentSavings, {
    fields: [savingsTransactions.accountId],
    references: [studentSavings.id],
  }),
  processedByUser: one(kurikulumStaff, {
    fields: [savingsTransactions.processedBy],
    references: [kurikulumStaff.id],
  }),
}))
```

---

## 8. Relationships and Constraints

### 8.1 Foreign Key Constraints

```sql
-- Core relationships
ALTER TABLE user_profiles ADD CONSTRAINT fk_user_profiles_user_id 
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

ALTER TABLE sessions ADD CONSTRAINT fk_sessions_user_id 
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

-- Academic relationships
ALTER TABLE students ADD CONSTRAINT fk_students_user_id 
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;
ALTER TABLE students ADD CONSTRAINT fk_students_class_id 
  FOREIGN KEY (class_id) REFERENCES classes(id) ON DELETE SET NULL;

ALTER TABLE teachers ADD CONSTRAINT fk_teachers_user_id 
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

ALTER TABLE classes ADD CONSTRAINT fk_classes_homeroom_teacher_id 
  FOREIGN KEY (homeroom_teacher_id) REFERENCES teachers(id) ON DELETE SET NULL;

-- Assessment relationships
ALTER TABLE grades ADD CONSTRAINT fk_grades_student_id 
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE;
ALTER TABLE grades ADD CONSTRAINT fk_grades_teacher_id 
  FOREIGN KEY (teacher_id) REFERENCES teachers(id) ON DELETE RESTRICT;
ALTER TABLE grades ADD CONSTRAINT fk_grades_subject_id 
  FOREIGN KEY (subject_id) REFERENCES subjects(id) ON DELETE RESTRICT;
```

### 8.2 Check Constraints

```sql
-- User role validation
ALTER TABLE users ADD CONSTRAINT ck_users_role 
  CHECK (role IN ('kurikulum', 'guru', 'siswa'));

-- Grade validation
ALTER TABLE grades ADD CONSTRAINT ck_grades_score_range 
  CHECK (score >= 0 AND score <= max_score);
ALTER TABLE grades ADD CONSTRAINT ck_grades_semester 
  CHECK (semester IN (1, 2));

-- Schedule validation
ALTER TABLE schedules ADD CONSTRAINT ck_schedules_day_of_week 
  CHECK (day_of_week >= 1 AND day_of_week <= 7);
ALTER TABLE schedules ADD CONSTRAINT ck_schedules_time_order 
  CHECK (start_time < end_time);

-- Class capacity validation
ALTER TABLE classes ADD CONSTRAINT ck_classes_capacity 
  CHECK (current_enrollment <= capacity);

-- Attendance validation
ALTER TABLE attendance ADD CONSTRAINT ck_attendance_check_time 
  CHECK (check_out_time IS NULL OR check_out_time > check_in_time);
```

### 8.3 Unique Constraints

```sql
-- Email uniqueness
ALTER TABLE users ADD CONSTRAINT uq_users_email UNIQUE (email);

-- Student ID uniqueness
ALTER TABLE students ADD CONSTRAINT uq_students_nis UNIQUE (nis);
ALTER TABLE students ADD CONSTRAINT uq_students_nisn UNIQUE (nisn);

-- Teacher ID uniqueness
ALTER TABLE teachers ADD CONSTRAINT uq_teachers_employee_id UNIQUE (employee_id);

-- Subject code uniqueness
ALTER TABLE subjects ADD CONSTRAINT uq_subjects_code UNIQUE (code);

-- Session token uniqueness
ALTER TABLE sessions ADD CONSTRAINT uq_sessions_session_token UNIQUE (session_token);
```

---

## 9. Indexes and Performance

### 9.1 Primary Indexes

```sql
-- User and authentication indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_is_active ON users(is_active);
CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_session_token ON sessions(session_token);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);

-- Academic indexes
CREATE INDEX idx_students_class_id ON students(class_id);
CREATE INDEX idx_students_nis ON students(nis);
CREATE INDEX idx_students_is_active ON students(is_active);
CREATE INDEX idx_teachers_employee_id ON teachers(employee_id);
CREATE INDEX idx_teachers_is_active ON teachers(is_active);
CREATE INDEX idx_classes_academic_year ON classes(academic_year);
CREATE INDEX idx_classes_grade_level ON classes(grade_level);

-- Assessment indexes
CREATE INDEX idx_grades_student_id ON grades(student_id);
CREATE INDEX idx_grades_subject_id ON grades(subject_id);
CREATE INDEX idx_grades_teacher_id ON grades(teacher_id);
CREATE INDEX idx_grades_assessment_date ON grades(assessment_date);
CREATE INDEX idx_grades_semester_year ON grades(semester, academic_year);
CREATE INDEX idx_attendance_student_date ON attendance(student_id, date);
CREATE INDEX idx_attendance_class_date ON attendance(class_id, date);

-- Content indexes
CREATE INDEX idx_content_files_subject_id ON content_files(subject_id);
CREATE INDEX idx_content_files_class_id ON content_files(class_id);
CREATE INDEX idx_content_files_content_type ON content_files(content_type);
CREATE INDEX idx_question_bank_subject_id ON question_bank(subject_id);
CREATE INDEX idx_question_bank_difficulty ON question_bank(difficulty);
CREATE INDEX idx_question_bank_topic ON question_bank(topic);

-- Communication indexes
CREATE INDEX idx_chat_messages_room_id ON chat_messages(room_id);
CREATE INDEX idx_chat_messages_sender_id ON chat_messages(sender_id);
CREATE INDEX idx_chat_messages_created_at ON chat_messages(created_at);

-- System indexes
CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_savings_transactions_account_id ON savings_transactions(account_id);
CREATE INDEX idx_savings_transactions_date ON savings_transactions(transaction_date);
```

### 9.2 Composite Indexes

```sql
-- Grade query optimization
CREATE INDEX idx_grades_composite ON grades(student_id, subject_id, semester, academic_year);
CREATE INDEX idx_grades_class_subject ON grades(class_id, subject_id, assessment_date);

-- Schedule optimization
CREATE INDEX idx_schedules_teacher_time ON schedules(teacher_id, day_of_week, start_time);
CREATE INDEX idx_schedules_class_day ON schedules(class_id, day_of_week);

-- Attendance optimization
CREATE INDEX idx_attendance_composite ON attendance(student_id, status, date);
CREATE INDEX idx_attendance_class_subject ON attendance(class_id, subject_id, date);

-- Quiz optimization
CREATE INDEX idx_quiz_attempts_composite ON quiz_attempts(quiz_id, student_id, status);
CREATE INDEX idx_quiz_attempts_student ON quiz_attempts(student_id, completed_at);
```

### 9.3 Full-Text Search Indexes

```sql
-- Content search
CREATE INDEX idx_content_files_search ON content_files USING gin(to_tsvector('english', title || ' ' || COALESCE(description, '')));
CREATE INDEX idx_question_bank_search ON question_bank USING gin(to_tsvector('english', question || ' ' || COALESCE(topic, '')));

-- User search
CREATE INDEX idx_users_search ON user_profiles USING gin(to_tsvector('english', first_name || ' ' || last_name || ' ' || email));
```

---

## 10. Migration Strategy

### 10.1 Migration Files Structure

```
drizzle/
├── migrations/
│   ├── 0001_initial_schema.sql
│   ├── 0002_add_academic_tables.sql
│   ├── 0003_add_assessment_tables.sql
│   ├── 0004_add_communication_tables.sql
│   ├── 0005_add_system_tables.sql
│   ├── 0006_add_indexes.sql
│   ├── 0007_add_constraints.sql
│   └── 0008_seed_data.sql
└── meta/
    ├── _journal.json
    └── 0001_snapshot.json
```

### 10.2 Example Migration Files

#### 10.2.1 Initial Schema Migration

```sql
-- 0001_initial_schema.sql
-- Initial schema with core authentication tables

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Users table
CREATE TABLE IF NOT EXISTS users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(255) NOT NULL UNIQUE,
  email_verified BOOLEAN DEFAULT false,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(20) NOT NULL CHECK (role IN ('kurikulum', 'guru', 'siswa')),
  is_active BOOLEAN DEFAULT true,
  last_login_at TIMESTAMP WITH TIME ZONE,
  password_changed_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- User profiles table
CREATE TABLE IF NOT EXISTS user_profiles (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  display_name VARCHAR(201),
  phone VARCHAR(20),
  avatar_url VARCHAR(500),
  bio TEXT,
  preferences JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Sessions table
CREATE TABLE IF NOT EXISTS sessions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  session_token VARCHAR(255) NOT NULL UNIQUE,
  access_token VARCHAR(500),
  refresh_token VARCHAR(500),
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
  ip_address VARCHAR(45),
  user_agent TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### 10.2.2 Academic Tables Migration

```sql
-- 0002_add_academic_tables.sql
-- Add academic structure tables

-- Subjects table
CREATE TABLE IF NOT EXISTS subjects (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  code VARCHAR(20) NOT NULL UNIQUE,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  credits INTEGER DEFAULT 1,
  department VARCHAR(100),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Classes table
CREATE TABLE IF NOT EXISTS classes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(100) NOT NULL,
  grade_level INTEGER NOT NULL,
  academic_year VARCHAR(9) NOT NULL,
  homeroom_teacher_id UUID,
  room VARCHAR(20),
  capacity INTEGER DEFAULT 40,
  current_enrollment INTEGER DEFAULT 0,
  is_active BOOLEAN DEFAULT true,
  metadata JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Students table
CREATE TABLE IF NOT EXISTS students (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  nis VARCHAR(20) NOT NULL UNIQUE,
  nisn VARCHAR(20) UNIQUE,
  class_id UUID REFERENCES classes(id) ON DELETE SET NULL,
  enrollment_date TIMESTAMP WITH TIME ZONE NOT NULL,
  graduation_date TIMESTAMP WITH TIME ZONE,
  is_active BOOLEAN DEFAULT true,
  parent_info JSONB,
  medical_info JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Teachers table
CREATE TABLE IF NOT EXISTS teachers (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  employee_id VARCHAR(20) NOT NULL UNIQUE,
  nik VARCHAR(20) UNIQUE,
  specialization VARCHAR(100),
  qualification VARCHAR(100),
  hire_date TIMESTAMP WITH TIME ZONE NOT NULL,
  is_active BOOLEAN DEFAULT true,
  is_homeroom BOOLEAN DEFAULT false,
  bank_info JSONB,
  emergency_contact JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### 10.3 Migration Commands

```bash
# Generate migration
npx drizzle-kit generate

# Run migrations
npx drizzle-kit migrate

# Push schema changes (development)
npx drizzle-kit push

# Check schema status
npx drizzle-kit introspect
```

---

## 11. Seed Data

### 11.1 Initial System Data

#### 11.1.1 Grade Categories Seed

```typescript
// db/seed/grade-categories.ts
import { db } from '../db'
import { gradeCategories } from '../db/schema/assessment'

export async function seedGradeCategories() {
  const categories = [
    {
      name: 'Tugas Harian',
      code: 'daily',
      description: 'Tugas-tugas yang diberikan secara rutin',
      weight: 20,
    },
    {
      name: 'Ulangan Harian',
      code: 'quiz',
      description: 'Tes singkat untuk evaluasi pemahaman',
      weight: 15,
    },
    {
      name: 'Tugas Besar',
      code: 'assignment',
      description: 'Proyek atau tugas komprehensif',
      weight: 25,
    },
    {
      name: 'Ujian Tengah Semester',
      code: 'midterm',
      description: 'Evaluasi tengah semester',
      weight: 20,
    },
    {
      name: 'Ujian Akhir Semester',
      code: 'final',
      description: 'Evaluasi akhir semester',
      weight: 20,
    },
  ]

  await db.insert(gradeCategories).values(categories)
}
```

#### 11.1.2 Subjects Seed

```typescript
// db/seed/subjects.ts
import { db } from '../db'
import { subjects } from '../db/schema/academic'

export async function seedSubjects() {
  const subjectList = [
    {
      code: 'MAT001',
      name: 'Matematika',
      description: 'Pelajaran matematika untuk jenjang SMA',
      credits: 4,
      department: 'MIPA',
    },
    {
      code: 'BIO001',
      name: 'Biologi',
      description: 'Pelajaran biologi untuk jenjang SMA',
      credits: 3,
      department: 'MIPA',
    },
    {
      code: 'FIS001',
      name: 'Fisika',
      description: 'Pelajaran fisika untuk jenjang SMA',
      credits: 3,
      department: 'MIPA',
    },
    {
      code: 'KIM001',
      name: 'Kimia',
      description: 'Pelajaran kimia untuk jenjang SMA',
      credits: 3,
      department: 'MIPA',
    },
    {
      code: 'IND001',
      name: 'Bahasa Indonesia',
      description: 'Pelajaran bahasa Indonesia',
      credits: 3,
      department: 'Bahasa',
    },
    {
      code: 'ENG001',
      name: 'Bahasa Inggris',
      description: 'Pelajaran bahasa Inggris',
      credits: 3,
      department: 'Bahasa',
    },
  ]

  await db.insert(subjects).values(subjectList)
}
```

#### 11.1.3 System Settings Seed

```typescript
// db/seed/system-settings.ts
import { db } from '../db'
import { systemSettings } from '../db/schema/system'

export async function seedSystemSettings() {
  const settings = [
    {
      key: 'school_name',
      value: 'SMA Negeri 1 Jakarta',
      description: 'Nama sekolah',
      category: 'general',
      isPublic: true,
    },
    {
      key: 'academic_year',
      value: '2024/2025',
      description: 'Tahun ajaran aktif',
      category: 'academic',
      isPublic: true,
    },
    {
      key: 'semester',
      value: 1,
      description: 'Semester aktif',
      category: 'academic',
      isPublic: true,
    },
    {
      key: 'passing_grade',
      value: 70,
      description: 'Nilai minimal kelulusan',
      category: 'academic',
      isPublic: false,
    },
    {
      key: 'max_file_size',
      value: 10485760, // 10MB
      description: 'Ukuran maksimal file upload (bytes)',
      category: 'system',
      isPublic: false,
    },
  ]

  await db.insert(systemSettings).values(settings)
}
```

### 11.2 Sample Data Generator

#### 11.2.1 Student Data Generator

```typescript
// db/seed/sample-students.ts
import { db } from '../db'
import { users, userProfiles, students, classes } from '../db/schema'
import { faker } from '@faker-js/faker/locale/id_ID'

export async function generateSampleStudents(count: number = 50) {
  const classes = await db.select().from(classes)
  
  for (let i = 0; i < count; i++) {
    const randomClass = classes[Math.floor(Math.random() * classes.length)]
    const firstName = faker.person.firstName()
    const lastName = faker.person.lastName()
    const email = faker.internet.email({ firstName, lastName })
    
    // Create user
    const [user] = await db.insert(users).values({
      email,
      passwordHash: 'hashed_password', // Should be properly hashed
      role: 'siswa',
    }).returning()
    
    // Create profile
    await db.insert(userProfiles).values({
      userId: user.id,
      firstName,
      lastName,
      displayName: `${firstName} ${lastName}`,
      phone: faker.phone.number(),
    })
    
    // Create student
    await db.insert(students).values({
      userId: user.id,
      nis: `2024${String(i + 1).padStart(4, '0')}`,
      classId: randomClass.id,
      enrollmentDate: faker.date.past({ years: 1 }),
      parentInfo: {
        fatherName: faker.person.fullName('male'),
        motherName: faker.person.fullName('female'),
        parentPhone: faker.phone.number(),
      },
    })
  }
}
```

---

## 12. Data Storage Strategy

### 12.1 Relational vs. Non-Relational Data

#### 12.1.1 PostgreSQL (Relational Data)
- **User Management:** Accounts, profiles, authentication
- **Academic Data:** Classes, subjects, schedules
- **Assessment Data:** Grades, attendance, quizzes
- **Financial Data:** Student savings, transactions
- **System Data:** Settings, audit logs

#### 12.1.2 S3/Cloud Storage (Files)
- **Profile Pictures:** User avatars and photos
- **Learning Materials:** PDFs, documents, presentations
- **Content Files:** Lesson plans, modules, references
- **Assignments:** Student submissions and files
- **Reports:** Generated PDF reports and documents

#### 12.1.3 Redis (Caching Layer)
- **Session Data:** Active user sessions
- **Frequent Queries:** Class schedules, user roles
- **Real-time Data:** Chat messages, online status
- **Temporary Data:** File upload progress, form drafts

### 12.2 File Storage Strategy

#### 12.2.1 S3 Bucket Structure

```
ems-files/
├── avatars/
│   ├── users/
│   └── teachers/
├── content/
│   ├── lesson-plans/
│   ├── modules/
│   ├── assignments/
│   └── references/
├── submissions/
│   ├── {class-id}/
│   │   ├── {subject-id}/
│   │   └── {student-id}/
├── reports/
│   ├── grades/
│   ├── attendance/
│   └── financial/
└── temp/
    ├── uploads/
    └── exports/
```

#### 12.2.2 File Metadata Storage

```typescript
interface FileMetadata {
  id: string
  originalName: string
  storedName: string
  mimeType: string
  size: number
  uploadedBy: string
  uploadedAt: Date
  category: 'avatar' | 'content' | 'submission' | 'report'
  accessibility: 'public' | 'private' | 'role-based'
  expiryDate?: Date
  downloadCount: number
  lastAccessedAt?: Date
}
```

### 12.3 Backup and Recovery Strategy

#### 12.3.1 Database Backup Schedule

```bash
# Daily full backup at 2 AM
0 2 * * * pg_dump -h localhost -U postgres -d ems_db | gzip > /backups/daily/ems_$(date +\%Y\%m\%d).sql.gz

# Weekly differential backup
0 3 * * 0 pg_dump -h localhost -U postgres -d ems_db --schema-only > /backups/weekly/schema_$(date +\%Y\%m\%d).sql

# Point-in-time recovery (WAL archiving)
archive_command = 'cp %p /backups/wal/%f'
```

#### 12.3.2 File Backup Strategy

```bash
# Sync S3 to backup location daily
aws s3 sync s3://ems-files/ s3://ems-backups/files/ --delete

# Cross-region replication for disaster recovery
aws s3api put-bucket-replication --bucket ems-files --replication-configuration file://replication-config.json
```

---

## Conclusion

This comprehensive database schema design provides a robust foundation for the Educational Management System. The schema addresses all functional requirements while maintaining data integrity, performance, and scalability.

Key features of this schema design:

1. **Normalization:** Proper normalization reduces data redundancy and improves consistency
2. **Type Safety:** Drizzle ORM provides TypeScript type safety throughout the application
3. **Performance:** Strategic indexing ensures optimal query performance
4. **Scalability:** Schema supports horizontal scaling and future growth
5. **Auditability:** Comprehensive audit trail for all data changes
6. **Flexibility:** JSONB columns provide flexibility for evolving requirements
7. **Security:** Row-level security and proper access controls

The schema is production-ready and can be easily extended as the system grows and new requirements emerge. Regular maintenance, monitoring, and optimization will ensure the database continues to perform optimally as the user base expands.