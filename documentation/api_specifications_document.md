# Educational Management System (EMS) - API Specifications

## Document Information

- **Document Version**: 1.0
- **Date**: October 19, 2025
- **Author**: EMS Development Team
- **Status**: Draft
- **API Version**: v1
- **Base URL**: `https://ems.school.edu/api/v1`

---

## Table of Contents

1. [API Overview](#1-api-overview)
2. [Authentication & Authorization](#2-authentication--authorization)
3. [Common Response Format](#3-common-response-format)
4. [Error Handling](#4-error-handling)
5. [Rate Limiting](#5-rate-limiting)
6. [Kurikulum (Admin) APIs](#6-kurikulum-admin-apis)
7. [Guru (Teacher) APIs](#7-guru-teacher-apis)
8. [Siswa (Student) APIs](#8-siswa-student-apis)
9. [Authentication APIs](#9-authentication-apis)
10. [Real-time APIs (WebSocket)](#10-real-time-apis-websocket)
11. [OpenAPI Specification](#11-openapi-specification)

---

## 1. API Overview

### 1.1 Design Principles

The EMS API follows RESTful design principles with:

- **Resource-based URLs**: Clear, hierarchical resource identification
- **HTTP Methods**: Proper use of GET, POST, PUT, DELETE, PATCH
- **Stateless**: Each request contains all necessary information
- **Consistent**: Uniform interface across all endpoints
- **Cacheable**: Responses indicate cacheability
- **Layered System**: Architecture supports scalable deployment

### 1.2 Base Architecture

```
https://ems.school.edu/api/v1/
├── auth/                 # Authentication endpoints
├── admin/                # Kurikulum (Admin) endpoints
├── teacher/              # Guru (Teacher) endpoints  
├── student/              # Siswa (Student) endpoints
├── chat/                 # Real-time chat endpoints
├── discussion/           # Forum/discussion endpoints
├── public/               # Public information endpoints
└── system/               # System health/maintenance endpoints
```

### 1.3 Supported HTTP Methods

| Method | Description | Idempotent |
|--------|-------------|------------|
| `GET` | Retrieve resources | ✅ |
| `POST` | Create new resources | ❌ |
| `PUT` | Replace entire resource | ✅ |
| `PATCH` | Partial resource update | ❌ |
| `DELETE` | Remove resource | ✅ |
| `OPTIONS` | Retrieve allowed methods | ✅ |

---

## 2. Authentication & Authorization

### 2.1 Authentication Methods

#### 2.1.1 Session-Based Authentication (Primary)
```http
Cookie: session-token=<session_id>
```

#### 2.1.2 JWT Bearer Token (Alternative)
```http
Authorization: Bearer <jwt_token>
```

### 2.2 Role-Based Access Control (RBAC)

| Role | Access Level | Description |
|------|--------------|-------------|
| `kurikulum` | Full System Access | Administrative functions, user management, system configuration |
| `guru` | Teacher Access | Grade management, class management, content creation |
| `siswa` | Student Access | View grades, schedules, learning materials |

### 2.3 Authorization Headers

All protected endpoints require valid authentication:

```http
# Session-based
Cookie: session-token=abc123def456...

# JWT Token
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Role Context (optional)
X-User-Role: guru
X-User-ID: 12345678-1234-1234-1234-123456789012
```

---

## 3. Common Response Format

### 3.1 Success Response Structure

```json
{
  "success": true,
  "data": {
    // Response data varies by endpoint
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00.000Z",
    "requestId": "req_123456789",
    "version": "v1",
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "totalPages": 8,
      "hasNext": true,
      "hasPrev": false
    }
  }
}
```

### 3.2 Error Response Structure

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": {
      "field": "email",
      "issue": "Invalid email format"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00.000Z",
    "requestId": "req_123456789",
    "version": "v1"
  }
}
```

### 3.3 Standard HTTP Status Codes

| Status | Meaning | Usage |
|--------|---------|-------|
| `200 OK` | Success | GET requests, successful updates |
| `201 Created` | Resource Created | POST requests creating new resources |
| `204 No Content` | Success, No Content | DELETE operations |
| `400 Bad Request` | Invalid Request | Validation errors, malformed data |
| `401 Unauthorized` | Authentication Required | Missing/invalid authentication |
| `403 Forbidden` | Insufficient Permissions | Valid auth but insufficient role |
| `404 Not Found` | Resource Not Found | Requested resource doesn't exist |
| `409 Conflict` | Resource Conflict | Duplicate creation, state conflicts |
| `422 Unprocessable Entity` | Validation Failed | Business logic validation errors |
| `429 Too Many Requests` | Rate Limited | Exceeded rate limits |
| `500 Internal Server Error` | Server Error | Unexpected server failures |

---

## 4. Error Handling

### 4.1 Error Codes Reference

| Error Code | HTTP Status | Description |
|------------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Authentication required or invalid |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Request validation failed |
| `CONFLICT` | 409 | Resource conflict |
| `RATE_LIMITED` | 429 | Rate limit exceeded |
| `INTERNAL_ERROR` | 500 | Internal server error |
| `SERVICE_UNAVAILABLE` | 503 | Service temporarily unavailable |
| `INVALID_ROLE` | 403 | Invalid user role for operation |
| `SESSION_EXPIRED` | 401 | User session has expired |
| `MAINTENANCE_MODE` | 503 | System under maintenance |

### 4.2 Validation Error Format

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": {
      "errors": [
        {
          "field": "email",
          "message": "Invalid email format",
          "value": "invalid-email"
        },
        {
          "field": "password",
          "message": "Password must be at least 8 characters",
          "value": "123"
        }
      ]
    }
  }
}
```

---

## 5. Rate Limiting

### 5.1 Rate Limit Configuration

| User Role | Requests per Minute | Burst Limit |
|-----------|---------------------|-------------|
| Anonymous | 30 | 10 |
| Student | 100 | 20 |
| Teacher | 200 | 50 |
| Admin | 500 | 100 |

### 5.2 Rate Limit Headers

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642248600
X-RateLimit-Retry-After: 30
```

### 5.3 Rate Limit Response

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests. Please try again later.",
    "details": {
      "retryAfter": 30,
      "limit": 100,
      "window": "1 minute"
    }
  }
}
```

---

## 6. Kurikulum (Admin) APIs

### 6.1 User Management

#### 6.1.1 List Users

```http
GET /api/v1/admin/users
```

**Query Parameters:**
- `role` (optional): Filter by role (`kurikulum`, `guru`, `siswa`)
- `search` (optional): Search by name or email
- `isActive` (optional): Filter by active status (`true`, `false`)
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 20, max: 100)

**Response:**
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "12345678-1234-1234-1234-123456789012",
        "email": "teacher@school.edu",
        "role": "guru",
        "profile": {
          "firstName": "John",
          "lastName": "Doe",
          "phone": "+62-812-3456-7890",
          "avatar": "https://cdn.school.edu/avatars/123.jpg"
        },
        "isActive": true,
        "lastLoginAt": "2024-01-15T09:00:00.000Z",
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ]
  },
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "totalPages": 8
    }
  }
}
```

#### 6.1.2 Create User

```http
POST /api/v1/admin/users
```

**Request Body:**
```json
{
  "email": "newteacher@school.edu",
  "password": "SecurePass123!",
  "role": "guru",
  "profile": {
    "firstName": "Jane",
    "lastName": "Smith",
    "phone": "+62-812-3456-7891",
    "employeeId": "EMP001"
  },
  "sendWelcomeEmail": true
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "87654321-4321-4321-4321-210987654321",
      "email": "newteacher@school.edu",
      "role": "guru",
      "profile": {
        "firstName": "Jane",
        "lastName": "Smith",
        "phone": "+62-812-3456-7891",
        "employeeId": "EMP001"
      },
      "isActive": true,
      "createdAt": "2024-01-15T10:30:00.000Z"
    },
    "temporaryPassword": "tempPass123!"
  }
}
```

#### 6.1.3 Update User

```http
PUT /api/v1/admin/users/{userId}
```

**Request Body:**
```json
{
  "profile": {
    "firstName": "Jane",
    "lastName": "Johnson",
    "phone": "+62-812-3456-7892"
  },
  "isActive": true,
  "role": "guru"
}
```

#### 6.1.4 Delete/Deactivate User

```http
DELETE /api/v1/admin/users/{userId}
```

**Query Parameters:**
- `permanent` (optional): Permanently delete vs deactivate (default: false)

### 6.2 Class Management

#### 6.2.1 List Classes

```http
GET /api/v1/admin/classes
```

**Query Parameters:**
- `academicYear` (optional): Filter by academic year
- `gradeLevel` (optional): Filter by grade level
- `isActive` (optional): Filter by active status

**Response:**
```json
{
  "success": true,
  "data": {
    "classes": [
      {
        "id": "class-123",
        "name": "10 IPA 1",
        "gradeLevel": 10,
        "academicYear": "2024/2025",
        "homeroomTeacher": {
          "id": "teacher-123",
          "name": "John Doe"
        },
        "studentCount": 32,
        "maxStudents": 35,
        "isActive": true
      }
    ]
  }
}
```

#### 6.2.2 Create Class

```http
POST /api/v1/admin/classes
```

**Request Body:**
```json
{
  "name": "11 IPA 2",
  "gradeLevel": 11,
  "academicYear": "2024/2025",
  "homeroomTeacherId": "teacher-456",
  "maxStudents": 35
}
```

#### 6.2.3 Update Class

```http
PUT /api/v1/admin/classes/{classId}
```

### 6.3 Subject Management

#### 6.3.1 List Subjects

```http
GET /api/v1/admin/subjects
```

**Response:**
```json
{
  "success": true,
  "data": {
    "subjects": [
      {
        "id": "subj-math",
        "code": "MAT001",
        "name": "Matematika",
        "description": "Matematika untuk Kelas 10",
        "credits": 4,
        "isActive": true
      }
    ]
  }
}
```

#### 6.3.2 Create Subject

```http
POST /api/v1/admin/subjects
```

**Request Body:**
```json
{
  "code": "PHY001",
  "name": "Fisika",
  "description": "Fisika untuk Kelas 10",
  "credits": 3
}
```

### 6.4 Schedule Management

#### 6.4.1 Create Schedule

```http
POST /api/v1/admin/schedules
```

**Request Body:**
```json
{
  "classId": "class-123",
  "teacherId": "teacher-456",
  "subjectId": "subj-math",
  "dayOfWeek": 1,
  "startTime": "07:00",
  "endTime": "08:30",
  "room": "Lab 101",
  "semester": 1,
  "academicYear": "2024/2025"
}
```

#### 6.4.2 Check Schedule Conflicts

```http
POST /api/v1/admin/schedules/conflicts
```

**Request Body:**
```json
{
  "teacherId": "teacher-456",
  "dayOfWeek": 1,
  "startTime": "07:00",
  "endTime": "08:30",
  "semester": 1,
  "academicYear": "2024/2025",
  "excludeScheduleId": "schedule-123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "hasConflicts": true,
    "conflicts": [
      {
        "type": "teacher_conflict",
        "message": "Teacher already scheduled at this time",
        "conflictingSchedule": {
          "id": "schedule-456",
          "subject": "Physics",
          "class": "10 IPA 2"
        }
      }
    ]
  }
}
```

#### 6.4.3 Publish Schedule

```http
POST /api/v1/admin/schedules/publish
```

**Request Body:**
```json
{
  "academicYear": "2024/2025",
  "semester": 1,
  "effectiveDate": "2024-01-15",
  "notificationMessage": "Schedule has been updated. Please check your new schedule."
}
```

### 6.5 Grade Oversight

#### 6.5.1 View All Grades

```http
GET /api/v1/admin/grades
```

**Query Parameters:**
- `classId` (optional): Filter by class
- `subjectId` (optional): Filter by subject
- `studentId` (optional): Filter by student
- `semester` (optional): Filter by semester
- `academicYear` (optional): Filter by academic year

**Response:**
```json
{
  "success": true,
  "data": {
    "grades": [
      {
        "id": "grade-123",
        "student": {
          "id": "student-123",
          "name": "Ahmad Rizki",
          "nis": "2024001"
        },
        "subject": {
          "id": "subj-math",
          "name": "Matematika"
        },
        "teacher": {
          "id": "teacher-456",
          "name": "John Doe"
        },
        "assessmentType": "midterm",
        "score": 85,
        "maxScore": 100,
        "assessmentDate": "2024-01-15",
        "semester": 1,
        "academicYear": "2024/2025"
      }
    ]
  }
}
```

#### 6.5.2 Adjust Grade

```http
PUT /api/v1/admin/grades/{gradeId}/adjust
```

**Request Body:**
```json
{
  "newScore": 88,
  "reason": "Grade adjustment based on re-evaluation",
  "approvedBy": "admin-123"
}
```

#### 6.5.3 Grade Analytics

```http
GET /api/v1/admin/grades/analytics
```

**Query Parameters:**
- `classId` (optional): Filter by class
- `subjectId` (optional): Filter by subject
- `semester` (optional): Filter by semester
- `academicYear` (optional): Filter by academic year

**Response:**
```json
{
  "success": true,
  "data": {
    "summary": {
      "totalGrades": 1250,
      "averageScore": 78.5,
      "highestScore": 100,
      "lowestScore": 45,
      "passRate": 0.85
    },
    "distribution": {
      "A": 0.15,
      "B": 0.35,
      "C": 0.30,
      "D": 0.15,
      "E": 0.05
    },
    "bySubject": [
      {
        "subject": "Matematika",
        "average": 75.2,
        "totalStudents": 120
      }
    ]
  }
}
```

### 6.6 Student Management

#### 6.6.1 List Students

```http
GET /api/v1/admin/students
```

#### 6.6.2 Create Student

```http
POST /api/v1/admin/students
```

**Request Body:**
```json
{
  "user": {
    "email": "student@school.edu",
    "password": "StudentPass123!",
    "profile": {
      "firstName": "Ahmad",
      "lastName": "Rizki",
      "phone": "+62-812-3456-7893"
    }
  },
  "studentData": {
    "nis": "2024001",
    "classId": "class-123",
    "enrollmentDate": "2024-01-15",
    "parentInfo": {
      "fatherName": "Budi Santoso",
      "motherName": "Siti Aminah",
      "guardianPhone": "+62-812-3456-7894"
    }
  }
}
```

#### 6.6.3 Student Transfer

```http
POST /api/v1/admin/students/{studentId}/transfer
```

**Request Body:**
```json
{
  "fromClassId": "class-123",
  "toClassId": "class-456",
  "transferDate": "2024-01-20",
  "reason": "Academic performance adjustment",
  "approvedBy": "admin-123"
}
```

---

## 7. Guru (Teacher) APIs

### 7.1 Grade Management

#### 7.1.1 Get Class Grades

```http
GET /api/v1/teacher/grades/{classId}
```

**Query Parameters:**
- `subjectId` (optional): Filter by subject
- `semester` (optional): Filter by semester
- `assessmentType` (optional): Filter by assessment type

**Response:**
```json
{
  "success": true,
  "data": {
    "class": {
      "id": "class-123",
      "name": "10 IPA 1",
      "subject": "Matematika"
    },
    "students": [
      {
        "id": "student-123",
        "name": "Ahmad Rizki",
        "nis": "2024001",
        "grades": [
          {
            "id": "grade-123",
            "assessmentType": "daily",
            "score": 85,
            "maxScore": 100,
            "assessmentDate": "2024-01-15"
          }
        ],
        "averageScore": 85.0
      }
    ],
    "summary": {
      "totalStudents": 32,
      "averageScore": 78.5,
      "submittedCount": 30,
      "pendingCount": 2
    }
  }
}
```

#### 7.1.2 Submit Grades

```http
POST /api/v1/teacher/grades
```

**Request Body:**
```json
{
  "classId": "class-123",
  "subjectId": "subj-math",
  "assessmentType": "daily",
  "assessmentDate": "2024-01-15",
  "maxScore": 100,
  "grades": [
    {
      "studentId": "student-123",
      "score": 85,
      "remarks": "Good performance"
    },
    {
      "studentId": "student-456",
      "score": 92,
      "remarks": "Excellent work"
    }
  ]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "batchId": "batch-123",
    "processedCount": 32,
    "successCount": 30,
    "errors": [
      {
        "studentId": "student-789",
        "error": "Score exceeds maximum"
      }
    ]
  }
}
```

#### 7.1.3 Update Grade

```http
PUT /api/v1/teacher/grades/{gradeId}
```

**Request Body:**
```json
{
  "score": 88,
  "remarks": "Updated after re-evaluation"
}
```

#### 7.1.4 Bulk Grade Import

```http
POST /api/v1/teacher/grades/bulk
```

**Request Body (multipart/form-data):**
```
file: grades.csv
classId: class-123
subjectId: subj-math
assessmentType: midterm
assessmentDate: 2024-01-15
maxScore: 100
```

**CSV Format:**
```csv
student_id,score,remarks
2024001,85,Good performance
2024002,92,Excellent work
```

### 7.2 Content Management

#### 7.2.1 Upload Content

```http
POST /api/v1/teacher/content/upload
```

**Request Body (multipart/form-data):**
```
file: lesson_plan.pdf
title: RPP Matematika Bab 1
description: Rencana Pelaksanaan Pembelajaran untuk bab 1
subjectId: subj-math
classId: class-123
contentType: lesson_plan
tags: ["matematika", "aljabar", "kelas-10"]
```

**Response:**
```json
{
  "success": true,
  "data": {
    "content": {
      "id": "content-123",
      "title": "RPP Matematika Bab 1",
      "description": "Rencana Pelaksanaan Pembelajaran untuk bab 1",
      "fileName": "lesson_plan.pdf",
      "fileSize": 2048576,
      "fileUrl": "https://cdn.school.edu/content/lesson_plan.pdf",
      "contentType": "lesson_plan",
      "subjectId": "subj-math",
      "classId": "class-123",
      "uploadedBy": "teacher-456",
      "uploadedAt": "2024-01-15T10:30:00.000Z",
      "tags": ["matematika", "aljabar", "kelas-10"]
    }
  }
}
```

#### 7.2.2 List Content

```http
GET /api/v1/teacher/content
```

**Query Parameters:**
- `subjectId` (optional): Filter by subject
- `classId` (optional): Filter by class
- `contentType` (optional): Filter by content type
- `tags` (optional): Filter by tags

**Response:**
```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": "content-123",
        "title": "RPP Matematika Bab 1",
        "fileName": "lesson_plan.pdf",
        "fileSize": 2048576,
        "contentType": "lesson_plan",
        "subject": {
          "id": "subj-math",
          "name": "Matematika"
        },
        "class": {
          "id": "class-123",
          "name": "10 IPA 1"
        },
        "downloadCount": 15,
        "uploadedAt": "2024-01-15T10:30:00.000Z"
      }
    ]
  }
}
```

#### 7.2.3 Update Content Metadata

```http
PUT /api/v1/teacher/content/{contentId}
```

**Request Body:**
```json
{
  "title": "Updated RPP Matematika Bab 1",
  "description": "Updated description",
  "tags": ["matematika", "aljabar", "kelas-10", "revised"]
}
```

#### 7.2.4 Delete Content

```http
DELETE /api/v1/teacher/content/{contentId}
```

#### 7.2.5 Download Content

```http
GET /api/v1/teacher/content/{contentId}/download
```

**Response:** File download with appropriate headers

### 7.3 Question Bank Management

#### 7.3.1 Create Question

```http
POST /api/v1/teacher/bank-soal
```

**Request Body:**
```json
{
  "subjectId": "subj-math",
  "question": "Jika x + 5 = 12, berapa nilai x?",
  "questionType": "multiple_choice",
  "options": [
    "5",
    "6",
    "7",
    "8"
  ],
  "correctAnswer": "6",
  "difficulty": "easy",
  "topic": "Aljabar Dasar",
  "tags": ["aljabar", "persamaan linear", "dasar"],
  "explanation": "x + 5 = 12, maka x = 12 - 5 = 7. Oops, correct answer should be 7."
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "question": {
      "id": "question-123",
      "subjectId": "subj-math",
      "question": "Jika x + 5 = 12, berapa nilai x?",
      "questionType": "multiple_choice",
      "options": ["5", "6", "7", "8"],
      "correctAnswer": "7",
      "difficulty": "easy",
      "topic": "Aljabar Dasar",
      "tags": ["aljabar", "persamaan linear", "dasar"],
      "createdBy": "teacher-456",
      "createdAt": "2024-01-15T10:30:00.000Z"
    }
  }
}
```

#### 7.3.2 List Questions

```http
GET /api/v1/teacher/bank-soal
```

**Query Parameters:**
- `subjectId` (optional): Filter by subject
- `difficulty` (optional): Filter by difficulty (`easy`, `medium`, `hard`)
- `topic` (optional): Filter by topic
- `tags` (optional): Filter by tags
- `questionType` (optional): Filter by question type

#### 7.3.3 Update Question

```http
PUT /api/v1/teacher/bank-soal/{questionId}
```

#### 7.3.4 Import Questions

```http
POST /api/v1/teacher/bank-soal/import
```

**Request Body (multipart/form-data):**
```
file: questions.csv
subjectId: subj-math
```

**CSV Format:**
```csv
question,question_type,options,correct_answer,difficulty,topic,tags
"Jika 2x = 10, berapa nilai x?",multiple_choice,"4,5,6,7",5,easy,"Aljabar","aljabar,persamaan linear"
```

#### 7.3.5 Export Questions

```http
GET /api/v1/teacher/bank-soal/export
```

**Query Parameters:**
- `format` (optional): Export format (`csv`, `json`, `excel`)
- `subjectId` (optional): Filter by subject
- `filters` (optional): Additional filters

### 7.4 Class Management

#### 7.4.1 Get My Classes

```http
GET /api/v1/teacher/classes
```

**Response:**
```json
{
  "success": true,
  "data": {
    "classes": [
      {
        "id": "class-123",
        "name": "10 IPA 1",
        "gradeLevel": 10,
        "subjects": [
          {
            "id": "subj-math",
            "name": "Matematika",
            "schedule": "Senin, 07:00-08:30"
          }
        ],
        "studentCount": 32,
        "homeroomTeacher": {
          "id": "teacher-456",
          "name": "John Doe",
          "isHomeroom": true
        }
      }
    ]
  }
}
```

#### 7.4.2 Get Class Students

```http
GET /api/v1/teacher/classes/{classId}/students
```

**Response:**
```json
{
  "success": true,
  "data": {
    "students": [
      {
        "id": "student-123",
        "nis": "2024001",
        "name": "Ahmad Rizki",
        "email": "ahmad@student.school.edu",
        "parentContact": "+62-812-3456-7894",
        "isActive": true
      }
    ]
  }
}
```

---

## 8. Siswa (Student) APIs

### 8.1 Schedule Viewing

#### 8.1.1 Get Daily Schedule

```http
GET /api/v1/student/schedule/daily
```

**Query Parameters:**
- `date` (optional): Specific date (default: today)
- `includeDetails` (optional): Include additional details (default: false)

**Response:**
```json
{
  "success": true,
  "data": {
    "date": "2024-01-15",
    "dayName": "Senin",
    "schedule": [
      {
        "id": "schedule-123",
        "time": "07:00-08:30",
        "subject": {
          "id": "subj-math",
          "name": "Matematika",
          "teacher": {
            "name": "John Doe"
          }
        },
        "room": "Lab 101",
        "status": "scheduled"
      },
      {
        "id": "schedule-456",
        "time": "08:30-10:00",
        "subject": {
          "id": "subj-physics",
          "name": "Fisika",
          "teacher": {
            "name": "Jane Smith"
          }
        },
        "room": "Lab 202",
        "status": "scheduled"
      }
    ]
  }
}
```

#### 8.1.2 Get Weekly Schedule

```http
GET /api/v1/student/schedule/weekly
```

**Query Parameters:**
- `week` (optional): Week start date (default: current week)
- `includeDetails` (optional): Include additional details

**Response:**
```json
{
  "success": true,
  "data": {
    "weekStart": "2024-01-15",
    "weekEnd": "2024-01-21",
    "schedule": {
      "senin": [
        {
          "time": "07:00-08:30",
          "subject": "Matematika",
          "room": "Lab 101",
          "teacher": "John Doe"
        }
      ],
      "selasa": [
        {
          "time": "07:00-08:30",
          "subject": "Fisika",
          "room": "Lab 202",
          "teacher": "Jane Smith"
        }
      ]
    }
  }
}
```

#### 8.1.3 Get Schedule Changes

```http
GET /api/v1/student/schedule/changes
```

**Query Parameters:**
- `fromDate` (optional): Start date for changes
- `toDate` (optional): End date for changes

**Response:**
```json
{
  "success": true,
  "data": {
    "changes": [
      {
        "type": "substitution",
        "originalSchedule": {
          "date": "2024-01-16",
          "time": "07:00-08:30",
          "subject": "Matematika",
          "teacher": "John Doe",
          "room": "Lab 101"
        },
        "newSchedule": {
          "teacher": "Jane Smith",
          "room": "Ruang 301"
        },
        "reason": "Teacher on leave",
        "notifiedAt": "2024-01-15T14:00:00.000Z"
      }
    ]
  }
}
```

### 8.2 Grade Reporting

#### 8.2.1 Get Student Grades

```http
GET /api/v1/student/grades
```

**Query Parameters:**
- `semester` (optional): Filter by semester
- `academicYear` (optional): Filter by academic year
- `subjectId` (optional): Filter by subject

**Response:**
```json
{
  "success": true,
  "data": {
    "summary": {
      "currentSemester": 1,
      "academicYear": "2024/2025",
      "overallAverage": 82.5,
      "classRank": 15,
      "totalStudents": 32
    },
    "subjects": [
      {
        "subject": {
          "id": "subj-math",
          "name": "Matematika"
        },
        "teacher": {
          "name": "John Doe"
        },
        "grades": [
          {
            "assessmentType": "daily",
            "score": 85,
            "maxScore": 100,
            "assessmentDate": "2024-01-15",
            "remarks": "Good performance"
          }
        ],
        "average": 82.0,
        "gradeDistribution": {
          "daily": 85.0,
          "midterm": 80.0,
          "final": 81.0
        }
      }
    ]
  }
}
```

#### 8.2.2 Get Grade History

```http
GET /api/v1/student/grades/history
```

#### 8.2.3 Get Progress Report

```http
GET /api/v1/student/report/progress
```

**Query Parameters:**
- `semester` (optional): Specific semester
- `academicYear` (optional): Specific academic year
- `format` (optional): Response format (`json`, `pdf`)

**Response:**
```json
{
  "success": true,
  "data": {
    "student": {
      "name": "Ahmad Rizki",
      "nis": "2024001",
      "class": "10 IPA 1",
      "semester": 1,
      "academicYear": "2024/2025"
    },
    "academicPerformance": {
      "overallAverage": 82.5,
      "classRank": 15,
      "totalSubjects": 8,
      "passedSubjects": 8,
      "failedSubjects": 0
    },
    "attendance": {
      "totalDays": 100,
      "present: 95,
      "absent": 3,
      "late": 2,
      "attendanceRate": 0.95
    },
    "subjects": [
      {
        "name": "Matematika",
        "average": 82.0,
        "rank": 12,
        "teacher": "John Doe",
        "remarks": "Good progress in algebra"
      }
    ],
    "teacherComments": [
      {
        "teacher": "John Doe",
        "subject": "Matematika",
        "comment": "Student shows good understanding of mathematical concepts",
        "date": "2024-01-15"
      }
    ]
  }
}
```

### 8.3 Practice Quizzes

#### 8.3.1 Get Available Quizzes

```http
GET /api/v1/student/quizzes
```

**Query Parameters:**
- `subjectId` (optional): Filter by subject
- `difficulty` (optional): Filter by difficulty
- `status` (optional): Filter by status (`available`, `completed`, `expired`)

**Response:**
```json
{
  "success": true,
  "data": {
    "quizzes": [
      {
        "id": "quiz-123",
        "title": "Quiz Aljabar Dasar",
        "description": "Quiz untuk materi aljabar dasar",
        "subject": {
          "id": "subj-math",
          "name": "Matematika"
        },
        "difficulty": "easy",
        "questionCount": 10,
        "timeLimit": 30,
        "attemptsAllowed": 3,
        "attemptsUsed": 1,
        "bestScore": 85,
        "status": "available",
        "expiresAt": "2024-01-31T23:59:59.000Z"
      }
    ]
  }
}
```

#### 8.3.2 Start Quiz

```http
POST /api/v1/student/quizzes/{quizId}/start
```

**Response:**
```json
{
  "success": true,
  "data": {
    "attemptId": "attempt-123",
    "quiz": {
      "id": "quiz-123",
      "title": "Quiz Aljabar Dasar",
      "timeLimit": 30,
      "questions": [
        {
          "id": "question-123",
          "question": "Jika x + 5 = 12, berapa nilai x?",
          "questionType": "multiple_choice",
          "options": ["5", "6", "7", "8"],
          "timeLimit": 60
        }
      ]
    },
    "startedAt": "2024-01-15T10:30:00.000Z",
    "expiresAt": "2024-01-15T11:00:00.000Z"
  }
}
```

#### 8.3.3 Submit Quiz Answers

```http
POST /api/v1/student/quizzes/{quizId}/submit
```

**Request Body:**
```json
{
  "attemptId": "attempt-123",
  "answers": [
    {
      "questionId": "question-123",
      "answer": "6",
      "timeSpent": 45
    }
  ],
  "completedAt": "2024-01-15T10:45:00.000Z"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "result": {
      "attemptId": "attempt-123",
      "score": 80,
      "totalQuestions": 10,
      "correctAnswers": 8,
      "timeSpent": 900,
      "completedAt": "2024-01-15T10:45:00.000Z",
      "answers": [
        {
          "questionId": "question-123",
          "userAnswer": "6",
          "correctAnswer": "7",
          "isCorrect": false,
          "explanation": "x + 5 = 12, maka x = 12 - 5 = 7"
        }
      ]
    }
  }
}
```

#### 8.3.4 Get Quiz Results

```http
GET /api/v1/student/quizzes/{quizId}/results
```

### 8.4 Financial Management (Tabungan)

#### 8.4.1 Get Savings Balance

```http
GET /api/v1/student/savings/balance
```

**Response:**
```json
{
  "success": true,
  "data": {
    "balance": 1500000,
    "currency": "IDR",
    "lastUpdated": "2024-01-15T10:30:00.000Z",
    "accountNumber": "S202400123"
  }
}
```

#### 8.4.2 Get Transaction History

```http
GET /api/v1/student/savings/transactions
```

**Query Parameters:**
- `fromDate` (optional): Start date
- `toDate` (optional): End date
- `type` (optional): Transaction type (`deposit`, `withdrawal`, `fee`)

**Response:**
```json
{
  "success": true,
  "data": {
    "transactions": [
      {
        "id": "trans-123",
        "type": "deposit",
        "amount": 100000,
        "description": "Tabungan bulanan Januari",
        "balanceAfter": 1500000,
        "transactionDate": "2024-01-15T10:30:00.000Z",
        "processedBy": "admin-123"
      }
    ],
    "summary": {
      "totalDeposits": 1500000,
      "totalWithdrawals": 0,
      "totalFees": 0,
      "netChange": 1500000
    }
  }
}
```

#### 8.4.3 Generate Statement

```http
POST /api/v1/student/savings/statement
```

**Request Body:**
```json
{
  "fromDate": "2024-01-01",
  "toDate": "2024-01-31",
  "format": "pdf",
  "includeDetails": true
}
```

### 8.5 Collaboration Tools

#### 8.5.1 Get Discussion Forums

```http
GET /api/v1/discussion/forums
```

**Query Parameters:**
- `subjectId` (optional): Filter by subject
- `classId` (optional): Filter by class

**Response:**
```json
{
  "success": true,
  "data": {
    "forums": [
      {
        "id": "forum-123",
        "title": "Diskusi Matematika Bab 1",
        "description": "Forum diskusi untuk materi aljabar dasar",
        "subject": {
          "id": "subj-math",
          "name": "Matematika"
        },
        "class": {
          "id": "class-123",
          "name": "10 IPA 1"
        },
        "createdBy": {
          "name": "John Doe",
          "role": "teacher"
        },
        "memberCount": 32,
        "messageCount": 45,
        "lastActivity": "2024-01-15T10:30:00.000Z"
      }
    ]
  }
}
```

#### 8.5.2 Get Forum Messages

```http
GET /api/v1/discussion/forums/{forumId}/messages
```

**Query Parameters:**
- `page` (optional): Page number
- `limit` (optional): Messages per page
- `after` (optional): Get messages after specific message ID

---

## 9. Authentication APIs

### 9.1 Sign In

```http
POST /api/v1/auth/signin
```

**Request Body:**
```json
{
  "email": "user@school.edu",
  "password": "userpassword123",
  "rememberMe": false
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user-123",
      "email": "user@school.edu",
      "role": "siswa",
      "profile": {
        "firstName": "Ahmad",
        "lastName": "Rizki"
      }
    },
    "session": {
      "expiresAt": "2024-01-15T11:30:00.000Z"
    }
  }
}
```

### 9.2 Sign Out

```http
POST /api/v1/auth/signout
```

### 9.3 Sign Up

```http
POST /api/v1/auth/signup
```

**Request Body:**
```json
{
  "email": "newuser@school.edu",
  "password": "newpassword123",
  "confirmPassword": "newpassword123",
  "role": "siswa",
  "profile": {
    "firstName": "New",
    "lastName": "User"
  }
}
```

### 9.4 Get Current User

```http
GET /api/v1/auth/me
```

### 9.5 Refresh Session

```http
POST /api/v1/auth/refresh
```

### 9.6 Forgot Password

```http
POST /api/v1/auth/forgot-password
```

**Request Body:**
```json
{
  "email": "user@school.edu"
}
```

### 9.7 Reset Password

```http
POST /api/v1/auth/reset-password
```

**Request Body:**
```json
{
  "token": "reset-token-123",
  "newPassword": "newpassword123",
  "confirmPassword": "newpassword123"
}
```

---

## 10. Real-time APIs (WebSocket)

### 10.1 Connection Establishment

```javascript
// WebSocket connection
const ws = new WebSocket('wss://ems.school.edu/api/v1/chat/consultation');

// Authentication message
ws.send(JSON.stringify({
  type: 'auth',
  token: 'jwt-token-or-session-id'
}));
```

### 10.2 Consultation Chat

#### 10.2.1 Connection URL

```
wss://ems.school.edu/api/v1/chat/consultation
```

#### 10.2.2 Message Format

**Client to Server:**
```json
{
  "type": "message",
  "data": {
    "receiverId": "student-123",
    "content": "How can I help you with your assignment?",
    "messageType": "text"
  }
}
```

**Server to Client:**
```json
{
  "type": "message",
  "data": {
    "id": "message-123",
    "senderId": "teacher-456",
    "receiverId": "student-123",
    "content": "How can I help you with your assignment?",
    "messageType": "text",
    "timestamp": "2024-01-15T10:30:00.000Z",
    "sender": {
      "name": "John Doe",
      "role": "teacher"
    }
  }
}
```

#### 10.2.3 Message Types

- `text`: Plain text messages
- `file`: File sharing messages
- `system`: System notifications
- `typing`: Typing indicators

#### 10.2.4 Status Messages

```json
{
  "type": "status",
  "data": {
    "userId": "student-123",
    "status": "online", // online, offline, away
    "lastSeen": "2024-01-15T10:25:00.000Z"
  }
}
```

### 10.3 Discussion Forum Real-time

#### 10.3.1 Connection URL

```
wss://ems.school.edu/api/v1/discussion/realtime
```

#### 10.3.2 Join Forum

```json
{
  "type": "join",
  "data": {
    "forumId": "forum-123"
  }
}
```

#### 10.3.3 New Message

```json
{
  "type": "new_message",
  "data": {
    "forumId": "forum-123",
    "message": {
      "id": "message-456",
      "senderId": "student-123",
      "content": "I have a question about the homework",
      "timestamp": "2024-01-15T10:30:00.000Z",
      "sender": {
        "name": "Ahmad Rizki",
        "role": "student"
      }
    }
  }
}
```

---

## 11. OpenAPI Specification

### 11.1 OpenAPI 3.0 Specification

```yaml
openapi: 3.0.3
info:
  title: Educational Management System API
  description: Comprehensive API for Educational Management System
  version: 1.0.0
  contact:
    name: EMS Development Team
    email: dev@ems.school.edu
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://ems.school.edu/api/v1
    description: Production server
  - url: https://staging.ems.school.edu/api/v1
    description: Staging server
  - url: http://localhost:3000/api/v1
    description: Development server

security:
  - bearerAuth: []
  - cookieAuth: []

paths:
  /auth/signin:
    post:
      tags:
        - Authentication
      summary: User sign in
      description: Authenticate user with email and password
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/SignInRequest'
      responses:
        '200':
          description: Successful authentication
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SignInResponse'
        '401':
          description: Invalid credentials
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

  /admin/users:
    get:
      tags:
        - Administration
      summary: List users
      description: Retrieve list of users with filtering options
      security:
        - bearerAuth: []
          cookieAuth: []
      parameters:
        - name: role
          in: query
          schema:
            type: string
            enum: [kurikulum, guru, siswa]
        - name: search
          in: query
          schema:
            type: string
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UsersListResponse'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    cookieAuth:
      type: apiKey
      in: cookie
      name: session-token

  schemas:
    SignInRequest:
      type: object
      required:
        - email
        - password
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8
        rememberMe:
          type: boolean
          default: false

    SignInResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: object
          properties:
            user:
              $ref: '#/components/schemas/User'
            session:
              type: object
              properties:
                expiresAt:
                  type: string
                  format: date-time

    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        role:
          type: string
          enum: [kurikulum, guru, siswa]
        profile:
          $ref: '#/components/schemas/UserProfile'
        isActive:
          type: boolean

    UserProfile:
      type: object
      properties:
        firstName:
          type: string
        lastName:
          type: string
        phone:
          type: string
        avatar:
          type: string
          format: uri

    ErrorResponse:
      type: object
      properties:
        success:
          type: boolean
          example: false
        error:
          type: object
          properties:
            code:
              type: string
            message:
              type: string
            details:
              type: object

    UsersListResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: object
          properties:
            users:
              type: array
              items:
                $ref: '#/components/schemas/User'
            pagination:
              $ref: '#/components/schemas/Pagination'

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer
        hasNext:
          type: boolean
        hasPrev:
          type: boolean
```

### 11.2 API Documentation Generation

The API documentation can be automatically generated using:

1. **Swagger UI**: Interactive API documentation
2. **Redoc**: Developer-friendly documentation
3. **Postman Collections**: API testing collections

### 11.3 SDK Generation

API clients can be generated for various languages:

- **TypeScript/JavaScript**: `@openapitools/openapi-generator-cli`
- **Python**: `openapi-python-client`
- **Java**: `openapi-generator`
- **PHP**: `openapi-generator`

---

## Conclusion

This API specification provides comprehensive documentation for all endpoints required to support the Educational Management System. The API follows RESTful principles and includes proper authentication, authorization, error handling, and real-time communication capabilities.

Key features of the API design:

- **Role-based access control** ensuring data security
- **Consistent response format** for easy client integration
- **Comprehensive error handling** with detailed error codes
- **Real-time communication** through WebSocket endpoints
- **Rate limiting** to ensure system stability
- **OpenAPI specification** for automated client generation

The API is designed to be scalable, secure, and developer-friendly, supporting all the functional requirements of the EMS system while maintaining best practices for API design and implementation.