# API Core, Authentication & Data Models
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0  
**Date:** December 2024  
**Document Type:** API Specification - Part A  
**Scope:** Core API, Authentication, Security & Data Models

---

## Executive Summary

This document provides the foundational API specifications for the CleanStation digitalization system, covering core architecture, authentication mechanisms, security protocols, and fundamental data models. This is Part A of a comprehensive 4-part API specification series.

### Document Series Overview
- **Part A (This Document)**: Core API, Authentication & Data Models
- **Part B**: Order & BOM Management APIs  
- **Part C**: Quality Control & Assembly APIs
- **Part D**: Service Department, Admin & Integration APIs

---

## API Architecture Overview

### Base Configuration
```typescript
interface APIConfiguration {
  baseURL: 'https://api.cleanstation.torvan.com/v1';
  authentication: 'JWT Bearer Token with RSA256 signing';
  contentType: 'application/json';
  versioning: 'URL path versioning (/v1, /v2)';
  rateLimit: {
    authenticated: '5000 requests/hour';
    admin: '10000 requests/hour';
    public: '100 requests/hour';
  };
  timeout: '30 seconds';
  retries: 3;
  websocket: 'wss://api.cleanstation.torvan.com/v1/ws';
}
```

### Core Endpoints Structure
```
/api/v1
├── /auth                    # Authentication & session management
│   ├── /login              # User authentication
│   ├── /logout             # Session termination
│   ├── /refresh            # Token refresh
│   ├── /reset-password     # Password reset workflow
│   └── /verify             # Token verification
├── /users                  # User management
│   ├── /profile           # User profile operations
│   ├── /preferences       # User preferences
│   └── /sessions          # Session management
├── /health                 # System health endpoints
│   ├── /status            # System status
│   ├── /metrics           # Performance metrics
│   └── /version           # API version info
└── /websocket             # Real-time connections
    ├── /connect           # WebSocket connection
    └── /events            # Event subscriptions
```

### API Design Principles
```typescript
interface APIDesignPrinciples {
  restful: 'RESTful design with standard HTTP methods';
  stateless: 'Stateless operations with JWT tokens';
  cacheable: 'Cacheable responses with appropriate headers';
  layered: 'Layered architecture with clear separation';
  uniform: 'Uniform interface across all endpoints';
  hypermedia: 'HATEOAS for discoverable API navigation';
}
```

---

## Authentication & Authorization

### JWT Token Implementation

#### Token Structure
```typescript
interface JWTHeader {
  alg: 'RS256';           // RSA signature with SHA-256
  typ: 'JWT';             // Token type
  kid: string;            // Key identifier
}

interface JWTPayload {
  // Standard claims
  iss: 'api.cleanstation.torvan.com';  // Issuer
  sub: string;                          // Subject (User ID)
  aud: 'cleanstation-app';             // Audience
  iat: number;                         // Issued at (timestamp)
  exp: number;                         // Expires at (timestamp)
  nbf: number;                         // Not before (timestamp)
  jti: string;                         // JWT ID (unique identifier)
  
  // Custom claims
  email: string;                       // User email
  role: UserRole;                      // Primary user role
  permissions: Permission[];           // Specific permissions array
  sessionId: string;                   // Session identifier
  department: string;                  // User department
  lastLogin: number;                   // Last login timestamp
  mfa: boolean;                        // MFA enabled flag
  scope: string[];                     // OAuth-style scopes
}

interface Permission {
  resource: string;        // Resource type (orders, bom, qc, etc.)
  actions: string[];       // Allowed actions (create, read, update, delete)
  conditions?: object;     // Additional conditions/constraints
}
```

#### Authentication Flow
```typescript
interface AuthenticationRequest {
  email: string;
  password: string;
  remember?: boolean;      // Extended session
  mfaCode?: string;        // Multi-factor authentication code
  deviceFingerprint?: string; // Device identification
}

interface AuthenticationResponse {
  accessToken: string;     // JWT access token (15 minutes)
  refreshToken: string;    // Refresh token (7 days)
  expiresIn: number;       // Access token expiration (seconds)
  tokenType: 'Bearer';     // Token type
  user: UserProfile;       // User profile information
  permissions: Permission[]; // User permissions
  sessionInfo: SessionInfo; // Session metadata
}

interface SessionInfo {
  sessionId: string;
  createdAt: Date;
  lastActivity: Date;
  deviceInfo: DeviceInfo;
  ipAddress: string;
  userAgent: string;
}
```

### Authentication Endpoints

#### Login Endpoint
```typescript
POST /api/v1/auth/login
Content-Type: application/json

Request Body:
{
  "email": "user@torvan.com",
  "password": "SecurePassword123!",
  "remember": false,
  "deviceFingerprint": "abc123def456"
}

Success Response (200):
{
  "accessToken": "eyJhbGciOiJSUzI1NiIs...",
  "refreshToken": "def456ghi789...",
  "expiresIn": 900,
  "tokenType": "Bearer",
  "user": {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@torvan.com",
    "fullName": "John Doe",
    "role": "OrderEntrySpecialist",
    "department": "Production",
    "isActive": true,
    "lastLogin": "2024-12-01T10:30:00Z"
  },
  "permissions": [
    {
      "resource": "orders",
      "actions": ["create", "read", "update"]
    }
  ],
  "sessionInfo": {
    "sessionId": "sess_abc123",
    "createdAt": "2024-12-01T10:30:00Z",
    "deviceInfo": {
      "type": "desktop",
      "os": "Windows 10",
      "browser": "Chrome 120"
    }
  }
}

Error Responses:
400 Bad Request:
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": {
      "field": "email",
      "validation": "email"
    }
  }
}

401 Unauthorized:
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password",
    "timestamp": "2024-12-01T10:30:00Z",
    "requestId": "req_abc123"
  }
}

429 Too Many Requests:
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many login attempts. Try again in 15 minutes.",
    "retryAfter": 900
  }
}
```

#### Token Refresh Endpoint
```typescript
POST /api/v1/auth/refresh
Content-Type: application/json

Request Body:
{
  "refreshToken": "def456ghi789..."
}

Success Response (200):
{
  "accessToken": "eyJhbGciOiJSUzI1NiIs...",
  "refreshToken": "ghi789jkl012...",
  "expiresIn": 900,
  "tokenType": "Bearer"
}

Error Responses:
401 Unauthorized:
{
  "error": {
    "code": "INVALID_REFRESH_TOKEN",
    "message": "Refresh token is invalid or expired"
  }
}
```

#### Logout Endpoint
```typescript
POST /api/v1/auth/logout
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...

Request Body:
{
  "everywhere": false  // Logout from all devices
}

Success Response (204):
// No content

Error Response:
401 Unauthorized:
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Token is invalid or expired"
  }
}
```

### Role-Based Access Control (RBAC)

#### User Roles Definition
```typescript
enum UserRole {
  OrderEntrySpecialist = 'OrderEntrySpecialist',
  ProcurementSpecialist = 'ProcurementSpecialist',
  Assembler = 'Assembler',
  QCPerson = 'QCPerson',
  ServiceDepartment = 'ServiceDepartment',
  Administrator = 'Administrator'
}

interface RolePermissions {
  [UserRole.OrderEntrySpecialist]: {
    orders: ['create', 'read', 'update'];
    customers: ['read'];
    catalog: ['read'];
    own_profile: ['read', 'update'];
  };
  
  [UserRole.ProcurementSpecialist]: {
    orders: ['read'];
    bom: ['read', 'update', 'approve', 'reject'];
    outsourcing: ['create', 'read', 'update'];
    suppliers: ['read'];
    own_profile: ['read', 'update'];
  };
  
  [UserRole.Assembler]: {
    orders: ['read'];
    assembly: ['read', 'update'];
    tasks: ['read', 'update', 'complete'];
    qc_checks: ['read', 'submit'];
    own_profile: ['read', 'update'];
  };
  
  [UserRole.QCPerson]: {
    orders: ['read'];
    qc: ['create', 'read', 'update', 'submit'];
    assembly: ['read'];
    testing: ['execute', 'record_results'];
    own_profile: ['read', 'update'];
  };
  
  [UserRole.ServiceDepartment]: {
    service_requests: ['create', 'read', 'update', 'approve'];
    parts_catalog: ['read'];
    customers: ['read'];
    orders: ['read'];
    own_profile: ['read', 'update'];
  };
  
  [UserRole.Administrator]: {
    '*': ['*'];  // Full access to all resources
  };
}
```

#### Permission Validation Middleware
```typescript
interface PermissionCheck {
  resource: string;
  action: string;
  conditions?: PermissionCondition[];
}

interface PermissionCondition {
  field: string;
  operator: 'equals' | 'not_equals' | 'in' | 'not_in' | 'greater_than' | 'less_than';
  value: any;
}

// Example permission checks
const ENDPOINT_PERMISSIONS: Record<string, PermissionCheck> = {
  'POST /api/v1/orders': {
    resource: 'orders',
    action: 'create'
  },
  'PUT /api/v1/orders/:id': {
    resource: 'orders',
    action: 'update',
    conditions: [
      {
        field: 'status',
        operator: 'in',
        value: ['Created', 'PartsOrderSent']
      }
    ]
  },
  'POST /api/v1/bom/:id/approve': {
    resource: 'bom',
    action: 'approve'
  }
};
```

---

## Security Framework

### Transport Security
```typescript
interface TransportSecurity {
  encryption: 'TLS 1.3 minimum';
  certificates: 'Extended Validation SSL certificates';
  hsts: 'Strict-Transport-Security header with 1 year max-age';
  certificatePinning: 'HTTP Public Key Pinning for mobile apps';
  ocspStapling: 'Online Certificate Status Protocol stapling';
}
```

### API Security Headers
```typescript
interface SecurityHeaders {
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload';
  'Content-Security-Policy': "default-src 'self'; script-src 'self' 'unsafe-inline'";
  'X-Content-Type-Options': 'nosniff';
  'X-Frame-Options': 'DENY';
  'X-XSS-Protection': '1; mode=block';
  'Referrer-Policy': 'strict-origin-when-cross-origin';
  'Permissions-Policy': 'camera=(), microphone=(), geolocation=()';
}
```

### Input Validation & Sanitization
```typescript
interface ValidationRules {
  email: {
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    maxLength: 254;
    sanitize: 'trim, lowercase';
  };
  
  password: {
    minLength: 12;
    pattern: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/;
    sanitize: 'none';  // Never log or expose passwords
  };
  
  poNumber: {
    pattern: /^[A-Z0-9\-_]{1,50}$/;
    sanitize: 'trim, uppercase';
  };
  
  buildNumber: {
    pattern: /^[A-Z0-9\-]{1,20}$/;
    sanitize: 'trim, uppercase';
  };
  
  notes: {
    maxLength: 2000;
    sanitize: 'trim, html_escape';
    allowedTags: [];  // No HTML tags allowed
  };
}
```

### Rate Limiting Strategy
```typescript
interface RateLimitConfiguration {
  global: {
    windowMs: 900000;      // 15 minutes
    max: 1000;             // Limit each IP to 1000 requests per windowMs
    message: 'Too many requests from this IP';
    standardHeaders: true;  // Return rate limit info in headers
    legacyHeaders: false;
  };
  
  authenticated: {
    windowMs: 3600000;     // 1 hour
    max: 5000;             // 5000 requests per hour for authenticated users
    keyGenerator: (req) => req.user.userId;
  };
  
  admin: {
    windowMs: 3600000;     // 1 hour
    max: 10000;            // 10000 requests per hour for admins
    keyGenerator: (req) => req.user.userId;
  };
  
  login: {
    windowMs: 900000;      // 15 minutes
    max: 5;                // Limit login attempts
    keyGenerator: (req) => req.ip + ':' + req.body.email;
    skipSuccessfulRequests: true;
  };
  
  passwordReset: {
    windowMs: 3600000;     // 1 hour
    max: 3;                // 3 password reset attempts per hour
    keyGenerator: (req) => req.body.email;
  };
}
```

---

## Core Data Models

### User Management Models
```typescript
interface User {
  userId: string;          // UUID
  email: string;           // Unique email address
  fullName: string;        // Full display name
  role: UserRole;          // Primary role
  department: string;      // Department/division
  isActive: boolean;       // Account status
  createdAt: Date;         // Account creation
  updatedAt: Date;         // Last profile update
  lastLogin?: Date;        // Last successful login
  emailVerified: boolean;  // Email verification status
  mfaEnabled: boolean;     // Multi-factor authentication
  preferences: UserPreferences;
  metadata: UserMetadata;
}

interface UserPreferences {
  language: 'en' | 'es' | 'fr';
  timezone: string;        // IANA timezone identifier
  dateFormat: 'MM/DD/YYYY' | 'DD/MM/YYYY' | 'YYYY-MM-DD';
  timeFormat: '12h' | '24h';
  notifications: NotificationPreferences;
  dashboard: DashboardPreferences;
}

interface NotificationPreferences {
  email: {
    orderUpdates: boolean;
    qcAlerts: boolean;
    systemMaintenance: boolean;
    weeklyReports: boolean;
  };
  inApp: {
    taskAssignments: boolean;
    approvalRequests: boolean;
    urgentAlerts: boolean;
    mentions: boolean;
  };
  frequency: 'immediate' | 'hourly' | 'daily';
}

interface UserMetadata {
  employeeId?: string;     // Employee identifier
  phoneNumber?: string;    // Contact number
  workLocation: string;    // Physical work location
  supervisor?: string;     // Supervisor user ID
  skills: string[];        // User skills/certifications
  trainingCompleted: string[]; // Completed training modules
  lastPasswordChange: Date;
  loginHistory: LoginHistoryEntry[];
}

interface LoginHistoryEntry {
  sessionId: string;
  loginTime: Date;
  logoutTime?: Date;
  ipAddress: string;
  userAgent: string;
  deviceFingerprint?: string;
  location?: GeoLocation;
}
```

### System Models
```typescript
interface APIResponse<T> {
  data: T;
  meta: ResponseMetadata;
  links?: HATEOASLinks;
}

interface ResponseMetadata {
  timestamp: Date;
  requestId: string;
  version: string;
  processingTime: number;  // milliseconds
  cached: boolean;
  cacheExpiry?: Date;
}

interface HATEOASLinks {
  self: string;
  next?: string;
  prev?: string;
  first?: string;
  last?: string;
  related?: Record<string, string>;
}

interface PaginatedResponse<T> {
  data: T[];
  pagination: PaginationMetadata;
  meta: ResponseMetadata;
  links: HATEOASLinks;
}

interface PaginationMetadata {
  page: number;
  limit: number;
  total: number;
  totalPages: number;
  hasNext: boolean;
  hasPrev: boolean;
}

interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: object;
    timestamp: Date;
    requestId: string;
    documentation?: string;
    suggestion?: string;
  };
}
```

### Audit & Logging Models
```typescript
interface AuditLogEntry {
  auditId: string;         // Unique audit entry ID
  userId?: string;         // User who performed action
  sessionId?: string;      // Session identifier
  action: AuditAction;     // Action performed
  resource: string;        // Resource type (orders, bom, etc.)
  resourceId?: string;     // Specific resource ID
  timestamp: Date;         // When action occurred
  ipAddress: string;       // Client IP address
  userAgent: string;       // Client user agent
  success: boolean;        // Whether action succeeded
  changes?: AuditChanges;  // What changed
  metadata: AuditMetadata;
}

enum AuditAction {
  CREATE = 'CREATE',
  READ = 'READ',
  UPDATE = 'UPDATE',
  DELETE = 'DELETE',
  LOGIN = 'LOGIN',
  LOGOUT = 'LOGOUT',
  APPROVE = 'APPROVE',
  REJECT = 'REJECT',
  SUBMIT = 'SUBMIT',
  EXPORT = 'EXPORT'
}

interface AuditChanges {
  before?: object;         // Previous values
  after?: object;          // New values
  fields: string[];        // Changed fields
}

interface AuditMetadata {
  complianceLevel: 'Standard' | 'Enhanced' | 'Critical';
  retentionPeriod: number; // Days to retain
  classification: 'Public' | 'Internal' | 'Confidential' | 'Restricted';
  regulatoryRequirement?: string; // ISO 13485, etc.
}
```

---

## Error Handling Framework

### Standard Error Codes
```typescript
enum APIErrorCode {
  // Client Errors (4xx)
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  INVALID_REQUEST = 'INVALID_REQUEST',
  UNAUTHORIZED = 'UNAUTHORIZED',
  FORBIDDEN = 'FORBIDDEN',
  NOT_FOUND = 'NOT_FOUND',
  METHOD_NOT_ALLOWED = 'METHOD_NOT_ALLOWED',
  CONFLICT = 'CONFLICT',
  GONE = 'GONE',
  PAYLOAD_TOO_LARGE = 'PAYLOAD_TOO_LARGE',
  UNSUPPORTED_MEDIA_TYPE = 'UNSUPPORTED_MEDIA_TYPE',
  UNPROCESSABLE_ENTITY = 'UNPROCESSABLE_ENTITY',
  RATE_LIMITED = 'RATE_LIMITED',
  
  // Server Errors (5xx)
  INTERNAL_ERROR = 'INTERNAL_ERROR',
  NOT_IMPLEMENTED = 'NOT_IMPLEMENTED',
  BAD_GATEWAY = 'BAD_GATEWAY',
  SERVICE_UNAVAILABLE = 'SERVICE_UNAVAILABLE',
  GATEWAY_TIMEOUT = 'GATEWAY_TIMEOUT',
  
  // Authentication Errors
  INVALID_CREDENTIALS = 'INVALID_CREDENTIALS',
  TOKEN_EXPIRED = 'TOKEN_EXPIRED',
  TOKEN_INVALID = 'TOKEN_INVALID',
  ACCOUNT_LOCKED = 'ACCOUNT_LOCKED',
  MFA_REQUIRED = 'MFA_REQUIRED',
  
  // Business Logic Errors
  INVALID_STATUS_TRANSITION = 'INVALID_STATUS_TRANSITION',
  INSUFFICIENT_PERMISSIONS = 'INSUFFICIENT_PERMISSIONS',
  RESOURCE_LOCKED = 'RESOURCE_LOCKED',
  DEPENDENCY_ERROR = 'DEPENDENCY_ERROR',
  BUSINESS_RULE_VIOLATION = 'BUSINESS_RULE_VIOLATION'
}
```

### Error Response Structure
```typescript
interface DetailedErrorResponse {
  error: {
    code: APIErrorCode;
    message: string;
    timestamp: Date;
    requestId: string;
    path: string;
    method: string;
    
    // Additional error details
    details?: {
      field?: string;
      value?: any;
      constraint?: string;
      validation?: string;
      allowedValues?: any[];
    };
    
    // Nested errors for batch operations
    errors?: SubError[];
    
    // Help information
    documentation?: string;
    suggestion?: string;
    supportContact?: string;
  };
}

interface SubError {
  code: string;
  message: string;
  field?: string;
  index?: number;  // For array items
}
```

### HTTP Status Code Mapping
```typescript
const ERROR_STATUS_MAPPING: Record<APIErrorCode, number> = {
  // 400 Bad Request
  [APIErrorCode.VALIDATION_ERROR]: 400,
  [APIErrorCode.INVALID_REQUEST]: 400,
  [APIErrorCode.BUSINESS_RULE_VIOLATION]: 400,
  
  // 401 Unauthorized
  [APIErrorCode.UNAUTHORIZED]: 401,
  [APIErrorCode.INVALID_CREDENTIALS]: 401,
  [APIErrorCode.TOKEN_EXPIRED]: 401,
  [APIErrorCode.TOKEN_INVALID]: 401,
  
  // 403 Forbidden
  [APIErrorCode.FORBIDDEN]: 403,
  [APIErrorCode.INSUFFICIENT_PERMISSIONS]: 403,
  [APIErrorCode.ACCOUNT_LOCKED]: 403,
  
  // 404 Not Found
  [APIErrorCode.NOT_FOUND]: 404,
  
  // 405 Method Not Allowed
  [APIErrorCode.METHOD_NOT_ALLOWED]: 405,
  
  // 409 Conflict
  [APIErrorCode.CONFLICT]: 409,
  [APIErrorCode.RESOURCE_LOCKED]: 409,
  [APIErrorCode.INVALID_STATUS_TRANSITION]: 409,
  
  // 422 Unprocessable Entity
  [APIErrorCode.UNPROCESSABLE_ENTITY]: 422,
  
  // 429 Too Many Requests
  [APIErrorCode.RATE_LIMITED]: 429,
  
  // 500 Internal Server Error
  [APIErrorCode.INTERNAL_ERROR]: 500,
  [APIErrorCode.DEPENDENCY_ERROR]: 500,
  
  // 503 Service Unavailable
  [APIErrorCode.SERVICE_UNAVAILABLE]: 503
};
```

---

## Data Validation Framework

### Validation Schema System
```typescript
interface ValidationSchema {
  type: 'object' | 'array' | 'string' | 'number' | 'boolean' | 'date';
  required?: boolean;
  properties?: Record<string, ValidationSchema>;
  items?: ValidationSchema;  // For arrays
  enum?: any[];
  pattern?: string;         // Regex pattern
  minLength?: number;
  maxLength?: number;
  min?: number;            // For numbers
  max?: number;            // For numbers
  format?: ValidationFormat;
  custom?: CustomValidator;
}

enum ValidationFormat {
  EMAIL = 'email',
  UUID = 'uuid',
  DATE = 'date',
  DATETIME = 'datetime',
  URI = 'uri',
  PHONE = 'phone',
  POSTAL_CODE = 'postal_code'
}

type CustomValidator = (value: any, context?: ValidationContext) => ValidationResult;

interface ValidationContext {
  user?: User;
  request?: any;
  resource?: any;
}

interface ValidationResult {
  isValid: boolean;
  errors?: ValidationError[];
}

interface ValidationError {
  field: string;
  code: string;
  message: string;
  value?: any;
}
```

### Common Validation Patterns
```typescript
const VALIDATION_PATTERNS = {
  email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  uuid: /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i,
  poNumber: /^[A-Z0-9\-_]{1,50}$/,
  buildNumber: /^[A-Z0-9\-]{1,20}$/,
  partNumber: /^[A-Z0-9\-\.]{1,30}$/,
  phoneNumber: /^\+?[1-9]\d{1,14}$/,
  strongPassword: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{12,}$/
};

const VALIDATION_SCHEMAS = {
  user: {
    type: 'object',
    required: true,
    properties: {
      email: {
        type: 'string',
        required: true,
        format: ValidationFormat.EMAIL,
        maxLength: 254
      },
      fullName: {
        type: 'string',
        required: true,
        minLength: 2,
        maxLength: 100,
        pattern: /^[a-zA-Z\s\-'\.]+$/
      },
      role: {
        type: 'string',
        required: true,
        enum: Object.values(UserRole)
      }
    }
  },
  
  orderBasic: {
    type: 'object',
    required: true,
    properties: {
      poNumber: {
        type: 'string',
        required: true,
        pattern: VALIDATION_PATTERNS.poNumber,
        maxLength: 50
      },
      customerName: {
        type: 'string',
        required: true,
        minLength: 2,
        maxLength: 100
      }
    }
  }
} as const;
```

---

## Caching Strategy

### Cache Configuration
```typescript
interface CacheConfiguration {
  // Redis cache settings
  redis: {
    host: 'redis.cleanstation.internal';
    port: 6379;
    password: '${REDIS_PASSWORD}';
    db: 0;
    keyPrefix: 'cleanstation:';
    ttl: {
      default: 3600;      // 1 hour
      short: 300;         // 5 minutes
      medium: 1800;       // 30 minutes
      long: 86400;        // 24 hours
    };
  };
  
  // HTTP cache headers
  httpCache: {
    publicAssets: 'public, max-age=31536000, immutable';
    apiResponses: 'private, max-age=300, must-revalidate';
    userProfile: 'private, max-age=900, must-revalidate';
    catalog: 'public, max-age=3600, must-revalidate';
    sensitive: 'no-cache, no-store, must-revalidate';
  };
  
  // Cache strategies by endpoint
  strategies: {
    '/api/v1/catalog/**': 'long';
    '/api/v1/users/profile': 'medium';
    '/api/v1/orders/**': 'short';
    '/api/v1/qc/**': 'no-cache';
    '/api/v1/auth/**': 'no-cache';
  };
}

interface CacheEntry<T> {
  key: string;
  value: T;
  timestamp: Date;
  ttl: number;
  tags: string[];        // For cache invalidation
  metadata: CacheMetadata;
}

interface CacheMetadata {
  version: string;
  userId?: string;
  role?: string;
  dependencies: string[];  // Related cache keys
}
```

---

## Performance & Monitoring

### Performance Metrics
```typescript
interface PerformanceMetrics {
  responseTime: {
    p50: number;         // 50th percentile
    p95: number;         // 95th percentile
    p99: number;         // 99th percentile
    average: number;
    maximum: number;
  };
  
  throughput: {
    requestsPerSecond: number;
    requestsPerMinute: number;
    requestsPerHour: number;
  };
  
  errorRates: {
    total: number;       // Total error rate
    client: number;      // 4xx errors
    server: number;      // 5xx errors
    timeout: number;     // Timeout errors
  };
  
  resources: {
    cpuUsage: number;    // Percentage
    memoryUsage: number; // Bytes
    diskUsage: number;   // Bytes
    networkIO: number;   // Bytes/second
  };
  
  database: {
    connectionPool: number;
    queryTime: number;
    activeQueries: number;
    deadlocks: number;
  };
}
```

### Health Check Endpoints
```typescript
GET /api/v1/health/status
Response:
{
  "status": "healthy" | "unhealthy" | "degraded",
  "timestamp": "2024-12-01T10:30:00Z",
  "version": "1.0.0",
  "uptime": 86400,
  "checks": {
    "database": {
      "status": "healthy",
      "responseTime": 15,
      "details": "Connected to primary database"
    },
    "redis": {
      "status": "healthy",
      "responseTime": 2,
      "details": "Cache operational"
    },
    "external_services": {
      "status": "degraded",
      "details": "Email service responding slowly"
    }
  }
}

GET /api/v1/health/metrics
Response:
{
  "performance": PerformanceMetrics,
  "timestamp": "2024-12-01T10:30:00Z",
  "period": "last_hour"
}
```

This concludes Part A of the API specification, covering core architecture, authentication, security, and fundamental data models. The next sections will cover specific functional APIs for orders, quality control, and administrative functions. 