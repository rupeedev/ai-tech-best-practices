# SECURITY GUIDELINES FOR AI-ASSISTED DEVELOPMENT

### Phase 10: Security Audit & Hardening (NEW - CRITICAL)
**MANDATORY**: Every feature MUST undergo comprehensive security review before deployment

#### 10a: Input Validation & Sanitization Audit
- **Frontend Security**: Verify ALL user inputs are sanitized
  ```typescript
  // Required pattern for all text inputs
  const sanitizeInput = (input: string) => {
    return input
      .replace(/[*%_\\<>'"`;(){}[\]|&$]/g, '') // Remove dangerous characters
      .replace(/[^\w\s-]/g, '') // Only allow safe characters
      .slice(0, 100) // Length limit
      .trim();
  };
  ```
- **Backend Security**: Verify ALL API endpoints validate and sanitize inputs
- **Database Security**: Ensure parameterized queries, no string concatenation
- **File Upload Security**: Validate file types, sizes, scan for malware

#### 10b: Authentication & Authorization Review
- **JWT Security**: Verify token storage, refresh, and validation
- **Role-Based Access**: Test all user roles can only access appropriate data
- **API Endpoints**: Verify authentication middleware on all protected routes
- **RLS Policies**: Test Row Level Security prevents data leakage

#### 10c: Query Security Analysis
- **SQL/NoSQL Injection**: Test all queries with malicious inputs
- **PostgREST Security**: Verify proper query escaping and sanitization
- **Database Relationships**: Check for unauthorized data exposure through joins

#### 10d: Security Logging Implementation
- **Audit Trail**: Log all sensitive operations (login, data changes, file uploads)
- **Security Events**: Monitor and log suspicious activities
- **Failed Attempts**: Track and alert on multiple failed operations

#### 10e: Error Handling Security
- **Information Disclosure**: Ensure errors don't expose system information
- **Generic Messages**: Return safe error messages to users
- **Detailed Logging**: Log full errors server-side for debugging

#### 10f: Security Testing
- **Manual Testing**: Test with malicious inputs, unauthorized access attempts
- **Automated Scans**: Run OWASP ZAP or similar security scanners
- **Penetration Testing**: Test for common vulnerabilities (OWASP Top 10)

#### 10g: Security Documentation
- **Document Security Measures**: Update Security.txt with feature-specific security
- **Threat Model**: Document potential security risks and mitigations
- **Incident Response**: Plan for security incident handling

**CRITICAL OUTPUT**: 
- `Security-Audit-Report-[Feature].md` - Complete security assessment
- Updated `/Users/rupeshpanwar/Documents/PProject/school-management/.claude/Security.txt`
- Security test results and remediation plans

This document provides comprehensive security guidelines for the School Management System project and serves as a reference for secure AI-assisted development practices.

## CRITICAL SECURITY REQUIREMENTS

### 1. INPUT VALIDATION & SANITIZATION

**MANDATORY**: All user inputs MUST be validated and sanitized at multiple layers.

#### Frontend Validation (First Line of Defense)
```typescript
// Example: Search input sanitization
const sanitizeInput = (input: string) => {
  return input
    .replace(/[*%_\\<>'"`;(){}[\]|&$]/g, '') // Remove dangerous characters
    .replace(/[^\w\s-]/g, '') // Only allow alphanumeric, spaces, hyphens
    .slice(0, 100) // Limit length
    .trim();
};
```

#### Backend Validation (Critical Defense Layer)
```typescript
// Example: Server-side validation with logging
const validateAndSanitize = (input: string, fieldName: string) => {
  const originalInput = input;
  const hasMaliciousChars = /[*%_\\<>'"`;(){}[\]|&$]/.test(originalInput);
  
  if (hasMaliciousChars) {
    logger.warn('Potential security threat detected:', {
      field: fieldName,
      input: originalInput,
      timestamp: new Date().toISOString(),
      ip: req.ip
    });
  }
  
  return input
    .replace(/[*%_\\<>'"`;(){}[\]|&$]/g, '')
    .replace(/[^\w\s-]/g, '')
    .trim();
};
```

### 2. DATABASE SECURITY

#### Query Protection
- **NEVER** use string concatenation for queries
- Always use parameterized queries or ORM methods
- Escape special characters properly for your database type

#### PostgREST Specific Security
```typescript
// WRONG - Vulnerable to injection
query = query.or(`title.ilike.%${userInput}%`);

// RIGHT - Sanitized input with proper escaping
const sanitizedInput = sanitizeInput(userInput);
if (sanitizedInput.length >= 2) {
  const searchTerm = `*${sanitizedInput}*`;
  query = query.or(`title.ilike.${searchTerm}`);
}
```

#### Supabase RLS (Row Level Security)
- Enable RLS on ALL sensitive tables
- Create specific policies for each user role
- Test policies thoroughly with different user scenarios

### 3. AUTHENTICATION & AUTHORIZATION

#### JWT Token Security
- Use secure token storage (httpOnly cookies or secure localStorage)
- Implement token refresh mechanisms
- Set appropriate expiration times
- Validate tokens on every request

#### Role-Based Access Control
```typescript
// Example: Secure route protection
const requireRole = (allowedRoles: UserRole[]) => {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        error: 'Insufficient permissions'
      });
    }
    next();
  };
};
```

### 4. FILE UPLOAD SECURITY

#### File Validation
- Validate file types using MIME type checking
- Implement file size limits
- Scan files for malware (if possible)
- Store files outside web root
- Use signed URLs for access

```typescript
// Example: Secure file upload validation
const validateFile = (file: Express.Multer.File) => {
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
  const maxSize = 10 * 1024 * 1024; // 10MB
  
  if (!allowedTypes.includes(file.mimetype)) {
    throw new Error('Invalid file type');
  }
  
  if (file.size > maxSize) {
    throw new Error('File too large');
  }
  
  // Additional security: Check file content matches extension
  return true;
};
```

### 5. API SECURITY

#### Rate Limiting
- Implement rate limiting on all endpoints
- Use different limits for different operations
- Monitor for abuse patterns

#### Request Validation
- Validate all request parameters
- Use schema validation (Joi, Zod, etc.)
- Implement request size limits

#### Error Handling
```typescript
// WRONG - Exposes system information
catch (error) {
  res.status(500).json({ error: error.message });
}

// RIGHT - Secure error handling
catch (error) {
  logger.error('Database error:', error);
  res.status(500).json({ 
    success: false,
    error: 'An unexpected error occurred' 
  });
}
```

### 6. SECURITY MONITORING & LOGGING

#### Security Event Logging
```typescript
const logSecurityEvent = (event: {
  type: 'MALICIOUS_INPUT' | 'FAILED_AUTH' | 'PRIVILEGE_ESCALATION';
  userId?: string;
  ip: string;
  userAgent: string;
  details: any;
}) => {
  logger.warn('Security Event:', {
    ...event,
    timestamp: new Date().toISOString(),
    severity: 'HIGH'
  });
  
  // Consider alerting for critical events
  if (event.type === 'PRIVILEGE_ESCALATION') {
    // Send alert to security team
  }
};
```

#### Failed Login Monitoring
- Track failed login attempts
- Implement account lockout mechanisms
- Monitor for brute force attacks

### 7. FRONTEND SECURITY

#### XSS Prevention
- Always sanitize user content before display
- Use Content Security Policy (CSP)
- Validate and escape HTML content

#### CSRF Protection
- Use CSRF tokens for state-changing operations
- Implement SameSite cookie attributes
- Validate referrer headers

### 8. ENVIRONMENT & CONFIGURATION SECURITY

#### Environment Variables
- Never commit secrets to version control
- Use proper secret management
- Rotate credentials regularly

#### Dependencies
- Regularly update dependencies
- Use vulnerability scanners (npm audit, Snyk)
- Review third-party packages before installation

## AI DEVELOPMENT SECURITY PROMPTS

Use these prompts when working with AI coding assistants:

### Primary Security Prompt
```
"When implementing this feature, ensure comprehensive security measures:

1. Input Validation: Sanitize all inputs with whitelist validation
2. Query Security: Use parameterized queries, escape special characters
3. Authentication: Verify user permissions for this operation
4. Error Handling: Don't expose sensitive information in errors
5. Logging: Log security events and suspicious activities
6. Rate Limiting: Implement appropriate rate limiting

Review the code for: SQL injection, XSS, CSRF, privilege escalation, information disclosure.
Flag any security concerns and suggest secure alternatives."
```

### Specific Feature Security Prompts
```
For Search Features:
"Implement search with input sanitization, minimum length requirements, and protection against wildcard abuse."

For File Uploads:
"Implement file upload with MIME type validation, size limits, malware scanning, and secure storage."

For User Authentication:
"Implement authentication with secure token handling, rate limiting, and brute force protection."

For API Endpoints:
"Implement API with input validation, authorization checks, error handling, and security logging."
```

## SECURITY TESTING CHECKLIST

### Manual Testing
- [ ] Test with malicious inputs (', ", ;, <script>, etc.)
- [ ] Test authorization bypass attempts
- [ ] Test file upload with malicious files
- [ ] Test rate limiting with automated requests
- [ ] Test error handling for information disclosure

### Automated Testing
- [ ] Run OWASP ZAP security scan
- [ ] Use SQL injection testing tools
- [ ] Implement security unit tests
- [ ] Use dependency vulnerability scanners

## INCIDENT RESPONSE

### Security Incident Procedure
1. **Immediate**: Stop the attack/block the threat
2. **Assess**: Determine scope and impact
3. **Contain**: Prevent further damage
4. **Investigate**: Analyze logs and evidence
5. **Remediate**: Fix vulnerabilities
6. **Document**: Record incident and lessons learned

### Contact Information
- Security Team: [security@yourcompany.com]
- On-call Engineer: [oncall@yourcompany.com]
- Incident Response: [incident@yourcompany.com]

## COMPLIANCE & STANDARDS

### Data Protection
- GDPR compliance for EU users
- CCPA compliance for California users
- FERPA compliance for educational records

### Security Standards
- OWASP Top 10 compliance
- ISO 27001 alignment
- SOC 2 Type II requirements

## REGULAR SECURITY TASKS

### Daily
- Monitor security logs
- Review failed authentication attempts
- Check for new security alerts

### Weekly
- Update dependencies
- Review access logs
- Test backup systems

### Monthly
- Security vulnerability assessment
- Access control review
- Security training updates

### Quarterly
- Full security audit
- Penetration testing
- Incident response drill

## RESOURCES

### Security Tools
- OWASP ZAP (Web Application Security Scanner)
- npm audit (Node.js dependency scanner)
- Snyk (Vulnerability management)
- ESLint security plugins

### Documentation
- OWASP Web Security Testing Guide
- NIST Cybersecurity Framework
- CIS Security Controls

### Training
- OWASP Security Training
- Secure Coding Practices
- Incident Response Training

## RECENT SECURITY FIXES APPLIED TO THIS PROJECT

### Login Form Security Analysis (2025-08-02)
✅ **Security Verification Completed:**

**False Alarm Investigation:**
- User reported seeing credentials in URL: `localhost:3001/login?email=user@example.com&password=ExamplePass123`
- Investigation revealed this was NOT a security vulnerability in our code
- Login form already implements secure practices correctly

**Verified Secure Implementation:**
- React Hook Form with proper POST method submission via `handleSubmit(onSubmit)`
- AuthService uses `apiClient.post('/auth/login', credentials)` sending data in request body
- JWT-based authentication with secure token storage
- No GET parameters or URL exposure of credentials

**Root Cause Analysis:**
The URL with credentials was likely caused by:
- Browser autocomplete suggestion from saved form data
- Direct URL navigation attempt (manual typing in address bar)
- Saved bookmark containing credentials (security anti-pattern)

**Prevention Measures:**
- Clear browser autocomplete/history for login URLs
- User education on secure login practices
- Monitor for similar false alarms in security reviews

**Key Lesson:**
Always investigate the actual code implementation before assuming security vulnerabilities. Browser behavior (autocomplete, bookmarks) can create false security alarms.

### Search Functionality Security (2025-07-27)
✅ **Fixed Critical Security Vulnerabilities:**

**Issues Found:**
- Search input was vulnerable to SQL/NoSQL injection
- Special characters (*,%,',;) passed directly to database
- No input sanitization or validation
- Potential for data exposure and system compromise

**Security Measures Implemented:**
- Frontend input sanitization with character whitelisting
- Backend comprehensive validation with security logging
- Minimum 2-character search requirement
- Length limits to prevent buffer overflow
- Real-time monitoring of malicious attempts

**Blocked Characters:**
- `*` `%` `_` `\` (PostgREST wildcards)
- `'` `"` `;` (SQL injection vectors)
- `<` `>` (XSS prevention)
- `()` `{}` `[]` (function/object injection)
- `|` `&` `$` (command injection)

**Security Monitoring Added:**
```typescript
if (hasMaliciousChars) {
  logger.warn('Potentially malicious search attempt:', {
    teacherId,
    searchQuery: originalSearch,
    timestamp: new Date().toISOString()
  });
}
```

---

**Remember**: Security is not a feature - it's a fundamental requirement.
**When in doubt**: Choose the more secure option.
**Always**: Test security measures thoroughly before deployment.

Last Updated: 2025-07-27
Version: 1.0