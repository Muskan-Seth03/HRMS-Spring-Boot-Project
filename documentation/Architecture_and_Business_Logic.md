# HRMS Spring Boot Project - Architecture & Design Documentation

## Table of Contents
1. [Architecture Overview](#1-architecture-overview)
2. [Layered Architecture Diagram](#2-layered-architecture-diagram)
3. [Role of Each Layer](#3-role-of-each-layer)
4. [Request-Response Flow](#4-request-response-flow)
5. [Entity Relationship Analysis](#5-entity-relationship-analysis)
6. [Business Logic Explanation](#6-business-logic-explanation)
7. [System Workflow](#7-system-workflow)

---

## 1. Architecture Overview

This Spring Boot project follows a **Layered Architecture** (also called N-Tier Architecture). Think of it like a restaurant:
- **Client/Presentation**: Customer placing order
- **Controller**: Waiter taking the order
- **Service**: Chef preparing the meal
- **Repository**: Pantry fetching ingredients
- **Database**: Storage of all ingredients

Each layer has a specific job and communicates only with adjacent layers.

---

## 2. Layered Architecture Diagram

### Simple Flow Diagram:

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT / API USER                         │
│              (Web Browser, Mobile App, etc.)                 │
└────────────────────────┬────────────────────────────────────┘
                         │
                         │ HTTP Request (JSON)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              CONTROLLER LAYER                                │
│   (JobAdvertisementController, EmployerController, etc.)    │
│   Role: Receive request, validate input, call service       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         │ Pass validated data
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              SERVICE LAYER                                   │
│  (JobAdvertisementManager, EmployerManager, etc.)           │
│  Role: Business logic, validation, data transformation      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         │ Request to fetch/save data
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              REPOSITORY LAYER                                │
│  (JobAdvertisementDao, EmployerDao, etc.)                   │
│  Role: Database operations using JPA/Hibernate              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         │ SQL Queries
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              DATABASE LAYER                                  │
│              (MySQL Database)                                │
│  Tables: cities, employers, job_positions,                  │
│          job_advertisements, job_seekers,                   │
│          job_applications, system_employee                  │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow - Request to Response:

```
┌──────────┐
│ REQUEST  │ (Client sends JSON data)
└────┬─────┘
     │
     ▼
┌──────────────────────────────────────┐
│ CONTROLLER                           │
│ 1. Receive HTTP request              │
│ 2. Validate using @Valid annotation  │
│ 3. Extract data from request body    │
└────┬─────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────┐
│ SERVICE                              │
│ 1. Apply business rules              │
│ 2. Check validations                 │
│ 3. Transform data if needed          │
│ 4. Call repository methods           │
└────┬─────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────┐
│ REPOSITORY                           │
│ 1. Execute JPA/Hibernate queries     │
│ 2. Communicate with database         │
│ 3. Return entity objects             │
└────┬─────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────┐
│ DATABASE                             │
│ 1. Execute SQL statements            │
│ 2. Fetch/Insert/Update/Delete data   │
│ 3. Return results                    │
└────┬─────────────────────────────────┘
     │
     ▼ (Response flows back up)
┌──────────────────────────────────────┐
│ REPOSITORY                           │
│ Returns entity data to service       │
└────┬─────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────┐
│ SERVICE                              │
│ Converts entity to DTO (Data Transfer│
│ Object) for API response             │
└────┬─────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────┐
│ CONTROLLER                           │
│ Returns JSON response with status    │
└────┬─────────────────────────────────┘
     │
     ▼
┌──────────┐
│ RESPONSE │ (Client receives JSON data)
└──────────┘
```

---

## 3. Role of Each Layer

### **Layer 1: Client / Presentation Layer**
- **What it is**: Your mobile app, web browser, or API testing tool (like Postman)
- **What it does**: Sends HTTP requests (GET, POST, PUT, DELETE) with data
- **Example**: User clicks "Apply Now" button → sends POST request to `/api/applications/apply`
- **Simple Terms**: "I'm the person asking for something"

### **Layer 2: Controller Layer**
- **What it is**: Spring REST Controllers (@RestController)
- **Location**: `hrms.hrms.controller` package
- **What it does**:
  1. Listens for incoming HTTP requests
  2. Validates request data using @Valid annotation
  3. Extracts data from request body
  4. Calls appropriate service method
  5. Returns JSON response
- **Example**: `JobAdvertisementController.add()` receives POST request and calls `jobAdvertisementService.add()`
- **Simple Terms**: "I'm the receptionist. I receive your request, check if it's valid, and pass it to the right department."
- **Key Annotations**:
  - `@RestController`: Marks class as REST endpoint
  - `@PostMapping`, `@GetMapping`: HTTP method mapping
  - `@RequestBody`: Gets data from request body
  - `@RequestParam`: Gets data from query parameters
  - `@Valid`: Validates input data

### **Layer 3: Service Layer**
- **What it is**: Business logic and service implementations
- **Location**: `hrms.hrms.business.abstracts` (interfaces) and `hrms.hrms.business.concretes` (implementations)
- **What it does**:
  1. Implements business rules and validations
  2. Processes data transformation
  3. Orchestrates repository calls
  4. Handles errors and exceptions
  5. Returns Result objects (success/error)
- **Example**: `JobAdvertisementManager.add()` checks if min salary < max salary, fetches related entities, and saves
- **Simple Terms**: "I'm the manager. I check if the request makes sense, apply company rules, and prepare everything before storing."
- **Key Patterns**:
  - Service interfaces define contracts
  - Implementations (@Service) contain actual logic
  - Dependency injection of repositories
  - Business validations before DB operations

### **Layer 4: Repository / Data Access Layer**
- **What it is**: Spring Data JPA repositories
- **Location**: `hrms.hrms.repository` package
- **What it does**:
  1. Provides methods to query database
  2. Translates Java calls to SQL queries
  3. Handles CRUD operations (Create, Read, Update, Delete)
  4. Implements custom query methods
- **Example**: `JobAdvertisementDao.findByActiveTrue()` returns only active job ads
- **Simple Terms**: "I'm the librarian. I know exactly where to find information and how to store it."
- **Methods**:
  - `findAll()`: Get all records
  - `findById()`: Get specific record
  - `save()`: Insert or update
  - `delete()`: Remove record
  - Custom methods: `findByActiveTrue()`, `findByEmployer_Id()`

### **Layer 5: Database Layer**
- **What it is**: MySQL database
- **What it does**:
  1. Stores all application data in tables
  2. Enforces data integrity with constraints
  3. Executes SQL queries
  4. Manages relationships between tables
- **Example**: Tables like `job_advertisements`, `employers`, `job_seekers`
- **Simple Terms**: "I'm the warehouse. I physically store everything and make sure data is organized."

### **Additional Layers**:

**Entity Layer** (Models)
- Java classes representing database tables
- Located in `hrms.hrms.entity` package
- Annotated with `@Entity`, `@Table`
- Define table structure and relationships

**DTO Layer** (Data Transfer Objects)
- Lightweight objects for API responses
- Located in `hrms.hrms.dto` package
- Protect entity data, reduce payload size
- Used to map entities to API responses

**Config Layer**
- Spring configuration classes
- Located in `hrms.hrms.config` package
- Example: `SecurityBeans` for password encoding

---

## 4. Request-Response Flow

### **Step-by-Step Flow Example: POST /api/jobPost/add**

```
STEP 1: CLIENT SENDS REQUEST
┌─────────────────────────────────────────────────────────┐
│ POST /api/jobPost/add                                   │
│ Content-Type: application/json                          │
│                                                         │
│ Body:                                                   │
│ {                                                       │
│   "jobPositionId": 1,                                   │
│   "cityId": 2,                                          │
│   "employerId": 3,                                      │
│   "description": "Looking for experienced developer",   │
│   "openPositionCount": 5,                               │
│   "minSalary": 50000,                                   │
│   "maxSalary": 70000,                                   │
│   "applicationDeadline": "2026-04-30"                   │
│ }                                                       │
└─────────────────────────────────────────────────────────┘
         │
         ▼
STEP 2: CONTROLLER RECEIVES & VALIDATES
┌─────────────────────────────────────────────────────────┐
│ JobAdvertisementController.add(request)                 │
│                                                         │
│ ✓ @Valid annotation checks:                            │
│   - openPositionCount > 0 (positive)                    │
│   - description length 10-5000                          │
│   - applicationDeadline in future                       │
│   - All required fields not null                        │
│                                                         │
│ Status: 400 Bad Request if validation fails             │
│ Status: 200 OK if validation passes                     │
│                                                         │
│ Action: Call jobAdvertisementService.add(request)       │
└─────────────────────────────────────────────────────────┘
         │
         ▼
STEP 3: SERVICE APPLIES BUSINESS LOGIC
┌─────────────────────────────────────────────────────────┐
│ JobAdvertisementManager.add(request)                    │
│                                                         │
│ Validation Rules:                                       │
│ 1. Check min salary < max salary                        │
│    → If NOT: return ErrorResult("Min > Max")            │
│                                                         │
│ 2. Fetch JobPosition from DB                            │
│    → If NOT found: throw IllegalArgumentException       │
│                                                         │
│ 3. Fetch City from DB                                   │
│    → If NOT found: throw IllegalArgumentException       │
│                                                         │
│ 4. Fetch Employer from DB                               │
│    → If NOT found: throw IllegalArgumentException       │
│                                                         │
│ 5. Create JobAdvertisement entity:                      │
│    - Set position, city, employer                       │
│    - Set description, salary info                       │
│    - Set releaseDate = today                            │
│    - Set active = true                                  │
│                                                         │
│ 6. Call jobAdvertisementDao.save(ad)                    │
└─────────────────────────────────────────────────────────┘
         │
         ▼
STEP 4: REPOSITORY EXECUTES DB OPERATION
┌─────────────────────────────────────────────────────────┐
│ JobAdvertisementDao.save(jobAdvertisement)              │
│                                                         │
│ JPA/Hibernate translates to SQL:                        │
│                                                         │
│ INSERT INTO job_advertisement                           │
│ (open_position_count, description, min_salary, ...)     │
│ VALUES (5, 'Looking for..', 50000, ...)                │
│                                                         │
│ Hibernate handles:                                      │
│ - Foreign key inserts (job_position_id, city_id, etc)   │
│ - Type conversions                                      │
│ - Transaction management                               │
└─────────────────────────────────────────────────────────┘
         │
         ▼
STEP 5: DATABASE EXECUTES & STORES
┌─────────────────────────────────────────────────────────┐
│ MySQL Database                                          │
│                                                         │
│ 1. Check primary key uniqueness                         │
│ 2. Validate foreign key constraints                     │
│ 3. Insert row into job_advertisement table              │
│ 4. Generate auto-increment ID                           │
│ 5. Return success                                       │
│                                                         │
│ Table: job_advertisement                                │
│ ┌────┬─────┬──────────┬─────────┬─────────┐             │
│ │ id │ ... │ position │  city   │employer │             │
│ ├────┼─────┼──────────┼─────────┼─────────┤             │
│ │ 5  │ ... │    1     │    2    │   3     │ ← NEW       │
│ └────┴─────┴──────────┴─────────┴─────────┘             │
└─────────────────────────────────────────────────────────┘
         │
         ▼
STEP 6: RESPONSE FLOWS BACK UP
┌─────────────────────────────────────────────────────────┐
│ Repository returns success                              │
         ▼
│ Service returns SuccessResult object                    │
         ▼
│ Controller converts to JSON                             │
         ▼
│ HTTP Response sent to client                            │
└─────────────────────────────────────────────────────────┘
         │
         ▼
STEP 7: CLIENT RECEIVES RESPONSE
┌─────────────────────────────────────────────────────────┐
│ HTTP Status: 200 OK                                     │
│ Response Body:                                          │
│ {                                                       │
│   "success": true,                                      │
│   "message": "Job advertisement created."               │
│ }                                                       │
└─────────────────────────────────────────────────────────┘
```

---

## 5. Entity Relationship Analysis

### **All Entities in the Project**

#### **1. City Entity**
```
┌─────────────────────┐
│       City          │
├─────────────────────┤
│ id (PK) [Integer]   │  ← Primary Key (Auto-increment)
│ cityName (String)   │  ← City name (unique, required)
└─────────────────────┘
        │
        │ 1 : Many
        ▼
  (JobAdvertisement)
```
- **Purpose**: Stores city information (e.g., "New York", "London")
- **Relationship**: One city can have MANY job advertisements
- **Why**: Jobs are posted in specific locations

#### **2. JobPosition Entity**
```
┌──────────────────────┐
│    JobPosition       │
├──────────────────────┤
│ id (PK) [Integer]    │  ← Primary Key (Auto-increment)
│ title (String)       │  ← Job title (e.g., "Developer")
└──────────────────────┘
        │
        │ 1 : Many
        ▼
  (JobAdvertisement)
```
- **Purpose**: Stores job titles/positions
- **Relationship**: One position can have MANY job advertisements
- **Why**: Multiple companies post the same position

#### **3. Employer Entity**
```
┌─────────────────────────────┐
│        Employer             │
├─────────────────────────────┤
│ id (PK) [Integer]           │  ← Primary Key
│ companyName (String)        │  ← Company name
│ companyWebPage (String)     │  ← Website URL
│ email (String, UNIQUE)      │  ← Email (unique)
│ phoneNumber (String)        │  ← Contact number
│ password (String, Encrypted)│  ← Hashed password
└─────────────────────────────┘
        │
        │ 1 : Many
        ▼
  (JobAdvertisement)
```
- **Purpose**: Stores employer/company information
- **Relationship**: One employer can post MANY job advertisements
- **Why**: Companies can have multiple job openings

#### **4. JobAdvertisement Entity** (CENTRAL ENTITY)
```
┌────────────────────────────────────┐
│      JobAdvertisement              │
├────────────────────────────────────┤
│ id (PK) [Integer]                  │
│ openPositionCount (Integer)        │
│ description (String)               │
│ minSalary (Integer)                │
│ maxSalary (Integer)                │
│ releaseDate (LocalDate)            │
│ applicationDeadline (LocalDate)    │
│ active (Boolean)                   │
├────────────────────────────────────┤
│ jobPosition_id (FK) ─────┐         │
│ city_id (FK) ────────┐   │         │
│ employer_id (FK) ──┐ │   │         │
└────────────────────┼─┼───┼─────────┘
                     │ │   │
        ┌────────────┘ │   │
        │              │   └─────────────┐
        │              │                 │
        ▼              ▼                 ▼
    JobPosition      City           Employer
    (ManyToOne)  (ManyToOne)      (ManyToOne)
```
- **Purpose**: Stores job posting details
- **Relationships**:
  - Many-to-One with JobPosition
  - Many-to-One with City
  - Many-to-One with Employer
  - One-to-Many with JobApplication
- **Why**: Core entity connecting employers, positions, cities, and applications

#### **5. JobSeeker Entity**
```
┌──────────────────────────────┐
│       JobSeeker              │
├──────────────────────────────┤
│ id (PK) [Integer]            │  ← Primary Key
│ name (String)                │  ← First name
│ lastName (String)            │  ← Last name
│ nationalId (String, UNIQUE)  │  ← ID number
│ birthDate (LocalDate)        │  ← Birth date
│ email (String, UNIQUE)       │  ← Email
│ password (String, Encrypted) │  ← Hashed password
└──────────────────────────────┘
        │
        │ 1 : Many
        ▼
  (JobApplication)
```
- **Purpose**: Stores job seeker profiles
- **Relationship**: One seeker can apply to MANY jobs
- **Why**: People can apply to multiple positions

#### **6. JobApplication Entity** (JUNCTION ENTITY)
```
┌─────────────────────────────────┐
│      JobApplication             │
├─────────────────────────────────┤
│ id (PK) [Integer]               │
│ applicationDate (LocalDateTime) │
│ status (JobApplicationStatus)   │
│   ├─ PENDING                    │
│   ├─ ACCEPTED                   │
│   └─ REJECTED                   │
├─────────────────────────────────┤
│ jobAdvertisement_id (FK) ───┐   │
│ jobSeeker_id (FK) ────────┐ │   │
└────────────────────────────┼─┼───┘
                             │ │
                ┌────────────┘ └──────────────┐
                │                            │
                ▼                            ▼
         JobAdvertisement              JobSeeker
            (ManyToOne)                (ManyToOne)
```
- **Purpose**: Connects job seekers with job advertisements (applications)
- **Relationships**:
  - Many-to-One with JobAdvertisement
  - Many-to-One with JobSeeker
- **Why**: Tracks who applied to which job and status
- **Special**: Unique constraint on (jobAdvertisement_id, jobSeeker_id) - prevents duplicate applications

#### **7. SystemEmployee Entity**
```
┌──────────────────────┐
│   SystemEmployee     │
├──────────────────────┤
│ id (PK) [Integer]    │
│ name (String)        │
│ lastName (String)    │
└──────────────────────┘
```
- **Purpose**: Admin/HR staff accounts
- **Relationships**: None currently
- **Why**: System administrators managing the platform

### **Complete ER Diagram (Text Format)**

```
┌─────────────┐                  ┌──────────────┐
│   City      │                  │ JobPosition  │
├─────────────┤                  ├──────────────┤
│ id (PK)     │                  │ id (PK)      │
│ cityName    │                  │ title        │
└──────┬──────┘                  └──────┬───────┘
       │                                │
       │ 1                              │ 1
       │ |                              │ |
       │ |------ Many ────┐             │ |------ Many ────┐
       │                  │             │                  │
       │                  ▼             │                  ▼
       │         ┌──────────────────────────────────────┐
       │         │    JobAdvertisement                  │
       │         ├──────────────────────────────────────┤
       │         │ id (PK)                              │
       │         │ openPositionCount                    │
       │         │ description                          │
       │         │ minSalary, maxSalary                 │
       │         │ releaseDate                          │
       │         │ applicationDeadline                  │
       │         │ active                               │
       │         │ FK: city_id                          │
       │         │ FK: jobPosition_id                   │
       │         │ FK: employer_id                      │
       └─────────┤                                      │
                 └──────────────────────────────────────┘
                            │ 1
                            │ |
                            │ |------ Many ────┐
                            │                  │
        ┌──────────────┐    │                  ▼
        │   Employer   │    │         ┌─────────────────────────┐
        ├──────────────┤    │         │   JobApplication        │
        │ id (PK)      │    │         ├─────────────────────────┤
        │ companyName  │    │         │ id (PK)                 │
        │ email        │    │         │ applicationDate         │
        │ password     │    │         │ status (PENDING/        │
        └──────┬───────┘    │         │         ACCEPTED/       │
               │ 1          │         │         REJECTED)       │
               │ |          │         │ FK: jobAdvertisement_id │
               │ |---Many ──┘         │ FK: jobSeeker_id        │
               │                      └─────────────────────────┘
               │                                    │ 1
               │                                    │ |
               └────────────────────────────────────┘

        ┌──────────────┐
        │  JobSeeker   │
        ├──────────────┤
        │ id (PK)      │
        │ name         │
        │ lastName     │
        │ nationalId   │
        │ birthDate    │
        │ email        │
        │ password     │
        └──────┬───────┘
               │ 1
               │ |
               │ |------ Many ──────────────────┐
               │                                │
               │             ┌─────────────────┴──────┐
               │             │ JobApplication         │
               │             │ (Referenced above)     │
               └─────────────▶│ (Links Seeker & Ad)    │
                              └────────────────────────┘

        ┌──────────────────┐
        │  SystemEmployee  │
        ├──────────────────┤
        │ id (PK)          │
        │ name             │
        │ lastName         │
        └──────────────────┘
        (No relationships)
```

### **Relationship Explanations in Simple Terms**

**1. City 1:Many JobAdvertisement**
- One city can have many job ads
- Example: "New York" city has 100 different job ads
- Foreign Key: `jobAdvertisement.city_id` → `city.id`

**2. JobPosition 1:Many JobAdvertisement**
- One position type can have many ads
- Example: Position "Developer" has 50 ads from different companies
- Foreign Key: `jobAdvertisement.jobPosition_id` → `jobPosition.id`

**3. Employer 1:Many JobAdvertisement**
- One company can post many job ads
- Example: "Google" company has 20 open positions
- Foreign Key: `jobAdvertisement.employer_id` → `employer.id`

**4. JobAdvertisement 1:Many JobApplication**
- One job ad can receive many applications
- Example: "Senior Developer" ad gets 50 applications
- Foreign Key: `jobApplication.jobAdvertisement_id` → `jobAdvertisement.id`

**5. JobSeeker 1:Many JobApplication**
- One person can apply to many jobs
- Example: "John Doe" applies to 10 different positions
- Foreign Key: `jobApplication.jobSeeker_id` → `jobSeeker.id`

---

## 6. Business Logic Explanation

### **Problem This System Solves**

**Without HRMS:**
- Companies manually manage job postings (spreadsheets, emails)
- Job seekers send emails to each company individually
- No centralized tracking of applications
- No organized way to manage hiring process
- Time-consuming, error-prone, chaotic

**With HRMS:**
- Centralized platform for job posting and applications
- Employers can post, track, and manage applications
- Job seekers can search and apply to multiple jobs
- Automated status tracking
- Organized, efficient, scalable

### **Core Business Processes**

#### **Process 1: Employer Registration**
```
Flow:
1. Employer accesses POST /api/employers/register
2. Provides: company name, email, website, phone, password
3. System validates:
   ✓ Passwords match
   ✓ Email not already used
4. If valid: Password encrypted with BCrypt, employer saved
5. If invalid: Error message returned
6. Response: Success or error message

Example:
INPUT:
{
  "companyName": "Tech Corp",
  "email": "hr@techcorp.com",
  "website": "www.techcorp.com",
  "phoneNumber": "123456789",
  "password": "secure123",
  "confirmPassword": "secure123"
}

OUTPUT (SUCCESS):
{
  "success": true,
  "message": "Employer registered."
}
```

#### **Process 2: Job Seeker Registration**
```
Flow:
1. Job seeker accesses POST /api/candidateController/register
2. Provides: name, email, national ID, birth date, password
3. System validates:
   ✓ All required fields present
   ✓ National ID unique
   ✓ Email unique
   ✓ Birth date is in past
4. If valid: Password encrypted, seeker saved
5. Response: Success or error

Business Rule: Age verification ensures only real adults can register
```

#### **Process 3: Post Job Advertisement**
```
Flow:
1. Employer accesses POST /api/jobPost/add
2. Provides: position, city, salary range, deadline, etc.
3. System validates:
   ✓ Min salary ≤ Max salary
   ✓ Deadline is in future
   ✓ Position exists in system
   ✓ City exists in system
   ✓ Employer exists and verified
4. If valid:
   a. Create JobAdvertisement record
   b. Set active = true
   c. Set releaseDate = today
   d. Save to database
5. Response: Success or error

Business Rules:
- Only verified employers can post
- Salary must make sense (min ≤ max)
- Deadline must be future date (no backdating)
- Default status: ACTIVE

Example:
INPUT:
{
  "jobPositionId": 5,
  "cityId": 2,
  "employerId": 3,
  "description": "Senior Java Developer with 5+ years experience",
  "openPositionCount": 3,
  "minSalary": 80000,
  "maxSalary": 120000,
  "applicationDeadline": "2026-05-31"
}

PROCESSING:
1. Check: 80000 ≤ 120000 ✓
2. Check: 2026-05-31 > today ✓
3. Fetch JobPosition ID 5 ✓
4. Fetch City ID 2 ✓
5. Fetch Employer ID 3 ✓
6. Create JobAdvertisement
7. Save to database ✓

OUTPUT:
{
  "success": true,
  "message": "Job advertisement created."
}
```

#### **Process 4: Apply for Job**
```
Flow:
1. Job seeker accesses POST /api/applications/apply
2. Provides: jobAdvertisementId, jobSeekerId
3. System validates:
   ✓ Job ad exists and is ACTIVE
   ✓ Job seeker exists
   ✓ Seeker hasn't already applied to this ad
4. If valid:
   a. Create JobApplication record
   b. Set status = PENDING
   c. Set applicationDate = now
   d. Save to database
5. Response: Success or error

Business Rules:
- Can only apply to ACTIVE ads (not closed ones)
- Cannot apply twice to same ad (duplicate prevention)
- Default status: PENDING (waiting for employer decision)

Example:
INPUT:
{
  "jobAdvertisementId": 10,
  "jobSeekerId": 5
}

VALIDATION:
1. Find Job Ad ID 10 → Found ✓
2. Check active = true ✓
3. Find Job Seeker ID 5 → Found ✓
4. Check if already applied → Not found ✓
5. All validations passed

CREATE APPLICATION:
- jobAdvertisement_id = 10
- jobSeeker_id = 5
- status = PENDING
- applicationDate = 2026-03-30 10:30:45

OUTPUT:
{
  "success": true,
  "message": "Application submitted."
}
```

#### **Process 5: Update Application Status**
```
Flow:
1. Employer accesses POST /api/applications/update-status
2. Provides: applicationId, newStatus (ACCEPTED/REJECTED)
3. System validates:
   ✓ Application exists
   ✓ Status is valid (PENDING, ACCEPTED, REJECTED)
4. If valid:
   a. Fetch JobApplication record
   b. Update status field
   c. Save to database
5. Response: Success or error

Business Rules:
- Only employers can update (not implemented yet - future feature)
- Status can be: PENDING → ACCEPTED, PENDING → REJECTED
- Once accepted/rejected, can be changed again

Example:
INPUT:
{
  "applicationId": 15,
  "status": "ACCEPTED"
}

PROCESSING:
1. Find Application ID 15 ✓
2. Update status from PENDING to ACCEPTED
3. Save changes ✓
4. Send notification to seeker (future feature)

OUTPUT:
{
  "success": true,
  "message": "Application status updated."
}
```

#### **Process 6: View Job Listings**
```
Flow:
1. Seeker accesses GET /api/jobPost/active
2. System:
   a. Query all JobAdvertisements where active = true
   b. Convert to DTOs (safe response format)
   c. Return list
3. Response: List of active ads

Why DTOs?: 
- Don't expose internal structure
- Faster transmission (smaller payload)
- Security: sensitive fields removed

Data Returned:
- Job ID
- Job Title
- Company Name
- City
- Salary Range
- Application Deadline
- Number of Positions
- Release Date
```

---

## 7. System Workflow

### **Complete System Operation Cycle**

```
┌─────────────────────────────────────────────────────────────┐
│              COMPLETE HRMS WORKFLOW                         │
└─────────────────────────────────────────────────────────────┘

╔═════════════════════════════════════════════════════════════╗
║ PHASE 1: SETUP (Initial Registration)                      ║
╚═════════════════════════════════════════════════════════════╝

1. EMPLOYER REGISTRATION
   ┌─────────────────────────────┐
   │ Company enters details:      │
   │ - Name: "TechCorp"          │
   │ - Email: hr@techcorp.com    │
   │ - Website: techcorp.com     │
   │ - Password: (encrypted)     │
   └──────────┬──────────────────┘
              │
              ▼ POST /api/employers/register
   ┌─────────────────────────────┐
   │ System validates            │
   │ - Email unique?             │
   │ - Password encrypted?       │
   │ - Required fields filled?   │
   └──────────┬──────────────────┘
              │
              ▼
   ┌─────────────────────────────┐
   │ Employer saved to database  │
   │ Status: REGISTERED ✓        │
   └─────────────────────────────┘

2. JOB SEEKER REGISTRATION
   ┌─────────────────────────────┐
   │ Seeker enters details:      │
   │ - Name: "John Doe"          │
   │ - Email: john@email.com     │
   │ - National ID: 123456789    │
   │ - Birth Date: 1990-01-01    │
   │ - Password: (encrypted)     │
   └──────────┬──────────────────┘
              │
              ▼ POST /api/candidateController/register
   ┌─────────────────────────────┐
   │ System validates            │
   │ - Email unique?             │
   │ - National ID unique?       │
   │ - Age > 18?                 │
   │ - All fields filled?        │
   └──────────┬──────────────────┘
              │
              ▼
   ┌─────────────────────────────┐
   │ Seeker saved to database    │
   │ Status: READY TO SEARCH ✓   │
   └─────────────────────────────┘

╔═════════════════════════════════════════════════════════════╗
║ PHASE 2: JOB POSTING (Employer Creates Opportunity)       ║
╚═════════════════════════════════════════════════════════════╝

3. CREATE JOB ADVERTISEMENT
   ┌──────────────────────────────────┐
   │ Employer post job:               │
   │ - Position: Senior Developer     │
   │ - City: New York                 │
   │ - Salary: $80k - $120k           │
   │ - Deadline: 2026-05-31           │
   │ - Positions: 3 openings          │
   └────────┬─────────────────────────┘
            │
            ▼ POST /api/jobPost/add
   ┌──────────────────────────────────┐
   │ System validates:                │
   │ ✓ Min salary ≤ Max salary        │
   │ ✓ Deadline is in future          │
   │ ✓ Position/City exists           │
   │ ✓ Employer verified              │
   └────────┬─────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────┐
   │ JobAdvertisement created:        │
   │ - Status: ACTIVE                 │
   │ - Release Date: Today            │
   │ - Visible to all seekers         │
   └──────────────────────────────────┘

╔═════════════════════════════════════════════════════════════╗
║ PHASE 3: JOB SEARCH & APPLICATION                          ║
╚═════════════════════════════════════════════════════════════╝

4. JOB SEEKER SEARCHES JOBS
   ┌──────────────────────────────────┐
   │ Seeker searches:                 │
   │ - View active listings           │
   │ - Filter by city, position, etc. │
   │ - Find matching job ad           │
   └────────┬─────────────────────────┘
            │
            ▼ GET /api/jobPost/active
   ┌──────────────────────────────────┐
   │ System returns:                  │
   │ ✓ All active job advertisements  │
   │ ✓ Filtered by criteria           │
   │ ✓ Sorted by deadline             │
   └──────────────────────────────────┘

5. JOB SEEKER APPLIES
   ┌──────────────────────────────────┐
   │ Seeker clicks "Apply"            │
   │ - Submits application            │
   │ - JobAdvertisementId: 10         │
   │ - JobSeekerId: 5                 │
   └────────┬─────────────────────────┘
            │
            ▼ POST /api/applications/apply
   ┌──────────────────────────────────┐
   │ System validates:                │
   │ ✓ Ad exists and active?          │
   │ ✓ Seeker exists?                 │
   │ ✓ Not already applied?           │
   └────────┬─────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────┐
   │ JobApplication created:          │
   │ - Status: PENDING                │
   │ - Application Date: Now          │
   │ - Awaiting employer review       │
   └──────────────────────────────────┘

╔═════════════════════════════════════════════════════════════╗
║ PHASE 4: APPLICATION REVIEW & DECISION                     ║
╚═════════════════════════════════════════════════════════════╝

6. EMPLOYER REVIEWS APPLICATIONS
   ┌──────────────────────────────────┐
   │ Employer views:                  │
   │ GET /api/applications/by-ad/{id} │
   │ - See all applicants for job     │
   │ - Review seeker profiles         │
   │ - Compare candidates             │
   └────────┬─────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────┐
   │ System returns:                  │
   │ - List of all applications       │
   │ - Applicant info                 │
   │ - Current status (PENDING)       │
   └──────────────────────────────────┘

7. EMPLOYER MAKES DECISION
   ┌──────────────────────────────────┐
   │ Employer decides:                │
   │ - Accept candidate A             │
   │ - Reject candidate B             │
   │ - Keep candidate C pending       │
   └────────┬─────────────────────────┘
            │
            ▼ POST /api/applications/update-status
   ┌──────────────────────────────────┐
   │ System updates:                  │
   │ - Application ID: 15             │
   │ - New Status: ACCEPTED           │
   │ - Timestamp: Updated             │
   └────────┬─────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────┐
   │ Application status changed:      │
   │ ✓ PENDING → ACCEPTED             │
   │ ✓ Seeker notified (future)       │
   │ ✓ Recorded in system             │
   └──────────────────────────────────┘

8. SEEKER CHECKS STATUS
   ┌──────────────────────────────────┐
   │ Seeker views:                    │
   │ GET /api/applications/           │
   │     by-jobseeker/{seekerId}      │
   │ - See all my applications        │
   │ - Check status of each           │
   └────────┬─────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────┐
   │ System returns:                  │
   │ - Application 1: ACCEPTED ✓      │
   │ - Application 2: REJECTED ✗      │
   │ - Application 3: PENDING ⏳      │
   └──────────────────────────────────┘

╔═════════════════════════════════════════════════════════════╗
║ PHASE 5: END OF PROCESS                                    │
╚═════════════════════════════════════════════════════════════╝

OUTCOMES:
├─ Employer fills position with accepted candidates
├─ Selected seekers get job offer
├─ Rejected seekers can apply to other jobs
└─ System maintains complete history for reporting
```

### **Data Flow Visualization**

```
EMPLOYER PERSPECTIVE:
┌─────────────┐
│  Employer   │
└──────┬──────┘
       │
       ├─→ Register Account
       │
       ├─→ Create Job Ad
       │    └─→ JobAdvertisement (ACTIVE)
       │        └─→ Receives Applications
       │
       ├─→ View Applications
       │    └─→ See JobApplication list
       │        ├─ Status: PENDING
       │        ├─ Applicant info
       │        └─ Timestamps
       │
       └─→ Review & Decide
            └─→ Update Application Status
                ├─ ACCEPTED
                └─ REJECTED


JOB SEEKER PERSPECTIVE:
┌──────────────┐
│  Job Seeker  │
└──────┬───────┘
       │
       ├─→ Register Account
       │
       ├─→ Search Jobs
       │    └─→ View Active Ads
       │        ├─ By City
       │        ├─ By Position
       │        └─ By Salary
       │
       ├─→ Apply to Job
       │    └─→ Create Application
       │        └─ Status: PENDING
       │
       └─→ Track Status
            └─→ View My Applications
                ├─ Status: ACCEPTED ✓
                ├─ Status: REJECTED ✗
                └─ Status: PENDING ⏳
```

### **Database State Changes**

```
TIMELINE OF DATABASE CHANGES:

T1 - After Employer Registration:
employers table:
│ id │ companyName │ email           │ password_hash │
├────┼─────────────┼─────────────────┼───────────────┤
│ 1  │ TechCorp    │ hr@techcorp.com │ $2a$... 256.. │

T2 - After Job Seeker Registration:
job_seekers table:
│ id │ name     │ email          │ nationalId │
├────┼──────────┼────────────────┼────────────┤
│ 1  │ John Doe │ john@email.com │ 123456789  │

T3 - After Job Advertisement Posted:
job_advertisements table:
│ id │ description      │ minSalary │ maxSalary │ active │ employer_id │
├────┼──────────────────┼───────────┼───────────┼────────┼─────────────┤
│ 1  │ Senior Developer │ 80000     │ 120000    │ true   │ 1           │

T4 - After Application Submitted:
job_applications table:
│ id │ status  │ application_date     │ job_advertisement_id │ job_seeker_id │
├────┼─────────┼──────────────────────┼──────────────────────┼───────────────┤
│ 1  │ PENDING │ 2026-03-30 10:30:45  │ 1                    │ 1             │

T5 - After Status Update (ACCEPTED):
job_applications table:
│ id │ status   │ application_date     │ job_advertisement_id │ job_seeker_id │
├────┼──────────┼──────────────────────┼──────────────────────┼───────────────┤
│ 1  │ ACCEPTED │ 2026-03-30 10:30:45  │ 1                    │ 1             │
```

---

## Summary

This HRMS project demonstrates a complete **layered architecture** where:

1. **Client** sends requests with data
2. **Controller** validates and routes requests
3. **Service** applies business logic and rules
4. **Repository** executes database operations
5. **Database** persists data in organized tables
6. **Response** flows back through all layers

The system solves the **job posting and application management problem** by:
- Centralizing job opportunities
- Simplifying the application process
- Tracking applications efficiently
- Organizing hiring data

All operations follow a **MVC pattern** with **separation of concerns**, making the code maintainable, testable, and scalable.
