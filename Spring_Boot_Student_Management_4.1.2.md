# Spring Boot Hands-On Training: Student Management REST API

A beginner-friendly theory and practical guide continuing from the JDBC Contact Book and JDBC Student Management projects. Students will rebuild the Student Management System as a Spring Boot REST API backed by MySQL.

## Project overview

| Item | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 4.1.2 |
| Build tool | Maven |
| Database | MySQL |
| Data access | Spring Data JPA + Hibernate |
| API | REST |
| Testing | Postman or curl |

**Why this version?** This guide follows the Spring Boot 4.1.2 version selected in your Spring Initializr screenshot. Spring Boot 4.1.2 supports Java 17 or later. Boot 4 uses updated Spring Framework and Jakarta EE components. For the web layer, this guide uses the current `spring-boot-starter-webmvc` starter. Do not copy dependency names from older Boot 2.x or 3.x tutorials without checking them.

### Final API

| HTTP method | URL | Operation |
|---|---|---|
| POST | `/api/students` | Add student |
| GET | `/api/students` | View all students |
| GET | `/api/students/{id}` | Search by ID |
| PUT | `/api/students/{id}` | Update student |
| DELETE | `/api/students/{id}` | Delete student |

The API receives and returns JSON.

## Beginner note: what you are building

You are taking the Student Management System you previously made with JDBC and rebuilding it as a web application.

- In the JDBC project, a user interacted with a console menu.
- In this Spring Boot project, a user or frontend sends an HTTP request.
- The controller receives that request.
- The service decides what operation should happen.
- The repository communicates with the database through JPA/Hibernate.
- MySQL stores the student records.

You do **not** need to learn every Spring feature before starting. Follow the parts in order, run the application after each major step, and compare each new class with the JDBC project you already understand.

### What you should have at the end

You will be able to add, view, search, update, and delete students through API requests. You will test those requests using Postman and verify the records in MySQL Workbench.

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

### What are these tools for?

| Tool | Why you need it |
|---|---|
| JDK 17 | Compiles and runs your Java code |
| IntelliJ IDEA / Eclipse / VS Code | Where you create and edit Java files |
| MySQL Server | Runs the database engine and stores student records |
| MySQL Workbench | A graphical tool for writing SQL and viewing tables |
| Postman | Sends GET, POST, PUT, and DELETE requests to your API |
| Maven Wrapper (`mvnw.cmd`) | Downloads the Maven version needed by the project and runs Maven commands |

**Important:** Spring Boot is a framework, not an IDE. You write your Java code in an IDE and use Spring Boot libraries to build and run the application.

### Check Java on Windows

Open Command Prompt or the VS Code terminal and run:

```bash
java -version
```

You should see Java version 17. If Windows says that `java` is not recognized, install JDK 17 and configure its PATH before continuing.

### Check MySQL

Open MySQL Workbench and connect to your local MySQL Server. If Workbench cannot connect, the MySQL Server service may not be running. Workbench is only the client; MySQL Server is the database engine.

## 9. Create the MySQL database

Open MySQL Workbench and connect to MySQL. Run:

```sql
CREATE DATABASE student_api_db;

USE student_api_db;
```

### Spring Boot 4 dependency note

Older Spring Boot tutorials commonly show `spring-boot-starter-web`. In Spring Boot 4.1, the preferred starter for the traditional Spring MVC / REST API application is `spring-boot-starter-webmvc`. The Initializr dependency is displayed as **Spring Web**. If you inspect `pom.xml`, check which artifact Initializr generated and do not add a second web starter manually.

The JPA, validation, and controller annotations used in this guide are Jakarta-based, for example `jakarta.persistence.*` and `jakarta.validation.*`. Keep those imports exactly as shown.

We will let Hibernate create the `students` table when the application starts. The development setting `spring.jpa.hibernate.ddl-auto=update` updates the schema based on entity mappings. This is for learning/development, not a recommended production schema-management strategy.

---

# PART 3 — CREATE THE SPRING BOOT PROJECT

## 10. Generate the project using Spring Initializr

Spring Initializr is an official website that creates the starting files for a Spring Boot project. It saves you from manually creating the Maven configuration and downloading each library yourself.

Open: https://start.spring.io/

Choose the following values:

| Field | Select / enter | What it means |
|---|---|---|
| Project | Maven | Maven manages libraries and builds the application |
| Language | Java | We are writing the application in Java |
| Spring Boot | 4.1.2 | The framework version selected in your screenshot |
| Group | `com.example` | A base identifier for your project |
| Artifact | `studentapi` | The project/build name |
| Name | `studentapi` | The application name |
| Description | `Student Management REST API` | A short explanation of the project |
| Package name | `com.example.studentapi` | The base Java package |
| Packaging | Jar | Builds one runnable JAR file |
| Configuration | Properties | Uses `application.properties` for settings |
| Java | 17 | The Java version used by the project |

### Add these five dependencies

Click **ADD DEPENDENCIES** and search for each item:

1. **Spring Web** — allows the application to receive HTTP requests and return responses. The current Spring Boot 4 MVC starter is `spring-boot-starter-webmvc`.
2. **Spring Data JPA** — gives you repository methods such as `save()`, `findAll()`, and `findById()`.
3. **MySQL Driver** — lets Java connect to MySQL using JDBC.
4. **Validation** — enables rules such as `@NotBlank`, `@Email`, `@Min`, and `@Max`.
5. **Spring Boot DevTools** — optional development support, including automatic restart when supported files change.

### Generate and extract the ZIP

1. Check that all fields and five dependencies are selected.
2. Click **GENERATE**.
3. Your browser downloads a ZIP file, usually named `studentapi.zip`.
4. Open your Windows **Downloads** folder.
5. Right-click `studentapi.zip` and choose **Extract All**.
6. Extract it to a simple location, for example `Documents/JavaProjects/studentapi`.
7. Do not try to edit Java files while they are still inside the ZIP archive.

### Open the project in your IDE

**IntelliJ IDEA:** choose **File → Open**, select the extracted `studentapi` folder, and open it. If asked whether to trust the project, trust it only if it is the ZIP you generated from Spring Initializr. Wait while IntelliJ imports Maven.

**Eclipse:** choose **File → Import → Maven → Existing Maven Projects**, select the extracted `studentapi` folder, and finish the import.

**VS Code:** choose **File → Open Folder**, select the extracted `studentapi` folder, and wait for the Java and Maven extensions to load the project.

### Wait for Maven

The first import can take a few minutes because Maven downloads the dependencies from the internet. Keep your internet connection active. Wait until the IDE finishes importing before creating Java classes.

If Maven shows a red error, do not delete project files. First check the error message and whether the internet connection is working.

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
- `mvnw` / `mvnw.cmd`: Maven Wrapper scripts. On Windows, use `mvnw.cmd`.

### What is Maven?

Maven is a build and dependency-management tool.

- **Dependency:** a library your project needs, such as Spring Web or the MySQL Driver.
- **Build:** compiling your Java code, running tests, and packaging the application.
- **`pom.xml`:** the file where Maven reads the project details and dependencies.
- **Maven Wrapper:** scripts included in the ZIP so you do not have to install Maven separately.

### Understand `pom.xml`

Open `pom.xml` in the root of the project. It is not a Java file; it is an XML configuration file. Initializr has already written the correct Spring Boot version and selected dependencies into it.

For Spring Boot 4, the web MVC starter is normally named `spring-boot-starter-webmvc`. The other important dependency names are `spring-boot-starter-data-jpa` and `spring-boot-starter-validation`. The MySQL connector is managed by Spring Boot's dependency management.

Do not replace the whole `pom.xml` with an old tutorial's version. Keep the generated file unless you have a specific build error to fix.

### What is a package?

A Java package is a way to group related classes. For example:

- `com.example.studentapi.entity` contains the Student entity.
- `com.example.studentapi.repository` contains database-access interfaces.
- `com.example.studentapi.service` contains business logic.
- `com.example.studentapi.controller` contains API endpoints.

The main application class stays in `com.example.studentapi`, the parent package. This allows Spring Boot to scan the subpackages for components.

---

# BEFORE CODING — HOW TO READ THE PROJECT

A Java file must be created in the matching package folder. For example, `Student.java` belongs in `src/main/java/com/example/studentapi/entity/`. The first line of the file must match its package:

```java
package com.example.studentapi.entity;
```

If the package declaration and folder do not match, the IDE may show errors.

When this guide says **Create a package**, right-click the `com.example.studentapi` package in your IDE, choose **New → Package**, and enter the package name. When it says **Create a class**, right-click the relevant package, choose **New → Java Class**, and enter the class name. For `StudentRepository`, create an **interface**, not a class.

### How to read a Java class

- `package` tells Java which package the file belongs to.
- `import` makes classes from other packages available.
- An annotation such as `@Service` gives Spring information about the class.
- A class contains fields (data) and methods (actions).
- A constructor is used when an object is created.
- A method's return type tells you what value it returns.

Do not paste all classes into one file. Create each file separately using the exact file name shown in the guide.

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

Replace `YOUR_MYSQL_PASSWORD` with the password you created for your local MySQL `root` account. Do not type the angle-bracket placeholder literally.

**Example only:** if your local MySQL password were `MyLocalPassword123`, the line would be:

```properties
spring.datasource.password=MyLocalPassword123
```

Use your own password, not this example. If your MySQL root account has no password, leave the value after `=` empty. Never upload a real password to GitHub.

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

Run `StudentapiApplication.java` from the IDE, or use the Windows Maven Wrapper command in the next section. The console should show that the application started and Tomcat is running on port `8080`.

If startup fails, inspect the first meaningful `Caused by` message.

### How to start the application on Windows

You can also start the project from the terminal. First, open a terminal in the folder that contains `pom.xml`.

```powershell
.\\mvnw.cmd spring-boot:run
```

What this command means:

- `.\` means run a script from the current folder.
- `mvnw.cmd` is the Windows Maven Wrapper.
- `spring-boot:run` asks Maven to compile and start the Spring Boot application.

Keep this terminal open while testing the API. To stop the application, click the terminal and press **Ctrl + C**.

### Check whether the server started

When startup succeeds, the console should show a message that the application has started and the embedded Tomcat server is listening on port `8080`.

Open this address in your browser:

```text
http://localhost:8080
```

You may see a 404 / Whitelabel error page because we have not created a route for `/`. That does **not** necessarily mean the server failed. It can simply mean there is no controller mapped to the root URL yet. The actual student API will be available after you create `StudentController`.

## 21. Step 10: Test with Postman

Base URL:

```text
http://localhost:8080/api/students
```

For POST and PUT, choose **Body → raw → JSON**.

### Understand the Postman screen

- **Method:** choose GET, POST, PUT, or DELETE.
- **URL:** enter the API address.
- **Body:** the data you send to the server. For POST and PUT, select **raw** and then **JSON**.
- **Send:** sends the request.
- **Status:** the HTTP result, such as `200 OK` or `201 Created`.
- **Response body:** the data returned by the API.

For GET and DELETE, you normally do not need a JSON request body.

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

### Important beginner-to-professional note

This is a learning version designed to teach the complete CRUD flow. Before using it for real users, improve it with DTOs, structured error handling, duplicate-email handling, secure external database credentials, tests, and production database migrations. The practice configuration `spring.jpa.hibernate.ddl-auto=update` is convenient for learning but should not be treated as a production database migration strategy.

---

# OFFICIAL REFERENCES

- Spring Boot: https://spring.io/projects/spring-boot/\n- Spring Boot 4.1 system requirements: https://docs.spring.io/spring-boot/system-requirements.html\n- Spring Boot 4.1 starter list: https://docs.spring.io/spring-boot/reference/using/build-systems.html\n- Spring Boot 4.1 first application tutorial: https://docs.spring.io/spring-boot/tutorial/first-application/index.html
- Spring Boot 4.1 reference: https://docs.spring.io/spring-boot/4.1/
- Spring Initializr: https://start.spring.io/
- Spring Data JPA: https://docs.spring.io/spring-data/jpa/reference/
- Hibernate ORM: https://hibernate.org/orm/documentation/
- MySQL: https://dev.mysql.com/doc/


---

## Version and dependency compatibility note

This edition is aligned to the Spring Boot **4.1.2** selection shown in the user's Spring Initializr screenshot. Official Spring Boot documentation states that 4.1.2 requires Java 17 or later. The Boot 4.1 starter reference lists `spring-boot-starter-webmvc` as the Spring MVC/Tomcat starter and marks the older `spring-boot-starter-web` starter as deprecated in favor of it.

Official references:
- System requirements: https://docs.spring.io/spring-boot/system-requirements.html
- Starter list: https://docs.spring.io/spring-boot/reference/using/build-systems.html
- First application tutorial: https://docs.spring.io/spring-boot/tutorial/first-application/index.html
