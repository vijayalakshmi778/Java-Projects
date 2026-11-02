# Java JDBC Hands-On Project: Student Management System

A complete beginner-friendly project to teach **JDBC before Spring
Boot**.

**Project:** Student Management System\
**Application type:** Java console application\
**Database:** MySQL\
**Build tool:** Maven\
**Core concepts:** JDBC, DriverManager, Connection, PreparedStatement,
ResultSet, CRUD, DAO pattern, exception handling, transactions, and
resource management.

------------------------------------------------------------------------

## 1. Learning outcome

By the end of this project, the student should be able to explain and
implement this flow:

``` text
User
  |
  v
Main.java (menu and input)
  |
  v
StudentService.java (application logic)
  |
  v
StudentDAO.java (SQL operations)
  |
  v
DBConnection.java (JDBC connection)
  |
  v
MySQL Database
```

The important idea: **Java does not directly manipulate MySQL tables.
JDBC provides the API that allows Java code to send SQL statements to
the database and read the results.**

### Concepts to teach in order

1.  What JDBC is and why it is needed.
2.  JDBC architecture and the role of the MySQL JDBC driver.
3.  Database connection using `DriverManager`.
4.  Executing an `INSERT` using `PreparedStatement`.
5.  Reading records using `ResultSet`.
6.  Updating and deleting records.
7.  Separating database code using the DAO pattern.
8.  Closing database resources safely with try-with-resources.
9.  Handling invalid input and SQL exceptions.
10. Using a transaction for multiple related SQL statements.

------------------------------------------------------------------------

## 2. Prerequisites

The student should already know:

-   Core Java: classes, objects, methods, constructors, access
    modifiers.
-   OOP: encapsulation and separation of responsibilities.
-   Collections: `List` and `ArrayList`.
-   SQL: `CREATE TABLE`, `INSERT`, `SELECT`, `UPDATE`, `DELETE`,
    `WHERE`.
-   HTML and CSS are not used in this console project. They can be
    connected later when the student builds a web UI.

Install:

-   JDK 17 or later.
-   IntelliJ IDEA, Eclipse, or VS Code with Java support.
-   MySQL Server.
-   MySQL Workbench (optional, but useful for viewing tables).
-   Maven support in the IDE.

Check Java and Maven from a terminal:

``` bash
java -version
mvn -version
```

If Maven is not installed separately, use the Maven support bundled with
the IDE or install Maven and configure `PATH`.

------------------------------------------------------------------------

## 3. First understand the JDBC architecture

JDBC stands for **Java Database Connectivity**. It is a Java API used to
connect Java applications to relational databases.

### Without JDBC

A Java program has no built-in understanding of how to communicate with
a MySQL server using MySQL's database protocol.

### With JDBC

``` text
Java application
      |
      | JDBC API
      v
MySQL JDBC Driver
      |
      | MySQL protocol
      v
MySQL Server
      |
      v
Database and tables
```

Explain these terms:

  -----------------------------------------------------------------------
  JDBC component                      Purpose
  ----------------------------------- -----------------------------------
  `DriverManager`                     Finds a suitable registered driver
                                      and creates a connection

  `Connection`                        Represents a session/connection
                                      with the database

  `PreparedStatement`                 Executes parameterized SQL safely

  `ResultSet`                         Holds rows returned by a `SELECT`
                                      query

  `SQLException`                      Represents database access errors
  -----------------------------------------------------------------------

**Teaching note:** JDBC is an API. The MySQL Connector/J library is the
driver implementation that lets JDBC communicate with MySQL.

------------------------------------------------------------------------

## 4. Create the MySQL database

Open MySQL Workbench or a MySQL command-line client.

Run:

``` sql
CREATE DATABASE jdbc_student_db;

USE jdbc_student_db;

CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    course VARCHAR(100) NOT NULL,
    age INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Insert two sample records:

``` sql
INSERT INTO students (name, email, course, age)
VALUES
('Arun Kumar', 'arun@example.com', 'Java Full Stack', 22),
('Priya Devi', 'priya@example.com', 'Python Full Stack', 21);
```

Verify:

``` sql
SELECT * FROM students;
```

### Explain the table

-   `id`: unique ID generated automatically by MySQL.
-   `name`: student's name.
-   `email`: unique email address.
-   `course`: enrolled course.
-   `age`: student's age.
-   `created_at`: automatically records when the row was created.

Ask the student: Why is `email` marked `UNIQUE`? What happens if two
students use the same email?

------------------------------------------------------------------------

## 5. Create the Maven project

Create a Maven project named:

``` text
jdbc-student-management
```

Use this folder structure:

``` text
jdbc-student-management/
│
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── student/
                        ├── Main.java
                        ├── DBConnection.java
                        ├── model/
                        │   └── Student.java
                        ├── dao/
                        │   └── StudentDAO.java
                        └── service/
                            └── StudentService.java
```

Create the packages and Java files exactly as shown.

### 5.1 Configure `pom.xml`

Replace the contents of `pom.xml` with:

``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>jdbc-student-management</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- MySQL JDBC driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.5.0</version>
        </dependency>
    </dependencies>

</project>
```

**Important:** Maven downloads the MySQL Connector/J driver and adds it
to the project's classpath. The version above is an example pinned
dependency; if your organization uses a different approved Connector/J
version, use that version consistently.

After saving, reload or synchronize the Maven project in the IDE.

------------------------------------------------------------------------

## 6. Step 1 --- Create the Student model

File: `src/main/java/com/example/student/model/Student.java`

``` java
package com.example.student.model;

public class Student {

    private int id;
    private String name;
    private String email;
    private String course;
    private int age;

    public Student() {
    }

    public Student(int id, String name, String email,
                   String course, int age) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.course = course;
        this.age = age;
    }

    public Student(String name, String email,
                   String course, int age) {
        this.name = name;
        this.email = email;
        this.course = course;
        this.age = age;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
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

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", course='" + course + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### Explain

This class represents one row in the `students` table.

  Java field   MySQL column
  ------------ --------------
  `id`         `id`
  `name`       `name`
  `email`      `email`
  `course`     `course`
  `age`        `age`

This is a Java object, not a database connection. It simply carries
student data between application layers.

**Ask the student:** If a table has five columns, why might a Java model
class also have five fields?

------------------------------------------------------------------------

## 7. Step 2 --- Create the database connection

File: `src/main/java/com/example/student/DBConnection.java`

``` java
package com.example.student;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DBConnection {

    private static final String URL =
            "jdbc:mysql://localhost:3306/jdbc_student_db";

    private static final String USER = "root";

    // Replace this with your local MySQL password.
    private static final String PASSWORD = "YOUR_MYSQL_PASSWORD";

    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
}
```

Replace `YOUR_MYSQL_PASSWORD` with the password configured for the local
MySQL user.

### Explain each part

-   `jdbc:mysql://`: identifies the JDBC protocol and MySQL driver.
-   `localhost`: MySQL is running on the same computer.
-   `3306`: default MySQL port (unless your installation uses another
    port).
-   `jdbc_student_db`: database name.
-   `DriverManager.getConnection(...)`: asks JDBC to establish a
    database connection.

### Test the connection before continuing

Temporarily add this method to `Main.java`:

``` java
package com.example.student;

import java.sql.Connection;

public class Main {

    public static void main(String[] args) {

        try (Connection connection = DBConnection.getConnection()) {
            System.out.println("Database connected successfully!");
        } catch (Exception e) {
            System.out.println("Database connection failed.");
            e.printStackTrace();
        }
    }
}
```

Run `Main.java`.

Expected output:

``` text
Database connected successfully!
```

If it fails, check:

1.  Is the MySQL server running?
2.  Is the database name correct?
3.  Are the username and password correct?
4.  Is MySQL listening on port `3306`?
5.  Did Maven download the MySQL Connector/J dependency?

**Security note:** Hard-coded credentials are acceptable only for this
local teaching exercise. Do not commit real passwords to GitHub. For a
more realistic version, read credentials from environment variables or
an external configuration file.

------------------------------------------------------------------------

## 8. Step 3 --- Understand the first JDBC INSERT

Before creating the DAO, explain this small JDBC example. It inserts one
student into the table.

``` java
String sql = """
        INSERT INTO students (name, email, course, age)
        VALUES (?, ?, ?, ?)
        """;

try (Connection connection = DBConnection.getConnection();
     PreparedStatement statement =
             connection.prepareStatement(sql)) {

    statement.setString(1, "Kavin");
    statement.setString(2, "kavin@example.com");
    statement.setString(3, "Java Full Stack");
    statement.setInt(4, 23);

    int rows = statement.executeUpdate();

    System.out.println(rows + " row inserted.");
}
```

### Explain line by line

1.  `Connection` opens a connection to MySQL.
2.  The SQL string contains four `?` placeholders.
3.  `prepareStatement(sql)` creates a parameterized SQL statement.
4.  `setString(1, ...)` supplies the first placeholder value.
5.  `setString(2, ...)` supplies the second placeholder value.
6.  `setString(3, ...)` supplies the third placeholder value.
7.  `setInt(4, ...)` supplies the fourth placeholder value.
8.  `executeUpdate()` runs an `INSERT`, `UPDATE`, or `DELETE`.
9.  The returned integer is the number of affected rows.

**Important:** JDBC parameter indexes start at `1`, not `0`.

### Why use PreparedStatement?

Avoid building SQL like this:

``` java
// Do not use this pattern with user input.
String sql = "SELECT * FROM students WHERE email = '" + email + "'";
```

String concatenation can create SQL injection vulnerabilities and
quoting bugs.

Use this instead:

``` java
String sql = "SELECT * FROM students WHERE email = ?";

PreparedStatement statement = connection.prepareStatement(sql);
statement.setString(1, email);
```

The values are bound separately from the SQL command.

------------------------------------------------------------------------

## 9. Step 4 --- Create StudentDAO

DAO means **Data Access Object**. Its responsibility is to communicate
with the database.

File: `src/main/java/com/example/student/dao/StudentDAO.java`

``` java
package com.example.student.dao;

import com.example.student.DBConnection;
import com.example.student.model.Student;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class StudentDAO {

    // CREATE: insert a student
    public boolean addStudent(Student student) throws SQLException {

        String sql = """
                INSERT INTO students (name, email, course, age)
                VALUES (?, ?, ?, ?)
                """;

        try (Connection connection = DBConnection.getConnection();
             PreparedStatement statement =
                     connection.prepareStatement(sql)) {

            statement.setString(1, student.getName());
            statement.setString(2, student.getEmail());
            statement.setString(3, student.getCourse());
            statement.setInt(4, student.getAge());

            return statement.executeUpdate() > 0;
        }
    }

    // READ: retrieve all students
    public List<Student> getAllStudents() throws SQLException {

        List<Student> students = new ArrayList<>();

        String sql = "SELECT id, name, email, course, age FROM students";

        try (Connection connection = DBConnection.getConnection();
             PreparedStatement statement =
                     connection.prepareStatement(sql);
             ResultSet resultSet = statement.executeQuery()) {

            while (resultSet.next()) {

                Student student = new Student(
                        resultSet.getInt("id"),
                        resultSet.getString("name"),
                        resultSet.getString("email"),
                        resultSet.getString("course"),
                        resultSet.getInt("age")
                );

                students.add(student);
            }
        }

        return students;
    }

    // READ: retrieve one student using ID
    public Student getStudentById(int id) throws SQLException {

        String sql = """
                SELECT id, name, email, course, age
                FROM students
                WHERE id = ?
                """;

        try (Connection connection = DBConnection.getConnection();
             PreparedStatement statement =
                     connection.prepareStatement(sql)) {

            statement.setInt(1, id);

            try (ResultSet resultSet = statement.executeQuery()) {

                if (resultSet.next()) {
                    return new Student(
                            resultSet.getInt("id"),
                            resultSet.getString("name"),
                            resultSet.getString("email"),
                            resultSet.getString("course"),
                            resultSet.getInt("age")
                    );
                }
            }
        }

        return null;
    }

    // UPDATE: modify a student's details
    public boolean updateStudent(Student student) throws SQLException {

        String sql = """
                UPDATE students
                SET name = ?, email = ?, course = ?, age = ?
                WHERE id = ?
                """;

        try (Connection connection = DBConnection.getConnection();
             PreparedStatement statement =
                     connection.prepareStatement(sql)) {

            statement.setString(1, student.getName());
            statement.setString(2, student.getEmail());
            statement.setString(3, student.getCourse());
            statement.setInt(4, student.getAge());
            statement.setInt(5, student.getId());

            return statement.executeUpdate() > 0;
        }
    }

    // DELETE: remove a student using ID
    public boolean deleteStudent(int id) throws SQLException {

        String sql = "DELETE FROM students WHERE id = ?";

        try (Connection connection = DBConnection.getConnection();
             PreparedStatement statement =
                     connection.prepareStatement(sql)) {

            statement.setInt(1, id);

            return statement.executeUpdate() > 0;
        }
    }
}
```

### Teach the two execution methods

  -----------------------------------------------------------------------
  Method                  Used for                Return value
  ----------------------- ----------------------- -----------------------
  `executeQuery()`        `SELECT`                `ResultSet`

  `executeUpdate()`       `INSERT`, `UPDATE`,     Number of affected rows
                          `DELETE`                
  -----------------------------------------------------------------------

### Explain ResultSet

A `ResultSet` is like a cursor over the rows returned by a `SELECT`.

``` java
while (resultSet.next()) {
    System.out.println(resultSet.getString("name"));
}
```

-   `next()` moves to the next row.
-   It returns `true` if a row exists.
-   It returns `false` when there are no more rows.
-   `getInt()` reads an integer column.
-   `getString()` reads a text column.

### Explain try-with-resources

``` java
try (Connection connection = ...;
     PreparedStatement statement = ...) {
    // database work
}
```

Java closes these resources automatically when the block finishes,
including when an exception occurs. A `ResultSet` is also closed
automatically when declared in a try-with-resources statement.

This is important because database connections and statements should not
be left open unnecessarily.

------------------------------------------------------------------------

## 10. Step 5 --- Create StudentService

The service layer contains application-level rules. For this first
project it will perform basic input validation and call the DAO.

File: `src/main/java/com/example/student/service/StudentService.java`

``` java
package com.example.student.service;

import com.example.student.dao.StudentDAO;
import com.example.student.model.Student;

import java.sql.SQLException;
import java.util.List;

public class StudentService {

    private final StudentDAO studentDAO = new StudentDAO();

    public boolean addStudent(Student student) throws SQLException {

        validateStudent(student);
        return studentDAO.addStudent(student);
    }

    public List<Student> getAllStudents() throws SQLException {
        return studentDAO.getAllStudents();
    }

    public Student getStudentById(int id) throws SQLException {

        if (id <= 0) {
            throw new IllegalArgumentException("ID must be greater than zero.");
        }

        return studentDAO.getStudentById(id);
    }

    public boolean updateStudent(Student student) throws SQLException {

        if (student.getId() <= 0) {
            throw new IllegalArgumentException("Invalid student ID.");
        }

        validateStudent(student);
        return studentDAO.updateStudent(student);
    }

    public boolean deleteStudent(int id) throws SQLException {

        if (id <= 0) {
            throw new IllegalArgumentException("Invalid student ID.");
        }

        return studentDAO.deleteStudent(id);
    }

    private void validateStudent(Student student) {

        if (student == null) {
            throw new IllegalArgumentException("Student cannot be null.");
        }

        if (student.getName() == null || student.getName().isBlank()) {
            throw new IllegalArgumentException("Name is required.");
        }

        if (student.getEmail() == null || student.getEmail().isBlank()) {
            throw new IllegalArgumentException("Email is required.");
        }

        if (student.getCourse() == null || student.getCourse().isBlank()) {
            throw new IllegalArgumentException("Course is required.");
        }

        if (student.getAge() < 16 || student.getAge() > 100) {
            throw new IllegalArgumentException("Age must be between 16 and 100.");
        }
    }
}
```

### Explain the responsibility of each layer

-   **Model:** carries student data.
-   **DAO:** contains SQL and JDBC code.
-   **Service:** applies application rules and calls the DAO.
-   **Main:** takes user input and displays results.
-   **DBConnection:** creates database connections.

This separation will make the transition to Spring Boot easier. In
Spring Boot, controllers, services, repositories, and model/entity
classes commonly separate these responsibilities.

------------------------------------------------------------------------

## 11. Step 6 --- Create the console menu

File: `src/main/java/com/example/student/Main.java`

``` java
package com.example.student;

import com.example.student.model.Student;
import com.example.student.service.StudentService;

import java.sql.SQLException;
import java.util.List;
import java.util.Scanner;

public class Main {

    private static final Scanner scanner = new Scanner(System.in);
    private static final StudentService studentService = new StudentService();

    public static void main(String[] args) {

        boolean running = true;

        while (running) {

            showMenu();

            int choice = readInt("Enter your choice: ");

            try {
                switch (choice) {

                    case 1 -> addStudent();

                    case 2 -> viewAllStudents();

                    case 3 -> searchStudent();

                    case 4 -> updateStudent();

                    case 5 -> deleteStudent();

                    case 6 -> {
                        running = false;
                        System.out.println("Application closed.");
                    }

                    default -> System.out.println("Invalid choice. Try again.");
                }

            } catch (SQLException e) {
                System.out.println("Database operation failed: " + e.getMessage());

            } catch (IllegalArgumentException e) {
                System.out.println("Invalid input: " + e.getMessage());
            }
        }

        scanner.close();
    }

    private static void showMenu() {

        System.out.println("\n===== STUDENT MANAGEMENT SYSTEM =====");
        System.out.println("1. Add Student");
        System.out.println("2. View All Students");
        System.out.println("3. Search Student by ID");
        System.out.println("4. Update Student");
        System.out.println("5. Delete Student");
        System.out.println("6. Exit");
    }

    private static void addStudent() throws SQLException {

        System.out.println("\n--- Add Student ---");

        String name = readText("Enter name: ");
        String email = readText("Enter email: ");
        String course = readText("Enter course: ");
        int age = readInt("Enter age: ");

        Student student = new Student(name, email, course, age);

        boolean inserted = studentService.addStudent(student);

        if (inserted) {
            System.out.println("Student added successfully!");
        } else {
            System.out.println("Student was not added.");
        }
    }

    private static void viewAllStudents() throws SQLException {

        System.out.println("\n--- All Students ---");

        List<Student> students = studentService.getAllStudents();

        if (students.isEmpty()) {
            System.out.println("No students found.");
            return;
        }

        for (Student student : students) {
            printStudent(student);
        }
    }

    private static void searchStudent() throws SQLException {

        System.out.println("\n--- Search Student ---");

        int id = readInt("Enter student ID: ");

        Student student = studentService.getStudentById(id);

        if (student == null) {
            System.out.println("Student not found.");
        } else {
            printStudent(student);
        }
    }

    private static void updateStudent() throws SQLException {

        System.out.println("\n--- Update Student ---");

        int id = readInt("Enter student ID to update: ");

        Student existingStudent = studentService.getStudentById(id);

        if (existingStudent == null) {
            System.out.println("Student not found.");
            return;
        }

        System.out.println("Current details:");
        printStudent(existingStudent);

        String name = readText("Enter new name: ");
        String email = readText("Enter new email: ");
        String course = readText("Enter new course: ");
        int age = readInt("Enter new age: ");

        Student updatedStudent =
                new Student(id, name, email, course, age);

        boolean updated = studentService.updateStudent(updatedStudent);

        System.out.println(updated
                ? "Student updated successfully!"
                : "No student record was updated.");
    }

    private static void deleteStudent() throws SQLException {

        System.out.println("\n--- Delete Student ---");

        int id = readInt("Enter student ID to delete: ");

        Student student = studentService.getStudentById(id);

        if (student == null) {
            System.out.println("Student not found.");
            return;
        }

        printStudent(student);

        String confirmation = readText("Are you sure? (yes/no): ");

        if (!confirmation.equalsIgnoreCase("yes")) {
            System.out.println("Delete cancelled.");
            return;
        }

        boolean deleted = studentService.deleteStudent(id);

        System.out.println(deleted
                ? "Student deleted successfully!"
                : "Student was not deleted.");
    }

    private static void printStudent(Student student) {

        System.out.println("--------------------------------");
        System.out.println("ID     : " + student.getId());
        System.out.println("Name   : " + student.getName());
        System.out.println("Email  : " + student.getEmail());
        System.out.println("Course : " + student.getCourse());
        System.out.println("Age    : " + student.getAge());
        System.out.println("--------------------------------");
    }

    private static String readText(String message) {

        System.out.print(message);
        return scanner.nextLine().trim();
    }

    private static int readInt(String message) {

        while (true) {

            System.out.print(message);
            String value = scanner.nextLine().trim();

            try {
                return Integer.parseInt(value);
            } catch (NumberFormatException e) {
                System.out.println("Please enter a valid whole number.");
            }
        }
    }
}
```

------------------------------------------------------------------------

## 12. Step 7 --- Run the complete project

Before running:

1.  Start MySQL Server.
2.  Confirm that `jdbc_student_db` exists.
3.  Confirm the `students` table exists.
4.  Check the JDBC URL, username, password, and port in
    `DBConnection.java`.
5.  Reload Maven dependencies.
6.  Run `Main.java`.

### Expected menu

``` text
===== STUDENT MANAGEMENT SYSTEM =====
1. Add Student
2. View All Students
3. Search Student by ID
4. Update Student
5. Delete Student
6. Exit
Enter your choice:
```

### Test 1: Add a student

``` text
Enter your choice: 1
Enter name: Kavin
Enter email: kavin@example.com
Enter course: Java Full Stack
Enter age: 23
Student added successfully!
```

Verify in MySQL:

``` sql
SELECT * FROM students;
```

### Test 2: View all students

``` text
Enter your choice: 2
```

The application should display all rows from the table.

### Test 3: Search by ID

``` text
Enter your choice: 3
Enter student ID: 1
```

It should display the matching student, or `Student not found.` if the
ID does not exist.

### Test 4: Update a student

``` text
Enter your choice: 4
Enter student ID to update: 1
```

Enter the new values and verify:

``` sql
SELECT * FROM students WHERE id = 1;
```

### Test 5: Delete a student

``` text
Enter your choice: 5
Enter student ID to delete: 2
Are you sure? (yes/no): yes
```

Verify:

``` sql
SELECT * FROM students WHERE id = 2;
```

### Test 6: Duplicate email

Try adding another student with an email already in the table.

MySQL should reject the insert because `email` has a `UNIQUE`
constraint. The application will display a database error. Explain that
this is a database constraint, not a JDBC syntax error.

------------------------------------------------------------------------

## 13. Draw the complete request flow for the student

Use the **Add Student** operation as the example.

``` text
1. User selects Add Student in Main.java
                 |
                 v
2. Main.java reads name, email, course and age
                 |
                 v
3. Main.java creates a Student object
                 |
                 v
4. StudentService validates the object
                 |
                 v
5. StudentDAO receives the Student object
                 |
                 v
6. DBConnection creates a JDBC Connection
                 |
                 v
7. StudentDAO prepares an INSERT statement
                 |
                 v
8. PreparedStatement binds the values
                 |
                 v
9. executeUpdate() sends SQL to MySQL
                 |
                 v
10. MySQL inserts the row
                 |
                 v
11. JDBC returns the affected-row count
                 |
                 v
12. Main.java displays the result
```

Ask the student to explain this flow without looking at the code. If
they can explain it clearly, they understand the basic JDBC
architecture.

------------------------------------------------------------------------

## 14. Important JDBC methods to revise

  -----------------------------------------------------------------------
  Method                              Purpose
  ----------------------------------- -----------------------------------
  `DriverManager.getConnection()`     Establish a database connection

  `connection.prepareStatement()`     Create a prepared SQL statement

  `statement.setString()`             Bind a string parameter

  `statement.setInt()`                Bind an integer parameter

  `statement.executeQuery()`          Execute a `SELECT`

  `statement.executeUpdate()`         Execute an `INSERT`, `UPDATE`, or
                                      `DELETE`

  `resultSet.next()`                  Move to the next returned row

  `resultSet.getInt()`                Read an integer column

  `resultSet.getString()`             Read a string column

  `connection.close()`                Close the connection

  `statement.close()`                 Close the statement
  -----------------------------------------------------------------------

With try-with-resources, Java closes the resources automatically, so
explicit `close()` calls are not required inside these blocks.

------------------------------------------------------------------------

## 15. Mini lesson --- Statement vs PreparedStatement

### Statement

``` java
Statement statement = connection.createStatement();

ResultSet resultSet =
        statement.executeQuery("SELECT * FROM students");
```

`Statement` is suitable for fixed SQL without user-supplied values, but
it is not the right choice for building SQL with user input.

### PreparedStatement

``` java
String sql = "SELECT * FROM students WHERE id = ?";

PreparedStatement statement = connection.prepareStatement(sql);

statement.setInt(1, 10);

ResultSet resultSet = statement.executeQuery();
```

`PreparedStatement` supports parameters, helps prevent SQL injection
when values are bound correctly, and is the preferred choice for this
project's CRUD operations.

**Teaching recommendation:** Teach `PreparedStatement` first and use it
throughout the project. Introduce `Statement` only to explain the
difference.

------------------------------------------------------------------------

## 16. Mini lesson --- What is a transaction?

A transaction groups multiple database operations into one logical unit.

For example, suppose a student is enrolled in a course and the
application must:

1.  Insert a student record.
2.  Insert a corresponding course-enrollment record.

If the first insert succeeds but the second fails, the database could be
left with incomplete data. A transaction allows both operations to
succeed together or be rolled back together.

### Transaction example

This is an additional teaching example; it is not required by the main
project.

Create another table:

``` sql
CREATE TABLE course_enrollments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT NOT NULL,
    course_name VARCHAR(100) NOT NULL,
    FOREIGN KEY (student_id) REFERENCES students(id)
);
```

Example JDBC transaction:

``` java
String studentSql = """
        INSERT INTO students (name, email, course, age)
        VALUES (?, ?, ?, ?)
        """;

String enrollmentSql = """
        INSERT INTO course_enrollments (student_id, course_name)
        VALUES (?, ?)
        """;

try (Connection connection = DBConnection.getConnection()) {

    connection.setAutoCommit(false);

    try {
        int generatedStudentId;

        try (PreparedStatement studentStatement =
                     connection.prepareStatement(
                             studentSql,
                             java.sql.Statement.RETURN_GENERATED_KEYS)) {

            studentStatement.setString(1, "Meena");
            studentStatement.setString(2, "meena@example.com");
            studentStatement.setString(3, "Java Full Stack");
            studentStatement.setInt(4, 22);

            studentStatement.executeUpdate();

            try (ResultSet keys = studentStatement.getGeneratedKeys()) {
                if (!keys.next()) {
                    throw new SQLException("Student ID was not generated.");
                }
                generatedStudentId = keys.getInt(1);
            }
        }

        try (PreparedStatement enrollmentStatement =
                     connection.prepareStatement(enrollmentSql)) {

            enrollmentStatement.setInt(1, generatedStudentId);
            enrollmentStatement.setString(2, "Java Full Stack");
            enrollmentStatement.executeUpdate();
        }

        connection.commit();
        System.out.println("Student and enrollment saved.");

    } catch (SQLException e) {
        connection.rollback();
        System.out.println("Transaction rolled back.");
        throw e;
    }
}
```

Explain:

-   `setAutoCommit(false)`: prevents each statement from being committed
    automatically.
-   `commit()`: saves all operations in the transaction.
-   `rollback()`: undoes the uncommitted operations if an error occurs.
-   `getGeneratedKeys()`: retrieves the auto-generated student ID.

A transaction should be kept short. In production code, also consider
how connection state is restored when connections are pooled.

------------------------------------------------------------------------

## 17. Common errors and how to fix them

  --------------------------------------------------------------------------------
  Error                            Likely reason           Fix
  -------------------------------- ----------------------- -----------------------
  `No suitable driver`             MySQL driver is missing Check Maven dependency
                                   from the classpath      and reload Maven

  `Access denied for user`         Wrong MySQL             Check credentials and
                                   username/password or    MySQL user grants
                                   permissions             

  `Unknown database`               Database does not exist Create the database and
                                   or URL is incorrect     verify the URL

  `Communications link failure`    MySQL server is stopped Start MySQL and check
                                   or host/port is wrong   port

  `Table doesn't exist`            Table was not created   Run the `CREATE TABLE`
                                   in the selected         SQL
                                   database                

  `Duplicate entry`                A unique column already Use a different email
                                   contains that value     

  `Column cannot be null`          Required data was not   Validate input and
                                   supplied                inspect SQL parameters

  `Parameter index out of range`   Placeholder count and   Count `?` placeholders
                                   parameter indexes do    carefully
                                   not match               
  --------------------------------------------------------------------------------

------------------------------------------------------------------------

## 18. Student practice tasks

Do not give all answers immediately. Let the student implement these
independently.

### Level 1 --- JDBC basics

-   [ ] Insert a student using `PreparedStatement`.
-   [ ] Display all student names using `ResultSet`.
-   [ ] Search for a student by email.
-   [ ] Display the total number of students using `SELECT COUNT(*)`.

### Level 2 --- CRUD extensions

-   [ ] Add a menu option to search by email.
-   [ ] Add a menu option to display students enrolled in a particular
    course.
-   [ ] Add a menu option to count students in each course.
-   [ ] Add an option to update only the student's course.
-   [ ] Add pagination using `LIMIT` and `OFFSET`.

### Level 3 --- Better application design

-   [ ] Move database credentials to environment variables.
-   [ ] Create a custom exception for a student-not-found case.
-   [ ] Prevent duplicate emails with a friendly message.
-   [ ] Add a confirmation step before deleting.
-   [ ] Write unit tests for service-layer validation.
-   [ ] Add logging instead of relying only on `System.out.println`.

------------------------------------------------------------------------

## 19. Questions to check understanding

Ask these questions after the student finishes the project.

1.  What is JDBC?
2.  Why do we need a JDBC driver?
3.  What is the responsibility of `DriverManager`?
4.  What does a `Connection` represent?
5.  What is the difference between `Statement` and `PreparedStatement`?
6.  Why do parameter indexes start from 1?
7.  What is the purpose of `ResultSet`?
8.  What does `ResultSet.next()` do?
9.  What is the difference between `executeQuery()` and
    `executeUpdate()`?
10. Why should database resources be closed?
11. What is try-with-resources?
12. What is SQL injection?
13. What is the DAO pattern?
14. Why do we have a service layer?
15. What is a transaction?
16. What is the difference between `commit()` and `rollback()`?
17. How does Java know which MySQL database to connect to?
18. What happens if the email already exists?

------------------------------------------------------------------------

## 20. Suggested teaching plan

  -----------------------------------------------------------------------
  Session                 Topic                   Practical outcome
  ----------------------- ----------------------- -----------------------
  1                       JDBC introduction and   Explain Java-to-MySQL
                          architecture            communication

  2                       MySQL setup and Maven   Create database and
                          driver                  connect

  3                       `Connection` and        Insert a student
                          `PreparedStatement`     

  4                       `ResultSet` and         Display and search
                          `executeQuery()`        students

  5                       Update and delete       Complete CRUD

  6                       Model, DAO and service  Organize the
                          layers                  application

  7                       Exceptions and          Handle failures and
                          try-with-resources      close resources

  8                       Transactions and        Understand commit and
                          practice                rollback
  -----------------------------------------------------------------------

A good teaching rhythm is: **explain the concept → write a small example
together → ask the student to predict the output → run it → let the
student modify it independently.**

------------------------------------------------------------------------

## 21. How this project prepares the student for Spring Boot

The student should understand the responsibility of each layer before
moving to Spring Boot.

  -----------------------------------------------------------------------
  This JDBC project                   Typical Spring Boot equivalent
  ----------------------------------- -----------------------------------
  `Main.java` console interaction     Controller / REST API entry point

  `StudentService.java`               `@Service` class

  `StudentDAO.java`                   `@Repository` class or Spring Data
                                      JPA repository

  `Student.java`                      Often a JPA `@Entity` (or DTO,
                                      depending on purpose)

  `DBConnection.java`                 DataSource and connection-pool
                                      configuration

  `PreparedStatement`                 Often handled by Spring JDBC or JPA

  Manual resource handling            Usually managed by Spring
                                      abstractions
  -----------------------------------------------------------------------

**Important distinction:** Spring Boot does not replace JDBC concepts.
Spring JDBC and JPA build on database connectivity concepts. A developer
who understands connections, SQL, prepared statements, result sets, and
transactions will find the database part of Spring Boot much easier.

### Recommended next learning sequence

``` text
Core Java
   |
OOP and Collections
   |
SQL and MySQL
   |
JDBC (this project)
   |
Servlets and HTTP basics
   |
Spring Core / Dependency Injection
   |
Spring Boot
   |
Spring Data JPA
   |
REST APIs
   |
Full Stack integration
```

The student does not need to master every advanced JDBC feature before
starting Spring Boot. They should be able to build this CRUD project,
explain the request flow, use `PreparedStatement`, map `ResultSet` rows
to Java objects, and understand transactions.

------------------------------------------------------------------------

## 22. Final project completion checklist

-   [ ] MySQL database created.
-   [ ] `students` table created.
-   [ ] Maven project created.
-   [ ] MySQL Connector/J dependency added.
-   [ ] Java-to-MySQL connection tested.
-   [ ] Student model created.
-   [ ] Add student implemented.
-   [ ] View all students implemented.
-   [ ] Search by ID implemented.
-   [ ] Update student implemented.
-   [ ] Delete student implemented.
-   [ ] DAO and service layers separated.
-   [ ] `PreparedStatement` used for all user-provided values.
-   [ ] `ResultSet` mapped to Student objects.
-   [ ] try-with-resources used.
-   [ ] SQL errors handled.
-   [ ] Student can explain the complete flow without copying code.

**Definition of done:** The student can independently create a Java
application that connects to MySQL, performs all four CRUD operations,
and explains how JDBC moves data between Java objects and database rows.
