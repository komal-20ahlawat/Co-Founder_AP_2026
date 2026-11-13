# Database Normalization

**Project:** Co-founder – Co-founder Matching Platform  
**Database:** PostgreSQL 

---

## 1. Introduction

Normalization is the process of organizing data into related tables to reduce data redundancy and prevent insertion, update, and deletion anomalies.

The Co-founder platform stores different types of information, including:

- User account and authentication details.
- Professional profiles and personal information.
- Technical and non-technical skills.
- Areas of interest.
- Startup and project details.
- Skills required by projects.
- Roles available in projects.
- Connection requests between users.
- Applications submitted by users.
- Project team members.
- Notifications.

Since the platform contains several related entities, storing all information in a single table would result in unnecessary data duplication and difficulty maintaining relationships.

Database normalization is used to organize the data into smaller, logically related tables.

The database is designed using the following normal forms:

1. Unnormalized Form (UNF)
2. First Normal Form (1NF)
3. Second Normal Form (2NF)
4. Third Normal Form (3NF)
5. Boyce-Codd Normal Form (BCNF)

The final database schema contains 13 tables.

---

## 2. Initial Data Structure (Unnormalized Form)

Before normalization, all information required by the platform could conceptually be stored in one large structure.

For example:

```text
USER_PROJECT_DATA(
    user_id,
    email,
    password_hash,
    full_name,
    profile_details,
    skills,
    interests,
    projects,
    project_skills,
    project_roles,
    connections,
    applications,
    project_members,
    notifications
)
```

This is an illustrative unnormalized structure, not an actual table in the final database.

A single user can have multiple skills, multiple interests, create multiple projects, and submit multiple applications.

If all these details were stored together, multiple values and repeating groups would appear in the same record.

### Problems in the Unnormalized Structure

**1. Data Redundancy**

User information could be repeated for every project, application, or connection associated with that user.

**2. Update Anomalies**

If a user's email or profile information changes, the same information might need to be updated in multiple records.

**3. Insertion Anomalies**

It would be difficult to add a new skill or interest independently without associating it with a user or project.

**4. Deletion Anomalies**

Deleting a project or application could unintentionally remove other information stored alongside it.

**5. Repeating Groups**

A user may have multiple skills and interests. Storing these as lists in a single field makes searching, updating, and maintaining individual values difficult.

To solve these problems, the data is divided into separate relations.

---

## 3. First Normal Form (1NF)

### Rule

A relation is in First Normal Form (1NF) if:

- Each column contains atomic values.
- Each cell contains only one value.
- There are no repeating groups or multivalued attributes.
- Each record can be uniquely identified using a primary key.

### Problems Before 1NF

In the initial structure, a user could have multiple skills and interests.

For example:

| user_id | full_name | skills | interests |
|---|---|---|---|
| 1 | Rahul | Python, React, Node.js | AI, EdTech |
| 2 | Priya | Java, UI/UX | FinTech, Design |

Here, the `skills` and `interests` columns contain multiple values in a single cell.

This makes it difficult to search for users having a particular skill or interest.

### Conversion to 1NF

To achieve 1NF, repeating and multivalued information is represented using separate records.

**USER_SKILLS**

| id | user_id | skill_id |
|---|---|---|
| 1 | 1 | 101 |
| 2 | 1 | 102 |
| 3 | 1 | 103 |
| 4 | 2 | 104 |
| 5 | 2 | 105 |

**USER_INTERESTS**

| id | user_id | interest_id |
|---|---|---|
| 1 | 1 | 201 |
| 2 | 1 | 202 |
| 3 | 2 | 203 |
| 4 | 2 | 204 |

The actual skill names and interest names are maintained in separate `skills` and `interests` tables.

Similarly, project skills are stored separately in `project_skills`.

### Changes Made

- Multiple skills are represented through individual records in `user_skills`.
- Multiple interests are represented through individual records in `user_interests`.
- Project requirements are represented through individual records in `project_skills`.
- Every table has a primary key to identify its records.
- Repeating groups are removed from the conceptual structure.

### Result

The data is organized into individual records, with each field storing one value.

Therefore, the database structure satisfies 1NF, assuming each text field is stored as one attribute value rather than as an internally maintained list.

---

## 4. Second Normal Form (2NF)

### Rule

A relation is in Second Normal Form (2NF) if:

- It is already in 1NF.
- Every non-prime attribute is fully functionally dependent on every candidate key.
- There are no partial dependencies, where a non-key attribute depends on only part of a composite candidate key.

### Understanding Partial Dependency

Consider a relation containing user and skill information:

```text
USER_SKILL_DETAILS(
    user_id,
    skill_id,
    user_name,
    skill_name,
    proficiency
)
```

The possible functional dependencies would be:

```text
user_id -> user_name

skill_id -> skill_name

(user_id, skill_id) -> proficiency
```

Here:

- `user_name` depends only on `user_id`.
- `skill_name` depends only on `skill_id`.
- `proficiency` depends on the complete user-skill combination.

If `(user_id, skill_id)` is the candidate key, the dependencies of `user_name` and `skill_name` on only part of that key are partial dependencies.

### Removing Partial Dependencies

To remove these dependencies, the information is divided into separate tables.

**USERS**

```text
USERS(
    id,
    email,
    password_hash,
    email_verified,
    created_at,
    updated_at
)
```

**SKILLS**

```text
SKILLS(
    id,
    name,
    created_at
)
```

**USER_SKILLS**

```text
USER_SKILLS(
    id,
    user_id,
    skill_id,
    created_at
)
```

Now:

- User account information is maintained in `users`.
- Skill information is maintained in `skills`.
- The relationship between users and skills is maintained in `user_skills`.

The same design principle is followed for user interests and project skills.

### Functional Dependencies in Relationship Tables

| Table | Candidate Key | Functional Dependency |
|---|---|---|
| `user_skills` | `(user_id, skill_id)` | `(user_id, skill_id) -> id, created_at` |
| `user_interests` | `(user_id, interest_id)` | `(user_id, interest_id) -> id, created_at` |
| `project_skills` | `(project_id, skill_id)` | `(project_id, skill_id) -> id, created_at` |

Each relationship is stored as a separate record.

### Result

The implemented schema uses a single-column primary key (`id`) in these tables and also declares the corresponding composite combinations as unique.

The composite combinations are candidate keys because their columns are declared `NOT NULL`.

The non-key attribute `created_at` depends on the complete association, not on just one side of the relationship.

The other tables also use single-column primary keys or declared alternate candidate keys. Based on the functional dependencies identified in the schema, no partial dependencies remain.

Therefore, the database is designed to satisfy 2NF.

---

## 5. Third Normal Form (3NF)

### Rule

A relation is in Third Normal Form (3NF) if:

- It is already in 2NF.
- It does not contain transitive dependencies of non-key attributes on candidate keys.
- Every non-key attribute depends on the key, the whole key, and nothing but the key.

### Understanding Transitive Dependency

A transitive dependency occurs when a non-key attribute depends on another non-key attribute instead of depending directly on the candidate key.

Consider the following hypothetical structure:

```text
PROJECT_DETAILS(
    project_id,
    owner_id,
    owner_email,
    project_name
)
```

Possible functional dependencies:

```text
project_id -> owner_id, project_name

owner_id -> owner_email
```

Here, the dependency becomes:

```text
project_id -> owner_id -> owner_email
```

The project owner's email depends on the owner, rather than directly on the project.

Keeping owner information in the project relation would cause repeated owner information when the same user creates multiple projects.

### Removing Transitive Dependencies

The database separates entities into dedicated tables.

**1. Users and Profiles**

The `users` table stores account and authentication information.

The `profiles` table stores additional professional information.

```text
USERS(
    id,
    email,
    password_hash,
    email_verified,
    created_at,
    updated_at
)

PROFILES(
    id,
    user_id,
    full_name,
    profile_picture,
    headline,
    bio,
    education,
    experience,
    github_url,
    portfolio_url,
    availability,
    collaboration_preferences,
    created_at,
    updated_at
)
```

The `user_id` column in `profiles` is both a foreign key and a unique column. This ensures that one user can have at most one profile.

**2. Skills and User Skills**

The `skills` table stores each skill once.

The `user_skills` table stores the association between a user and a skill.

```text
SKILLS(
    id,
    name,
    created_at
)

USER_SKILLS(
    id,
    user_id,
    skill_id,
    created_at
)
```

**3. Interests and User Interests**

The `interests` table stores the available interests.

The `user_interests` table connects users with their selected interests.

```text
INTERESTS(
    id,
    name,
    created_at
)

USER_INTERESTS(
    id,
    user_id,
    interest_id,
    created_at
)
```

**4. Projects and Project Skills**

The `projects` table stores project information.

The `project_skills` table stores the skills required by each project.

```text
PROJECTS(
    id,
    owner_id,
    project_name,
    description,
    problem_statement,
    industry,
    project_stage,
    collaboration_type,
    expected_commitment,
    created_at,
    updated_at
)

PROJECT_SKILLS(
    id,
    project_id,
    skill_id,
    created_at
)
```

**5. Connections and Applications**

Connection requests and project applications are maintained independently.

```text
CONNECTIONS(
    id,
    requester_id,
    recipient_id,
    status,
    created_at,
    updated_at
)

APPLICATIONS(
    id,
    project_id,
    applicant_id,
    desired_role,
    application_message,
    status,
    created_at,
    updated_at
)
```

This prevents connection information from being unnecessarily mixed with project application information.

### Functional Dependencies in the Main Tables

| Table | Functional Dependency |
|---|---|
| `users` | `id -> email, password_hash, email_verified, created_at, updated_at` |
| `profiles` | `id -> user_id, full_name, profile_picture, headline, bio, education, experience, github_url, portfolio_url, availability, collaboration_preferences, created_at, updated_at` |
| `skills` | `id -> name, created_at` |
| `interests` | `id -> name, created_at` |
| `projects` | `id -> owner_id, project_name, description, problem_statement, industry, project_stage, collaboration_type, expected_commitment, created_at, updated_at` |
| `project_roles` | `id -> project_id, role_name, description, required_members, created_at, updated_at` |
| `connections` | `id -> requester_id, recipient_id, status, created_at, updated_at` |
| `applications` | `id -> project_id, applicant_id, desired_role, application_message, status, created_at, updated_at` |
| `project_members` | `id -> project_id, user_id, role_name, joined_at` |
| `notifications` | `id -> user_id, actor_id, notification_type, message, is_read, created_at` |

### Result

Information about users, profiles, skills, interests, and projects is stored in separate relations.

This avoids unnecessary repetition and ensures that attributes are maintained with their corresponding entities.

No additional transitive dependencies are identified from the functional dependencies declared in the schema.

Therefore, the database is designed to satisfy 3NF based on the declared dependencies.

---

## 6. Boyce-Codd Normal Form (BCNF)

### Rule

A relation is in Boyce-Codd Normal Form (BCNF) if, for every non-trivial functional dependency:

```text
X -> Y
```

`X` must be a superkey of the relation.

In simple terms, every determinant must be a candidate key or a superkey.

BCNF is stricter than 3NF because it does not allow a non-superkey determinant even in cases where 3NF may allow a dependency.

### Candidate Keys and Functional Dependencies

The database uses primary keys and unique constraints to identify records.

The following candidate keys are declared in the schema:

| Table | Candidate Keys |
|---|---|
| `users` | `id`, `email` |
| `profiles` | `id`, `user_id` |
| `skills` | `id`, `name` |
| `user_skills` | `id`, `(user_id, skill_id)` |
| `interests` | `id`, `name` |
| `user_interests` | `id`, `(user_id, interest_id)` |
| `projects` | `id` |
| `project_skills` | `id`, `(project_id, skill_id)` |
| `project_roles` | `id` |
| `connections` | `id`, `(requester_id, recipient_id)` |
| `applications` | `id`, `(project_id, applicant_id)` |
| `project_members` | `id`, `(project_id, user_id)` |
| `notifications` | `id` |

### BCNF Analysis

The following functional dependencies are based on the primary keys and unique constraints declared in the SQL schema.

**1. Users**

```text
id -> email, password_hash, email_verified,
      created_at, updated_at

email -> id, password_hash, email_verified,
         created_at, updated_at
```

Both `id` and `email` are candidate keys.

Therefore, the determinants are candidate keys.

**2. Profiles**

```text
id -> user_id, full_name, profile_picture, headline,
      bio, education, experience, github_url,
      portfolio_url, availability,
      collaboration_preferences, created_at, updated_at

user_id -> id, full_name, profile_picture, headline,
           bio, education, experience, github_url,
           portfolio_url, availability,
           collaboration_preferences, created_at, updated_at
```

Both `id` and `user_id` are candidate keys because `user_id` is declared unique and not null.

Therefore, the relation satisfies BCNF under the declared dependencies.

**3. Skills**

```text
id -> name, created_at

name -> id, created_at
```

Both `id` and `name` are candidate keys.

Therefore, the relation satisfies BCNF.

**4. User Skills**

```text
id -> user_id, skill_id, created_at

(user_id, skill_id) -> id, created_at
```

Both the primary key and the unique composite key identify a record.

Therefore, the relation satisfies BCNF.

**5. Interests**

```text
id -> name, created_at

name -> id, created_at
```

Both `id` and `name` are candidate keys.

Therefore, the relation satisfies BCNF.

**6. User Interests**

```text
id -> user_id, interest_id, created_at

(user_id, interest_id) -> id, created_at
```

Both the primary key and the unique composite key identify a record.

Therefore, the relation satisfies BCNF.

**7. Projects**

```text
id -> owner_id, project_name, description,
      problem_statement, industry, project_stage,
      collaboration_type, expected_commitment,
      created_at, updated_at
```

The project ID uniquely identifies a project.

The `owner_id` column is a foreign key and is not declared unique, since one user can create multiple projects.

Therefore, `id` is the declared candidate key and the relation satisfies BCNF under the declared dependencies.

**8. Project Skills**

```text
id -> project_id, skill_id, created_at

(project_id, skill_id) -> id, created_at
```

The composite combination is declared unique.

Therefore, the relation satisfies BCNF.

**9. Project Roles**

```text
id -> project_id, role_name, description,
      required_members, created_at, updated_at
```

The project role ID identifies each role record.

The schema does not declare `(project_id, role_name)` as unique. Therefore, it is not treated as a candidate key.

Based on the declared dependencies, the relation satisfies BCNF.

**10. Connections**

```text
id -> requester_id, recipient_id, status,
      created_at, updated_at

(requester_id, recipient_id) -> id, status,
                                created_at, updated_at
```

The composite combination is declared unique and both columns are not null.

Therefore, the relation satisfies BCNF.

**11. Applications**

```text
id -> project_id, applicant_id, desired_role,
      application_message, status, created_at, updated_at

(project_id, applicant_id) -> id, desired_role,
                              application_message, status,
                              created_at, updated_at
```

The combination of project and applicant is declared unique.

Therefore, the relation satisfies BCNF.

**12. Project Members**

```text
id -> project_id, user_id, role_name, joined_at

(project_id, user_id) -> id, role_name, joined_at
```

The combination of project and user is declared unique.

Therefore, the relation satisfies BCNF.

**13. Notifications**

```text
id -> user_id, actor_id, notification_type,
      message, is_read, created_at
```

The notification ID uniquely identifies each notification.

The `actor_id` column is optional and is not declared unique.

Therefore, the relation satisfies BCNF under the declared dependencies.

### BCNF Conclusion

Under the functional dependencies represented by the declared primary keys and unique constraints, the determinants in these examples are candidate keys or superkeys.

Therefore, the relations satisfy BCNF under those assumptions.

**Important:** A complete BCNF proof depends on all real business functional dependencies, not only the declared primary keys. If additional business rules introduce dependencies, the relations must be reassessed.

---

## 7. Final Normalized Database Structure

After normalization, the Co-founder database consists of 13 relations.

| No. | Table | Description |
|---|---|---|
| 1 | `users` | Stores user account and authentication information. |
| 2 | `profiles` | Stores professional profile information associated with users. |
| 3 | `skills` | Stores available technical and non-technical skills. |
| 4 | `user_skills` | Maintains the many-to-many relationship between users and skills. |
| 5 | `interests` | Stores available areas of interest. |
| 6 | `user_interests` | Maintains the many-to-many relationship between users and interests. |
| 7 | `projects` | Stores startup and project information created by users. |
| 8 | `project_skills` | Maintains the skills required by projects. |
| 9 | `project_roles` | Stores roles and the number of members required for projects. |
| 10 | `connections` | Stores connection requests between users. |
| 11 | `applications` | Stores applications submitted by users for projects. |
| 12 | `project_members` | Stores users who are part of project teams. |
| 13 | `notifications` | Stores notifications received by users. |

### Relationship Overview

```text
USERS
 |
 +---- PROFILES
 |
 +---- USER_SKILLS -------- SKILLS
 |
 +---- USER_INTERESTS ----- INTERESTS
 |
 +---- CONNECTIONS
 |
 +---- PROJECTS
 |        |
 |        +---- PROJECT_SKILLS ---- SKILLS
 |        |
 |        +---- PROJECT_ROLES
 |        |
 |        +---- APPLICATIONS
 |        |
 |        +---- PROJECT_MEMBERS
 |
 +---- APPLICATIONS
 |
 +---- PROJECT_MEMBERS
 |
 +---- NOTIFICATIONS
```

### Important Constraints

The database also uses constraints to maintain data integrity.

| Constraint | Purpose |
|---|---|
| `PRIMARY KEY` | Uniquely identifies each record. |
| `FOREIGN KEY` | Maintains relationships between tables. |
| `UNIQUE` | Prevents duplicate values or associations. |
| `NOT NULL` | Ensures required fields contain values. |
| `CHECK` | Validates values against specified conditions. |
| `DEFAULT` | Provides default values when a value is not supplied. |

Examples from the schema:

- A user cannot register with an already registered email address.
- A user can have at most one professional profile.
- A user-skill combination cannot be inserted more than once.
- A user-interest combination cannot be inserted more than once.
- A project-skill combination cannot be inserted more than once.
- A user cannot send a connection request to themselves.
- Connection and application statuses are restricted to `pending`, `accepted`, or `rejected`.
- A project role must require at least one member.
- A user cannot apply to the same project more than once.
- A user cannot be added to the same project team more than once.

---

## 8. Anomalies Prevented Through Normalization

Normalization helps prevent the following anomalies:

| Anomaly | Problem Before Normalization | Solution in the Normalized Database |
|---|---|---|
| Insertion Anomaly | A new skill or interest could not be conveniently maintained independently of user records. | Skills and interests are stored in separate tables. |
| Update Anomaly | Changing repeated user or skill information could require updating multiple records. | User and skill information is stored once in their respective tables. |
| Deletion Anomaly | Removing a project-related record could unintentionally remove other information stored alongside it. | Independent entities are maintained in separate relations. |
| Data Redundancy | User, skill, and interest information could be unnecessarily repeated. | Separate entity and mapping tables reduce duplication. |
| Relationship Duplication | The same user could be associated with the same skill or project more than once. | Composite unique constraints prevent duplicate associations. |

---

## 9. Benefits of Normalization

The normalized database provides several benefits:

**1. Reduced Data Redundancy**

Information such as user details, skills, and interests is stored in dedicated tables instead of being repeated unnecessarily.

**2. Improved Data Consistency**

Changes to shared information can be made in one place, reducing the possibility of inconsistent records.

**3. Better Data Integrity**

Primary keys, foreign keys, unique constraints, and check constraints help maintain valid data.

**4. Easier Database Maintenance**

The separation of different entities makes the database easier to understand, update, and manage.

**5. Prevention of Data Anomalies**

Normalization reduces insertion, update, and deletion anomalies.

**6. Better Relationship Management**

Many-to-many relationships between users, skills, interests, and projects are maintained through dedicated mapping tables.

**7. Scalability**

The platform can support additional users, skills, interests, projects, applications, and connections without storing everything in a single large table.

---

## 10. Conclusion

The Co-founder database is designed using normalization principles to organize platform information into 13 related tables.

The initial conceptual structure contained multiple types of information together, which could lead to data redundancy, repeating groups, and insertion, update, and deletion anomalies.

Through 1NF, repeating groups and multivalued attributes were separated into individual records. Through 2NF, partial dependencies were eliminated from the illustrated composite-key relations. Through 3NF, different entities and their attributes were maintained in separate relations to avoid the illustrated transitive dependencies. Finally, BCNF was evaluated using the candidate keys and functional dependencies declared in the schema.

The resulting database uses primary keys, foreign keys, unique constraints, and check constraints to maintain data integrity and consistency.

This normalized structure provides a well-organized foundation for the Co-founder platform and supports its core features, including professional profiles, skill-based discovery, project creation, connection requests, applications, and project team management.