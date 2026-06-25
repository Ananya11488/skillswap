# SkillSwap

## Corporate Knowledge Exchange Platform

SkillSwap is a full-stack enterprise web application that enables employees to exchange skills through peer-to-peer mentorship. The platform allows users to discover skills, request mentorship sessions, schedule meetings, provide feedback, and track professional development using role-based dashboards.

---

## Features

- Role-based authentication using Spring Security
- Skill catalog with category and proficiency filters
- Mentorship request workflow
- Session scheduling and management
- Feedback and rating system
- Notifications for mentorship activities
- Skill taxonomy management
- Analytics dashboard for learning progress

---

## User Roles

- Mentee
- Mentor
- Head of Department (HOD)
- Administrator

---

## Tech Stack

### Backend

- Java 17
- Spring Boot
- Spring Security
- Hibernate / JPA
- MySQL

### Frontend

- Thymeleaf
- HTML
- CSS

### Build Tool

- Maven

---

## Architecture

The application follows the MVC architecture.

```
Presentation Layer
        ↓
Controllers
        ↓
Services
        ↓
Repositories
        ↓
MySQL Database
```

Major modules include:

- Authentication
- Skill Management
- Mentorship Requests
- Session Scheduling
- Feedback
- Notifications
- Admin Dashboard

---

## Project Structure

```
skillswap/
│
├── src/
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── entity/
│   └── resources/
│
├── pom.xml
└── README.md
```

---

## How to Run

Clone the repository

```bash
git clone https://github.com/Ananya11488/skillswap.git
```

Navigate to the project

```bash
cd skillswap
```

Configure MySQL in

```
application.properties
```

Build the project

```bash
mvn clean install
```

Run

```bash
mvn spring-boot:run
```

Application URL

```
http://localhost:8080
```

---

## Screenshots

Add screenshots here.

- Home Page
- Dashboard
- Skill Catalog
- Session Management

---

## Future Improvements

- Video conferencing integration
- Email notifications
- Skill recommendation engine
- Mobile application
- Advanced analytics

  ---
  ## Additional Documentation

For detailed project documentation, refer to:

- [Project Overview](docs/PROJECT_OVERVIEW.md)
- [Development Guide](docs/DEVELOPMENT_GUIDE.md)
- [Setup Guide](docs/SETUP_GUIDE.md)
- [Use Cases](docs/USE_CASES.md)
- [Development Checklist](docs/CHECKLIST.md)
