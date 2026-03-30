# API Documentation for JobAdvertisementController

This controller handles job advertisements in the HRMS system. It lets employers add jobs and users view them.

## 1. Add Job Advertisement
- **API URL**: `/api/jobPost/add`
- **HTTP Method**: POST
- **Description**: Adds a new job advertisement. Employer provides job details like position, city, salary, etc.
- **Request Body (JSON)**:
  ```json
  {
    "jobPositionId": 1,
    "cityId": 2,
    "employerId": 3,
    "description": "We need a developer...",
    "openPositionCount": 5,
    "minSalary": 50000,
    "maxSalary": 70000,
    "applicationDeadline": "2026-04-30"
  }
  ```
- **Response Body (JSON)**:
  ```json
  {
    "success": true,
    "message": "Job advertisement created."
  }
  ```
- **Status Codes**:
  - 200: Success
  - 400: Bad request (validation error, e.g., min salary > max)
  - 404: Not found (position, city, or employer not found)
  - 500: Server error
- **Step-by-step flow**:
  1. Controller gets request, validates data.
  2. Service checks rules (e.g., salaries), fetches position/city/employer from DB.
  3. Repository saves new job ad to DB.
  4. DB stores it.
  5. Response: Success message back to user.

## 2. Get All Job Advertisements
- **API URL**: `/api/jobPost/getAll`
- **HTTP Method**: GET
- **Description**: Lists all job advertisements.
- **Request Body**: None
- **Response Body (JSON)**:
  ```json
  {
    "success": true,
    "message": "Job advertisements listed.",
    "data": [
      {
        "id": 1,
        "jobTitle": "Developer",
        "companyName": "ABC Corp",
        "city": "New York",
        "openPositionCount": 5,
        "minSalary": 50000,
        "maxSalary": 70000,
        "releaseDate": "2026-03-30",
        "applicationDeadline": "2026-04-30",
        "active": true
      }
    ]
  }
  ```
- **Status Codes**:
  - 200: Success
  - 500: Server error
- **Step-by-step flow**:
  1. Controller calls service.
  2. Service gets all ads from repository.
  3. Repository queries DB for all ads.
  4. DB returns data.
  5. Service maps to DTOs.
  6. Response: List of ads back to user.

## 3. Get Active Job Advertisements
- **API URL**: `/api/jobPost/active`
- **HTTP Method**: GET
- **Description**: Lists only active (open) job ads.
- **Request Body**: None
- **Response Body**: Same as above, but only active ones.
- **Status Codes**: 200, 500
- **Step-by-step flow**: Similar to getAll, but repository filters by active=true.

## 4. Get Active Ads by Employer
- **API URL**: `/api/jobPost/active/by-employer?employerId=1`
- **HTTP Method**: GET
- **Description**: Lists active ads for a specific employer.
- **Request Body**: None (ID in query param)
- **Response Body**: List of ads for that employer.
- **Status Codes**: 200, 404 (employer not found), 500
- **Step-by-step flow**: Repository finds ads by employer ID and active=true.

## 5. Get Ads Sorted by Deadline
- **API URL**: `/api/jobPost/sorted-by-deadline`
- **HTTP Method**: GET
- **Description**: Lists ads sorted by application deadline (earliest first).
- **Request Body**: None
- **Response Body**: Sorted list of ads.
- **Status Codes**: 200, 500
- **Step-by-step flow**: Repository orders by deadline asc.

## 6. Get Ads by Deadline
- **API URL**: `/api/jobPost/by-deadline?date=2026-04-30`
- **HTTP Method**: GET
- **Description**: Lists ads with a specific deadline.
- **Request Body**: None (date in query param)
- **Response Body**: Ads for that date.
- **Status Codes**: 200, 500
- **Step-by-step flow**: Repository finds by deadline date.
