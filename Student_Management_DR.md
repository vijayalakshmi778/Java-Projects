# Deploying a Spring Boot Student Management System to Railway

## Project Overview

This project contains:

-   Spring Boot backend
-   MySQL database
-   HTML, CSS and JavaScript frontend
-   Maven build configuration

The frontend files are stored inside the Spring Boot project under:

``` text
src/main/resources/static/
```

The JavaScript API base path is:

``` javascript
const API = "/api/students";
```

Because the frontend and backend are deployed together, the frontend can
call the backend using the same domain.

------------------------------------------------------------------------

## Step 1: Prepare the Project for Deployment

### 1.1 Check the project structure

The project should contain the Maven wrapper, `pom.xml`, Java source
files and static frontend files.

``` text
studentapi/
├── .mvn/
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/example/studentapi/
        └── resources/
            ├── application.properties
            └── static/
                ├── index.html
                ├── style.css
                └── script.js
```

### 1.2 Update `application.properties`

Open:

``` text
src/main/resources/application.properties
```

Replace the contents with:

``` properties
spring.application.name=studentapi

# Database configuration
spring.datasource.url=${DB_URL:jdbc:mysql://localhost:3306/student_api_db}
spring.datasource.username=${DB_USERNAME:root}
spring.datasource.password=${DB_PASSWORD:}
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.open-in-view=false

# Server configuration
server.port=${PORT:8080}
```

The default values allow the project to use the local MySQL database
when the Railway environment variables are not set. On Railway, the
`DB_URL`, `DB_USERNAME`, `DB_PASSWORD` and `PORT` environment variables
are used.

**Security:** Do not put a real database password in this file or commit
database credentials to GitHub.

### 1.3 Check `.gitignore`

Keep build output and IDE files out of Git. In particular, make sure
`target/` is ignored.

Do not add an exception that forces sensitive local configuration or
credentials into the repository. The `application.properties` file above
contains environment-variable placeholders rather than a real password,
so it can be committed if needed.

------------------------------------------------------------------------

## Step 2: Build the Project Locally

Open PowerShell or the VS Code terminal in the project root folder (the
folder containing `pom.xml`).

On Windows, use the Maven Wrapper:

``` powershell
.\mvnw.cmd clean package
```

Do not use `mvn clean package` if Maven is not installed or not
configured in PATH.

A successful build ends with:

``` text
[INFO] BUILD SUCCESS
```

The packaged application JAR is created in:

``` text
target/studentapi-0.0.1-SNAPSHOT.jar
```

The build in this deployment process completed successfully, with one
test passing and no failures.

------------------------------------------------------------------------

## Step 3: Push the Project to GitHub

1.  Sign in to GitHub.

2.  Create a repository named:

    ``` text
    student-management-system
    ```

3.  Choose the repository visibility you want.

4.  Push the complete Spring Boot project to the repository.

5.  Confirm that `pom.xml`, `mvnw`, `mvnw.cmd`, `.mvn/` and `src/` are
    present in the repository.

If the code has already been pushed, continue to the next step.

------------------------------------------------------------------------

## Step 4: Create a Railway Project from GitHub

1.  Open [Railway](https://railway.com/).
2.  Sign in using GitHub.
3.  Create a new project.
4.  Choose **Deploy from GitHub Repo**.
5.  Select the `student-management-system` repository.
6.  Click **Deploy Repo**.
7.  Wait for Railway to build the project.

Railway detects the Java/Maven project and builds it using Maven.

The initial deployment may build successfully but fail to start because
the online MySQL database has not yet been created or connected.
Continue with the database setup below.

------------------------------------------------------------------------

## Step 5: Add a MySQL Database to Railway

1.  Open the Railway project canvas.
2.  Click **+ Add**.
3.  Choose **Database**.
4.  Select **MySQL**.
5.  Wait until the MySQL service shows that it is online or active.

The project canvas should now show two services:

-   `student-management-system` --- Spring Boot application
-   `MySQL` --- database service

------------------------------------------------------------------------

## Step 6: Configure Database Variables on the Spring Boot Service

The database connection variables must be added to the
**student-management-system application service**, not to the MySQL
service.

1.  Return to the Railway project canvas.
2.  Click the `student-management-system` service.
3.  Open the **Variables** tab.
4.  Click **New Variable**.
5.  Add the following three variables.

### Variable 1: `DB_URL`

**Name:**

``` text
DB_URL
```

**Value:**

``` text
jdbc:mysql://${{MySQL.MYSQLHOST}}:${{MySQL.MYSQLPORT}}/${{MySQL.MYSQLDATABASE}}
```

### Variable 2: `DB_USERNAME`

**Name:**

``` text
DB_USERNAME
```

**Value:**

``` text
${{MySQL.MYSQLUSER}}
```

### Variable 3: `DB_PASSWORD`

**Name:**

``` text
DB_PASSWORD
```

**Value:**

``` text
${{MySQL.MYSQLPASSWORD}}
```

Use Railway's variable-reference picker if available. Select the MySQL
service and the matching variable. The service name in the reference
must match the actual MySQL service name in the Railway project.

Do not type or share the actual database password. The reference allows
Railway to provide it to the application service.

### Important placement check

-   `DB_URL`, `DB_USERNAME` and `DB_PASSWORD` belong on
    `student-management-system`.
-   The original `MYSQL_...` variables belong to the MySQL service.
-   Do not delete the MySQL service's generated database variables.

If the three `DB_` variables were accidentally created on the MySQL
service, remove those misplaced entries and add them to
`student-management-system`.

------------------------------------------------------------------------

## Step 7: Apply the Variable Changes and Redeploy

1.  Check that all three `DB_` variables are present on the
    `student-management-system` service.
2.  Click **Apply Changes** or **Deploy** in Railway.
3.  Wait for the deployment to finish.
4.  Open the latest deployment and check its status.
5.  Confirm that the deployment shows **SUCCESS**.

The Spring Boot service should now be able to connect to the Railway
MySQL database.

------------------------------------------------------------------------

## Step 8: Generate a Public Domain

1.  Open the `student-management-system` service.
2.  Select **Settings**.
3.  Find **Networking** or **Public Networking**.
4.  Select **Generate Domain**.
5.  Wait for Railway to create the public domain.
6.  Copy the generated HTTPS URL.

The generated URL will have a format similar to:

``` text
https://your-service-name-production.up.railway.app
```

Use the exact domain generated in your Railway project. The example
above is only a format example.

------------------------------------------------------------------------

## Step 9: Open and Test the Website

1.  Open the generated Railway HTTPS URL in a browser.

2.  Confirm that the Student Management System frontend loads.

3.  Test the following features:

    -   Add Student
    -   View Student Records
    -   Edit Student
    -   Delete Student

4.  Refresh the browser and check whether saved records remain
    available. This verifies that the application is using the hosted
    database.

------------------------------------------------------------------------

## Troubleshooting

### Maven is not recognized in PowerShell

If this command fails:

``` powershell
mvn clean package
```

Use the Maven Wrapper instead:

``` powershell
.\mvnw.cmd clean package
```

### Railway deployment crashes with a Hibernate dialect error

Example:

``` text
Unable to determine Dialect without JDBC metadata
```

Check the database connection variables on the Spring Boot service:

-   `DB_URL`
-   `DB_USERNAME`
-   `DB_PASSWORD`

### MySQL connection error

Example:

``` text
Communications link failure
Connection refused
```

Check that:

1.  The MySQL service is online.
2.  The application service has the three `DB_` variables.
3.  The variable references point to the correct MySQL service and
    variable names.
4.  The changes have been applied and the application redeployed.

### Build warning about a missing production Maven profile

A message such as:

``` text
The requested profile "production" could not be activated because it does not exist.
```

means the build command requested a Maven profile that is not defined in
the project. This warning alone does not prove that the application
failed. Check the deployment/runtime logs for the actual startup error.

------------------------------------------------------------------------

## Deployment Checklist

-   [ ] `application.properties` uses environment-variable placeholders.
-   [ ] No real database password is committed to GitHub.
-   [ ] Local Maven Wrapper build completes with `BUILD SUCCESS`.
-   [ ] Project code is pushed to GitHub.
-   [ ] Railway project is created from the GitHub repository.
-   [ ] MySQL service is created and online.
-   [ ] `DB_URL` is added to the Spring Boot service.
-   [ ] `DB_USERNAME` is added to the Spring Boot service.
-   [ ] `DB_PASSWORD` is added to the Spring Boot service.
-   [ ] Railway changes are applied and deployment succeeds.
-   [ ] Public domain is generated.
-   [ ] Website opens using the public HTTPS URL.
-   [ ] Add, View, Edit and Delete features are tested.
