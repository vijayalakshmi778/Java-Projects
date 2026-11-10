# Spring Boot Hands-On Training: Student Management REST API

A beginner-friendly theory and practical guide continuing from the JDBC Contact Book and JDBC Student Management projects. Students will rebuild the Student Management System as a Spring Boot REST API backed by MySQL.

## Project overview

| Item | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.5.16 |
| Build tool | Maven |
| Database | MySQL |
| Data access | Spring Data JPA + Hibernate |
| API | REST |
| Testing | Postman or curl |

**Why this version?** This guide uses the stable Spring Boot 3.5.x line and Java 17. It uses the familiar `spring-boot-starter-web` dependency. Spring Boot 4.x is also available, but its starter layout can differ. Keep the versions and dependency names in this guide consistent.

### Final API

| HTTP method | URL | Operation |
|---|---|---|
| POST | `/api/students` | Add student |
| GET | `/api/students` | View all students |
| GET | `/api/students/{id}` | Search by ID |
| PUT | `/api/students/{id}` | Update student |
| DELETE | `/api/students/{id}` | Delete student |

The API receives and returns JSON.

---

# PART 1 — THEORY BEFORE CODING

## 1. What is Spring Framework?

Spring Framework is a Java framework for building applications. It provides infrastructure for Dependency Injection (DI), Inversion of Control (IoC), web applications, database access, transaction management, and testing.

## 2. What is Spring Boot?

Spring Boot simplifies creating and running Spring applications. Its main features include:

1. **Auto-configuration:** configures common components based on project dependencies and settings.
2. **Starter dependencies:** groups commonly used libraries into convenient dependencies.
3. **Embedded server:** includes a server such as Tomcat for web applications.
4. **External configuration:** keeps settings such as database details outside Java code.
5. **Production features:** supports health checks, metrics, and monitoring through additional configuration and dependencies.

### Spring Framework vs Spring Boot

| Spring Framework | Spring Boot |
|---|---|
| Core framework for Java applications | Simplifies creating and running Spring applications |
| More configuration may be needed | Provides sensible defaults and auto-configuration |
| Provides DI, IoC, MVC, transactions, etc. | Builds on Spring Framework |
| Does not automatically mean an embedded server is configured | Common web starter includes an embedded server |

Spring Boot does not replace Spring Framework; it builds on it.

## 3. What changes when moving from JDBC to Spring Boot?

In the JDBC Student Management project, the code used `DriverManager`, `Connection`, `PreparedStatement`, `ResultSet`, SQL statements, a `DBConnection` class, and a DAO class.

In this project, Spring Data JPA provides repository methods. Hibernate maps Java objects to database rows and generates SQL for common operations.

**Important:** JDBC is not removed. JPA/Hibernate generally uses JDBC underneath to communicate with a relational database. Spring Data JPA reduces the amount of JDBC code developers write.

| Previous JDBC project | New Spring Boot project |
|---|---|
| `DBConnection` | Spring datasource configuration |
| `Student` model | `Student` JPA entity |
| `StudentDAO` with SQL | `StudentRepository` |
| `PreparedStatement` | Repository methods such as `save()` |
| Manual CRUD SQL | JPA/Hibernate generates common CRUD SQL |
| `Main` console menu | REST controller |
| Console input/output | JSON request/response |

## 4. What is a REST API?

A REST API exposes resources through URLs and uses HTTP methods to perform operations. In this project, the resource is a student.

Base URL:

```text
http://localhost:8080/api/students
```

| HTTP method | Purpose |
|---|---|
| GET | Read data |
| POST | Create a resource |
| PUT | Update/replace a resource |
| DELETE | Delete a resource |

### What is JSON?

JSON (JavaScript Object Notation) is a text format for exchanging structured data.

```json
{
  "name": "Arun Kumar",
  "email": "arun@example.com",
  "course": "Java Full Stack",
  "age": 22
}
```

Spring converts incoming JSON into a Java object and converts returned Java objects into JSON.

## 5. Important Spring annotations

| Annotation | Purpose |
|---|---|
| `@SpringBootApplication` | Main Boot configuration; enables auto-configuration and component scanning |
| `@RestController` | Declares a REST controller |
| `@RequestMapping` | Defines a base URL |
| `@GetMapping` | Maps GET requests |
| `@PostMapping` | Maps POST requests |
| `@PutMapping` | Maps PUT requests |
| `@DeleteMapping` | Maps DELETE requests |
| `@PathVariable` | Reads a value from the URL |
| `@RequestBody` | Converts request JSON into a Java object |
| `@Entity` | Marks a JPA entity |
| `@Id` | Marks the primary key |
| `@GeneratedValue` | Configures primary-key generation |
| `@Column` | Configures a database column |
| `@Service` | Marks an application service component |
| `@Valid` | Triggers request-object validation |
| `@NotBlank`, `@Email`, `@Min`, `@Max` | Declare validation rules |
| `@Transactional` | Defines a transaction boundary |

## 6. IoC, Dependency Injection, and beans

**Inversion of Control (IoC):** the framework takes responsibility for creating and managing application objects.

**Dependency Injection (DI):** an object receives the other objects it needs instead of creating them itself.

Constructor injection example:

```java
@Service
public class StudentService {

    private final StudentRepository repository;

    public StudentService(StudentRepository repository) {
        this.repository = repository;
    }
}
```

Spring creates and manages the repository and supplies it to the service. An object managed by Spring is called a **Spring bean**.

## 7. Understand the application architecture

```text
Client (Postman / Frontend)
          |
          | HTTP request + JSON
          v
   StudentController
          |
          v
     StudentService
          |
          v
   StudentRepository
          |
          v
 Spring Data JPA / Hibernate
          |
          v
         JDBC
          |
          v
     MySQL Database
```

| Layer | Responsibility |
|---|---|
| Controller | Receives HTTP requests and returns responses |
| Service | Contains application/business logic |
| Repository | Provides database access methods |
| Entity | Represents a persisted database object |
| Configuration | Stores application settings |

Keep the main application class in the root package `com.example.studentapi` so component scanning can find the subpackages.

---

# PART 2 — SOFTWARE SETUP

## 8. Required software

1. JDK 17.
2. IntelliJ IDEA, Eclipse, or VS Code with Java support.
3. MySQL Server.
4. MySQL Workbench (optional).
5. Postman (or curl).
6. Internet access for the initial Maven dependency download.

Check Java:

```bash
java -version
```

Check Maven if installed separately:

```bash
mvn -version
```

A separate Maven installation is not essential if the IDE or Maven Wrapper is used.

## 9. Create the MySQL database

Open MySQL Workbench and connect to MySQL. Run:

```sql
CREATE DATABASE student_api_db;

USE student_api_db;
```

We will let Hibernate create the `students` table when the application starts. The development setting `spring.jpa.hibernate.ddl-auto=update` updates the schema based on entity mappings. This is for learning/development, not a recommended production schema-management strategy.

---

# PART 3 — CREATE THE SPRING BOOT PROJECT

## 10. Generate the project using Spring Initializr

Open https://start.spring.io/

Choose:

| Field | Value |
|---|---|
| Project | Maven |
| Language | Java |
| Spring Boot | 3.5.16 |
| Group | `com.example` |
| Artifact | `studentapi` |
| Name | `studentapi` |
| Description | `Student Management REST API` |
| Package name | `com.example.studentapi` |
| Packaging | Jar |
| Java | 17 |

Add these dependencies:

1. **Spring Web** — builds REST APIs and handles HTTP requests.
2. **Spring Data JPA** — repository abstractions for relational databases.
3. **MySQL Driver** — JDBC driver for MySQL.
4. **Validation** — supports `@NotBlank`, `@Email`, etc.
5. **Spring Boot DevTools** — optional development-time restart support.

Download the ZIP, extract it, and open the project in your IDE.

If Initializr no longer offers the exact patch version, select the latest available Spring Boot 3.5.x version. Keep Java and dependency choices consistent.

## 11. Understand the generated project

```text
studentapi/
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/example/studentapi/
    │   │   └── StudentapiApplication.java
    │   └── resources/
    │       └── application.properties
    └── test/
```

- `pom.xml`: Maven project configuration and dependencies.
- `src/main/java`: Java source code.
- `src/main/resources`: configuration and other resources.
- `application.properties`: server and database settings.
- `mvnw` / `mvnw.cmd`: Maven Wrapper scripts.

---

# PART 4 — BUILD THE APPLICATION STEP BY STEP

## 12. Step 1: Understand the main application class

Open `StudentapiApplication.java`:

```java
package com.example.studentapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StudentapiApplication {

    public static void main(String[] args) {
        SpringApplication.run(StudentapiApplication.class, args);
    }
}
```

**Theory**

- `@SpringBootApplication` marks the main Boot configuration class.
- `SpringApplication.run(...)` starts the application.
- Spring creates the application context and manages configured beans.
- With Spring Web present, the embedded web server starts.
- Default server port: `8080`.

Run the class. Confirm the startup logs show that the application started and Tomcat is listening on port 8080. Stop it before continuing.

## 13. Step 2: Configure MySQL

Open `src/main/resources/application.properties` and add:

```properties
spring.application.name=studentapi

# MySQL connection
spring.datasource.url=jdbc:mysql://localhost:3306/student_api_db
spring.datasource.username=root
spring.datasource.password=YOUR_MYSQL_PASSWORD
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA and Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Web server
server.port=8080
```

Replace `YOUR_MYSQL_PASSWORD` with the local MySQL password.

| Property | Meaning |
|---|---|
| `spring.datasource.url` | MySQL database address |
| `spring.datasource.username` | MySQL username |
| `spring.datasource.password` | MySQL password |
| `spring.datasource.driver-class-name` | MySQL JDBC driver class |
| `spring.jpa.hibernate.ddl-auto=update` | Updates development schema from entity mappings |
| `spring.jpa.show-sql=true` | Prints Hibernate-generated SQL |
| `server.port=8080` | Web server port |

**Security:** This local password setup is for classroom use only. Never commit real database credentials to GitHub. Use environment variables or a secrets manager for a safer setup.

## 14. Step 3: Create the package structure

Inside `com.example.studentapi`, create:

```text
com.example.studentapi
├── StudentapiApplication.java
├── controller/
│   └── StudentController.java
├── entity/
│   └── Student.java
├── repository/
│   └── StudentRepository.java
└── service/
    └── StudentService.java
```

## 15. Step 4: Create the Student entity

File: `entity/Student.java`

```java
package com.example.studentapi.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.Max;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;

@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Name is required")
    @Column(nullable = false)
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Enter a valid email address")
    @Column(nullable = false, unique = true)
    private String email;

    @NotBlank(message = "Course is required")
    @Column(nullable = false)
    private String course;

    @Min(value = 16, message = "Age must be at least 16")
    @Max(value = 100, message = "Age cannot be greater than 100")
    @Column(nullable = false)
    private int age;

    public Student() {
    }

    public Student(String name, String email, String course, int age) {
        this.name = name;
        this.email = email;
        this.course = course;
        this.age = age;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getCourse() {
        return course;
    }

    public void setCourse(String course) {
        this.course = course;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

### Theory: Entity and ORM mapping

An entity is a Java class mapped to a database table.

| Java | Annotation | Database |
|---|---|---|
| `Student` | `@Entity` | `students` table |
| `id` | `@Id` | Primary key |
| ID generation | `@GeneratedValue` | Auto-generated ID |
| `name` | `@Column` | `name` column |
| `email` | `@Column(unique=true)` | Unique email |

- `@Entity`: marks the class as persisted by JPA.
- `@Table`: specifies the table name.
- `@Id`: marks the primary key.
- `@GeneratedValue`: configures primary-key generation.
- `@Column(nullable=false)`: maps a non-null column.
- `@Column(unique=true)`: requests a uniqueness constraint.
- `@NotBlank`, `@Email`, `@Min`, `@Max`: define validation rules.

JPA requires a no-argument constructor. Getters and setters allow Java code and the persistence provider to access fields.

Start the application once to let Hibernate create/update the table. In MySQL Workbench:

```sql
USE student_api_db;
SHOW TABLES;
DESCRIBE students;
```

## 16. Step 5: Create StudentRepository

File: `repository/StudentRepository.java`

```java
package com.example.studentapi.repository;

import com.example.studentapi.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentRepository extends JpaRepository<Student, Long> {
}
```

### Theory: Repository

A repository is the data-access layer. `JpaRepository<Student, Long>` means:

- `Student`: entity type.
- `Long`: primary-key type.

Common inherited methods:

| Method | Purpose |
|---|---|
| `save(student)` | Insert or update |
| `findAll()` | Retrieve all records |
| `findById(id)` | Retrieve one record |
| `existsById(id)` | Check whether ID exists |
| `deleteById(id)` | Delete by ID |
| `count()` | Count records |

We do not write an implementation class. Spring Data JPA creates a runtime implementation and registers it as a Spring bean.

**JDBC comparison:** the JDBC DAO manually wrote `SELECT`, `INSERT`, `UPDATE`, and `DELETE`. Here, the service calls repository methods and JPA/Hibernate handles the common database operations.

## 17. Step 6: Create StudentService

File: `service/StudentService.java`

```java
package com.example.studentapi.service;

import com.example.studentapi.entity.Student;
import com.example.studentapi.repository.StudentRepository;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.server.ResponseStatusException;

import java.util.List;

@Service
public class StudentService {

    private final StudentRepository studentRepository;

    public StudentService(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }

    @Transactional
    public Student addStudent(Student student) {
        return studentRepository.save(student);
    }

    @Transactional(readOnly = true)
    public List<Student> getAllStudents() {
        return studentRepository.findAll();
    }

    @Transactional(readOnly = true)
    public Student getStudentById(Long id) {
        return studentRepository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(
                        HttpStatus.NOT_FOUND,
                        "Student not found with ID: " + id
                ));
    }

    @Transactional
    public Student updateStudent(Long id, Student updatedStudent) {
        Student existingStudent = studentRepository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(
                        HttpStatus.NOT_FOUND,
                        "Student not found with ID: " + id
                ));

        existingStudent.setName(updatedStudent.getName());
        existingStudent.setEmail(updatedStudent.getEmail());
        existingStudent.setCourse(updatedStudent.getCourse());
        existingStudent.setAge(updatedStudent.getAge());

        return studentRepository.save(existingStudent);
    }

    @Transactional
    public void deleteStudent(Long id) {
        if (!studentRepository.existsById(id)) {
            throw new ResponseStatusException(
                    HttpStatus.NOT_FOUND,
                    "Student not found with ID: " + id
            );
        }

        studentRepository.deleteById(id);
    }
}
```

### Theory: Service layer

The service contains application/business logic and sits between controller and repository.

For an update:

1. Find the student.
2. If missing, raise `404 Not Found`.
3. Change the fields.
4. Save through the repository.

Annotations/concepts:

- `@Service`: registers the class as a Spring-managed service bean.
- `@Transactional`: runs the method within a database transaction.
- `readOnly = true`: marks read operations as read-only.
- Constructor injection: Spring supplies the repository dependency.
- `ResponseStatusException`: raises an HTTP-aware exception with a status code.

Do not use `new StudentRepository()`. It is an interface whose implementation is created and managed by Spring Data JPA.

## 18. Step 7: Create StudentController

File: `controller/StudentController.java`

```java
package com.example.studentapi.controller;

import com.example.studentapi.entity.Student;
import com.example.studentapi.service.StudentService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/students")
public class StudentController {

    private final StudentService studentService;

    public StudentController(StudentService studentService) {
        this.studentService = studentService;
    }

    // CREATE
    @PostMapping
    public ResponseEntity<Student> addStudent(
            @Valid @RequestBody Student student) {
        Student savedStudent = studentService.addStudent(student);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedStudent);
    }

    // READ ALL
    @GetMapping
    public ResponseEntity<List<Student>> getAllStudents() {
        return ResponseEntity.ok(studentService.getAllStudents());
    }

    // READ ONE
    @GetMapping("/{id}")
    public ResponseEntity<Student> getStudentById(@PathVariable Long id) {
        return ResponseEntity.ok(studentService.getStudentById(id));
    }

    // UPDATE
    @PutMapping("/{id}")
    public ResponseEntity<Student> updateStudent(
            @PathVariable Long id,
            @Valid @RequestBody Student student) {
        return ResponseEntity.ok(studentService.updateStudent(id, student));
    }

    // DELETE
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteStudent(@PathVariable Long id) {
        studentService.deleteStudent(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Theory: Controller

A controller receives HTTP requests and returns HTTP responses.

- `@RestController`: returned objects are written to the response body, normally as JSON.
- `@RequestMapping`: defines the common URL path.
- `@PostMapping`, `@GetMapping`, `@PutMapping`, `@DeleteMapping`: map HTTP methods.
- `@PathVariable`: reads the ID from a URL such as `/api/students/1`.
- `@RequestBody`: converts JSON into a Java object.
- `@Valid`: triggers validation.
- `ResponseEntity`: lets the controller specify status and response body.

### Understand the POST request

1. Postman sends POST.
2. Spring reads the JSON.
3. Spring converts it into a `Student`.
4. `@Valid` checks the validation rules.
5. Controller calls the service.
6. Service calls the repository.
7. JPA/Hibernate persists the entity.
8. Controller returns the saved student with `201 Created`.

## 19. Step 8: Final project structure

```text
studentapi/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/example/studentapi/
    │   │   ├── StudentapiApplication.java
    │   │   ├── controller/StudentController.java
    │   │   ├── entity/Student.java
    │   │   ├── repository/StudentRepository.java
    │   │   └── service/StudentService.java
    │   └── resources/application.properties
    └── test/
```

---

# PART 5 — RUN AND TEST THE API

## 20. Step 9: Start the application

Check:

1. MySQL Server is running.
2. Database `student_api_db` exists.
3. `application.properties` has the correct credentials.
4. Maven dependencies are downloaded.
5. Main class is `StudentapiApplication.java`.

Run the main class. The console should show that the application started and Tomcat is running on port `8080`.

If startup fails, inspect the first meaningful `Caused by` message.

## 21. Step 10: Test with Postman

Base URL:

```text
http://localhost:8080/api/students
```

For POST and PUT, choose **Body → raw → JSON**.

### Test 1 — Add a student

- Method: `POST`
- URL: `http://localhost:8080/api/students`
- Body:

```json
{
  "name": "Arun Kumar",
  "email": "arun@example.com",
  "course": "Java Full Stack",
  "age": 22
}
```

Expected status: `201 Created`.

Example response (generated ID may differ):

```json
{
  "id": 1,
  "name": "Arun Kumar",
  "email": "arun@example.com",
  "course": "Java Full Stack",
  "age": 22
}
```

### Test 2 — Add a second student

POST to the same URL:

```json
{
  "name": "Priya Devi",
  "email": "priya@example.com",
  "course": "Python Full Stack",
  "age": 21
}
```

Expected: `201 Created`.

### Test 3 — View all students

- Method: `GET`
- URL: `http://localhost:8080/api/students`
- Expected: `200 OK` and a JSON array.

### Test 4 — Search by ID

- Method: `GET`
- URL: `http://localhost:8080/api/students/1`
- Expected: `200 OK` if the ID exists; otherwise `404 Not Found`.

### Test 5 — Update a student

- Method: `PUT`
- URL: `http://localhost:8080/api/students/1`
- Body:

```json
{
  "name": "Arun Kumar Updated",
  "email": "arun.updated@example.com",
  "course": "Spring Boot",
  "age": 23
}
```

Expected: `200 OK`.

The ID in the URL determines which record is updated. The service does not use an ID from the JSON body to choose the record.

### Test 6 — Delete a student

- Method: `DELETE`
- URL: `http://localhost:8080/api/students/1`
- Expected: `204 No Content`.

Try deleting the same ID again. Expected: `404 Not Found`.

## 22. Verify records in MySQL

```sql
USE student_api_db;

SELECT * FROM students;

SELECT COUNT(*) FROM students;

SELECT * FROM students WHERE id = 1;
```

Confirm that changes made through Postman appear in MySQL.

---

# PART 6 — COMPLETE REQUEST FLOW

## 23. Add-student flow

```text
1. User sends POST request from Postman
                  |
                  v
2. JSON reaches StudentController
                  |
                  v
3. Spring converts JSON into Student object
                  |
                  v
4. @Valid checks input
                  |
                  v
5. Controller calls StudentService
                  |
                  v
6. Service calls StudentRepository.save()
                  |
                  v
7. Spring Data JPA delegates to Hibernate
                  |
                  v
8. Hibernate generates SQL
                  |
                  v
9. JDBC sends SQL to MySQL
                  |
                  v
10. MySQL inserts the record
                  |
                  v
11. Saved object returns through the layers
                  |
                  v
12. Spring converts object into JSON
                  |
                  v
13. Client receives HTTP 201 Created
```

## 24. Compare the JDBC and Spring Boot flows

### JDBC

```text
Console input -> Main.java -> StudentService -> StudentDAO
             -> DBConnection -> JDBC -> MySQL
```

### Spring Boot

```text
HTTP/JSON -> StudentController -> StudentService
          -> StudentRepository -> JPA/Hibernate
          -> JDBC -> MySQL
```

The console menu becomes HTTP endpoints. Spring manages components. The repository replaces most manually written CRUD DAO code. JPA/Hibernate maps objects to rows. JDBC remains underneath the persistence layer.

---

# PART 7 — THEORY REVISION

## 25. Spring Boot, Spring MVC, JPA, Hibernate, and JDBC

| Technology | Responsibility |
|---|---|
| Spring Boot | Simplifies setup, configuration, and running |
| Spring MVC | Handles HTTP requests and controllers |
| Spring Data JPA | Provides repository abstractions |
| JPA | Java persistence specification |
| Hibernate | Common JPA implementation |
| JDBC | Low-level Java database API |
| MySQL | Relational database |
| Maven | Builds project and manages dependencies |

## 26. What is ORM?

ORM means **Object-Relational Mapping**. It maps Java objects to relational database tables.

```java
Student student = new Student(
        "Arun",
        "arun@example.com",
        "Java Full Stack",
        22
);

studentRepository.save(student);
```

The application works with a Java object. JPA/Hibernate maps it to a row in the `students` table.

## 27. What is Hibernate?

Hibernate is an ORM framework and a widely used implementation of the JPA specification.

1. `Student` defines the entity mapping.
2. The repository requests persistence operations.
3. Hibernate translates operations into SQL.
4. JDBC communicates with MySQL.

## 28. What is auto-configuration?

Spring Boot examines the libraries on the classpath and the application configuration. It configures common infrastructure when suitable conditions are met.

For example, Spring Web provides web infrastructure. With JPA dependencies, a MySQL driver, and datasource properties, Boot can configure datasource and JPA infrastructure.

Auto-configuration does not make every application decision. Developers still define entities, endpoints, application rules, and settings.

## 29. What is embedded Tomcat?

Tomcat is a Java web server/servlet container. A Spring Boot web application can include Tomcat as an embedded server, so the application starts the server itself.

Default address:

```text
http://localhost:8080
```

## 30. HTTP status codes

| Status | Meaning | Example |
|---|---|---|
| `200 OK` | Request succeeded | Read/update student |
| `201 Created` | Resource created | Student added |
| `204 No Content` | Success with no response body | Student deleted |
| `400 Bad Request` | Invalid request | Validation failure |
| `404 Not Found` | Resource missing | Student ID not found |
| `500 Internal Server Error` | Unexpected server error | Unhandled application/database failure |

---

# PART 8 — COMMON ERRORS AND FIXES

## 31. Troubleshooting

| Error / symptom | Possible cause | Check |
|---|---|---|
| `Communications link failure` | MySQL stopped or wrong host/port | Start MySQL; check port 3306 and URL |
| `Access denied for user` | Wrong credentials/permissions | Check datasource username/password |
| `Unknown database` | Database not created | Run `CREATE DATABASE student_api_db;` |
| `Table doesn't exist` | Entity/table creation did not complete | Check startup logs and JPA config |
| `Port 8080 was already in use` | Another process uses port | Stop it or set `server.port=8081` |
| `Not a managed type` | Entity not detected or missing `@Entity` | Check package and annotation |
| `No qualifying bean` | Component not discovered or configuration issue | Check package structure |
| `404 Not Found` | Wrong URL/method or missing record | Check mapping and ID |
| `400 Bad Request` | Invalid JSON or validation failure | Check JSON and required fields |
| Unique constraint error | Email already exists | Use another email or handle the conflict |

### If Hibernate does not create the table

Check:

1. `@Entity` is present on `Student`.
2. Main class is in `com.example.studentapi`.
3. Entity is in a subpackage such as `com.example.studentapi.entity`.
4. `spring.jpa.hibernate.ddl-auto=update` is present.
5. Database connection succeeds.
6. Startup logs contain no schema/datasource error.

---

# PART 9 — STUDENT PRACTICE

## 32. Level 1 — Understand the code

- [ ] Explain `@SpringBootApplication`.
- [ ] Explain `@RestController`.
- [ ] Explain `@Entity`, `@Id`, and `@GeneratedValue`.
- [ ] Explain why the repository extends `JpaRepository`.
- [ ] Explain constructor injection.
- [ ] Explain the request flow from Postman to MySQL.

## 33. Level 2 — Extend the API

- [ ] Search students by email.
- [ ] Search students by course.
- [ ] Count all students.
- [ ] Count students in a selected course.
- [ ] Sort students by name.
- [ ] Add pagination to GET-all.

## 34. Level 3 — Improve the application

- [ ] Return a clear error response for duplicate emails.
- [ ] Create request/response DTOs instead of exposing the entity directly.
- [ ] Add global exception handling with `@RestControllerAdvice`.
- [ ] Add service unit tests.
- [ ] Add controller tests using Spring Boot test tools.
- [ ] Move credentials to environment variables.
- [ ] Add OpenAPI/Swagger documentation.

---

# PART 10 — QUESTIONS TO CHECK UNDERSTANDING

## 35. Theory questions

1. What is Spring Framework?
2. What is Spring Boot?
3. What problems does Spring Boot solve?
4. What is the difference between Spring and Spring Boot?
5. What is auto-configuration?
6. What is a starter dependency?
7. What is an embedded Tomcat server?
8. What is REST?
9. What is JSON?
10. Difference between GET, POST, PUT, and DELETE?
11. What is IoC?
12. What is Dependency Injection?
13. What is a Spring bean?
14. What is the purpose of `@RestController`?
15. What is the purpose of `@Service`?
16. What is the purpose of `@Entity`?
17. What is JPA?
18. What is Hibernate?
19. What is ORM?
20. What is `JpaRepository`?
21. Does Spring Data JPA remove JDBC completely?
22. Difference between controller, service, and repository?
23. What does `@RequestBody` do?
24. What does `@PathVariable` do?
25. What is `@Transactional`?

## 36. Practical questions

1. Create a student using Postman.
2. Retrieve all students.
3. Retrieve one student by ID.
4. Update a student's course.
5. Delete a student.
6. Send an invalid email and observe validation.
7. Request a missing ID and observe the response.
8. Verify changes in MySQL Workbench.
9. Explain the complete POST request flow.
10. Explain which component generates SQL.

---

# PART 11 — WHAT TO LEARN NEXT

After students can explain and implement the CRUD request flow, continue in this order:

1. DTOs — separate API request/response objects from entities.
2. Global exception handling — `@RestControllerAdvice` and `@ExceptionHandler`.
3. Advanced Spring Data JPA — derived queries, JPQL, pagination, sorting, relationships.
4. Validation — custom validation and structured error responses.
5. Testing — JUnit, Mockito, and Spring Boot test support.
6. Spring Security — authentication and authorization.
7. JWT — token-based authentication.
8. API documentation — OpenAPI/Swagger.
9. Spring Boot Actuator — health and metrics.
10. Deployment — package as a JAR and deploy to a server or cloud platform.

Do not begin security, JWT, or microservices until students understand the CRUD request flow.

---

# FINAL PROJECT CHECKLIST

- [ ] JDK 17 installed.
- [ ] MySQL Server running.
- [ ] `student_api_db` created.
- [ ] Spring Boot project generated.
- [ ] Maven dependencies downloaded.
- [ ] `application.properties` configured.
- [ ] Main application starts.
- [ ] Student entity created.
- [ ] Student repository created.
- [ ] Student service created.
- [ ] Student controller created.
- [ ] POST endpoint tested.
- [ ] GET-all endpoint tested.
- [ ] GET-by-ID endpoint tested.
- [ ] PUT endpoint tested.
- [ ] DELETE endpoint tested.
- [ ] Validation tested.
- [ ] Database records verified in MySQL.
- [ ] Student can explain the request flow.

**Project complete:** The JDBC console-based Student Management System has been converted into a Spring Boot REST API backed by MySQL.

---

# OFFICIAL REFERENCES

- Spring Boot: https://spring.io/projects/spring-boot/
- Spring Boot 3.5 reference: https://docs.spring.io/spring-boot/3.5/
- Spring Initializr: https://start.spring.io/
- Spring Data JPA: https://docs.spring.io/spring-data/jpa/reference/
- Hibernate ORM: https://hibernate.org/orm/documentation/
- MySQL: https://dev.mysql.com/doc/
