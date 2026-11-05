# JDBC Mini Project: Contact Book

A simple Java console application that stores and manages contacts in a
MySQL database using JDBC.

**Project name:** JDBC Contact Book\
**Application:** Java console application\
**Database:** MySQL\
**Build tool:** Maven\
**Operations:** Add contact, view all contacts, search by ID, and delete
contact.

------------------------------------------------------------------------

## 1. Project overview

The application accepts contact details from the user and stores them in
a MySQL table.

``` text
User
  |
  v
Main.java
  |
  v
JDBC API
  |
  v
MySQL JDBC Driver
  |
  v
MySQL Database
```

The Java application uses JDBC to send SQL statements to MySQL and
retrieve query results.

## 2. Software required

1.  JDK 17 or later.
2.  IntelliJ IDEA, Eclipse, or another Java IDE.
3.  MySQL Server.
4.  MySQL Workbench (optional, for running SQL and viewing records).

Maven support is available in IntelliJ IDEA. The MySQL JDBC driver will
be downloaded through the Maven dependency in `pom.xml`.

Verify Java:

``` bash
java -version
javac -version
```

## 3. Create the database

Open MySQL Workbench and connect to your MySQL Server.

Run:

``` sql
CREATE DATABASE jdbc_contact_db;

USE jdbc_contact_db;

CREATE TABLE contacts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    phone VARCHAR(20) NOT NULL,
    email VARCHAR(150)
);
```

Insert two sample contacts:

``` sql
INSERT INTO contacts (name, phone, email)
VALUES
('Arun', '9876543210', 'arun@example.com'),
('Priya', '9876501234', 'priya@example.com');
```

Check the records:

``` sql
SELECT * FROM contacts;
```

### Table structure

  Column    Description
  --------- -------------------------------------------
  `id`      Unique contact ID generated automatically
  `name`    Contact name
  `phone`   Contact phone number
  `email`   Contact email address

## 4. Create a Maven project

Open IntelliJ IDEA.

1.  Select **New Project**.
2.  Choose **Java**.
3.  Select **Maven** as the build system, if shown.
4.  Select JDK 17 or later.
5.  Set the project name to `jdbc-contact-book`.
6.  Create the project.

Project structure:

``` text
jdbc-contact-book/
│
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── contact/
                        ├── DBConnection.java
                        └── Main.java
```

Create the package `com.example.contact`, then create the two Java
classes inside it.

## 5. Add the MySQL JDBC dependency

Open `pom.xml` and use this configuration:

``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>jdbc-contact-book</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.5.0</version>
        </dependency>
    </dependencies>

</project>
```

Save the file and reload or synchronize Maven in the IDE. Maven
downloads the MySQL JDBC driver and makes it available to the project.

## 6. Create the database connection

File: `DBConnection.java`

``` java
package com.example.contact;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DBConnection {

    private static final String URL =
            "jdbc:mysql://localhost:3306/jdbc_contact_db";

    private static final String USER = "root";

    // Replace this with your MySQL password.
    private static final String PASSWORD = "YOUR_MYSQL_PASSWORD";

    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
}
```

Replace `YOUR_MYSQL_PASSWORD` with the password of your local MySQL
user.

  Part                Meaning
  ------------------- ----------------------------------
  `jdbc:mysql://`     JDBC URL prefix for MySQL
  `localhost`         MySQL server is on this computer
  `3306`              MySQL port (default)
  `jdbc_contact_db`   Database name
  `root`              MySQL username
  `getConnection()`   Opens a connection to MySQL

## 7. Test the database connection

Temporarily create this `Main.java`:

``` java
package com.example.contact;

import java.sql.Connection;
import java.sql.SQLException;

public class Main {

    public static void main(String[] args) {

        try (Connection connection = DBConnection.getConnection()) {

            System.out.println("Database connected successfully!");

        } catch (SQLException e) {

            System.out.println("Database connection failed.");
            System.out.println(e.getMessage());
        }
    }
}
```

Run `Main.java`.

Expected output:

``` text
Database connected successfully!
```

After confirming the connection, replace this temporary `Main.java` with
the complete application in the next section.

## 8. Create the complete Contact Book application

File: `Main.java`

``` java
package com.example.contact;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Scanner;

public class Main {

    private static final Scanner scanner = new Scanner(System.in);

    public static void main(String[] args) {

        boolean running = true;

        while (running) {

            System.out.println("\n===== CONTACT BOOK =====");
            System.out.println("1. Add Contact");
            System.out.println("2. View All Contacts");
            System.out.println("3. Search Contact by ID");
            System.out.println("4. Delete Contact");
            System.out.println("5. Exit");

            int choice = readInt("Enter your choice: ");

            try {
                switch (choice) {

                    case 1 -> addContact();
                    case 2 -> viewContacts();
                    case 3 -> searchContact();
                    case 4 -> deleteContact();

                    case 5 -> {
                        running = false;
                        System.out.println("Application closed.");
                    }

                    default -> System.out.println("Invalid choice.");
                }

            } catch (SQLException e) {
                System.out.println("Database error: " + e.getMessage());
            }
        }

        scanner.close();
    }

    // CREATE: Add a new contact
    private static void addContact() throws SQLException {

        System.out.println("\n--- Add Contact ---");

        System.out.print("Enter name: ");
        String name = scanner.nextLine().trim();

        System.out.print("Enter phone: ");
        String phone = scanner.nextLine().trim();

        System.out.print("Enter email: ");
        String email = scanner.nextLine().trim();

        if (name.isBlank() || phone.isBlank()) {
            System.out.println("Name and phone are required.");
            return;
        }

        String sql = """
                INSERT INTO contacts (name, phone, email)
                VALUES (?, ?, ?)
                """;

        try (Connection connection = DBConnection.getConnection();
             PreparedStatement statement =
                     connection.prepareStatement(sql)) {

            statement.setString(1, name);
            statement.setString(2, phone);

            if (email.isBlank()) {
                statement.setNull(3, java.sql.Types.VARCHAR);
            } else {
                statement.setString(3, email);
            }

            int rows = statement.executeUpdate();

            if (rows > 0) {
                System.out.println("Contact added successfully!");
            }
        }
    }

    // READ: Display all contacts
    private static void viewContacts() throws SQLException {

        System.out.println("\n--- All Contacts ---");

        String sql = "SELECT id, name, phone, email FROM contacts";

        try (Connection connection = DBConnection.getConnection();
             PreparedStatement statement =
                     connection.prepareStatement(sql);
             ResultSet resultSet = statement.executeQuery()) {

            boolean found = false;

            while (resultSet.next()) {

                found = true;

                System.out.println("---------------------------");
                System.out.println("ID    : " + resultSet.getInt("id"));
                System.out.println("Name  : " + resultSet.getString("name"));
                System.out.println("Phone : " + resultSet.getString("phone"));
                System.out.println("Email : " + resultSet.getString("email"));
            }

            if (!found) {
                System.out.println("No contacts found.");
            }
        }
    }

    // READ: Search one contact by ID
    private static void searchContact() throws SQLException {

        System.out.println("\n--- Search Contact ---");

        int id = readInt("Enter contact ID: ");

        String sql = """
                SELECT id, name, phone, email
                FROM contacts
                WHERE id = ?
                """;

        try (Connection connection = DBConnection.getConnection();
             PreparedStatement statement =
                     connection.prepareStatement(sql)) {

            statement.setInt(1, id);

            try (ResultSet resultSet = statement.executeQuery()) {

                if (resultSet.next()) {

                    System.out.println("---------------------------");
                    System.out.println("ID    : " + resultSet.getInt("id"));
                    System.out.println("Name  : " + resultSet.getString("name"));
                    System.out.println("Phone : " + resultSet.getString("phone"));
                    System.out.println("Email : " + resultSet.getString("email"));

                } else {
                    System.out.println("Contact not found.");
                }
            }
        }
    }

    // DELETE: Delete one contact by ID
    private static void deleteContact() throws SQLException {

        System.out.println("\n--- Delete Contact ---");

        int id = readInt("Enter contact ID to delete: ");

        String sql = "DELETE FROM contacts WHERE id = ?";

        try (Connection connection = DBConnection.getConnection();
             PreparedStatement statement =
                     connection.prepareStatement(sql)) {

            statement.setInt(1, id);

            int rows = statement.executeUpdate();

            if (rows > 0) {
                System.out.println("Contact deleted successfully!");
            } else {
                System.out.println("Contact not found.");
            }
        }
    }

    // Read an integer safely
    private static int readInt(String message) {

        while (true) {

            System.out.print(message);
            String input = scanner.nextLine().trim();

            try {
                return Integer.parseInt(input);

            } catch (NumberFormatException e) {
                System.out.println("Enter a valid number.");
            }
        }
    }
}
```

## 9. Run the application

Before running:

1.  Start MySQL Server.
2.  Confirm that the `jdbc_contact_db` database exists.
3.  Confirm that the `contacts` table exists.
4.  Check the username, password, and port in `DBConnection.java`.
5.  Confirm Maven has downloaded the MySQL Connector/J dependency.
6.  Run `Main.java`.

Expected menu:

``` text
===== CONTACT BOOK =====
1. Add Contact
2. View All Contacts
3. Search Contact by ID
4. Delete Contact
5. Exit

Enter your choice:
```

## 10. Understand each operation

### Operation 1: Add Contact --- INSERT

SQL:

``` sql
INSERT INTO contacts (name, phone, email)
VALUES (?, ?, ?);
```

Java code:

``` java
String sql = "INSERT INTO contacts (name, phone, email) VALUES (?, ?, ?)";

try (Connection connection = DBConnection.getConnection();
     PreparedStatement statement = connection.prepareStatement(sql)) {

    statement.setString(1, name);
    statement.setString(2, phone);
    statement.setString(3, email);

    int rows = statement.executeUpdate();
}
```

The `?` symbols are placeholders. `setString()` assigns values to those
placeholders. `executeUpdate()` executes the INSERT and returns the
number of affected rows.

### Operation 2: View Contacts --- SELECT

SQL:

``` sql
SELECT * FROM contacts;
```

Java code:

``` java
String sql = "SELECT * FROM contacts";

try (Connection connection = DBConnection.getConnection();
     PreparedStatement statement = connection.prepareStatement(sql);
     ResultSet resultSet = statement.executeQuery()) {

    while (resultSet.next()) {
        System.out.println(resultSet.getString("name"));
    }
}
```

`executeQuery()` executes the SELECT statement and returns a
`ResultSet`.

`resultSet.next()` moves to the next available row. The loop reads each
row until no more rows remain.

### Operation 3: Search Contact --- SELECT with WHERE

SQL:

``` sql
SELECT * FROM contacts WHERE id = ?;
```

Java code:

``` java
String sql = "SELECT * FROM contacts WHERE id = ?";

try (Connection connection = DBConnection.getConnection();
     PreparedStatement statement = connection.prepareStatement(sql)) {

    statement.setInt(1, id);

    try (ResultSet resultSet = statement.executeQuery()) {

        if (resultSet.next()) {
            System.out.println(resultSet.getString("name"));
        } else {
            System.out.println("Contact not found.");
        }
    }
}
```

The ID supplied by the user is assigned to the first placeholder. The
query returns the matching contact, if one exists.

### Operation 4: Delete Contact --- DELETE

SQL:

``` sql
DELETE FROM contacts WHERE id = ?;
```

Java code:

``` java
String sql = "DELETE FROM contacts WHERE id = ?";

try (Connection connection = DBConnection.getConnection();
     PreparedStatement statement = connection.prepareStatement(sql)) {

    statement.setInt(1, id);

    int rows = statement.executeUpdate();
}
```

`executeUpdate()` returns the number of deleted rows. If it returns `1`,
one contact was deleted. If it returns `0`, no matching contact existed.

## 11. Important JDBC classes and methods

  JDBC class or method   Purpose
  ---------------------- -------------------------------------------
  `DriverManager`        Creates a database connection
  `Connection`           Represents the connection to MySQL
  `PreparedStatement`    Executes SQL with parameters
  `ResultSet`            Holds rows returned by SELECT
  `SQLException`         Represents database errors
  `getConnection()`      Opens the database connection
  `prepareStatement()`   Creates a prepared SQL statement
  `setString()`          Assigns a text value to a placeholder
  `setInt()`             Assigns an integer value to a placeholder
  `executeQuery()`       Executes SELECT
  `executeUpdate()`      Executes INSERT, UPDATE, or DELETE
  `next()`               Moves to the next row in a ResultSet

## 12. How the data moves

### Adding a contact

``` text
User enters contact details
          |
          v
Main.java reads the input
          |
          v
SQL INSERT statement is created
          |
          v
DBConnection opens a connection
          |
          v
PreparedStatement receives values
          |
          v
executeUpdate() executes INSERT
          |
          v
MySQL stores the contact
          |
          v
Java displays success message
```

### Viewing contacts

``` text
User selects View All Contacts
          |
          v
Java creates SELECT query
          |
          v
JDBC sends query to MySQL
          |
          v
MySQL returns matching rows
          |
          v
JDBC provides ResultSet
          |
          v
resultSet.next() reads each row
          |
          v
Java displays contact details
```

## 13. Verify the results in MySQL

After adding, viewing, searching, or deleting contacts, run these SQL
queries in MySQL Workbench.

Display all contacts:

``` sql
SELECT * FROM contacts;
```

Search by ID:

``` sql
SELECT * FROM contacts WHERE id = 1;
```

Count contacts:

``` sql
SELECT COUNT(*) FROM contacts;
```

Delete a contact directly using SQL:

``` sql
DELETE FROM contacts WHERE id = 1;
```

## 14. Common errors

  --------------------------------------------------------------------------------
  Error                            Possible reason         Resolution
  -------------------------------- ----------------------- -----------------------
  `No suitable driver`             MySQL JDBC driver is    Reload Maven and verify
                                   missing                 `pom.xml`

  `Access denied for user`         Incorrect MySQL         Check credentials
                                   username or password    

  `Unknown database`               Database does not exist Create
                                                           `jdbc_contact_db`

  `Table doesn't exist`            Contacts table is       Run the `CREATE TABLE`
                                   missing                 statement

  `Communications link failure`    MySQL server is stopped Start MySQL and verify
                                   or port is incorrect    port

  `Parameter index out of range`   Placeholder index does  Check the number of `?`
                                   not match SQL           placeholders
  --------------------------------------------------------------------------------

## 15. JDBC resource management

The project uses try-with-resources:

``` java
try (Connection connection = DBConnection.getConnection();
     PreparedStatement statement = connection.prepareStatement(sql)) {

    // Execute the SQL operation.

}
```

Java automatically closes the `Connection` and `PreparedStatement` when
execution leaves the block, including when an exception occurs.

When a `ResultSet` is used, it is also declared as a resource:

``` java
try (ResultSet resultSet = statement.executeQuery()) {

    while (resultSet.next()) {
        System.out.println(resultSet.getString("name"));
    }
}
```

This ensures that database resources are released after use.

## 16. JDBC and SQL execution methods

  SQL operation   JDBC method         Result
  --------------- ------------------- -------------------------
  INSERT          `executeUpdate()`   Number of inserted rows
  SELECT          `executeQuery()`    `ResultSet`
  UPDATE          `executeUpdate()`   Number of updated rows
  DELETE          `executeUpdate()`   Number of deleted rows

JDBC parameter indexes start from `1`.

For example:

``` java
String sql = "SELECT * FROM contacts WHERE id = ? AND name = ?";

PreparedStatement statement = connection.prepareStatement(sql);

statement.setInt(1, 5);
statement.setString(2, "Arun");
```

The first parameter is assigned to the first `?`, and the second
parameter is assigned to the second `?`.

## 17. Final project structure

``` text
jdbc-contact-book/
│
├── pom.xml
│
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── contact/
                        ├── DBConnection.java
                        └── Main.java
```

## 18. Project completion checklist

-   [ ] Java JDK installed.
-   [ ] MySQL Server running.
-   [ ] Database `jdbc_contact_db` created.
-   [ ] Table `contacts` created.
-   [ ] Maven project created.
-   [ ] MySQL Connector/J dependency downloaded.
-   [ ] Database connection successful.
-   [ ] Add contact operation working.
-   [ ] View all contacts operation working.
-   [ ] Search contact by ID working.
-   [ ] Delete contact operation working.
-   [ ] Results verified in MySQL Workbench.

**Project complete:** The Java application connects to MySQL using JDBC
and performs INSERT, SELECT, and DELETE operations using `Connection`,
`PreparedStatement`, and `ResultSet`.
