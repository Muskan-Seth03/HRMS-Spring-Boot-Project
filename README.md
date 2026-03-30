
# HRMS (Human Resource Management System)

## Project Description
HRMS (Human Resource Management System) is a Spring Boot-based web service application designed for managing job postings, applications, employer and candidate information.  
It follows the REST API architecture and implements **DTO**, **Request-Response Pattern**, **Validation**, and **Global Exception Handling**.

---

##  Features
- **City Management**: Add and list cities.
- **Job Position Management**: Add and list job positions.
- **Employer Management**: Register and list employers.
- **Candidate Management**: Register and list job seekers.
- **Job Advertisement Management**: Add, list, and filter job ads.
- **Job Application Management**: Allow candidates to apply for job ads.
- **Error Handling**: Global exception handling with `@ControllerAdvice`.
- **Validation**: Field validation with annotations like `@NotBlank`, `@Size`.

---

##  Technologies Used
- **Java 17**
- **Spring Boot**
- **Spring Data JPA (Hibernate)**
- **PostgreSQL**
- **Lombok**
- **Validation API (Jakarta Validation)**
- **Jackson**
- **Postman** for API testing

---

##  Project Layers
- **Entity**: Represents database tables.
- **DTO**: Data Transfer Objects for API responses.
- **Request**: Classes for incoming API data.
- **Service**: Business logic layer.
- **Repository (DAO)**: Database access layer.
- **Controller**: REST API endpoints.
- **Core Utilities**: Common response classes like `Result`, `DataResult`, `SuccessResult`, `ErrorResult`.

**Result Structure:**
- `Result`: Returns success/failure status and a message.
- `DataResult<T>`: Returns status + data.
- `SuccessResult`, `ErrorResult`: Ready-made classes for success/error cases.

---

##  Sample API Endpoints
| HTTP | Endpoint | Description |
|------|----------|-------------|
| POST | `/api/employers/register` | Register a new employer |
| GET  | `/api/employers/getAll` | Get all employers |
| POST | `/api/candidateController/register` | Register a new candidate |
| GET  | `/api/candidateController/getAll` | Get all candidates |
| POST | `/api/jobAdvertisements/add` | Add a new job advertisement |
| GET  | `/api/jobAdvertisements/getAll` | Get all job advertisements |
| POST | `/api/jobApplications/apply` | Apply for a job advertisement |

---

##  Sample JSON Requests

**Employer Registration:**
```json
{
  "companyName": "Tech Solutions Ltd.",
  "companyWebPage": "https://techsolutions.com",
  "email": "contact@techsolutions.com",
  "phoneNumber": "+1-555-123-4567",
  "password": "password123",
  "confirmPassword": "password123"
}
```
**Candidate Registration:**
```json
{
    "name": "Aysu",
    "lastName": "Ay",
    "nationalId": "12345678901",
    "birthDate": 2000,
    "email": "aysu@example.com",
    "password": "password123",
    "confirmPassword": "password123"
}

```
