
# LeaveFlow Auth Service - Copilot Instructions

## 1. Project Overview

This microservice handles authentication and authorization for LeaveFlow, an Employee Leave Management System built using microservices architecture.

It is responsible for user registration, login, password management and JWT token generation.

## 2. Technology Stack

- Java 21
- Spring Boot 3.5.x
- Spring Security
- Spring Data JPA
- Hibernate
- PostgreSQL
- Maven
- Lombok
- Bean Validation
- Spring Boot Actuator
- JUnit 5
- Mockito
- JWT using RSA asymmetric signing

## 3. Application Configuration

Service name: leaveflow-auth-service

Port: 8081

Database: leaveflow_auth

Package name:
com.leaveflow.auth

## 4. Package Structure

Follow this package structure:

com.leaveflow.auth
config
controller
service
repository
entity
dto
request
response
exception
security
util

Keep authentication logic separate from controllers and database operations.

## 5. User Entity

Create a User entity with:

- id (UUID)
- firstName (String)
- lastName (String)
- email (String, unique)
- password (String)
- role (enum)
- active (Boolean)
- createdAt (LocalDateTime)
- updatedAt (LocalDateTime)

Roles:

- EMPLOYEE
- MANAGER
- ADMIN

Requirements:

- Email must be unique.
- Store passwords using BCrypt hashing.
- Never store plain-text passwords.
- Default role for public registration must be EMPLOYEE.
- Never allow users to assign themselves MANAGER or ADMIN during public registration.
- Administrative role assignment must be restricted to a trusted administrative operation.

## 6. API Endpoints

### Registration

POST /api/auth/register

Request:

{
"firstName": "John",
"lastName": "Smith",
"email": "john@example.com",
"password": "Password123"
}

Requirements:

- Validate request fields.
- Check whether email already exists.
- Hash password using BCrypt.
- Assign EMPLOYEE role by default.
- Save user in PostgreSQL.
- Return safe user details.
- Never return password hashes.

### Login

POST /api/auth/login

Request:

{
"email": "john@example.com",
"password": "Password123"
}

Requirements:

- Authenticate using Spring Security AuthenticationManager.
- Verify password using BCrypt.
- Reject invalid credentials.
- Reject inactive accounts.
- Generate JWT access token.
- Return token and user details.

### Additional endpoints

GET /api/auth/me

Returns the authenticated user's profile.

GET /api/auth/health

Returns service health information.

POST /api/auth/logout

For the initial MVP, logout should clear the frontend's in-memory token.
Do not claim that a stateless JWT has been revoked unless a revocation mechanism is implemented.

## 7. JWT Implementation

Use RSA asymmetric cryptography.

Auth Service:
- Generates JWT tokens.
- Signs JWT using RSA private key.
- Keeps private key outside source control.
- Loads private key through secure configuration.

API Gateway:
- Verifies JWT signatures using the corresponding public key.
- Never requires the Auth Service private key.

JWT claims:

- sub: User UUID
- email: User email
- roles: User roles
- iss: leaveflow-auth-service
- iat: Issued timestamp
- exp: Expiration timestamp

Token requirements:

- Use RS256 algorithm.
- Configure token expiration.
- Validate issuer and expiration.
- Never put passwords or sensitive information in JWT claims.
- Never log JWT tokens.
- Use a configurable signing key location.
- Do not hardcode private keys.

Generate and configure RSA keys securely for local development.
Do not commit private or public key files containing real deployment secrets.

Use Spring Security OAuth2 JOSE / Nimbus JWT support where appropriate.
Avoid unnecessary third-party JWT libraries.

## 8. Spring Security Configuration

Implement:

- SecurityFilterChain
- PasswordEncoder using BCryptPasswordEncoder
- AuthenticationManager
- UserDetailsService
- Custom UserDetails implementation if required.

Authorization:

- POST /api/auth/register: Public
- POST /api/auth/login: Public
- GET /api/auth/health: Public
- GET /api/auth/me: Authenticated

Other administrative operations must require appropriate roles.

Use stateless session management.

Disable CSRF for stateless REST authentication endpoints where appropriate.

Configure CORS for the React frontend through a consistent gateway policy.

Do not create unnecessary custom authentication filters when Spring Security already provides the required functionality.

## 9. Authentication Flow

Registration:

1. React sends registration request.
2. Auth Controller receives the request.
3. Service validates the user.
4. BCrypt hashes the password.
5. User is persisted in PostgreSQL.
6. Return registration response.

Login:

1. React sends email and password.
2. Auth Controller receives the request.
3. AuthenticationManager authenticates credentials.
4. Spring Security verifies the BCrypt password.
5. JWT service generates an RSA-signed JWT.
6. Return access token and user details.

Subsequent requests:

1. React sends JWT to API Gateway.
2. Gateway verifies JWT using RSA public key.
3. Gateway extracts user identity and roles.
4. Gateway forwards authorized requests to downstream services.
5. Backend services trust only authenticated gateway traffic and must not accept spoofable client-supplied identity headers.

## 10. Database Configuration

Use PostgreSQL.

Database name: leaveflow_auth

Configure through environment variables:

DB_URL
DB_USERNAME
DB_PASSWORD

JWT_PRIVATE_KEY_LOCATION
JWT_ISSUER
JWT_EXPIRATION

Never hardcode database credentials or private keys.

Use JPA and Hibernate.

Configure appropriate database indexes and unique constraints.

## 11. Exception Handling

Implement global exception handling.

Handle:

- Duplicate email registration.
- Invalid login credentials.
- User not found.
- Inactive user.
- Validation failures.
- JWT generation failures.
- Database exceptions.

Return consistent error responses.

Never expose password hashes, private key information or internal stack traces.

## 12. Coding Standards

- Follow layered architecture.
- Use constructor injection.
- Use DTOs for requests and responses.
- Never expose JPA entities directly.
- Use Bean Validation.
- Use meaningful exception classes.
- Use @Transactional where required.
- Keep business logic in service classes.
- Avoid unnecessary abstractions.
- Write clean, maintainable Java code.

## 13. Testing

Use JUnit 5 and Mockito.

Implement tests for:

- Successful registration.
- Duplicate email registration.
- Password hashing.
- Successful login.
- Invalid password.
- Unknown email.
- Inactive account.
- JWT generation.
- JWT signature verification.
- Token expiration.
- Role claims.
- Public and protected endpoint access.

## 14. Important Development Rules

Before implementing:

1. Inspect existing pom.xml.
2. Inspect existing Java packages.
3. Inspect application.properties.
4. Identify missing dependencies.
5. Present the implementation plan.

Do not regenerate the existing Spring Boot project.

Do not modify unrelated files.

Do not add arbitrary dependency versions.

Do not hardcode credentials or signing keys.

Do not implement all features in one large change.

Implement incrementally and run Maven tests after each major feature.

Explain important implementation decisions.
