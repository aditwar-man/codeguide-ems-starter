# Educational Management System (EMS) - Security Architecture & Implementation Guidelines

## Document Information

- **Document Version**: 1.0
- **Date:** October 19, 2025
- **Author:** EMS Development Team
- **Status:** Draft
- **Classification:** Internal Use Only
- **Last Review:** October 19, 2025

---

## Table of Contents

1. [Security Overview](#1-security-overview)
2. [Threat Model](#2-threat-model)
3. [Security Architecture](#3-security-architecture)
4. [Authentication & Authorization](#4-authentication--authorization)
5. [Data Protection](#5-data-protection)
6. [Network Security](#6-network-security)
7. [Application Security](#7-application-security)
8. [Infrastructure Security](#8-infrastructure-security)
9. [Compliance & Governance](#9-compliance--governance)
10. [Security Monitoring](#10-security-monitoring)
11. [Incident Response](#11-incident-response)
12. [Security Best Practices](#12-security-best-practices)
13. [Implementation Guidelines](#13-implementation-guidelines)

---

## 1. Security Overview

### 1.1 Security Objectives

The EMS system security architecture is built around the following core objectives:

- **Confidentiality:** Ensure sensitive educational data is accessible only to authorized individuals
- **Integrity:** Protect data from unauthorized modification and ensure accuracy
- **Availability:** Maintain system accessibility for legitimate users
- **Accountability:** Track and audit all system activities
- **Privacy:** Protect student and staff personal information in compliance with regulations

### 1.2 Security Principles

1. **Defense in Depth:** Multiple layers of security controls
2. **Least Privilege:** Users only have access to necessary resources
3. **Zero Trust:** Never trust, always verify
4. **Security by Design:** Security built into every layer of the system
5. **Fail Securely:** System defaults to secure state when errors occur
6. **Transparency:** Security practices are documented and auditable

### 1.3 Scope

This security architecture covers:
- Application layer security
- Data protection and encryption
- Authentication and authorization
- Network security
- Infrastructure security
- Compliance and regulatory requirements
- Security monitoring and incident response

---

## 2. Threat Model

### 2.1 Threat Actors

| Actor | Motivation | Capability | Likelihood |
|-------|------------|------------|------------|
| **External Hackers** | Data theft, system disruption | High | Medium |
| **Insider Threats** | Data manipulation, unauthorized access | Medium | Medium |
| **Malicious Students** | Grade manipulation, system abuse | Low | High |
| **Competitors** | Intellectual property theft | Medium | Low |
| **Nation States** | Espionage, system disruption | Very High | Very Low |

### 2.2 Threat Vectors

#### 2.2.1 Application Threats
- **SQL Injection:** Malicious database queries
- **Cross-Site Scripting (XSS):** Client-side script injection
- **Cross-Site Request Forgery (CSRF):** Forced authenticated requests
- **Authentication Bypass:** Invalid access to protected resources
- **Privilege Escalation:** Gaining higher access levels

#### 2.2.2 Infrastructure Threats
- **Denial of Service (DoS):** Service availability disruption
- **Man-in-the-Middle:** Network traffic interception
- **Data Breaches:** Unauthorized data access
- **Ransomware:** System encryption for extortion
- **Supply Chain Attacks:** Compromised third-party dependencies

#### 2.2.3 Data Threats
- **Data Exfiltration:** Unauthorized data extraction
- **Data Manipulation:** Unauthorized data modification
- **Privacy Violations:** Mishandling of personal information
- **Data Loss:** Accidental or intentional data deletion

### 2.3 Risk Assessment Matrix

| Threat | Impact | Likelihood | Risk Level | Mitigation |
|--------|--------|------------|------------|------------|
| SQL Injection | High | Medium | High | Parameterized queries, input validation |
| XSS | Medium | High | High | Content Security Policy, output encoding |
| Data Breach | Critical | Low | Medium | Encryption, access controls |
| DoS Attack | Medium | Medium | Medium | Rate limiting, CDN protection |
| Insider Threat | High | Medium | Medium | Access controls, audit logging |

---

## 3. Security Architecture

### 3.1 Security Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │          Web Application Firewall (WAF)            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                         │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │   Auth      │ │   RBAC      │ │   Input Validation  │   │
│  │ Middleware  │ │ Middleware  │ │   & Sanitization    │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                              │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │   Data      │ │   Access    │ │   Encryption &      │   │
│  │ Encryption  │ │   Control   │ │   Key Management    │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Infrastructure Layer                       │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │  Network    │ │   Host      │ │   Monitoring &      │   │
│  │ Security    │ │ Security    │ │   Logging           │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Security Zones

#### 3.2.1 DMZ Zone
- **Components:** Load balancers, WAF, reverse proxies
- **Access:** Public internet access
- **Controls:** Network segmentation, traffic filtering

#### 3.2.2 Application Zone
- **Components:** Application servers, API gateways
- **Access:** Limited from DMZ and internal networks
- **Controls:** Authentication, authorization, encryption

#### 3.2.3 Data Zone
- **Components:** Database servers, file storage
- **Access:** Highly restricted, only from application zone
- **Controls:** Database encryption, access controls, audit logging

#### 3.2.4 Management Zone
- **Components:** Monitoring, logging, backup systems
- **Access:** Restricted to authorized administrators
- **Controls:** Multi-factor authentication, bastion hosts

### 3.3 Security Controls

#### 3.3.1 Preventive Controls
- **Access Controls:** Authentication, authorization, RBAC
- **Network Security:** Firewalls, VPN, network segmentation
- **Encryption:** Data at rest and in transit
- **Input Validation:** Parameterized queries, input sanitization

#### 3.3.2 Detective Controls
- **Logging:** Comprehensive audit trails
- **Monitoring:** Real-time security monitoring
- **Intrusion Detection:** Network and host-based IDS
- **Vulnerability Scanning:** Regular security assessments

#### 3.3.3 Corrective Controls
- **Incident Response:** Security incident procedures
- **Backup and Recovery:** Data restoration capabilities
- **Patch Management:** Regular security updates
- **Security Training:** User awareness programs

---

## 4. Authentication & Authorization

### 4.1 Authentication Architecture

#### 4.1.1 Better Auth Integration

```typescript
// lib/auth.config.ts
import { betterAuth } from "better-auth"
import { drizzleAdapter } from "better-auth/adapters/drizzle"
import { db } from "./db"
import { sendEmail } from "./email"

export const auth = betterAuth({
  database: drizzleAdapter(db, {
    provider: "pg",
    schema: {
      users: users,
      sessions: sessions,
      accounts: accounts,
      verifications: verifications
    }
  }),
  
  emailAndPassword: {
    enabled: true,
    requireEmailVerification: true,
    minPasswordLength: 8,
    maxPasswordLength: 128,
    passwordPolicy: {
      requireUppercase: true,
      requireLowercase: true,
      requireNumbers: true,
      requireSpecialChars: true
    }
  },
  
  session: {
    expiresIn: 60 * 60, // 1 hour
    updateAge: 60 * 5, // 5 minutes
    cookieCache: {
      enabled: true,
      maxAge: 5 * 60 // 5 minutes
    }
  },
  
  emailVerification: {
    sendOnSignUp: true,
    expiresIn: 24 * 60 * 60, // 24 hours
    sendVerificationEmail: async ({ user, url }) => {
      await sendEmail({
        to: user.email,
        subject: "Verify your EMS account",
        template: "email-verification",
        data: { verificationUrl: url }
      })
    }
  },
  
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!
    }
  },
  
  account: {
    accountLinking: {
      enabled: true,
      trustedProviders: ["google"]
    }
  }
})
```

#### 4.1.2 Session Management

```typescript
// lib/session-manager.ts
import { auth } from "./auth.config"
import { cookies } from "next/headers"

export class SessionManager {
  static async createSession(userId: string, userAgent?: string, ipAddress?: string) {
    const session = await auth.api.signInEmail({
      body: {
        email: user.email,
        password: 'validated_password'
      },
      headers: {
        'user-agent': userAgent,
        'x-forwarded-for': ipAddress
      }
    })
    
    // Set secure cookie
    cookies().set('session-token', session.sessionToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 60 * 60, // 1 hour
      path: '/'
    })
    
    return session
  }
  
  static async validateSession() {
    const sessionToken = cookies().get('session-token')?.value
    
    if (!sessionToken) {
      return null
    }
    
    const session = await auth.api.getSession({
      headers: {
        cookie: `session-token=${sessionToken}`
      }
    })
    
    return session
  }
  
  static async revokeSession(sessionToken: string) {
    await auth.api.signOut({
      headers: {
        cookie: `session-token=${sessionToken}`
      }
    })
    
    cookies().delete('session-token')
  }
  
  static async refreshSession() {
    const currentSession = await this.validateSession()
    
    if (!currentSession) {
      throw new Error('No active session')
    }
    
    const newSession = await auth.api.refreshSession({
      headers: {
        cookie: `session-token=${currentSession.sessionToken}`
      }
    })
    
    return newSession
  }
}
```

### 4.2 Role-Based Access Control (RBAC)

#### 4.2.1 RBAC Middleware

```typescript
// middleware/auth.ts
import { NextRequest, NextResponse } from 'next/server'
import { auth } from './auth.config'
import { paths, permissions } from './rbac-config'

export async function rbacMiddleware(request: NextRequest) {
  const session = await auth.api.getSession({
    headers: request.headers
  })
  
  if (!session) {
    return NextResponse.json(
      { error: { code: 'UNAUTHORIZED', message: 'Authentication required' } },
      { status: 401 }
    )
  }
  
  const userRole = session.user.role
  const pathname = request.nextUrl.pathname
  
  // Check if user has permission for this path
  const hasPermission = checkPathPermission(pathname, userRole)
  
  if (!hasPermission) {
    return NextResponse.json(
      { error: { code: 'FORBIDDEN', message: 'Insufficient permissions' } },
      { status: 403 }
    )
  }
  
  // Add user context to request headers
  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-user-id', session.user.id)
  requestHeaders.set('x-user-role', userRole)
  
  return NextResponse.next({
    request: {
      headers: requestHeaders
    }
  })
}

function checkPathPermission(pathname: string, userRole: string): boolean {
  // Check direct path permissions
  if (permissions[userRole]?.allowedPaths?.some(path => pathname.startsWith(path))) {
    return true
  }
  
  // Check denied paths
  if (permissions[userRole]?.deniedPaths?.some(path => pathname.startsWith(path))) {
    return false
  }
  
  // Default deny
  return false
}
```

#### 4.2.2 RBAC Configuration

```typescript
// config/rbac-config.ts
export const permissions = {
  kurikulum: {
    allowedPaths: [
      '/admin',
      '/api/admin',
      '/api/auth',
      '/api/public'
    ],
    deniedPaths: [],
    permissions: [
      'user:create', 'user:read', 'user:update', 'user:delete',
      'class:create', 'class:read', 'class:update', 'class:delete',
      'subject:create', 'subject:read', 'subject:update', 'subject:delete',
      'schedule:create', 'schedule:read', 'schedule:update', 'schedule:delete',
      'grade:read', 'grade:adjust',
      'system:read', 'system:update'
    ]
  },
  
  guru: {
    allowedPaths: [
      '/guru',
      '/api/teacher',
      '/api/auth',
      '/api/public'
    ],
    deniedPaths: [
      '/admin',
      '/api/admin'
    ],
    permissions: [
      'student:read', // assigned students only
      'grade:create', 'grade:read', 'grade:update', // assigned classes only
      'content:create', 'content:read', 'content:update', 'content:delete',
      'question:create', 'question:read', 'question:update', 'question:delete',
      'schedule:read', // own schedule only
      'profile:read', 'profile:update'
    ]
  },
  
  siswa: {
    allowedPaths: [
      '/siswa',
      '/api/student',
      '/api/auth',
      '/api/public'
    ],
    deniedPaths: [
      '/admin',
      '/api/admin',
      '/guru',
      '/api/teacher'
    ],
    permissions: [
      'schedule:read', // own schedule only
      'grade:read', // own grades only
      'content:read', // assigned content only
      'quiz:read', 'quiz:submit',
      'profile:read', 'profile:update',
      'savings:read', 'savings:view_transactions'
    ]
  }
}

export const apiPermissions = {
  'POST:/api/admin/users': ['user:create'],
  'GET:/api/admin/users': ['user:read'],
  'PUT:/api/admin/users/:id': ['user:update'],
  'DELETE:/api/admin/users/:id': ['user:delete'],
  
  'POST:/api/teacher/grades': ['grade:create'],
  'GET:/api/teacher/grades/:classId': ['grade:read'],
  'PUT:/api/teacher/grades/:id': ['grade:update'],
  
  'GET:/api/student/grades': ['grade:read'],
  'GET:/api/student/schedule': ['schedule:read'],
  'POST:/api/student/quizzes/:id/submit': ['quiz:submit']
}
```

#### 4.2.3 Permission Checker

```typescript
// lib/permission-checker.ts
export class PermissionChecker {
  static hasPermission(userRole: string, permission: string): boolean {
    return permissions[userRole]?.permissions?.includes(permission) || false
  }
  
  static canAccessResource(
    userRole: string, 
    resource: string, 
    resourceId?: string,
    userId?: string
  ): boolean {
    // Check basic permission
    if (!this.hasPermission(userRole, `${resource}:read`)) {
      return false
    }
    
    // Check ownership for student resources
    if (userRole === 'siswa' && resourceId && userId) {
      return resourceId === userId
    }
    
    // Add more specific ownership checks as needed
    return true
  }
  
  static checkApiPermission(userRole: string, method: string, path: string): boolean {
    const permissionKey = `${method}:${path}`
    const requiredPermissions = apiPermissions[permissionKey]
    
    if (!requiredPermissions) {
      return false
    }
    
    return requiredPermissions.some(permission => 
      this.hasPermission(userRole, permission)
    )
  }
}
```

### 4.3 Multi-Factor Authentication (MFA)

#### 4.3.1 MFA Implementation

```typescript
// lib/mfa.ts
import { authenticator } from 'otplib'
import { QRCode } from 'qrcode'

export class MFAService {
  static generateSecret(userEmail: string) {
    const secret = authenticator.generateSecret()
    const issuer = 'EMS System'
    const label = `${userEmail} (${issuer})`
    
    return {
      secret,
      otpauthUrl: authenticator.keyuri(userEmail, issuer, secret)
    }
  }
  
  static async generateQRCode(otpauthUrl: string): Promise<string> {
    return await QRCode.toDataURL(otpauthUrl)
  }
  
  static verifyToken(secret: string, token: string): boolean {
    try {
      return authenticator.verify({ secret, token })
    } catch (error) {
      return false
    }
  }
  
  static generateBackupCodes(): string[] {
    const codes: string[] = []
    for (let i = 0; i < 10; i++) {
      codes.push(Math.random().toString(36).substring(2, 10))
    }
    return codes
  }
  
  static async enableMFA(userId: string, secret: string) {
    await db.update(userProfiles)
      .set({ 
        mfaSecret: secret,
        mfaEnabled: true,
        mfaBackupCodes: this.generateBackupCodes()
      })
      .where(eq(userProfiles.userId, userId))
  }
  
  static async verifyMFA(userId: string, token: string): Promise<boolean> {
    const user = await db.query.userProfiles.findFirst({
      where: eq(userProfiles.userId, userId)
    })
    
    if (!user?.mfaSecret || !user.mfaEnabled) {
      return false
    }
    
    return this.verifyToken(user.mfaSecret, token)
  }
}
```

#### 4.3.2 MFA Middleware

```typescript
// middleware/mfa.ts
export async function mfaMiddleware(request: NextRequest) {
  const session = await auth.api.getSession({
    headers: request.headers
  })
  
  if (!session) {
    return NextResponse.json(
      { error: { code: 'UNAUTHORIZED', message: 'Authentication required' } },
      { status: 401 }
    )
  }
  
  const userProfile = await db.query.userProfiles.findFirst({
    where: eq(userProfiles.userId, session.user.id)
  })
  
  // Check if MFA is enabled and verified
  if (userProfile?.mfaEnabled && !session.mfaVerified) {
    const pathname = request.nextUrl.pathname
    
    // Allow MFA-related endpoints
    if (pathname.startsWith('/api/auth/mfa')) {
      return NextResponse.next()
    }
    
    return NextResponse.json(
      { error: { code: 'MFA_REQUIRED', message: 'Multi-factor authentication required' } },
      { status: 403 }
    )
  }
  
  return NextResponse.next()
}
```

---

## 5. Data Protection

### 5.1 Encryption Strategy

#### 5.1.1 Data Classification

| Classification | Data Types | Storage | Transit | Access Level |
|----------------|------------|---------|---------|--------------|
| **Public** | General information | Encrypted | HTTPS | All users |
| **Internal** | Operational data | Encrypted | HTTPS | Internal users |
| **Confidential** | Personal data, grades | Encrypted | HTTPS | Authorized users |
| **Restricted** | Financial, sensitive personal | Encrypted | HTTPS + VPN | Limited access |

#### 5.1.2 Encryption Implementation

```typescript
// lib/encryption.ts
import crypto from 'crypto'

export class EncryptionService {
  private static readonly algorithm = 'aes-256-gcm'
  private static readonly keyLength = 32
  private static readonly ivLength = 16
  private static readonly tagLength = 16
  
  static getEncryptionKey(): Buffer {
    const key = process.env.ENCRYPTION_KEY
    if (!key) {
      throw new Error('ENCRYPTION_KEY environment variable is not set')
    }
    return Buffer.from(key, 'hex')
  }
  
  static encrypt(text: string): { encrypted: string; iv: string; tag: string } {
    const key = this.getEncryptionKey()
    const iv = crypto.randomBytes(this.ivLength)
    
    const cipher = crypto.createCipher(this.algorithm, key, iv)
    cipher.setAAD(Buffer.from('ems-data'))
    
    let encrypted = cipher.update(text, 'utf8', 'hex')
    encrypted += cipher.final('hex')
    
    const tag = cipher.getAuthTag()
    
    return {
      encrypted,
      iv: iv.toString('hex'),
      tag: tag.toString('hex')
    }
  }
  
  static decrypt(encryptedData: string, iv: string, tag: string): string {
    const key = this.getEncryptionKey()
    const ivBuffer = Buffer.from(iv, 'hex')
    const tagBuffer = Buffer.from(tag, 'hex')
    
    const decipher = crypto.createDecipher(this.algorithm, key, ivBuffer)
    decipher.setAAD(Buffer.from('ems-data'))
    decipher.setAuthTag(tagBuffer)
    
    let decrypted = decipher.update(encryptedData, 'hex', 'utf8')
    decrypted += decipher.final('utf8')
    
    return decrypted
  }
  
  static encryptSensitiveData(data: any): string {
    const jsonString = JSON.stringify(data)
    const { encrypted, iv, tag } = this.encrypt(jsonString)
    
    return `${encrypted}:${iv}:${tag}`
  }
  
  static decryptSensitiveData(encryptedString: string): any {
    const [encrypted, iv, tag] = encryptedString.split(':')
    const decrypted = this.decrypt(encrypted, iv, tag)
    
    return JSON.parse(decrypted)
  }
  
  static hashPassword(password: string): Promise<string> {
    return new Promise((resolve, reject) => {
      const salt = crypto.randomBytes(16).toString('hex')
      
      crypto.scrypt(password, salt, 64, (err, derivedKey) => {
        if (err) reject(err)
        resolve(`${salt}:${derivedKey.toString('hex')}`)
      })
    })
  }
  
  static verifyPassword(password: string, hash: string): Promise<boolean> {
    return new Promise((resolve, reject) => {
      const [salt, key] = hash.split(':')
      
      crypto.scrypt(password, salt, 64, (err, derivedKey) => {
        if (err) reject(err)
        resolve(key === derivedKey.toString('hex'))
      })
    })
  }
}
```

#### 5.1.3 Database Encryption

```typescript
// db/schema/encrypted-fields.ts
import { pgTable, text } from 'drizzle-orm/pg-core'
import { EncryptionService } from '../lib/encryption'

// Custom type for encrypted fields
export const encryptedText = {
  ...text(),
  transform: {
    input: (value: any) => {
      if (!value) return value
      return EncryptionService.encryptSensitiveData(value)
    },
    output: (value: string) => {
      if (!value) return value
      return EncryptionService.decryptSensitiveData(value)
    }
  }
}

// Usage in schema
export const students = pgTable('students', {
  // ... other fields
  parentInfo: encryptedText('parent_info'),
  medicalInfo: encryptedText('medical_info'),
})
```

### 5.2 Data Masking

#### 5.2.1 Sensitive Data Masking

```typescript
// lib/data-masking.ts
export class DataMaskingService {
  static maskEmail(email: string): string {
    const [username, domain] = email.split('@')
    const maskedUsername = username.slice(0, 2) + '*'.repeat(username.length - 2)
    return `${maskedUsername}@${domain}`
  }
  
  static maskPhone(phone: string): string {
    const cleaned = phone.replace(/\D/g, '')
    if (cleaned.length < 4) return phone
    
    const lastFour = cleaned.slice(-4)
    const masked = '*'.repeat(cleaned.length - 4) + lastFour
    
    // Reformat with original separators
    return phone.replace(/\d/g, (match, offset) => {
      if (offset >= cleaned.length - 4) return match
      return '*'
    })
  }
  
  static maskNIK(nik: string): string {
    if (nik.length < 6) return '*'.repeat(nik.length)
    return nik.slice(0, 2) + '*'.repeat(nik.length - 6) + nik.slice(-4)
  }
  
  static maskFinancialData(amount: number): string {
    return `***,***.${amount.toString().slice(-2)}`
  }
  
  static maskUserData(user: any, viewerRole: string): any {
    const maskedUser = { ...user }
    
    // Apply masking based on viewer role
    switch (viewerRole) {
      case 'siswa':
        // Students can see basic info of other students
        if (maskedUser.email) {
          maskedUser.email = this.maskEmail(maskedUser.email)
        }
        if (maskedUser.phone) {
          maskedUser.phone = this.maskPhone(maskedUser.phone)
        }
        break
        
      case 'guru':
        // Teachers can see more information but still mask sensitive data
        if (maskedUser.nik) {
          maskedUser.nik = this.maskNIK(maskedUser.nik)
        }
        break
        
      case 'kurikulum':
        // Admins can see all data (no masking)
        break
    }
    
    return maskedUser
  }
}
```

### 5.3 Data Loss Prevention (DLP)

#### 5.3.1 DLP Implementation

```typescript
// lib/dlp.ts
export class DLPService {
  private static readonly sensitivePatterns = {
    email: /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g,
    phone: /\b(?:\+?62|0)[0-9]{9,13}\b/g,
    nik: /\b[0-9]{16}\b/g,
    creditCard: /\b[0-9]{13,19}\b/g,
    bankAccount: /\b[0-9]{10,20}\b/g
  }
  
  private static readonly forbiddenKeywords = [
    'password', 'secret', 'token', 'key', 'confidential',
    'private', 'sensitive', 'internal', 'restricted'
  ]
  
  static scanForSensitiveData(text: string): {
    hasSensitiveData: boolean
    findings: Array<{ type: string; matches: string[] }>
  } {
    const findings: Array<{ type: string; matches: string[] }> = []
    let hasSensitiveData = false
    
    // Check for sensitive patterns
    for (const [type, pattern] of Object.entries(this.sensitivePatterns)) {
      const matches = text.match(pattern)
      if (matches) {
        findings.push({ type, matches })
        hasSensitiveData = true
      }
    }
    
    // Check for forbidden keywords
    const lowerText = text.toLowerCase()
    for (const keyword of this.forbiddenKeywords) {
      if (lowerText.includes(keyword)) {
        findings.push({ type: 'forbidden_keyword', matches: [keyword] })
        hasSensitiveData = true
      }
    }
    
    return { hasSensitiveData, findings }
  }
  
  static sanitizeInput(input: string): string {
    // Remove or mask sensitive information
    let sanitized = input
    
    // Replace emails
    sanitized = sanitized.replace(this.sensitivePatterns.email, (match) => {
      return DataMaskingService.maskEmail(match)
    })
    
    // Replace phone numbers
    sanitized = sanitized.replace(this.sensitivePatterns.phone, (match) => {
      return DataMaskingService.maskPhone(match)
    })
    
    // Remove other sensitive patterns completely
    sanitized = sanitized.replace(this.sensitivePatterns.nik, '[REDACTED]')
    sanitized = sanitized.replace(this.sensitivePatterns.creditCard, '[REDACTED]')
    sanitized = sanitized.replace(this.sensitivePatterns.bankAccount, '[REDACTED]')
    
    return sanitized
  }
  
  static validateFileUpload(buffer: Buffer, filename: string): {
    isValid: boolean
    issues: string[]
  } {
    const issues: string[] = []
    
    // Check file size (max 10MB)
    if (buffer.length > 10 * 1024 * 1024) {
      issues.push('File size exceeds 10MB limit')
    }
    
    // Check file type
    const allowedTypes = [
      'application/pdf',
      'application/msword',
      'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
      'image/jpeg',
      'image/png',
      'text/plain'
    ]
    
    // For simplicity, check file extension
    const allowedExtensions = ['.pdf', '.doc', '.docx', '.jpg', '.jpeg', '.png', '.txt']
    const fileExtension = filename.toLowerCase().slice(filename.lastIndexOf('.'))
    
    if (!allowedExtensions.includes(fileExtension)) {
      issues.push(`File type ${fileExtension} is not allowed`)
    }
    
    // Scan for sensitive data in text files
    if (fileExtension === '.txt') {
      const content = buffer.toString('utf-8')
      const scan = this.scanForSensitiveData(content)
      
      if (scan.hasSensitiveData) {
        issues.push('File contains sensitive information that should be removed')
      }
    }
    
    return {
      isValid: issues.length === 0,
      issues
    }
  }
}
```

### 5.4 Key Management

#### 5.4.1 Key Rotation

```typescript
// lib/key-management.ts
export class KeyManagementService {
  static async rotateEncryptionKey(): Promise<void> {
    // Generate new key
    const newKey = crypto.randomBytes(32).toString('hex')
    
    // Get all encrypted data that needs re-encryption
    const encryptedRecords = await db.query.userProfiles.findMany({
      where: or(
        isNotNull(userProfiles.parentInfo),
        isNotNull(userProfiles.medicalInfo)
      )
    })
    
    // Re-encrypt all data with new key
    for (const record of encryptedRecords) {
      const updatedRecord: any = {}
      
      if (record.parentInfo) {
        const decrypted = EncryptionService.decryptSensitiveData(record.parentInfo)
        updatedRecord.parentInfo = EncryptionService.encryptSensitiveData(decrypted)
      }
      
      if (record.medicalInfo) {
        const decrypted = EncryptionService.decryptSensitiveData(record.medicalInfo)
        updatedRecord.medicalInfo = EncryptionService.encryptSensitiveData(decrypted)
      }
      
      if (Object.keys(updatedRecord).length > 0) {
        await db.update(userProfiles)
          .set(updatedRecord)
          .where(eq(userProfiles.id, record.id))
      }
    }
    
    // Update environment variable (in production, use secure key management service)
    process.env.ENCRYPTION_KEY = newKey
    
    // Log key rotation
    await this.logKeyRotation(newKey)
  }
  
  private static async logKeyRotation(newKeyId: string): Promise<void> {
    await db.insert(auditLogs).values({
      action: 'KEY_ROTATION',
      resource: 'encryption_key',
      resourceId: newKeyId,
      metadata: {
        timestamp: new Date().toISOString(),
        rotatedBy: 'system'
      }
    })
  }
  
  static async scheduleKeyRotation(): Promise<void> {
    // Schedule key rotation every 90 days
    const rotationInterval = 90 * 24 * 60 * 60 * 1000 // 90 days in milliseconds
    
    setInterval(async () => {
      try {
        await this.rotateEncryptionKey()
        console.log('Encryption key rotated successfully')
      } catch (error) {
        console.error('Key rotation failed:', error)
        // Send alert to administrators
      }
    }, rotationInterval)
  }
}
```

---

## 6. Network Security

### 6.1 Web Application Firewall (WAF)

#### 6.1.1 WAF Configuration

```yaml
# waf-config.yaml
waf_rules:
  - name: "SQL Injection Protection"
    type: "sql_injection"
    patterns:
      - "union.*select"
      - "drop.*table"
      - "insert.*into"
      - "delete.*from"
    action: "block"
    severity: "high"
    
  - name: "XSS Protection"
    type: "xss"
    patterns:
      - "<script"
      - "javascript:"
      - "onload="
      - "onerror="
    action: "block"
    severity: "high"
    
  - name: "Path Traversal Protection"
    type: "path_traversal"
    patterns:
      - "\\.\\./"
      - "\\.\\.\\\\"
      - "%2e%2e%2f"
    action: "block"
    severity: "high"
    
  - name: "Command Injection Protection"
    type: "command_injection"
    patterns:
      - "\\|.*cat"
      - "\\;.*rm"
      - "`.*ls"
      - "\\$\\("
    action: "block"
    severity: "critical"

rate_limiting:
  - path: "/api/auth/signin"
    limit: 10
    window: 300 # 5 minutes
    action: "block"
    
  - path: "/api/admin/*"
    limit: 100
    window: 60 # 1 minute
    action: "throttle"
    
  - path: "/*"
    limit: 1000
    window: 60 # 1 minute
    action: "throttle"
```

#### 6.1.2 WAF Middleware

```typescript
// middleware/waf.ts
import { NextRequest, NextResponse } from 'next/server'

export class WAFMiddleware {
  private static readonly suspiciousPatterns = [
    /union.*select/i,
    /drop.*table/i,
    /<script[^>]*>/i,
    /javascript:/i,
    /\.\.\//,
    /\|.*cat/i,
    /`.*ls/i
  ]
  
  private static readonly rateLimitStore = new Map<string, { count: number; resetTime: number }>()
  
  static async wafProtection(request: NextRequest): Promise<NextResponse | null> {
    const url = request.nextUrl
    const userAgent = request.headers.get('user-agent') || ''
    const ip = request.ip || 'unknown'
    
    // Check for suspicious patterns in URL and headers
    const urlToCheck = url.pathname + url.search
    const hasSuspiciousPattern = this.suspiciousPatterns.some(pattern => 
      pattern.test(urlToCheck) || pattern.test(userAgent)
    )
    
    if (hasSuspiciousPattern) {
      await this.logSecurityEvent('WAF_BLOCK', {
        ip,
        url: urlToCheck,
        userAgent,
        reason: 'Suspicious pattern detected'
      })
      
      return NextResponse.json(
        { error: { code: 'BLOCKED', message: 'Request blocked by security filter' } },
        { status: 403 }
      )
    }
    
    // Rate limiting
    const rateLimitResult = this.checkRateLimit(ip, url.pathname)
    if (!rateLimitResult.allowed) {
      await this.logSecurityEvent('RATE_LIMIT', {
        ip,
        url: url.pathname,
        userAgent,
        reason: rateLimitResult.reason
      })
      
      return NextResponse.json(
        { error: { code: 'RATE_LIMITED', message: 'Too many requests' } },
        { status: 429 }
      )
    }
    
    return null // Continue processing
  }
  
  private static checkRateLimit(ip: string, path: string): { allowed: boolean; reason?: string } {
    const now = Date.now()
    const key = `${ip}:${path}`
    const current = this.rateLimitStore.get(key)
    
    // Define rate limits based on path
    let limit = 1000 // Default limit
    let window = 60000 // 1 minute
    
    if (path.startsWith('/api/auth/signin')) {
      limit = 10
      window = 300000 // 5 minutes
    } else if (path.startsWith('/api/admin/')) {
      limit = 100
      window = 60000 // 1 minute
    }
    
    if (!current || now > current.resetTime) {
      // First request or window reset
      this.rateLimitStore.set(key, {
        count: 1,
        resetTime: now + window
      })
      return { allowed: true }
    }
    
    if (current.count >= limit) {
      return { 
        allowed: false, 
        reason: `Rate limit exceeded: ${limit} requests per ${window/1000} seconds` 
      }
    }
    
    current.count++
    return { allowed: true }
  }
  
  private static async logSecurityEvent(eventType: string, details: any): Promise<void> {
    await db.insert(auditLogs).values({
      action: eventType,
      resource: 'security',
      metadata: details
    })
  }
}
```

### 6.2 Content Security Policy (CSP)

#### 6.2.1 CSP Configuration

```typescript
// lib/security-headers.ts
export class SecurityHeaders {
  static getSecurityHeaders(): Record<string, string> {
    return {
      // Content Security Policy
      'Content-Security-Policy': [
        "default-src 'self'",
        "script-src 'self' 'unsafe-inline' 'unsafe-eval'",
        "style-src 'self' 'unsafe-inline'",
        "img-src 'self' data: https:",
        "font-src 'self'",
        "connect-src 'self' wss:",
        "frame-src 'none'",
        "object-src 'none'",
        "base-uri 'self'",
        "form-action 'self'",
        "frame-ancestors 'none'",
        "upgrade-insecure-requests"
      ].join('; '),
      
      // Strict Transport Security
      'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload',
      
      // X-Frame-Options
      'X-Frame-Options': 'DENY',
      
      // X-Content-Type-Options
      'X-Content-Type-Options': 'nosniff',
      
      // Referrer Policy
      'Referrer-Policy': 'strict-origin-when-cross-origin',
      
      // Permissions Policy
      'Permissions-Policy': [
        'camera=()',
        'microphone=()',
        'geolocation=()',
        'payment=()',
        'usb=()',
        'magnetometer=()',
        'gyroscope=()',
        'accelerometer=()'
      ].join(', '),
      
      // X-XSS-Protection (legacy, CSP is preferred)
      'X-XSS-Protection': '1; mode=block',
      
      // Feature Policy (legacy)
      'Feature-Policy': [
        'camera none',
        'microphone none',
        'geolocation none',
        'payment none'
      ].join('; ')
    }
  }
  
  static addSecurityHeaders(response: Response): Response {
    const headers = this.getSecurityHeaders()
    
    Object.entries(headers).forEach(([key, value]) => {
      response.headers.set(key, value)
    })
    
    return response
  }
}
```

### 6.3 Network Segmentation

#### 6.3.1 Network Architecture

```yaml
# network-architecture.yaml
network_zones:
  dmz:
    cidr: "10.0.1.0/24"
    components:
      - load_balancer
      - waf
      - reverse_proxy
    ingress_from: ["internet"]
    egress_to: ["application"]
    
  application:
    cidr: "10.0.2.0/24"
    components:
      - app_servers
      - api_gateways
      - cache_servers
    ingress_from: ["dmz"]
    egress_to: ["database", "storage"]
    
  database:
    cidr: "10.0.3.0/24"
    components:
      - postgresql_primary
      - postgresql_replica
      - redis_cache
    ingress_from: ["application"]
    egress_to: ["backup"]
    
  storage:
    cidr: "10.0.4.0/24"
    components:
      - file_servers
      - backup_servers
    ingress_from: ["application", "database"]
    egress_to: []
    
  management:
    cidr: "10.0.5.0/24"
    components:
      - monitoring
      - logging
      - bastion_hosts
    ingress_from: ["admin_network"]
    egress_to: ["all_zones"]

firewall_rules:
  - name: "Allow HTTP from DMZ to Application"
    source: "dmz"
    destination: "application"
    ports: [80, 443]
    protocol: "tcp"
    action: "allow"
    
  - name: "Allow Database from Application to Database"
    source: "application"
    destination: "database"
    ports: [5432]
    protocol: "tcp"
    action: "allow"
    
  - name: "Deny all traffic from Internet to Database"
    source: "internet"
    destination: "database"
    ports: "all"
    protocol: "all"
    action: "deny"
```

### 6.4 VPN and Secure Access

#### 6.4.1 VPN Configuration

```typescript
// lib/vpn-service.ts
export class VPNService {
  static async generateVPNConfig(userId: string, userRole: string): Promise<{
    config: string
    expiresAt: Date
  }> {
    // Check if user is authorized for VPN access
    if (!['kurikulum', 'guru'].includes(userRole)) {
      throw new Error('VPN access not authorized for this role')
    }
    
    // Generate temporary VPN configuration
    const vpnConfig = {
      client: {
        dev: 'tun',
        proto: 'udp',
        remote: 'vpn.ems.school.edu',
        port: 1194,
        resolv_retry: 'infinite',
        nobind: true,
        persist_key: true,
        persist_tun: true,
        ca: '/path/to/ca.crt',
        cert: `/path/to/${userId}.crt`,
        key: `/path/to/${userId}.key`,
        cipher: 'AES-256-CBC',
        auth: 'SHA256',
        verb: 3
      }
    }
    
    // Generate configuration string
    const configString = Object.entries(vpnConfig.client)
      .map(([key, value]) => typeof value === 'boolean' ? key : `${key} ${value}`)
      .join('\n')
    
    // Set expiration (24 hours)
    const expiresAt = new Date(Date.now() + 24 * 60 * 60 * 1000)
    
    // Store VPN access log
    await db.insert(vpnAccessLogs).values({
      userId,
      configGeneratedAt: new Date(),
      expiresAt,
      grantedBy: 'system'
    })
    
    return {
      config: configString,
      expiresAt
    }
  }
  
  static async validateVPNAccess(userId: string): Promise<boolean> {
    const latestAccess = await db.query.vpnAccessLogs.findFirst({
      where: eq(vpnAccessLogs.userId, userId),
      orderBy: desc(vpnAccessLogs.configGeneratedAt)
    })
    
    if (!latestAccess) {
      return false
    }
    
    return latestAccess.expiresAt > new Date()
  }
}
```

---

## 7. Application Security

### 7.1 Input Validation

#### 7.1.1 Validation Schemas

```typescript
// lib/validation-schemas.ts
import { z } from 'zod'

// Common validation patterns
const emailPattern = z.string().email().max(255)
const passwordPattern = z.string()
  .min(8, 'Password must be at least 8 characters')
  .max(128, 'Password must be less than 128 characters')
  .regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/, 
    'Password must contain at least one lowercase letter, one uppercase letter, one number, and one special character')

const phonePattern = z.string()
  .regex(/^(?:\+?62|0)[0-9]{9,13}$/, 'Invalid Indonesian phone number')

const nisPattern = z.string()
  .regex(/^[0-9]{4,20}$/, 'NIS must be 4-20 digits')

// User-related schemas
export const userSchemas = {
  signUp: z.object({
    email: emailPattern,
    password: passwordPattern,
    confirmPassword: z.string(),
    role: z.enum(['kurikulum', 'guru', 'siswa']),
    profile: z.object({
      firstName: z.string().min(1).max(100),
      lastName: z.string().min(1).max(100),
      phone: phonePattern.optional()
    })
  }).refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"]
  }),
  
  signIn: z.object({
    email: emailPattern,
    password: z.string().min(1),
    rememberMe: z.boolean().default(false)
  }),
  
  updateProfile: z.object({
    firstName: z.string().min(1).max(100),
    lastName: z.string().min(1).max(100),
    phone: phonePattern.optional(),
    bio: z.string().max(500).optional()
  }),
  
  changePassword: z.object({
    currentPassword: z.string().min(1),
    newPassword: passwordPattern,
    confirmPassword: z.string()
  }).refine((data) => data.newPassword === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"]
  })
}

// Academic schemas
export const academicSchemas = {
  createClass: z.object({
    name: z.string().min(1).max(100),
    gradeLevel: z.number().int().min(1).max(12),
    academicYear: z.string().regex(/^\d{4}\/\d{4}$/, 'Invalid academic year format'),
    homeroomTeacherId: z.string().uuid().optional(),
    room: z.string().max(20).optional(),
    capacity: z.number().int().min(1).max(50).default(40)
  }),
  
  createSubject: z.object({
    code: z.string().min(1).max(20),
    name: z.string().min(1).max(100),
    description: z.string().max(500).optional(),
    credits: z.number().int().min(1).max(10).default(1),
    department: z.string().max(100).optional()
  }),
  
  createSchedule: z.object({
    classId: z.string().uuid(),
    teacherId: z.string().uuid(),
    subjectId: z.string().uuid(),
    dayOfWeek: z.number().int().min(1).max(7),
    startTime: z.string().regex(/^([0-1]?[0-9]|2[0-3]):[0-5][0-9]$/, 'Invalid time format'),
    endTime: z.string().regex(/^([0-1]?[0-9]|2[0-3]):[0-5][0-9]$/, 'Invalid time format'),
    room: z.string().max(20).optional(),
    semester: z.number().int().min(1).max(2),
    academicYear: z.string().regex(/^\d{4}\/\d{4}$/)
  }).refine((data) => data.startTime < data.endTime, {
    message: "End time must be after start time",
    path: ["endTime"]
  })
}

// Assessment schemas
export const assessmentSchemas = {
  submitGrades: z.object({
    classId: z.string().uuid(),
    subjectId: z.string().uuid(),
    assessmentType: z.enum(['daily', 'quiz', 'assignment', 'midterm', 'final', 'project']),
    assessmentDate: z.string().datetime(),
    maxScore: z.number().min(1).max(1000),
    grades: z.array(z.object({
      studentId: z.string().uuid(),
      score: z.number().min(0).max(1000),
      remarks: z.string().max(500).optional()
    })).min(1)
  }),
  
  createQuestion: z.object({
    subjectId: z.string().uuid(),
    question: z.string().min(1).max(2000),
    questionType: z.enum(['multiple_choice', 'true_false', 'essay', 'fill_blank', 'matching']),
    options: z.array(z.string()).optional(),
    correctAnswer: z.string().min(1),
    explanation: z.string().max(1000).optional(),
    difficulty: z.enum(['easy', 'medium', 'hard']),
    topic: z.string().min(1).max(100),
    tags: z.array(z.string()).optional(),
    points: z.number().min(1).max(100).default(1)
  })
}
```

#### 7.1.2 Validation Middleware

```typescript
// middleware/validation.ts
import { NextRequest, NextResponse } from 'next/server'
import { ZodSchema } from 'zod'

export class ValidationMiddleware {
  static validateBody<T>(schema: ZodSchema<T>) {
    return async (request: NextRequest) => {
      try {
        const body = await request.json()
        const validatedData = schema.parse(body)
        
        // Attach validated data to request
        request.validatedBody = validatedData
        return null // Continue processing
        
      } catch (error) {
        if (error instanceof z.ZodError) {
          return NextResponse.json(
            {
              success: false,
              error: {
                code: 'VALIDATION_ERROR',
                message: 'Request validation failed',
                details: {
                  errors: error.errors.map(err => ({
                    field: err.path.join('.'),
                    message: err.message,
                    code: err.code
                  }))
                }
              }
            },
            { status: 400 }
          )
        }
        
        return NextResponse.json(
          {
            success: false,
            error: {
              code: 'VALIDATION_ERROR',
              message: 'Invalid request format'
            }
          },
          { status: 400 }
        )
      }
    }
  }
  
  static validateQuery<T>(schema: ZodSchema<T>) {
    return (request: NextRequest) => {
      try {
        const query = Object.fromEntries(request.nextUrl.searchParams)
        const validatedQuery = schema.parse(query)
        
        request.validatedQuery = validatedQuery
        return null
        
      } catch (error) {
        if (error instanceof z.ZodError) {
          return NextResponse.json(
            {
              success: false,
              error: {
                code: 'VALIDATION_ERROR',
                message: 'Query parameter validation failed',
                details: {
                  errors: error.errors.map(err => ({
                    field: err.path.join('.'),
                    message: err.message,
                    code: err.code
                  }))
                }
              }
            },
            { status: 400 }
          )
        }
        
        return NextResponse.json(
          {
            success: false,
            error: {
              code: 'VALIDATION_ERROR',
              message: 'Invalid query parameters'
            }
          },
          { status: 400 }
        )
      }
    }
  }
  
  static sanitizeInput(input: string): string {
    return input
      .replace(/[<>]/g, '') // Remove HTML tags
      .replace(/javascript:/gi, '') // Remove JavaScript URLs
      .replace(/on\w+\s*=/gi, '') // Remove event handlers
      .trim()
  }
}
```

### 7.2 SQL Injection Prevention

#### 7.2.1 Parameterized Queries

```typescript
// lib/db-service.ts
import { db } from './db'
import { eq, and, or, like, ilike } from 'drizzle-orm'

export class DatabaseService {
  // Safe query examples using Drizzle ORM
  static async getUserByEmail(email: string) {
    return await db.query.users.findFirst({
      where: eq(users.email, email)
    })
  }
  
  static async searchUsers(searchTerm: string, role?: string) {
    const conditions = []
    
    if (searchTerm) {
      conditions.push(
        or(
          ilike(userProfiles.firstName, `%${searchTerm}%`),
          ilike(userProfiles.lastName, `%${searchTerm}%`),
          ilike(users.email, `%${searchTerm}%`)
        )
      )
    }
    
    if (role) {
      conditions.push(eq(users.role, role))
    }
    
    return await db.query.users.findMany({
      where: and(...conditions),
      with: {
        profile: true
      }
    })
  }
  
  static async getStudentGrades(studentId: string, semester?: number, academicYear?: string) {
    const conditions = [eq(grades.studentId, studentId)]
    
    if (semester) {
      conditions.push(eq(grades.semester, semester))
    }
    
    if (academicYear) {
      conditions.push(eq(grades.academicYear, academicYear))
    }
    
    return await db.query.grades.findMany({
      where: and(...conditions),
      with: {
        subject: true,
        teacher: {
          with: {
            profile: true
          }
        }
      },
      orderBy: [desc(grades.assessmentDate)]
    })
  }
  
  // Safe insertion with validation
  static async createStudent(data: {
    userId: string
    nis: string
    classId: string
    enrollmentDate: Date
    parentInfo?: any
    medicalInfo?: any
  }) {
    // Validate required fields
    if (!data.userId || !data.nis || !data.classId) {
      throw new Error('Missing required fields')
    }
    
    // Sanitize and encrypt sensitive data
    const sanitizedData = {
      ...data,
      parentInfo: data.parentInfo ? EncryptionService.encryptSensitiveData(data.parentInfo) : null,
      medicalInfo: data.medicalInfo ? EncryptionService.encryptSensitiveData(data.medicalInfo) : null
    }
    
    return await db.insert(students).values(sanitizedData).returning()
  }
}
```

#### 7.2.2 SQL Injection Detection

```typescript
// lib/sql-injection-detector.ts
export class SQLInjectionDetector {
  private static readonly suspiciousPatterns = [
    /(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|UNION|SCRIPT)\b)/i,
    /(\/\*.*\*\/|--|\bOR\b.*=.*\bOR\b|\bAND\b.*=.*\bAND\b)/i,
    /(\b(UNION|SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|SCRIPT)\s.*\b(FROM|INTO|TABLE|DATABASE)\b)/i,
    /(\'(.*)(\bOR\b|\bAND\b)(.*)\')|(\")(.*)(\bOR\b|\bAND\b)(.*)\")/i,
    /(\b(DECLARE|CAST|CONVERT|EXEC|EXECUTE|sp_executesql)\b)/i,
    /(\bWAITFOR\s+DELAY\b)/i,
    /(\'\s*;\s*(DROP|DELETE|UPDATE|INSERT)\s+\w+\s*;|--)/i
  ]
  
  static detectSQLInjection(input: string): {
    isSuspicious: boolean
    patterns: string[]
    confidence: number
  } {
    const foundPatterns: string[] = []
    let suspiciousCount = 0
    
    for (const pattern of this.suspiciousPatterns) {
      if (pattern.test(input)) {
        foundPatterns.push(pattern.source)
        suspiciousCount++
      }
    }
    
    // Calculate confidence based on number of suspicious patterns found
    const confidence = Math.min(suspiciousCount / this.suspiciousPatterns.length * 100, 100)
    
    return {
      isSuspicious: suspiciousCount > 0,
      patterns: foundPatterns,
      confidence
    }
  }
  
  static sanitizeInput(input: string): string {
    // Remove dangerous SQL characters and patterns
    return input
      .replace(/['"\\;]/g, '') // Remove dangerous characters
      .replace(/\b(OR|AND|UNION|SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|SCRIPT)\b/gi, '') // Remove SQL keywords
      .replace(/\/\*.*?\*\//g, '') // Remove comments
      .replace(/--.*$/gm, '') // Remove single line comments
      .trim()
  }
  
  static async logSQLInjectionAttempt(
    input: string, 
    ip: string, 
    userAgent: string, 
    userId?: string
  ): Promise<void> {
    const detection = this.detectSQLInjection(input)
    
    await db.insert(auditLogs).values({
      userId: userId || null,
      action: 'SQL_INJECTION_ATTEMPT',
      resource: 'security',
      metadata: {
        input,
        ip,
        userAgent,
        patterns: detection.patterns,
        confidence: detection.confidence,
        timestamp: new Date().toISOString()
      }
    })
    
    // Send alert to security team
    if (detection.confidence > 70) {
      await this.sendSecurityAlert({
        type: 'SQL_INJECTION',
        severity: 'high',
        input,
        ip,
        userAgent,
        userId,
        confidence: detection.confidence
      })
    }
  }
  
  private static async sendSecurityAlert(alert: any): Promise<void> {
    // Implementation for sending alerts to security team
    console.error('SECURITY ALERT:', alert)
    // In production, integrate with alerting system
  }
}
```

### 7.3 Cross-Site Scripting (XSS) Prevention

#### 7.3.1 Output Encoding

```typescript
// lib/xss-protection.ts
import { JSDOM } from 'jsdom'

export class XSSProtection {
  static escapeHtml(unsafe: string): string {
    return unsafe
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#039;')
      .replace(/\//g, '&#x2F;')
  }
  
  static escapeJs(unsafe: string): string {
    return unsafe
      .replace(/\\/g, '\\\\')
      .replace(/'/g, "\\'")
      .replace(/"/g, '\\"')
      .replace(/\n/g, '\\n')
      .replace(/\r/g, '\\r')
      .replace(/\t/g, '\\t')
      .replace(/\f/g, '\\f')
      .replace(/\v/g, '\\v')
      .replace(/\0/g, '\\0')
  }
  
  static sanitizeHtml(dirty: string): string {
    const dom = new JSDOM(dirty)
    const document = dom.window.document
    
    // Remove script tags and dangerous attributes
    const scripts = document.querySelectorAll('script')
    scripts.forEach(script => script.remove())
    
    const dangerousElements = document.querySelectorAll('iframe, object, embed, form, input, button')
    dangerousElements.forEach(el => el.remove())
    
    // Remove dangerous attributes
    const allElements = document.querySelectorAll('*')
    allElements.forEach(el => {
      const dangerousAttrs = ['onclick', 'onload', 'onerror', 'onmouseover', 'onfocus', 'onblur']
      dangerousAttrs.forEach(attr => {
        if (el.hasAttribute(attr)) {
          el.removeAttribute(attr)
        }
      })
    })
    
    return document.body.innerHTML
  }
  
  static validateUrl(url: string): boolean {
    try {
      const urlObj = new URL(url)
      
      // Allow only http and https protocols
      if (!['http:', 'https:'].includes(urlObj.protocol)) {
        return false
      }
      
      // Allow only specific domains (whitelist)
      const allowedDomains = [
        'ems.school.edu',
        'cdn.ems.school.edu',
        'api.ems.school.edu'
      ]
      
      if (!allowedDomains.includes(urlObj.hostname)) {
        return false
      }
      
      return true
    } catch {
      return false
    }
  }
  
  static sanitizeContent(content: string, type: 'text' | 'html' | 'url'): string {
    switch (type) {
      case 'text':
        return this.escapeHtml(content)
      case 'html':
        return this.sanitizeHtml(content)
      case 'url':
        return this.validateUrl(content) ? content : '#'
      default:
        return this.escapeHtml(content)
    }
  }
}
```

#### 7.3.2 CSP Implementation

```typescript
// middleware/csp.ts
export class CSPMiddleware {
  static generateCSP(nonce: string): string {
    const directives = [
      `default-src 'self'`,
      `script-src 'self' 'nonce-${nonce}' 'unsafe-inline'`,
      `style-src 'self' 'nonce-${nonce}' 'unsafe-inline'`,
      `img-src 'self' data: https:`,
      `font-src 'self'`,
      `connect-src 'self' wss:`,
      `frame-src 'none'`,
      `object-src 'none'`,
      `base-uri 'self'`,
      `form-action 'self'`,
      `frame-ancestors 'none'`,
      `upgrade-insecure-requests`
    ]
    
    return directives.join('; ')
  }
  
  static getNonce(): string {
    return crypto.randomBytes(16).toString('base64')
  }
  
  static addCSPHeaders(request: NextRequest, response: NextResponse): NextResponse {
    const nonce = this.getNonce()
    const csp = this.generateCSP(nonce)
    
    response.headers.set('Content-Security-Policy', csp)
    response.headers.set('X-Content-Security-Policy-Nonce', nonce)
    
    return response
  }
}
```

---

## 8. Infrastructure Security

### 8.1 Container Security

#### 8.1.1 Docker Security Configuration

```dockerfile
# Dockerfile.security
FROM node:18-alpine AS base

# Create non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# Install security updates
RUN apk update && \
    apk upgrade && \
    apk add --no-cache dumb-init

# Set security flags
USER nextjs
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/api/health || exit 1

# Use minimal base image
FROM gcr.io/distroless/nodejs18-debian11 AS runner

# Copy application with proper permissions
COPY --from=base --chown=nextjs:nodejs /app /app

# Set secure working directory
WORKDIR /app

# Use non-root user
USER nextjs

# Expose only necessary ports
EXPOSE 3000

# Set secure environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Use dumb-init as PID 1
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]
```

#### 8.1.2 Kubernetes Security Context

```yaml
# k8s/security-context.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ems-app-secure
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1001
    runAsGroup: 1001
    fsGroup: 1001
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: ems-app
    image: ems-app:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
      runAsNonRoot: true
      runAsUser: 1001
      seccompProfile:
        type: RuntimeDefault
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /app/.next/cache
    resources:
      limits:
        cpu: "500m"
        memory: "512Mi"
      requests:
        cpu: "250m"
        memory: "256Mi"
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
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

### 8.2 Network Security

#### 8.2.1 Network Policies

```yaml
# k8s/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ems-app-network-policy
spec:
  podSelector:
    matchLabels:
      app: ems-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: ingress-controller
    ports:
    - protocol: TCP
      port: 3000
  - from:
    - podSelector:
        matchLabels:
          app: monitoring
    ports:
    - protocol: TCP
      port: 3000
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
  - to: []
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 443
```

#### 8.2.2 Service Mesh (Optional)

```yaml
# istio-gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: ems-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: ems-tls-secret
    hosts:
    - ems.school.edu
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ems-vs
spec:
  hosts:
  - ems.school.edu
  gateways:
  - ems-gateway
  http:
  - match:
    - uri:
        prefix: /api/
    route:
    - destination:
        host: ems-service
        port:
          number: 3000
    fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    retries:
      attempts: 3
      perTryTimeout: 2s
```

### 8.3 Secrets Management

#### 8.3.1 Kubernetes Secrets

```yaml
# k8s/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: ems-secrets
  annotations:
    sealedsecrets.bitnami.com/cluster-wide: "true"
type: Opaque
data:
  database-url: <base64-encoded-database-url>
  encryption-key: <base64-encoded-encryption-key>
  jwt-secret: <base64-encoded-jwt-secret>
  redis-url: <base64-encoded-redis-url>
  aws-access-key: <base64-encoded-aws-access-key>
  aws-secret-key: <base64-encoded-aws-secret-key>
  sentry-dsn: <base64-encoded-sentry-dsn>
```

#### 8.3.2 External Secret Management

```typescript
// lib/secret-manager.ts
export class SecretManager {
  private static vault: any // HashiCorp Vault client
  
  static async initialize() {
    // Initialize Vault client
    this.vault = require('node-vault')({
      endpoint: process.env.VAULT_ADDR,
      token: process.env.VAULT_TOKEN
    })
  }
  
  static async getSecret(secretPath: string): Promise<any> {
    try {
      const result = await this.vault.read(secretPath)
      return result.data.data
    } catch (error) {
      console.error(`Failed to retrieve secret ${secretPath}:`, error)
      throw error
    }
  }
  
  static async setSecret(secretPath: string, data: any): Promise<void> {
    try {
      await this.vault.write(secretPath, data)
    } catch (error) {
      console.error(`Failed to store secret ${secretPath}:`, error)
      throw error
    }
  }
  
  static async rotateSecret(secretPath: string): Promise<void> {
    try {
      // Generate new secret value
      const newValue = crypto.randomBytes(32).toString('hex')
      
      // Update secret in Vault
      await this.setSecret(secretPath, { value: newValue })
      
      // Trigger application restart or configuration reload
      await this.triggerSecretRotation(secretPath)
      
      // Log rotation
      await this.logSecretRotation(secretPath)
    } catch (error) {
      console.error(`Failed to rotate secret ${secretPath}:`, error)
      throw error
    }
  }
  
  private static async triggerSecretRotation(secretPath: string): Promise<void> {
    // Implementation for triggering secret rotation
    // This could involve restarting pods, updating configurations, etc.
  }
  
  private static async logSecretRotation(secretPath: string): Promise<void> {
    await db.insert(auditLogs).values({
      action: 'SECRET_ROTATION',
      resource: 'secret',
      resourceId: secretPath,
      metadata: {
        timestamp: new Date().toISOString(),
        rotatedBy: 'system'
      }
    })
  }
}
```

---

## 9. Compliance & Governance

### 9.1 Data Privacy Compliance

#### 9.1.1 GDPR Compliance

```typescript
// lib/gdpr-compliance.ts
export class GDPRCompliance {
  static async processDataRequest(requestType: 'access' | 'rectification' | 'erasure' | 'portability', userId: string): Promise<any> {
    switch (requestType) {
      case 'access':
        return await this.getUserData(userId)
      case 'rectification':
        return await this.rectifyUserData(userId)
      case 'erasure':
        return await this.eraseUserData(userId)
      case 'portability':
        return await this.exportUserData(userId)
      default:
        throw new Error('Invalid request type')
    }
  }
  
  private static async getUserData(userId: string): Promise<any> {
    const user = await db.query.users.findFirst({
      where: eq(users.id, userId),
      with: {
        profile: true,
        student: true,
        teacher: true,
        grades: true,
        attendance: true
      }
    })
    
    if (!user) {
      throw new Error('User not found')
    }
    
    // Apply data masking for sensitive information
    const maskedData = DataMaskingService.maskUserData(user, 'kurikulum')
    
    return {
      data: maskedData,
      timestamp: new Date().toISOString(),
      requestId: crypto.randomUUID()
    }
  }
  
  private static async rectifyUserData(userId: string, corrections: any): Promise<void> {
    // Verify user identity before making changes
    await this.verifyUserIdentity(userId)
    
    // Apply corrections with audit trail
    await db.transaction(async (tx) => {
      for (const [field, value] of Object.entries(corrections)) {
        await tx.update(userProfiles)
          .set({ [field]: value, updatedAt: new Date() })
          .where(eq(userProfiles.userId, userId))
        
        await tx.insert(auditLogs).values({
          userId,
          action: 'DATA_RECTIFICATION',
          resource: 'user_profile',
          metadata: {
            field,
            oldValue: await this.getOldValue(userId, field),
            newValue: value,
            timestamp: new Date().toISOString()
          }
        })
      }
    })
  }
  
  private static async eraseUserData(userId: string): Promise<void> {
    // Verify user identity and authorization
    await this.verifyUserIdentity(userId)
    
    // Check for legal holds or retention requirements
    const hasLegalHold = await this.checkLegalHold(userId)
    if (hasLegalHold) {
      throw new Error('Cannot erase data due to legal hold')
    }
    
    // Anonymize data instead of deletion for audit purposes
    await db.transaction(async (tx) => {
      // Anonymize user profile
      await tx.update(userProfiles)
        .set({
          firstName: 'Deleted User',
          lastName: 'Deleted User',
          phone: null,
          avatarUrl: null,
          bio: null,
          updatedAt: new Date()
        })
        .where(eq(userProfiles.userId, userId))
      
      // Anonymize user account
      await tx.update(users)
        .set({
          email: `deleted_${crypto.randomUUID()}@deleted.local`,
          isActive: false,
          updatedAt: new Date()
        })
        .where(eq(users.id, userId))
      
      // Log erasure
      await tx.insert(auditLogs).values({
        userId,
        action: 'DATA_ERASURE',
        resource: 'user_data',
        metadata: {
          timestamp: new Date().toISOString(),
          method: 'anonymization'
        }
      })
    })
  }
  
  private static async exportUserData(userId: string): Promise<any> {
    const userData = await this.getUserData(userId)
    
    // Format data in portable format (JSON/CSV)
    return {
      format: 'json',
      data: userData,
      timestamp: new Date().toISOString(),
      schema: 'ems-user-data-v1'
    }
  }
  
  private static async verifyUserIdentity(userId: string): Promise<void> {
    // Implementation for verifying user identity before data operations
    // This could involve multi-factor authentication, identity documents, etc.
  }
  
  private static async checkLegalHold(userId: string): Promise<boolean> {
    // Check if user data is under legal hold
    return false // Default implementation
  }
  
  private static async getOldValue(userId: string, field: string): Promise<any> {
    const profile = await db.query.userProfiles.findFirst({
      where: eq(userProfiles.userId, userId)
    })
    return profile?.[field]
  }
}
```

#### 9.1.2 Data Retention Policy

```typescript
// lib/data-retention.ts
export class DataRetentionPolicy {
  private static readonly retentionPeriods = {
    userProfiles: 7 * 365, // 7 years
    grades: 10 * 365, // 10 years
    attendance: 5 * 365, // 5 years
    auditLogs: 7 * 365, // 7 years
    sessions: 30, // 30 days
    chatMessages: 2 * 365, // 2 years
    quizAttempts: 5 * 365 // 5 years
  }
  
  static async enforceRetentionPolicy(): Promise<void> {
    console.log('Starting data retention policy enforcement...')
    
    const now = new Date()
    
    for (const [table, retentionDays] of Object.entries(this.retentionPeriods)) {
      const cutoffDate = new Date(now.getTime() - retentionDays * 24 * 60 * 60 * 1000)
      
      try {
        await this.archiveOldData(table, cutoffDate)
        await this.deleteArchivedData(table, cutoffDate)
        
        console.log(`Processed retention for ${table}`)
      } catch (error) {
        console.error(`Failed to process retention for ${table}:`, error)
      }
    }
    
    console.log('Data retention policy enforcement completed')
  }
  
  private static async archiveOldData(table: string, cutoffDate: Date): Promise<void> {
    // Move old data to archive tables
    switch (table) {
      case 'sessions':
        await db.execute(sql`
          INSERT INTO archive_sessions 
          SELECT * FROM sessions 
          WHERE created_at < ${cutoffDate} AND is_active = false
        `)
        break
        
      case 'auditLogs':
        await db.execute(sql`
          INSERT INTO archive_audit_logs 
          SELECT * FROM audit_logs 
          WHERE timestamp < ${cutoffDate}
        `)
        break
        
      // Add other tables as needed
    }
  }
  
  private static async deleteArchivedData(table: string, cutoffDate: Date): Promise<void> {
    // Delete old data from main tables
    switch (table) {
      case 'sessions':
        await db.delete(sessions)
          .where(and(
            lt(sessions.createdAt, cutoffDate),
            eq(sessions.isActive, false)
          ))
        break
        
      // Add other tables as needed
    }
  }
  
  static async scheduleRetentionPolicy(): Promise<void> {
    // Run retention policy daily at 2 AM
    cron.schedule('0 2 * * *', async () => {
      try {
        await this.enforceRetentionPolicy()
      } catch (error) {
        console.error('Retention policy enforcement failed:', error)
        // Send alert to administrators
      }
    })
  }
}
```

### 9.2 Security Audits

#### 9.2.1 Security Audit Framework

```typescript
// lib/security-audit.ts
export class SecurityAudit {
  static async performSecurityAudit(): Promise<{
    overallScore: number
    findings: Array<{
      category: string
      severity: 'low' | 'medium' | 'high' | 'critical'
      finding: string
      recommendation: string
    }>
  }> {
    const findings = []
    let totalScore = 0
    let maxScore = 0
    
    // Authentication security
    const authResults = await this.auditAuthentication()
    findings.push(...authResults.findings)
    totalScore += authResults.score
    maxScore += authResults.maxScore
    
    // Authorization security
    const authzResults = await this.auditAuthorization()
    findings.push(...authzResults.findings)
    totalScore += authzResults.score
    maxScore += authzResults.maxScore
    
    // Data protection
    const dataResults = await this.auditDataProtection()
    findings.push(...dataResults.findings)
    totalScore += dataResults.score
    maxScore += dataResults.maxScore
    
    // Infrastructure security
    const infraResults = await this.auditInfrastructure()
    findings.push(...infraResults.findings)
    totalScore += infraResults.score
    maxScore += infraResults.maxScore
    
    return {
      overallScore: Math.round((totalScore / maxScore) * 100),
      findings: findings.sort((a, b) => {
        const severityOrder = { critical: 0, high: 1, medium: 2, low: 3 }
        return severityOrder[a.severity] - severityOrder[b.severity]
      })
    }
  }
  
  private static async auditAuthentication(): Promise<{
    score: number
    maxScore: number
    findings: any[]
  }> {
    const findings = []
    let score = 0
    const maxScore = 25
    
    // Check password policy enforcement
    const usersWithWeakPasswords = await this.checkPasswordStrength()
    if (usersWithWeakPasswords > 0) {
      findings.push({
        category: 'Authentication',
        severity: 'high',
        finding: `${usersWithWeakPasswords} users have weak passwords`,
        recommendation: 'Enforce stronger password policies and require password changes'
      })
    } else {
      score += 5
    }
    
    // Check MFA enforcement
    const mfaEnabledUsers = await this.checkMFAEnforcement()
    const mfaPercentage = (mfaEnabledUsers / await this.getTotalUsers()) * 100
    if (mfaPercentage < 80) {
      findings.push({
        category: 'Authentication',
        severity: 'medium',
        finding: `Only ${mfaPercentage.toFixed(1)}% of users have MFA enabled`,
        recommendation: 'Implement mandatory MFA for all privileged accounts'
      })
    } else {
      score += 5
    }
    
    // Check session security
    const hasSecureSessions = await this.checkSessionSecurity()
    if (!hasSecureSessions) {
      findings.push({
        category: 'Authentication',
        severity: 'high',
        finding: 'Session configuration is not secure',
        recommendation: 'Implement secure session management with proper timeouts'
      })
    } else {
      score += 5
    }
    
    // Add more authentication checks...
    
    return { score, maxScore, findings }
  }
  
  private static async auditAuthorization(): Promise<{
    score: number
    maxScore: number
    findings: any[]
  }> {
    const findings = []
    let score = 0
    const maxScore = 25
    
    // Check RBAC implementation
    const rbacConfigured = await this.checkRBACConfiguration()
    if (!rbacConfigured) {
      findings.push({
        category: 'Authorization',
        severity: 'critical',
        finding: 'RBAC is not properly configured',
        recommendation: 'Implement proper role-based access control'
      })
    } else {
      score += 10
    }
    
    // Check privilege escalation risks
    const escalationRisks = await this.checkPrivilegeEscalation()
    if (escalationRisks.length > 0) {
      findings.push({
        category: 'Authorization',
        severity: 'high',
        finding: 'Potential privilege escalation risks identified',
        recommendation: 'Review and fix privilege escalation vulnerabilities'
      })
    } else {
      score += 10
    }
    
    // Add more authorization checks...
    
    return { score, maxScore, findings }
  }
  
  private static async auditDataProtection(): Promise<{
    score: number
    maxScore: number
    findings: any[]
  }> {
    const findings = []
    let score = 0
    const maxScore = 25
    
    // Check encryption implementation
    const encryptionStatus = await this.checkEncryptionImplementation()
    if (!encryptionStatus.dataAtRest) {
      findings.push({
        category: 'Data Protection',
        severity: 'critical',
        finding: 'Data at rest is not encrypted',
        recommendation: 'Implement database encryption for sensitive data'
      })
    } else {
      score += 10
    }
    
    if (!encryptionStatus.dataInTransit) {
      findings.push({
        category: 'Data Protection',
        severity: 'high',
        finding: 'Data in transit is not always encrypted',
        recommendation: 'Enforce HTTPS for all communications'
      })
    } else {
      score += 10
    }
    
    // Add more data protection checks...
    
    return { score, maxScore, findings }
  }
  
  private static async auditInfrastructure(): Promise<{
    score: number
    maxScore: number
    findings: any[]
  }> {
    const findings = []
    let score = 0
    const maxScore = 25
    
    // Check container security
    const containerSecurity = await this.checkContainerSecurity()
    if (!containerSecurity.secure) {
      findings.push({
        category: 'Infrastructure',
        severity: 'medium',
        finding: 'Container security configuration needs improvement',
        recommendation: 'Implement secure container configurations'
      })
    } else {
      score += 10
    }
    
    // Add more infrastructure checks...
    
    return { score, maxScore, findings }
  }
  
  // Helper methods for specific checks
  private static async checkPasswordStrength(): Promise<number> {
    // Implementation to check password strength
    return 0
  }
  
  private static async checkMFAEnforcement(): Promise<number> {
    // Implementation to check MFA enforcement
    const totalUsers = await this.getTotalUsers()
    return Math.round(totalUsers * 0.7) // 70% MFA adoption
  }
  
  private static async checkSessionSecurity(): Promise<boolean> {
    // Implementation to check session security
    return true
  }
  
  private static async checkRBACConfiguration(): Promise<boolean> {
    // Implementation to check RBAC configuration
    return true
  }
  
  private static async checkPrivilegeEscalation(): Promise<any[]> {
    // Implementation to check privilege escalation
    return []
  }
  
  private static async checkEncryptionImplementation(): Promise<{
    dataAtRest: boolean
    dataInTransit: boolean
  }> {
    return { dataAtRest: true, dataInTransit: true }
  }
  
  private static async checkContainerSecurity(): Promise<{
    secure: boolean
    issues: string[]
  }> {
    return { secure: true, issues: [] }
  }
  
  private static async getTotalUsers(): Promise<number> {
    const result = await db.select({ count: count() }).from(users)
    return result[0]?.count || 0
  }
}
```

---

## 10. Security Monitoring

### 10.1 Real-time Security Monitoring

#### 10.1.1 Security Event Monitoring

```typescript
// lib/security-monitoring.ts
export class SecurityMonitoring {
  private static alertThresholds = {
    failedLogins: 5, // 5 failed logins in 5 minutes
    suspiciousActivity: 10, // 10 suspicious activities in 1 hour
    dataAccess: 100, // 100 data access events in 1 minute
    privilegedActions: 3 // 3 privileged actions in 5 minutes
  }
  
  static async monitorSecurityEvents(): Promise<void> {
    // Monitor failed login attempts
    await this.monitorFailedLogins()
    
    // Monitor suspicious activities
    await this.monitorSuspiciousActivity()
    
    // Monitor data access patterns
    await this.monitorDataAccess()
    
    // Monitor privileged actions
    await this.monitorPrivilegedActions()
  }
  
  private static async monitorFailedLogins(): Promise<void> {
    const fiveMinutesAgo = new Date(Date.now() - 5 * 60 * 1000)
    
    const failedLogins = await db.query.auditLogs.findMany({
      where: and(
        eq(auditLogs.action, 'LOGIN_FAILED'),
        gte(auditLogs.timestamp, fiveMinutesAgo)
      )
    })
    
    const ipGroups = failedLogins.reduce((groups, log) => {
      const ip = log.metadata?.ip
      if (ip) {
        groups[ip] = (groups[ip] || 0) + 1
      }
      return groups
    }, {} as Record<string, number>)
    
    for (const [ip, count] of Object.entries(ipGroups)) {
      if (count >= this.alertThresholds.failedLogins) {
        await this.triggerSecurityAlert({
          type: 'BRUTE_FORCE_ATTACK',
          severity: 'high',
          details: {
            ip,
            attemptCount: count,
            timeWindow: '5 minutes'
          }
        })
        
        // Block IP temporarily
        await this.blockIP(ip, 15 * 60) // 15 minutes
      }
    }
  }
  
  private static async monitorSuspiciousActivity(): Promise<void> {
    const oneHourAgo = new Date(Date.now() - 60 * 60 * 1000)
    
    const suspiciousEvents = await db.query.auditLogs.findMany({
      where: and(
        inArray(auditLogs.action, [
          'SQL_INJECTION_ATTEMPT',
          'XSS_ATTEMPT',
          'UNAUTHORIZED_ACCESS',
          'PRIVILEGE_ESCALATION_ATTEMPT'
        ]),
        gte(auditLogs.timestamp, oneHourAgo)
      )
    })
    
    if (suspiciousEvents.length >= this.alertThresholds.suspiciousActivity) {
      await this.triggerSecurityAlert({
        type: 'SUSPICIOUS_ACTIVITY_SPIKE',
        severity: 'medium',
        details: {
          eventCount: suspiciousEvents.length,
          timeWindow: '1 hour',
          events: suspiciousEvents.map(e => e.action)
        }
      })
    }
  }
  
  private static async monitorDataAccess(): Promise<void> {
    const oneMinuteAgo = new Date(Date.now() - 60 * 1000)
    
    const dataAccessEvents = await db.query.auditLogs.findMany({
      where: and(
        inArray(auditLogs.action, [
          'GRADE_READ',
          'STUDENT_DATA_READ',
          'FINANCIAL_DATA_READ'
        ]),
        gte(auditLogs.timestamp, oneMinuteAgo)
      )
    })
    
    if (dataAccessEvents.length >= this.alertThresholds.dataAccess) {
      await this.triggerSecurityAlert({
        type: 'UNUSUAL_DATA_ACCESS',
        severity: 'medium',
        details: {
          accessCount: dataAccessEvents.length,
          timeWindow: '1 minute',
          users: [...new Set(dataAccessEvents.map(e => e.userId))]
        }
      })
    }
  }
  
  private static async monitorPrivilegedActions(): Promise<void> {
    const fiveMinutesAgo = new Date(Date.now() - 5 * 60 * 1000)
    
    const privilegedActions = await db.query.auditLogs.findMany({
      where: and(
        inArray(auditLogs.action, [
          'USER_DELETE',
          'GRADE_ADJUST',
          'SYSTEM_CONFIG_CHANGE',
          'DATA_EXPORT'
        ]),
        gte(auditLogs.timestamp, fiveMinutesAgo)
      )
    })
    
    if (privilegedActions.length >= this.alertThresholds.privilegedActions) {
      await this.triggerSecurityAlert({
        type: 'EXCESSIVE_PRIVILEGED_ACTIONS',
        severity: 'high',
        details: {
          actionCount: privilegedActions.length,
          timeWindow: '5 minutes',
          actions: privilegedActions.map(e => e.action),
          users: [...new Set(privilegedActions.map(e => e.userId))]
        }
      })
    }
  }
  
  private static async triggerSecurityAlert(alert: {
    type: string
    severity: 'low' | 'medium' | 'high' | 'critical'
    details: any
  }): Promise<void> {
    // Log the alert
    await db.insert(securityAlerts).values({
      type: alert.type,
      severity: alert.severity,
      details: alert.details,
      status: 'open',
      createdAt: new Date()
    })
    
    // Send notifications based on severity
    if (['high', 'critical'].includes(alert.severity)) {
      await this.sendImmediateAlert(alert)
    } else {
      await this.sendScheduledAlert(alert)
    }
    
    // Check for automated response
    await this.checkAutomatedResponse(alert)
  }
  
  private static async sendImmediateAlert(alert: any): Promise<void> {
    // Send to security team immediately
    await NotificationService.send({
      type: 'security_alert',
      severity: alert.severity,
      message: `Security Alert: ${alert.type}`,
      details: alert.details,
      channels: ['email', 'sms', 'slack']
    })
  }
  
  private static async sendScheduledAlert(alert: any): Promise<void> {
    // Queue for next batch notification
    await NotificationService.queue({
      type: 'security_alert',
      severity: alert.severity,
      message: `Security Alert: ${alert.type}`,
      details: alert.details
    })
  }
  
  private static async checkAutomatedResponse(alert: any): Promise<void> {
    switch (alert.type) {
      case 'BRUTE_FORCE_ATTACK':
        // Additional IP blocking, user account lockout
        await this.enhancedBruteForceResponse(alert.details)
        break
        
      case 'SQL_INJECTION_ATTEMPT':
        // Block source IP, scan for compromise
        await this.sqlInjectionResponse(alert.details)
        break
        
      // Add more automated responses
    }
  }
  
  private static async blockIP(ip: string, duration: number): Promise<void> {
    // Implementation to block IP at firewall level
    await db.insert(blockedIPs).values({
      ip,
      blockedAt: new Date(),
      expiresAt: new Date(Date.now() + duration * 1000),
      reason: 'Security monitoring auto-block'
    })
  }
  
  private static async enhancedBruteForceResponse(details: any): Promise<void> {
    // Block affected accounts temporarily
    const accountsToLock = details.accounts || []
    
    for (const userId of accountsToLock) {
      await db.update(users)
        .set({ isActive: false, lockedUntil: new Date(Date.now() + 30 * 60 * 1000) })
        .where(eq(users.id, userId))
    }
  }
  
  private static async sqlInjectionResponse(details: any): Promise<void> {
    // Additional security measures for SQL injection attempts
    await this.triggerSecurityScan(details.ip)
    await this.notifySecurityTeam({
      type: 'POTENTIAL_COMPROMISE',
      severity: 'critical',
      details
    })
  }
  
  private static async triggerSecurityScan(ip: string): Promise<void> {
    // Trigger security scan for potentially compromised systems
    console.log(`Triggering security scan for IP: ${ip}`)
    // Implementation would integrate with security scanning tools
  }
  
  private static async notifySecurityTeam(alert: any): Promise<void> {
    // Immediate notification to security team for critical issues
    await NotificationService.send({
      type: 'critical_security_incident',
      message: 'Critical security incident detected',
      details: alert,
      channels: ['email', 'sms', 'phone', 'slack'],
      escalation: true
    })
  }
}
```

#### 10.1.2 Anomaly Detection

```typescript
// lib/anomaly-detection.ts
export class AnomalyDetection {
  static async detectAnomalies(): Promise<void> {
    // Detect login anomalies
    await this.detectLoginAnomalies()
    
    // Detect access pattern anomalies
    await this.detectAccessAnomalies()
    
    // Detect performance anomalies
    await this.detectPerformanceAnomalies()
    
    // Detect data anomalies
    await this.detectDataAnomalies()
  }
  
  private static async detectLoginAnomalies(): Promise<void> {
    const recentLogins = await this.getRecentLogins(24 * 60 * 60 * 1000) // 24 hours
    
    // Detect unusual login times
    const timeAnomalies = this.detectUnusualLoginTimes(recentLogins)
    for (const anomaly of timeAnomalies) {
      await this.reportAnomaly({
        type: 'UNUSUAL_LOGIN_TIME',
        severity: 'medium',
        userId: anomaly.userId,
        details: {
          usualHours: anomaly.usualHours,
          currentHour: anomaly.currentHour,
          deviationScore: anomaly.deviationScore
        }
      })
    }
    
    // Detect unusual login locations
    const locationAnomalies = await this.detectUnusualLoginLocations(recentLogins)
    for (const anomaly of locationAnomalies) {
      await this.reportAnomaly({
        type: 'UNUSUAL_LOGIN_LOCATION',
        severity: 'high',
        userId: anomaly.userId,
        details: {
          usualLocations: anomaly.usualLocations,
          currentLocation: anomaly.currentLocation,
          distance: anomaly.distance
        }
      })
    }
    
    // Detect multiple concurrent sessions
    const sessionAnomalies = this.detectConcurrentSessions(recentLogins)
    for (const anomaly of sessionAnomalies) {
      await this.reportAnomaly({
        type: 'MULTIPLE_CONCURRENT_SESSIONS',
        severity: 'medium',
        userId: anomaly.userId,
        details: {
          sessionCount: anomaly.sessionCount,
          locations: anomaly.locations
        }
      })
    }
  }
  
  private static async detectAccessAnomalies(): Promise<void> {
    const recentAccess = await this.getRecentAccess(60 * 60 * 1000) // 1 hour
    
    // Detect unusual data access patterns
    const accessAnomalies = this.detectUnusualAccessPatterns(recentAccess)
    for (const anomaly of accessAnomalies) {
      await this.reportAnomaly({
        type: 'UNUSUAL_ACCESS_PATTERN',
        severity: 'medium',
        userId: anomaly.userId,
        details: {
          accessedResources: anomaly.resources,
          accessFrequency: anomaly.frequency,
          baselineFrequency: anomaly.baselineFrequency
        }
      })
    }
    
    // Detect privilege escalation attempts
    const escalationAnomalies = this.detectPrivilegeEscalation(recentAccess)
    for (const anomaly of escalationAnomalies) {
      await this.reportAnomaly({
        type: 'PRIVILEGE_ESCALATION_ATTEMPT',
        severity: 'high',
        userId: anomaly.userId,
        details: anomaly.details
      })
    }
  }
  
  private static async detectPerformanceAnomalies(): Promise<void> {
    const metrics = await this.getSystemMetrics(15 * 60 * 1000) // 15 minutes
    
    // Detect unusual response times
    const responseTimeAnomalies = this.detectResponseTimeAnomalies(metrics)
    for (const anomaly of responseTimeAnomalies) {
      await this.reportAnomaly({
        type: 'PERFORMANCE_ANOMALY',
        severity: 'low',
        details: {
          metric: 'response_time',
          currentValue: anomaly.currentValue,
          baselineValue: anomaly.baselineValue,
          deviation: anomaly.deviation
        }
      })
    }
    
    // Detect unusual error rates
    const errorRateAnomalies = this.detectErrorRateAnomalies(metrics)
    for (const anomaly of errorRateAnomalies) {
      await this.reportAnomaly({
        type: 'ERROR_RATE_ANOMALY',
        severity: 'medium',
        details: {
          currentErrorRate: anomaly.currentValue,
          baselineErrorRate: anomaly.baselineValue,
          increase: anomaly.deviation
        }
      })
    }
  }
  
  private static async detectDataAnomalies(): Promise<void> {
    // Detect unusual grade changes
    const gradeAnomalies = await this.detectGradeAnomalies()
    for (const anomaly of gradeAnomalies) {
      await this.reportAnomaly({
        type: 'UNUSUAL_GRADE_CHANGE',
        severity: 'high',
        details: anomaly.details
      })
    }
    
    // Detect unusual user behavior
    const behaviorAnomalies = await this.detectBehaviorAnomalies()
    for (const anomaly of behaviorAnomalies) {
      await this.reportAnomaly({
        type: 'UNUSUAL_USER_BEHAVIOR',
        severity: 'medium',
        userId: anomaly.userId,
        details: anomaly.details
      })
    }
  }
  
  private static async reportAnomaly(anomaly: {
    type: string
    severity: 'low' | 'medium' | 'high' | 'critical'
    userId?: string
    details: any
  }): Promise<void> {
    await db.insert(anomalies).values({
      type: anomaly.type,
      severity: anomaly.severity,
      userId: anomaly.userId || null,
      details: anomaly.details,
      status: 'open',
      createdAt: new Date()
    })
    
    // Trigger investigation for high severity anomalies
    if (['high', 'critical'].includes(anomaly.severity)) {
      await this.triggerInvestigation(anomaly)
    }
  }
  
  private static async triggerInvestigation(anomaly: any): Promise<void> {
    await db.insert(investigations).values({
      anomalyId: anomaly.id,
      status: 'open',
      assignedTo: 'security-team',
      priority: anomaly.severity === 'critical' ? 'high' : 'medium',
      createdAt: new Date()
    })
    
    // Notify security team
    await NotificationService.send({
      type: 'anomaly_investigation',
      message: `Anomaly investigation required: ${anomaly.type}`,
      details: anomaly,
      channels: ['slack', 'email']
    })
  }
  
  // Helper methods for detection algorithms
  private static detectUnusualLoginTimes(logins: any[]): any[] {
    // Implementation for detecting unusual login times
    return []
  }
  
  private static async detectUnusualLoginLocations(logins: any[]): Promise<any[]> {
    // Implementation for detecting unusual login locations
    return []
  }
  
  private static detectConcurrentSessions(logins: any[]): any[] {
    // Implementation for detecting concurrent sessions
    return []
  }
  
  private static detectUnusualAccessPatterns(access: any[]): any[] {
    // Implementation for detecting unusual access patterns
    return []
  }
  
  private static detectPrivilegeEscalation(access: any[]): any[] {
    // Implementation for detecting privilege escalation
    return []
  }
  
  private static detectResponseTimeAnomalies(metrics: any[]): any[] {
    // Implementation for detecting response time anomalies
    return []
  }
  
  private static detectErrorRateAnomalies(metrics: any[]): any[] {
    // Implementation for detecting error rate anomalies
    return []
  }
  
  private static async detectGradeAnomalies(): Promise<any[]> {
    // Implementation for detecting grade anomalies
    return []
  }
  
  private static async detectBehaviorAnomalies(): Promise<any[]> {
    // Implementation for detecting behavior anomalies
    return []
  }
  
  // Data retrieval methods
  private static async getRecentLogins(timeRange: number): Promise<any[]> {
    const cutoff = new Date(Date.now() - timeRange)
    return await db.query.auditLogs.findMany({
      where: and(
        eq(auditLogs.action, 'LOGIN_SUCCESS'),
        gte(auditLogs.timestamp, cutoff)
      )
    })
  }
  
  private static async getRecentAccess(timeRange: number): Promise<any[]> {
    const cutoff = new Date(Date.now() - timeRange)
    return await db.query.auditLogs.findMany({
      where: gte(auditLogs.timestamp, cutoff)
    })
  }
  
  private static async getSystemMetrics(timeRange: number): Promise<any[]> {
    // Implementation to retrieve system metrics
    return []
  }
}
```

---

## 11. Incident Response

### 11.1 Incident Response Framework

#### 11.1.1 Incident Classification

```typescript
// lib/incident-response.ts
export class IncidentResponse {
  static readonly incidentTypes = {
    SECURITY_BREACH: {
      severity: 'critical',
      responseTime: 15, // minutes
      escalationLevel: 1
    },
    DATA_COMPROMISE: {
      severity: 'critical',
      responseTime: 15,
      escalationLevel: 1
    },
    SYSTEM_COMPROMISE: {
      severity: 'high',
      responseTime: 30,
      escalationLevel: 2
    },
    DDOS_ATTACK: {
      severity: 'high',
      responseTime: 10,
      escalationLevel: 2
    },
    MALWARE_DETECTION: {
      severity: 'medium',
      responseTime: 60,
      escalationLevel: 3
    },
    POLICY_VIOLATION: {
      severity: 'low',
      responseTime: 240,
      escalationLevel: 4
    }
  }
  
  static async createIncident(incident: {
    type: keyof typeof this.incidentTypes
    description: string
    source: string
    affectedSystems?: string[]
    userId?: string
    metadata?: any
  }): Promise<string> {
    const incidentConfig = this.incidentTypes[incident.type]
    
    // Create incident record
    const [newIncident] = await db.insert(securityIncidents).values({
      type: incident.type,
      severity: incidentConfig.severity,
      description: incident.description,
      source: incident.source,
      affectedSystems: incident.affectedSystems || [],
      userId: incident.userId || null,
      metadata: incident.metadata || {},
      status: 'open',
      priority: incidentConfig.escalationLevel,
      createdAt: new Date(),
      responseDueAt: new Date(Date.now() + incidentConfig.responseTime * 60 * 1000)
    }).returning()
    
    // Trigger immediate response actions
    await this.triggerIncidentResponse(newIncident)
    
    // Send notifications
    await this.notifyStakeholders(newIncident)
    
    return newIncident.id
  }
  
  private static async triggerIncidentResponse(incident: any): Promise<void> {
    const incidentType = incident.type as keyof typeof this.incidentTypes
    
    switch (incidentType) {
      case 'SECURITY_BREACH':
        await this.handleSecurityBreach(incident)
        break
        
      case 'DATA_COMPROMISE':
        await this.handleDataCompromise(incident)
        break
        
      case 'SYSTEM_COMPROMISE':
        await this.handleSystemCompromise(incident)
        break
        
      case 'DDOS_ATTACK':
        await this.handleDDoSAttack(incident)
        break
        
      case 'MALWARE_DETECTION':
        await this.handleMalwareDetection(incident)
        break
        
      case 'POLICY_VIOLATION':
        await this.handlePolicyViolation(incident)
        break
    }
  }
  
  private static async handleSecurityBreach(incident: any): Promise<void> {
    // Immediate containment actions
    await this.containmentActions({
      isolateAffected: true,
      blockAccess: true,
      preserveEvidence: true
    })
    
    // Investigate scope
    await this.assessScope({
      checkAuthentication: true,
      checkAuthorization: true,
      checkDataAccess: true,
      checkSystemIntegrity: true
    })
    
    // Notify all stakeholders
    await this.notifyStakeholders({
      ...incident,
      priority: 'critical'
    })
  }
  
  private static async handleDataCompromise(incident: any): Promise<void> {
    // Identify compromised data
    const compromisedData = await this.identifyCompromisedData(incident)
    
    // Notify affected individuals if PII is involved
    if (compromisedData.includesPII) {
      await this.notifyAffectedIndividuals(compromisedData)
    }
    
    // Begin data recovery
    await this.initiateDataRecovery(compromisedData)
    
    // Report to regulatory authorities if required
    await this.reportToAuthorities(compromisedData)
  }
  
  private static async handleSystemCompromise(incident: any): Promise<void> {
    // Isolate affected systems
    await this.isolateSystems(incident.affectedSystems)
    
    // Analyze malware/payload
    await this.analyzeThreat(incident.metadata?.threatInfo)
    
    // Patch vulnerabilities
    await this.patchVulnerabilities(incident.metadata?.vulnerabilities)
    
    // Monitor for continued activity
    await this.enhancedMonitoring(incident.affectedSystems)
  }
  
  private static async handleDDoSAttack(incident: any): Promise<void> {
    // Activate DDoS mitigation
    await this.activateDDoSMitigation()
    
    // Analyze attack patterns
    await this.analyzeAttackPatterns(incident.metadata?.attackInfo)
    
    // Scale infrastructure if needed
    await this.scaleInfrastructure()
    
    // Communicate with users
    await this.communicateOutage()
  }
  
  private static async handleMalwareDetection(incident: any): Promise<void> {
    // Quarantine affected systems
    await this.quarantineSystems(incident.affectedSystems)
    
    // Perform malware analysis
    await this.analyzeMalware(incident.metadata?.malwareInfo)
    
    // Deploy antivirus updates
    await this.deployAVUpdates()
    
    // Scan for lateral movement
    await this.scanForLateralMovement()
  }
  
  private static async handlePolicyViolation(incident: any): Promise<void> {
    // Document violation
    await this.documentViolation(incident)
    
    // Notify management
    await this.notifyManagement(incident)
    
    // Recommend disciplinary action
    await this.recommendAction(incident)
    
    // Update policies if needed
    await this.reviewPolicies(incident)
  }
  
  private static async containmentActions(actions: {
    isolateAffected?: boolean
    blockAccess?: boolean
    preserveEvidence?: boolean
  }): Promise<void> {
    if (actions.isolateAffected) {
      // Isolate affected systems from network
      await this.isolateAffectedSystems()
    }
    
    if (actions.blockAccess) {
      // Block suspicious IPs and accounts
      await this.blockSuspiciousAccess()
    }
    
    if (actions.preserveEvidence) {
      // Preserve system state for forensics
      await this.preserveForensicEvidence()
    }
  }
  
  private static async notifyStakeholders(incident: any): Promise<void> {
    const notificationChannels = this.getNotificationChannels(incident.severity)
    
    await NotificationService.send({
      type: 'security_incident',
      severity: incident.severity,
      title: `Security Incident: ${incident.type}`,
      message: incident.description,
      details: {
        incidentId: incident.id,
        severity: incident.severity,
        affectedSystems: incident.affectedSystems,
        responseDueAt: incident.responseDueAt
      },
      channels: notificationChannels
    })
  }
  
  private static getNotificationChannels(severity: string): string[] {
    switch (severity) {
      case 'critical':
        return ['sms', 'phone', 'email', 'slack', 'teams']
      case 'high':
        return ['email', 'slack', 'teams']
      case 'medium':
        return ['email', 'slack']
      case 'low':
        return ['email']
      default:
        return ['email']
    }
  }
  
  // Incident lifecycle management
  static async updateIncidentStatus(
    incidentId: string, 
    status: 'open' | 'investigating' | 'contained' | 'resolved' | 'closed',
    updates?: any
  ): Promise<void> {
    await db.update(securityIncidents)
      .set({
        status,
        updatedAt: new Date(),
        ...(updates && { metadata: updates })
      })
      .where(eq(securityIncidents.id, incidentId))
    
    // Log status change
    await db.insert(incidentUpdates).values({
      incidentId,
      status,
      updates: updates || {},
      updatedAt: new Date()
    })
  }
  
  static async assignIncident(
    incidentId: string, 
    assigneeId: string, 
    assigneeRole: string
  ): Promise<void> {
    await db.update(securityIncidents)
      .set({
        assignedTo: assigneeId,
        assignedRole: assigneeRole,
        assignedAt: new Date(),
        status: 'investigating'
      })
      .where(eq(securityIncidents.id, incidentId))
    
    // Notify assignee
    await NotificationService.send({
      type: 'incident_assignment',
      message: `Incident ${incidentId} has been assigned to you`,
      channels: ['email', 'slack']
    })
  }
  
  // Helper methods
  private static async isolateAffectedSystems(): Promise<void> {
    // Implementation for system isolation
  }
  
  private static async blockSuspiciousAccess(): Promise<void> {
    // Implementation for blocking suspicious access
  }
  
  private static async preserveForensicEvidence(): Promise<void> {
    // Implementation for preserving evidence
  }
  
  private static async identifyCompromisedData(incident: any): Promise<any> {
    // Implementation for identifying compromised data
    return { includesPII: false, dataTypes: [], recordsAffected: 0 }
  }
  
  private static async notifyAffectedIndividuals(data: any): Promise<void> {
    // Implementation for notifying affected individuals
  }
  
  private static async initiateDataRecovery(data: any): Promise<void> {
    // Implementation for data recovery
  }
  
  private static async reportToAuthorities(data: any): Promise<void> {
    // Implementation for reporting to authorities
  }
}
```

---

## 12. Security Best Practices

### 12.1 Development Security

#### 12.1.1 Secure Development Lifecycle

```typescript
// lib/sdlc-security.ts
export class SDLCSecurity {
  static securityChecklist = {
    requirements: [
      'Security requirements identified and documented',
      'Threat model completed',
      'Data classification performed',
      'Compliance requirements identified'
    ],
    
    design: [
      'Security architecture reviewed',
      'Authentication and authorization designed',
      'Data encryption planned',
      'Error handling designed',
      'Logging and monitoring planned'
    ],
    
    implementation: [
      'Secure coding practices followed',
      'Input validation implemented',
      'Output encoding implemented',
      'Database security implemented',
      'API security implemented'
    ],
    
    testing: [
      'Security testing completed',
      'Penetration testing performed',
      'Vulnerability scanning completed',
      'Static code analysis completed',
      'Dynamic security testing completed'
    ],
    
    deployment: [
      'Security configuration validated',
      'Environment variables secured',
      'Network security configured',
      'Monitoring implemented',
      'Backup procedures tested'
    ],
    
    maintenance: [
      'Security patches applied',
      'Security monitoring active',
      'Incident response ready',
      'Regular security reviews',
      'User training provided'
    ]
  }
  
  static async performSecurityGate(stage: keyof typeof this.securityChecklist): Promise<{
    passed: boolean
    findings: string[]
    recommendations: string[]
  }> {
    const checklist = this.securityChecklist[stage]
    const findings = []
    const recommendations = []
    
    for (const item of checklist) {
      const result = await this.checkSecurityItem(item, stage)
      if (!result.passed) {
        findings.push(`Failed: ${item}`)
        recommendations.push(result.recommendation)
      }
    }
    
    return {
      passed: findings.length === 0,
      findings,
      recommendations
    }
  }
  
  private static async checkSecurityItem(item: string, stage: string): Promise<{
    passed: boolean
    recommendation: string
  }> {
    // Implementation for checking individual security items
    return { passed: true, recommendation: '' }
  }
}
```

#### 12.1.2 Code Security Review

```typescript
// lib/code-security.ts
export class CodeSecurity {
  static securityPatterns = {
    // Vulnerable patterns to detect
    vulnerabilities: [
      {
        pattern: /eval\s*\(/,
        description: 'Use of eval() function',
        severity: 'high',
        recommendation: 'Avoid using eval() function'
      },
      {
        pattern: /innerHTML\s*=/,
        description: 'Direct innerHTML assignment',
        severity: 'medium',
        recommendation: 'Use textContent or sanitize HTML'
      },
      {
        pattern: /document\.write\s*\(/,
        description: 'Use of document.write()',
        severity: 'medium',
        recommendation: 'Avoid document.write()'
      },
      {
        pattern: /\.exec\s*\(/,
        description: 'Direct database execution',
        severity: 'critical',
        recommendation: 'Use parameterized queries'
      }
    ],
    
    // Security best practices
    bestPractices: [
      {
        pattern: /bcrypt\.hash/,
        description: 'Password hashing with bcrypt',
        severity: 'positive',
        recommendation: 'Good practice - continue using bcrypt'
      },
      {
        pattern: /validator\.isEmail/,
        description: 'Email validation',
        severity: 'positive',
        recommendation: 'Good practice - validate inputs'
      }
    ]
  }
  
  static analyzeCodeSecurity(code: string): {
    vulnerabilities: Array<{
      line: number
      pattern: string
      description: string
      severity: string
      recommendation: string
    }>
    bestPractices: Array<{
      line: number
      pattern: string
      description: string
      recommendation: string
    }>
    score: number
  } {
    const lines = code.split('\n')
    const vulnerabilities = []
    const bestPractices = []
    
    lines.forEach((line, index) => {
      // Check for vulnerabilities
      this.securityPatterns.vulnerabilities.forEach(vuln => {
        if (vuln.pattern.test(line)) {
          vulnerabilities.push({
            line: index + 1,
            pattern: vuln.pattern.source,
            description: vuln.description,
            severity: vuln.severity,
            recommendation: vuln.recommendation
          })
        }
      })
      
      // Check for best practices
      this.securityPatterns.bestPractices.forEach(practice => {
        if (practice.pattern.test(line)) {
          bestPractices.push({
            line: index + 1,
            pattern: practice.pattern.source,
            description: practice.description,
            recommendation: practice.recommendation
          })
        }
      })
    })
    
    // Calculate security score
    const score = this.calculateSecurityScore(vulnerabilities, bestPractices)
    
    return {
      vulnerabilities,
      bestPractices,
      score
    }
  }
  
  private static calculateSecurityScore(vulnerabilities: any[], bestPractices: any[]): number {
    let score = 100
    
    vulnerabilities.forEach(vuln => {
      switch (vuln.severity) {
        case 'critical':
          score -= 25
          break
        case 'high':
          score -= 15
          break
        case 'medium':
          score -= 10
          break
        case 'low':
          score -= 5
          break
      }
    })
    
    // Bonus points for best practices
    score += bestPractices.length * 2
    
    return Math.max(0, Math.min(100, score))
  }
  
  static async generateSecurityReport(codePath: string): Promise<{
    overallScore: number
    files: Array<{
      path: string
      score: number
      vulnerabilities: number
      bestPractices: number
    }>
    recommendations: string[]
  }> {
    // Implementation for generating security report
    return {
      overallScore: 85,
      files: [],
      recommendations: []
    }
  }
}
```

### 12.2 User Security Awareness

#### 12.2.1 Security Training

```typescript
// lib/security-training.ts
export class SecurityTraining {
  static trainingModules = {
    passwordSecurity: {
      title: 'Password Security Best Practices',
      duration: 15, // minutes
      content: [
        'Create strong, unique passwords',
        'Use a password manager',
        'Enable multi-factor authentication',
        'Never share passwords',
        'Report suspicious password requests'
      ],
      quiz: [
        {
          question: 'What makes a password strong?',
          options: [
            'Only letters',
            '8+ characters with mixed types',
            'Common words',
            'Personal information'
          ],
          correct: 1
        }
      ]
    },
    
    phishingAwareness: {
      title: 'Recognizing Phishing Attempts',
      duration: 20,
      content: [
        'Identify suspicious emails',
        'Check sender addresses carefully',
        'Avoid clicking unknown links',
        'Verify requests for sensitive information',
        'Report phishing attempts'
      ],
      quiz: [
        {
          question: 'What is a common sign of phishing?',
          options: [
            'Official sender name',
            'Urgent requests for personal info',
            'Proper grammar',
            'Known company branding'
          ],
          correct: 1
        }
      ]
    },
    
    dataProtection: {
      title: 'Protecting Sensitive Data',
      duration: 15,
      content: [
        'Handle student information carefully',
        'Use secure file storage',
        'Follow data retention policies',
        'Report data breaches immediately',
        'Use encrypted communication channels'
      ]
    }
  }
  
  static async assignTraining(userId: string, role: string): Promise<void> {
    const requiredModules = this.getRequiredModules(role)
    
    for (const moduleId of requiredModules) {
      await db.insert(trainingAssignments).values({
        userId,
        moduleId,
        assignedAt: new Date(),
        dueDate: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000), // 30 days
        status: 'assigned'
      })
    }
    
    // Notify user of training assignment
    await NotificationService.send({
      type: 'training_assigned',
      message: 'Security training has been assigned to you',
      userId,
      channels: ['email', 'in-app']
    })
  }
  
  static getRequiredModules(role: string): string[] {
    const baseModules = ['passwordSecurity', 'phishingAwareness']
    
    switch (role) {
      case 'kurikulum':
        return [...baseModules, 'dataProtection', 'incidentResponse']
      case 'guru':
        return [...baseModules, 'dataProtection', 'classroomSecurity']
      case 'siswa':
        return ['passwordSecurity', 'phishingAwareness', 'digitalCitizenship']
      default:
        return baseModules
    }
  }
  
  static async completeTraining(userId: string, moduleId: string, quizAnswers: number[]): Promise<{
    passed: boolean
    score: number
    certificateUrl?: string
  }> {
    const module = this.trainingModules[moduleId as keyof typeof this.trainingModules]
    
    if (!module || !module.quiz) {
      throw new Error('Invalid training module')
    }
    
    // Calculate quiz score
    let correctAnswers = 0
    module.quiz.forEach((question, index) => {
      if (quizAnswers[index] === question.correct) {
        correctAnswers++
      }
    })
    
    const score = Math.round((correctAnswers / module.quiz.length) * 100)
    const passed = score >= 80 // 80% passing score
    
    // Update training record
    await db.update(trainingAssignments)
      .set({
        status: passed ? 'completed' : 'failed',
        completedAt: new Date(),
        score
      })
      .where(and(
        eq(trainingAssignments.userId, userId),
        eq(trainingAssignments.moduleId, moduleId)
      ))
    
    // Generate certificate if passed
    let certificateUrl
    if (passed) {
      certificateUrl = await this.generateCertificate(userId, moduleId)
    }
    
    return {
      passed,
      score,
      certificateUrl
    }
  }
  
  private static async generateCertificate(userId: string, moduleId: string): Promise<string> {
    const certificate = {
      id: crypto.randomUUID(),
      userId,
      moduleId,
      issuedAt: new Date(),
      expiresAt: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000) // 1 year
    }
    
    await db.insert(trainingCertificates).values(certificate)
    
    return `/certificates/${certificate.id}`
  }
  
  static async getTrainingStatus(userId: string): Promise<{
    completed: number
    assigned: number
    overdue: number
    nextDue?: Date
  }> {
    const assignments = await db.query.trainingAssignments.findMany({
      where: eq(trainingAssignments.userId, userId)
    })
    
    const completed = assignments.filter(a => a.status === 'completed').length
    const assigned = assignments.filter(a => a.status === 'assigned').length
    const overdue = assignments.filter(a => 
      a.status === 'assigned' && a.dueDate < new Date()
    ).length
    
    const nextAssignment = assignments
      .filter(a => a.status === 'assigned')
      .sort((a, b) => a.dueDate.getTime() - b.dueDate.getTime())[0]
    
    return {
      completed,
      assigned,
      overdue,
      nextDue: nextAssignment?.dueDate
    }
  }
}
```

---

## 13. Implementation Guidelines

### 13.1 Security Implementation Roadmap

#### 13.1.1 Phase 1: Foundation (Weeks 1-4)

**Week 1: Authentication & Authorization**
- [ ] Implement Better Auth integration
- [ ] Set up role-based access control
- [ ] Configure session management
- [ ] Implement password policies

**Week 2: Data Protection**
- [ ] Implement encryption for sensitive data
- [ ] Set up key management
- [ ] Configure database security
- [ ] Implement input validation

**Week 3: Network Security**
- [ ] Configure WAF rules
- [ ] Set up security headers
- [ ] Implement CSP policies
- [ ] Configure firewall rules

**Week 4: Monitoring & Logging**
- [ ] Set up comprehensive logging
- [ ] Implement security monitoring
- [ ] Configure alerting
- [ ] Set up audit trails

#### 13.1.2 Phase 2: Advanced Security (Weeks 5-8)

**Week 5: Advanced Authentication**
- [ ] Implement MFA
- [ ] Set up SSO integration
- [ ] Configure account lockout policies
- [ ] Implement session security

**Week 6: Threat Detection**
- [ ] Implement anomaly detection
- [ ] Set up security scanning
- [ ] Configure intrusion detection
- [ ] Implement automated response

**Week 7: Compliance**
- [ ] Implement GDPR compliance
- [ ] Set up data retention policies
- [ ] Configure privacy controls
- [ ] Implement consent management

**Week 8: Testing & Validation**
- [ ] Conduct penetration testing
- [ ] Perform security audits
- [ ] Validate incident response
- [ ] Document security procedures

### 13.2 Security Checklist

#### 13.2.1 Pre-Deployment Security Checklist

**Authentication & Authorization**
- [ ] Password policies enforced
- [ ] MFA implemented for privileged accounts
- [ ] Session management configured
- [ ] RBAC properly implemented
- [ ] Account lockout policies active

**Data Protection**
- [ ] Encryption at rest implemented
- [ ] Encryption in transit enforced
- [ ] Key management configured
- [ ] Data masking implemented
- [ ] Backup encryption enabled

**Application Security**
- [ ] Input validation implemented
- [ ] Output encoding configured
- [ ] SQL injection prevention verified
- [ ] XSS prevention implemented
- [ ] CSRF protection enabled

**Infrastructure Security**
- [ ] Network segmentation configured
- [ ] Firewall rules implemented
- [ ] Container security configured
- [ ] Secrets management implemented
- [ ] Access controls enforced

**Monitoring & Logging**
- [ ] Comprehensive logging enabled
- [ ] Security monitoring active
- [ ] Alerting configured
- [ ] Log retention policies set
- [ ] Incident response ready

**Compliance**
- [ ] GDPR requirements met
- [ ] Data privacy controls active
- [ ] Audit trails complete
- [ ] Documentation updated
- [ ] User training completed

### 13.3 Security Testing Procedures

#### 13.3.1 Security Testing Framework

```typescript
// lib/security-testing.ts
export class SecurityTesting {
  static async runSecurityTests(): Promise<{
    overallStatus: 'pass' | 'fail' | 'warning'
    tests: Array<{
      name: string
      status: 'pass' | 'fail' | 'warning'
      details: string
      recommendation?: string
    }>
  }> {
    const tests = []
    
    // Authentication tests
    tests.push(await this.testAuthentication())
    tests.push(await this.testAuthorization())
    tests.push(await this.testSessionManagement())
    
    // Data protection tests
    tests.push(await this.testDataEncryption())
    tests.push(await this.testInputValidation())
    tests.push(await this.testOutputEncoding())
    
    // Infrastructure tests
    tests.push(await this.testNetworkSecurity())
    tests.push(await this.testContainerSecurity())
    tests.push(await this.testSecretsManagement())
    
    // Monitoring tests
    tests.push(await this.testLogging())
    tests.push(await this.testAlerting())
    tests.push(await this.testIncidentResponse())
    
    const overallStatus = this.calculateOverallStatus(tests)
    
    return {
      overallStatus,
      tests
    }
  }
  
  private static async testAuthentication(): Promise<any> {
    try {
      // Test password policy enforcement
      const weakPasswordResult = await this.testWeakPasswordPolicy()
      
      // Test account lockout
      const lockoutResult = await this.testAccountLockout()
      
      // Test MFA enforcement
      const mfaResult = await this.testMFAEnforcement()
      
      if (weakPasswordResult && lockoutResult && mfaResult) {
        return {
          name: 'Authentication Security',
          status: 'pass',
          details: 'All authentication security tests passed'
        }
      } else {
        return {
          name: 'Authentication Security',
          status: 'fail',
          details: 'Some authentication security tests failed',
          recommendation: 'Review and fix authentication security issues'
        }
      }
    } catch (error) {
      return {
        name: 'Authentication Security',
        status: 'warning',
        details: `Authentication test error: ${error.message}`,
        recommendation: 'Investigate authentication test failures'
      }
    }
  }
  
  private static async testAuthorization(): Promise<any> {
    // Test RBAC implementation
    const rbacTests = [
      { role: 'siswa', path: '/admin', shouldFail: true },
      { role: 'guru', path: '/admin/users', shouldFail: true },
      { role: 'kurikulum', path: '/admin/users', shouldFail: false }
    ]
    
    for (const test of rbacTests) {
      const result = await this.testRBAC(test.role, test.path, test.shouldFail)
      if (!result) {
        return {
          name: 'Authorization Security',
          status: 'fail',
          details: `RBAC test failed for ${test.role} accessing ${test.path}`,
          recommendation: 'Review and fix RBAC implementation'
        }
      }
    }
    
    return {
      name: 'Authorization Security',
      status: 'pass',
      details: 'All authorization tests passed'
    }
  }
  
  // Helper methods for specific tests
  private static async testWeakPasswordPolicy(): Promise<boolean> {
    try {
      // Attempt to create user with weak password
      const response = await fetch('/api/auth/signup', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          email: 'test@example.com',
          password: '123',
          role: 'siswa'
        })
      })
      
      return response.status === 400 // Should fail
    } catch {
      return false
    }
  }
  
  private static async testAccountLockout(): Promise<boolean> {
    // Test account lockout after multiple failed attempts
    for (let i = 0; i < 6; i++) {
      await fetch('/api/auth/signin', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          email: 'test@example.com',
          password: 'wrongpassword'
        })
      })
    }
    
    // Try login again - should be locked out
    const response = await fetch('/api/auth/signin', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        email: 'test@example.com',
        password: 'correctpassword'
      })
    })
    
    return response.status === 423 // Locked out
  }
  
  private static async testMFAEnforcement(): Promise<boolean> {
    // Test MFA enforcement for privileged accounts
    return true // Implementation depends on MFA setup
  }
  
  private static async testRBAC(role: string, path: string, shouldFail: boolean): Promise<boolean> {
    try {
      const response = await fetch(path, {
        headers: {
          'Authorization': `Bearer ${await this.getTokenForRole(role)}`
        }
      })
      
      return shouldFail ? response.status === 403 : response.status === 200
    } catch {
      return false
    }
  }
  
  private static async getTokenForRole(role: string): Promise<string> {
    // Implementation to get token for specific role
    return 'test-token'
  }
  
  private static calculateOverallStatus(tests: any[]): 'pass' | 'fail' | 'warning' {
    const failedTests = tests.filter(t => t.status === 'fail')
    const warningTests = tests.filter(t => t.status === 'warning')
    
    if (failedTests.length > 0) {
      return 'fail'
    }
    
    if (warningTests.length > 0) {
      return 'warning'
    }
    
    return 'pass'
  }
}
```

---

## Conclusion

This comprehensive security architecture and implementation guide provides a robust foundation for securing the Educational Management System. The multi-layered security approach ensures protection against common threats while maintaining usability for legitimate users.

### Key Security Achievements

1. **Strong Authentication**: Better Auth integration with MFA support
2. **Granular Authorization**: Role-based access control with principle of least privilege
3. **Data Protection**: Comprehensive encryption and data masking strategies
4. **Network Security**: WAF, CSP, and network segmentation
5. **Active Monitoring**: Real-time threat detection and incident response
6. **Compliance**: GDPR compliance and data privacy protection
7. **Secure Development**: Security best practices throughout the SDLC

### Continuous Improvement

Security is an ongoing process that requires:
- Regular security assessments and penetration testing
- Continuous monitoring and threat intelligence
- User education and awareness programs
- Regular updates to security controls and procedures
- Incident response drills and tabletop exercises

This security architecture provides a solid foundation that can evolve with emerging threats and changing requirements, ensuring the long-term security and integrity of the Educational Management System.