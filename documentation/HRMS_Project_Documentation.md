# HRMS (Human Resource Management System) Project Documentation

## 1. Project Overview
This is a simple HRMS (Human Resource Management System) built with Java and Spring Boot. It helps manage job seekers, employers, job advertisements, and job applications. Think of it as a basic job portal where employers can post jobs, job seekers can apply, and admins can manage everything. It's like a mini LinkedIn for jobs, but just the backend API.

## 2. Technologies Used
- **Java 17**: The programming language.
- **Spring Boot 3.2.2**: Framework for building web apps quickly.
- **Spring Data JPA**: For database operations (saving, finding data).
- **MySQL**: Database to store data.
- **Maven**: Tool to manage project dependencies and build.
- **Lombok**: Library to reduce boilerplate code (like getters/setters).
- **Spring Validation**: For checking input data.
- **Spring Security Crypto**: For password hashing (but no full login yet).

## 3. Project Architecture (Layered Architecture)
The project uses a layered architecture, like building a house with floors. Each layer has its own job:

- **Controller Layer**: Handles web requests (like API calls). It's the front door.
- **Service Layer**: Contains business logic (rules, like "salary must be positive").
- **Repository Layer**: Talks to the database (saves/finds data).
- **Entity Layer**: Represents database tables as Java classes.
- **DTO Layer**: Simple objects to send data to/from APIs.
- **Config Layer**: Settings, like password encoding.

This keeps code organized and easy to change.

## 4. Flow of Request (Step-by-step from Controller → Service → Repository → Database)
Imagine a user wants to add a job ad:

1. **Controller**: Receives the request (e.g., POST /api/jobPost/add). Validates input.
2. **Service**: Checks business rules (e.g., min salary < max salary). Prepares data.
3. **Repository**: Calls database methods (e.g., save the job ad).
4. **Database**: Stores the data in MySQL.

Response goes back: Database → Repository → Service → Controller → User.

## 5. Database Design (Entities and Relationships)
The database has tables for:
- **City**: Stores city names (e.g., "New York").
- **Employer**: Company details (name, email, etc.).
- **JobPosition**: Job titles (e.g., "Developer").
- **JobAdvertisement**: Job posts (linked to position, city, employer).
- **JobSeeker**: People looking for jobs (name, email).
- **JobApplication**: Applications (links seeker to ad).
- **SystemEmployee**: Admins.

Relationships:
- One city can have many job ads.
- One employer can post many ads.
- One position can have many ads.
- One ad can have many applications.
- One seeker can apply to many ads.

## 6. ER Diagram
![ER Diagram](./ER%20Diagram.png)

## 7. API Endpoints Summary

| Module          | HTTP Method | Endpoint                                      | Description |
|-----------------|-------------|-----------------------------------------------|-------------|
| Cities          | POST        | /api/cities/add                               | Add a new city |
| Cities          | GET         | /api/cities/getAll                            | Get all cities |
| Employers       | POST        | /api/employers/register                       | Register a new employer |
| Employers       | GET         | /api/employers/getAll                         | Get all employers |
| Job Positions   | POST        | /api/jobPosition/add                          | Add a new job position |
| Job Positions   | GET         | /api/jobPosition/getAll                       | Get all job positions |
| Job Ads         | POST        | /api/jobPost/add                              | Add a new job advertisement |
| Job Ads         | GET         | /api/jobPost/getAll                           | Get all job advertisements |
| Job Ads         | GET         | /api/jobPost/active                           | Get active job advertisements |
| Job Ads         | GET         | /api/jobPost/active/by-employer?employerId=X  | Get active ads by employer |
| Job Ads         | GET         | /api/jobPost/sorted-by-deadline               | Get ads sorted by deadline |
| Job Ads         | GET         | /api/jobPost/by-deadline?date=YYYY-MM-DD      | Get ads by specific deadline |
| Job Applications| POST        | /api/applications/apply                       | Apply for a job |
| Job Applications| POST        | /api/applications/update-status               | Update application status |
| Job Applications| GET         | /api/applications/by-advertisement/{id}       | Get applications by ad ID |
| Job Applications| GET         | /api/applications/by-jobseeker/{id}           | Get applications by seeker ID |
| Job Seekers     | POST        | /api/candidateController/register             | Register a new job seeker |
| Job Seekers     | GET         | /api/candidateController/getAll               | Get all job seekers |
| System Employees| POST        | /api/hrController/add                         | Add a new system employee |
| System Employees| GET         | /api/hrController/getAll/systemEmployee       | Get all system employees |

## 8. Security Flow (JWT/Auth if used)
No JWT or full authentication yet. Only password hashing with BCrypt. No login/logout. To add security, use Spring Security with JWT for tokens.

## 9. Design Patterns Used
- **Dependency Injection**: Spring injects services into controllers.
- **Repository Pattern**: DAOs handle data access.
- **Service Layer Pattern**: Business logic in services.
- **DTO Pattern**: Separates data transfer from entities.
- **Singleton**: Spring beans are singletons.

## 10. How to Run the Project
1. Install Java 17, Maven, MySQL.
2. Create MySQL database: `hrms_apis`.
3. Update `application.properties` with your DB password.
4. Run: `mvn spring-boot:run` or `./mvnw spring-boot:run`.
5. App runs on http://localhost:9091.

## 11. Conclusion (What this project does)
This project is a backend API for an HRMS. It lets employers post jobs, seekers apply, and manages data. It's basic but shows Spring Boot skills. Expand with auth, frontend, or more features.
