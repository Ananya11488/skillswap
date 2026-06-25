# SkillSwap Development Guide

## Architecture Overview

SkillSwap follows a layered architecture with clear separation of concerns:

```
┌─────────────────────────────────────────┐
│           View Layer                    │
│    (Thymeleaf Templates - HTML)         │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│        Controller Layer                 │
│  (Request Handling & Routing)           │
├─────────────────────────────────────────┤
│ - AuthController                        │
│ - DashboardController                   │
│ - SkillController                       │
│ - RequestController                     │
│ - SessionController                     │
│ - FeedbackController                    │
│ - AdminController                       │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│        Service Layer                    │
│  (Business Logic)                       │
├─────────────────────────────────────────┤
│ - UserService                           │
│ - SkillService                          │
│ - RequestService                        │
│ - SessionService                        │
│ - FeedbackService                       │
│ - NotificationService                   │
│ - SkillTaxonomyService                  │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│     Repository/Data Access Layer        │
│        (JPA Repositories)               │
├─────────────────────────────────────────┤
│ - UserRepository                        │
│ - SkillRepository                       │
│ - RequestRepository                     │
│ - SessionRepository                     │
│ - FeedbackRepository                    │
│ - NotificationRepository                │
│ - SkillTaxonomyRepository               │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│           Database Layer                │
│            (MySQL)                      │
└─────────────────────────────────────────┘
```

## Design Patterns Used

### 1. MVC Pattern
- **Model**: Entity classes (`@Entity` annotated classes)
- **View**: Thymeleaf templates (`.html` files)
- **Controller**: Request handlers managing business flow

### 2. Repository Pattern
- Data access abstraction through `JpaRepository`
- Encapsulates database queries
- Facilitates testing and switching databases

### 3. Service Layer Pattern
- Business logic separation from controllers
- Reusable service methods
- Transaction management with `@Transactional`

### 4. Dependency Injection
- Spring manages object creation and wiring
- `@Autowired` annotation for automatic injection
- Loose coupling between components

### 5. Entity Inheritance (Joined Table)
- User table has role-based inheritance
- Different user types can be managed uniformly
- Extensible for future role additions

## Database Schema

### Entity-Relationship Diagram

```
┌─────────────┐
│   Users     │
│─────────────│
│ user_id (PK)│
│ email       │
│ password    │
│ first_name  │
│ last_name   │
│ role        │
│ department  │
└──────┬──────┘
       │
       ├─── (One-to-Many) ──→ Requests
       │      (mentee_id)
       │      (mentor_id)
       │
       ├─── (One-to-Many) ──→ Sessions
       │      (mentee_id)
       │      (mentor_id)
       │
       ├─── (Many-to-Many) ──→ Skills
       │      (user_skills table)
       │
       ├─── (One-to-Many) ──→ Feedback
       │      (given_by_id)
       │      (given_to_id)
       │
       └─── (One-to-Many) ──→ Notifications
              (user_id)

┌──────────────────┐
│  Requests        │
│──────────────────│
│ request_id (PK)  │
│ mentee_id (FK)   │
│ mentor_id (FK)   │
│ skill_to_learn   │
│ status           │
│ message          │
└────────┬─────────┘
         │
         └─── (One-to-One) ──→ Sessions

┌──────────────────┐      ┌──────────────────┐
│  Skills          │      │ SkillTaxonomy    │
│──────────────────│      │──────────────────│
│ skill_id (PK)    │      │ taxonomy_id (PK) │
│ skill_name       │──┐   │ category_name    │
│ category         │  └──→│ description      │
│ proficiency      │      └──────────────────┘
│ taxonomy_id (FK) │
└──────────────────┘

┌───────────────────┐
│  Sessions         │
│───────────────────│
│ session_id (PK)   │
│ request_id (FK)   │
│ mentee_id (FK)    │
│ mentor_id (FK)    │
│ scheduled_start   │
│ scheduled_end     │
│ status            │
│ notes             │
└─────────┬─────────┘
          │
          └─── (One-to-One) ──→ Feedback

┌──────────────────┐
│  Feedback        │
│──────────────────│
│ feedback_id (PK) │
│ session_id (FK)  │
│ given_by_id (FK) │
│ given_to_id (FK) │
│ rating           │
│ comment          │
└──────────────────┘
```

## Core Entity Relationships

### User
- **Roles**: MENTEE, MENTOR, HOD, ADMIN
- **Relationships**:
  - One-to-Many with Requests (as mentee/mentor)
  - One-to-Many with Sessions (as mentee/mentor)
  - Many-to-Many with Skills
  - One-to-Many with Feedback

### Request (Core Workflow Entity)
- **Status Flow**: PROPOSED → ACCEPTED → COMPLETED
- **Alternative Status**: DECLINED, RESCHEDULED
- **Relationships**:
  - Belongs to Mentee (User)
  - Belongs to Mentor (User)
  - Links to two Skills (to learn, to teach)
  - One-to-One with Session (after acceptance)

### Session
- **Status Flow**: SCHEDULED → IN_PROGRESS → COMPLETED
- **Alternative Status**: CANCELLED, RESCHEDULED
- **Relationships**:
  - One-to-One with Request
  - Belongs to Mentee and Mentor
  - One-to-One with Feedback

### Feedback
- **Purpose**: Post-session evaluation
- **Relationships**:
  - One-to-One with Session
  - Belongs to two Users (giver, receiver)

## Key Business Logic

### Request Workflow

```
1. Mentee browses Skill Catalog
   ↓
2. Mentee searches for available mentors
   ↓
3. Mentee creates Request (PROPOSED)
   ↓
4. Mentor receives Notification
   ↓
5. Mentor reviews Request
   ├─→ ACCEPTED → Session created (SCHEDULED)
   │       ↓
   │    Session scheduled between parties
   │       ↓
   │    Session occurs
   │       ↓
   │    Session marked COMPLETED
   │       ↓
   │    Both parties submit Feedback
   │
   └─→ DECLINED → Request rejected with reason
```

### Session Lifecycle

```
SCHEDULED (After request accepted)
    ↓
IN_PROGRESS (When session starts)
    ↓
COMPLETED (When session ends)
    ├─→ Feedback submission window opens
    │
CANCELLED (Cancelled by either party)
    │
RESCHEDULED (Moved to different time)
```

## Service Layer Responsibilities

### UserService
- User registration with password encryption
- User retrieval by email/ID/role/department
- User activation/deactivation
- User profile management

### SkillService
- Skill creation and management
- Skill filtering by category
- Skill activation/deactivation
- Skill taxonomy association

### RequestService
- Create new requests
- Accept/decline requests
- Retrieve user-specific requests
- Get pending requests for mentors
- Request status updates

### SessionService
- Create sessions from requests
- Retrieve upcoming sessions
- Complete sessions
- Cancel sessions
- Reschedule sessions
- Track session history

### FeedbackService
- Submit feedback after sessions
- Retrieve feedback received/given
- Calculate average ratings
- Track feedback history

### NotificationService
- Create notifications for events
- Retrieve user notifications
- Mark notifications as read
- Send system alerts

### SkillTaxonomyService
- Manage skill categories
- CRUD operations for taxonomy
- Activate/deactivate categories

## Controller Responsibilities

### AuthController
- Handle login page rendering
- Process login requests
- Handle registration page rendering
- Process new user registration

### DashboardController
- Render role-specific dashboards
- Coordinate between services for dashboard data

### SkillController
- Display skill catalog
- Filter skills by category/proficiency
- Show individual skill details

### RequestController
- Display user's requests
- Show pending requests (for mentors)
- Handle accept/decline actions
- Create new requests

### SessionController
- List user's sessions
- Show session details
- Cancel sessions
- Mark sessions as complete
- Reschedule sessions

### FeedbackController
- Display feedback forms
- Submit feedback
- Show received feedback
- Calculate and display ratings

### AdminController
- Manage skill taxonomy
- Manage available skills
- CRUD operations for system admin

## How to Extend the Application

### Adding a New Use Case

Follow this pattern:

1. **Create Entity** (if needed)
   ```java
   @Entity
   @Table(name = "new_use_case")
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class NewUseCase {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;
       // fields
   }
   ```

2. **Create Repository**
   ```java
   @Repository
   public interface NewUseCaseRepository extends JpaRepository<NewUseCase, Long> {
       // custom query methods
   }
   ```

3. **Create Service**
   ```java
   @Service
   public class NewUseCaseService {
       @Autowired
       private NewUseCaseRepository repository;
       
       public NewUseCase createNewUseCase(NewUseCase entity) {
           return repository.save(entity);
       }
       // more methods
   }
   ```

4. **Create Controller**
   ```java
   @Controller
   @RequestMapping("/use-case")
   public class NewUseCaseController {
       @Autowired
       private NewUseCaseService service;
       
       @GetMapping
       public String listItems(Model model) {
           model.addAttribute("items", service.findAll());
           return "use-case/list";
       }
   }
   ```

5. **Create Templates**
   - Create folder `src/main/resources/templates/use-case/`
   - Add `.html` files for each view

### Adding Role-Based Access Control

Update `SecurityConfig.java`:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .antMatchers("/mentor/**").hasRole("MENTOR")
        .antMatchers("/mentee/**").hasRole("MENTEE")
        // ... more patterns
}
```

### Adding Email Notifications

1. Add Spring Mail dependency to pom.xml
2. Create `EmailService`
3. Call from services after important events

### Adding Analytics Dashboard

1. Create `Analytics` entity
2. Create queries in repositories for analytics data
3. Add new controller endpoints for analytics
4. Create analytics dashboard templates

## Testing Strategy

### Unit Testing
Test services in isolation:

```java
@SpringBootTest
class UserServiceTest {
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testRegisterUser() {
        // test logic
    }
}
```

### Integration Testing
Test full request-response cycles

### Manual Testing Checklist
- [ ] Register new users
- [ ] Login with different roles
- [ ] Browse skill catalog
- [ ] Create requests
- [ ] Accept/decline requests
- [ ] Schedule sessions
- [ ] Submit feedback
- [ ] View analytics (HOD)
- [ ] Manage skills (Admin)

## Performance Optimization

### Caching
```java
@Cacheable("skills")
public List<Skill> findAllActiveSkills() {
    return skillRepository.findByIsActiveTrue();
}
```

### Lazy Loading
Use `@Lazy` for expensive relationships:
```java
@OneToMany
@Lazy
private Set<Session> sessions;
```

### Query Optimization
Use `@EntityGraph` to avoid N+1 queries:
```java
@EntityGraph(attributePaths = {"skills"})
List<User> findAll();
```

## Security Considerations

1. **Password Encoding**: BCrypt via Spring Security
2. **CSRF Protection**: Enabled in SecurityConfig
3. **Role-Based Access**: Authority checks on endpoints
4. **SQL Injection**: Prevented by JPA parameterized queries
5. **XSS Protection**: Thymeleaf escapes by default

## Logging

Configure logging in `application.properties`:

```properties
logging.level.root=INFO
logging.level.com.skillswap=DEBUG
logging.level.org.springframework.web=DEBUG
```

## Common Issues & Solutions

### N+1 Query Problem
Use `@EntityGraph` or eager loading:
```java
@Query("SELECT u FROM User u LEFT JOIN FETCH u.skills")
List<User> findAll();
```

### Lazy Initialization Exception
Move session access to service layer or use `@Transactional`

### Circular Dependencies
Restructure relationships or use DTOs

## Version Control Best Practices

```bash
# Clone strategy
git clone <repo>

# Branch naming
git checkout -b feature/use-case-name
git checkout -b fix/bug-name

# Commit messages
git commit -m "Add user request validation"

# Merge strategy
git merge --no-ff feature/use-case-name
```

---

**Last Updated:** March 2024
**Version:** 1.0.0
