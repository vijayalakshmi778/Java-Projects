# Spring Boot Movie Ticket Booking System --- Step-by-Step Project Flow

**Project type:** Movie/cinema ticket booking website\
**Stack:** Java 17, Spring Boot 4.1.2, Maven, Spring Data JPA, MySQL,
HTML, CSS, JavaScript, Postman, Git/GitHub.

> Assumption: "Ticket booking" means movie tickets. This is a learning
> project. Payment is simulated; it does not collect real card details
> or transfer real money.

This project follows the same layered structure as the Student
Management REST API: **Controller → Service → Repository → JPA/Hibernate
→ MySQL**. Build each phase, test it, and only then continue.

------------------------------------------------------------------------

# 1. Project Goal and Features

## Customer features

1.  Register, log in, and log out.
2.  Browse/search movies and view details.
3.  Select city, theatre, date, and showtime.
4.  View a screen's seat layout.
5.  Select multiple available seats.
6.  Temporarily hold seats while checking out.
7.  Complete simulated payment.
8.  Receive a booking reference/ticket.
9.  View booking history and cancel eligible bookings.

## Admin features

1.  Admin login and protected dashboard.
2.  Add, edit, publish, and deactivate movies.
3.  Manage cities, theatres, screens, and physical seats.
4.  Create showtimes and set prices.
5.  View bookings and basic reports.

## Technology plan

  Layer             Technology
  ----------------- ------------------------------------------
  Language          Java 17
  Framework         Spring Boot 4.1.2
  Build             Maven
  Backend           Spring MVC / REST
  Frontend          HTML, CSS, JavaScript
  Database          MySQL
  ORM               Hibernate
  Persistence       Spring Data JPA
  Validation        Jakarta Bean Validation
  API testing       Postman
  Database tool     MySQL Workbench
  Version control   Git and GitHub
  Hosting           Railway or another Java hosting platform

For Spring Boot 4 MVC, use the Spring Initializr **Spring Web**
dependency and verify the generated artifact is
`spring-boot-starter-webmvc`. Avoid copying dependency names from old
tutorials without checking compatibility.

------------------------------------------------------------------------

# 2. Complete Application Flow

## Customer flow

``` text
Website → Register/Login → Browse Movies
        → Select Movie → Select City/Theatre
        → Select Date/Showtime → View Seats
        → Select Seats → Review Price
        → Hold Seats → Simulated Payment
        → Payment Success → Confirm Booking
        → Booking Reference / Booking History
```

If payment fails or the hold expires, release the seats.

## Admin flow

``` text
Admin Login → Add Movie → Add City/Theatre
            → Create Screen → Configure Seats
            → Create Showtime → Set Price
            → Publish Showtime → Customers Can Book
```

## Backend request flow

``` text
HTML/CSS/JavaScript
        |
   HTTP + JSON
        v
 REST Controller
        v
     Service
        v
   Repository
        v
 Spring Data JPA
        v
 Hibernate / JDBC
        v
      MySQL
```

-   Controller receives requests and returns responses.
-   Service contains business and booking rules.
-   Repository accesses the database.
-   Entity maps a Java class to a table.
-   DTO defines API request/response data.
-   MySQL stores the application data.

------------------------------------------------------------------------

# 3. Business Rules --- Decide Before Coding

1.  A customer must be logged in to book.
2.  Email addresses must be unique.
3.  Customers can access only their own bookings.
4.  Admin APIs must be protected by an admin role.
5.  A theatre has multiple screens; a screen has physical seats.
6.  A show connects one movie to one screen at a date/time.
7.  The same screen cannot have overlapping shows.
8.  A seat's availability is specific to a show, not global to the
    physical seat.
9.  The backend rechecks availability when a seat is held; browser
    status is never authoritative.
10. A hold expires after a short period, such as five minutes.
11. The server calculates the final price. Never trust a total sent by
    the browser.
12. Use `BigDecimal` for money, not `double`.
13. Never store plain-text passwords.
14. For this project, payment is simulated. Never ask for real card
    numbers, CVVs, UPI PINs, or banking passwords.

Suggested states:

-   Seat inventory: `AVAILABLE`, `HELD`, `BOOKED`
-   Booking: `PENDING_PAYMENT`, `CONFIRMED`, `CANCELLED`,
    `PAYMENT_FAILED`, `EXPIRED`
-   Payment: `PENDING`, `SUCCESS`, `FAILED`, `REFUNDED`

------------------------------------------------------------------------

# 4. Software Setup

Install: 1. JDK 17. 2. IntelliJ IDEA, Eclipse, or VS Code with Java
support. 3. MySQL Server. 4. MySQL Workbench (optional). 5. Postman. 6.
Git. 7. Browser and internet connection for Maven downloads.

Verify:

``` bash
java -version
git --version
```

A separate Maven installation is not required if you use the Maven
Wrapper generated by Spring Initializr.

------------------------------------------------------------------------

# 5. Generate the Spring Boot Project

Open https://start.spring.io/

Select:

  Field           Value
  --------------- ---------------------------------
  Project         Maven
  Language        Java
  Spring Boot     4.1.2
  Group           `com.zetlan`
  Artifact        `movie-ticket-booking`
  Name            `movie-ticket-booking`
  Description     `Movie Ticket Booking System`
  Package         `com.zetlan.movieticketbooking`
  Packaging       Jar
  Configuration   Properties
  Java            17

If 4.1.2 is no longer offered, select a currently available compatible
Spring Boot 4.1.x patch and use it consistently.

Add dependencies: 1. **Spring Web** (Boot 4 MVC starter:
`spring-boot-starter-webmvc`) --- REST APIs. 2. **Spring Data JPA** ---
repositories and persistence. 3. **MySQL Driver** --- JDBC connection to
MySQL. 4. **Validation** --- validate request fields. 5. **Spring Boot
DevTools** --- optional development restart support.

Add **Spring Security** in the authentication phase if you want to build
and test the catalogue first. Do not add the same dependency twice.

Click Generate, extract the ZIP, open the extracted folder in the IDE,
and wait for Maven import to finish.

## Generated project

``` text
movie-ticket-booking/
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/zetlan/movieticketbooking/
    │   │   └── MovieTicketBookingApplication.java
    │   └── resources/
    │       └── application.properties
    └── test/
```

Keep the main application class in the root package so Spring scans all
subpackages.

------------------------------------------------------------------------

# 6. Create the MySQL Database

Open MySQL Workbench, connect to MySQL Server, and run:

``` sql
CREATE DATABASE movie_ticket_booking_db;
USE movie_ticket_booking_db;
```

Do not manually create all tables yet. In the learning phase, JPA
entities define the table structure and Hibernate can create/update it.

------------------------------------------------------------------------

# 7. Configure application.properties

File: `src/main/resources/application.properties`

``` properties
spring.application.name=movie-ticket-booking

spring.datasource.url=${DB_URL:jdbc:mysql://localhost:3306/movie_ticket_booking_db}
spring.datasource.username=${DB_USERNAME:root}
spring.datasource.password=${DB_PASSWORD:}
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.open-in-view=false

server.port=${PORT:8080}
```

Provide your local MySQL password through `DB_PASSWORD` or a local-only
configuration excluded from Git. Never commit credentials.

  Setting             Meaning
  ------------------- -------------------------------------------------
  `DB_URL`            Database URL
  `DB_USERNAME`       MySQL username
  `DB_PASSWORD`       MySQL password
  `ddl-auto=update`   Updates development schema from entity mappings
  `show-sql=true`     Prints generated SQL
  `server.port`       Uses hosting platform port or local 8080

`ddl-auto=update` is for learning. Use controlled migrations such as
Flyway or Liquibase for production.

------------------------------------------------------------------------

# 8. Database Design

## Tables

  Table                   Purpose
  ----------------------- ------------------------------------------
  `users`                 Customer/admin accounts
  `movies`                Movie information
  `cities`                City names
  `theatres`              Theatre details and city
  `screens`               Screens inside theatres
  `seats`                 Physical seats for each screen
  `shows`                 Movie, screen, date/time, ticket price
  `show_seat_inventory`   Seat status for each show
  `bookings`              Booking owner, reference, amount, status
  `booking_seats`         Seats included in a booking
  `payments`              Simulated payment reference and status

## Relationships

``` text
CITY 1 -------- many THEATRE
THEATRE 1 ----- many SCREEN
SCREEN 1 ------ many SEAT

MOVIE 1 ------- many SHOW
SCREEN 1 ------ many SHOW

USER 1 -------- many BOOKING
SHOW 1 -------- many BOOKING
BOOKING 1 ----- many BOOKING_SEAT
SEAT 1 -------- many BOOKING_SEAT

SHOW 1 -------- many SHOW_SEAT_INVENTORY
SEAT 1 -------- many SHOW_SEAT_INVENTORY
BOOKING 1 ----- many PAYMENT records/attempts
```

## Critical design: show-specific seat inventory

Do not put the only `AVAILABLE`/`BOOKED` status on the physical `seats`
table. Seat A1 may be booked for the 10 AM show but available for the 2
PM show.

`show_seat_inventory` should include: - `id` - `show_id` - `seat_id` -
`status` - `held_by_user_id` - `hold_expires_at`

Add a unique database constraint on `(show_id, seat_id)`. When a show is
created, insert one inventory row for each active physical seat on its
screen, initially `AVAILABLE`.

------------------------------------------------------------------------

# 9. Java Package Structure

Create these packages under `com.zetlan.movieticketbooking`:

``` text
com.zetlan.movieticketbooking
├── MovieTicketBookingApplication.java
├── config/
│   ├── SecurityConfig.java
│   └── WebConfig.java
├── controller/
│   ├── AuthController.java
│   ├── MovieController.java
│   ├── CityController.java
│   ├── TheatreController.java
│   ├── ShowController.java
│   ├── SeatController.java
│   ├── BookingController.java
│   └── PaymentController.java
├── dto/
│   ├── RegisterRequest.java
│   ├── LoginRequest.java
│   ├── MovieRequest.java
│   ├── ShowRequest.java
│   ├── SeatHoldRequest.java
│   └── BookingResponse.java
├── entity/
│   ├── User.java
│   ├── Movie.java
│   ├── City.java
│   ├── Theatre.java
│   ├── Screen.java
│   ├── Seat.java
│   ├── Show.java
│   ├── ShowSeatInventory.java
│   ├── Booking.java
│   ├── BookingSeat.java
│   └── Payment.java
├── enums/
│   ├── Role.java
│   ├── SeatStatus.java
│   ├── BookingStatus.java
│   └── PaymentStatus.java
├── exception/
│   ├── GlobalExceptionHandler.java
│   ├── ResourceNotFoundException.java
│   └── SeatUnavailableException.java
├── repository/
│   └── [one repository interface per entity]
├── service/
│   └── [one service class per feature]
└── util/
    └── BookingReferenceGenerator.java
```

This is the target structure, not a requirement to create every file
immediately. Create each Java class in its own file and match package
declarations to folders.

------------------------------------------------------------------------

# 10. Development Phases --- Follow This Exact Order

## PHASE 1 --- Start the empty application

1.  Open `MovieTicketBookingApplication.java`.
2.  Confirm it has `@SpringBootApplication`.
3.  Run it from the IDE.
4.  Confirm startup succeeds and Tomcat listens on port 8080.

Open `http://localhost:8080`. A 404 at `/` is normal until you create a
homepage route or static file.

**Checkpoint:** Application starts without errors.

## PHASE 2 --- Movie catalogue CRUD

### Create Movie entity

Suggested fields:

  Field             Type
  ----------------- ------------------
  id                Long, generated
  title             String, required
  description       Text
  durationMinutes   Integer
  language          String
  genre             String
  certification     String
  releaseDate       LocalDate
  posterUrl         String
  active            Boolean

Create: 1. `Movie` entity with JPA annotations and validation. 2.
`MovieRepository extends JpaRepository<Movie, Long>`. 3. `MovieService`
with add, list active, find by ID, update, deactivate, and search
methods. 4. `MovieController` with REST mappings.

Endpoints:

  Method   URL                              Purpose
  -------- -------------------------------- --------------------
  POST     `/api/movies`                    Add movie (admin)
  GET      `/api/movies`                    List active movies
  GET      `/api/movies/{id}`               Movie details
  PUT      `/api/movies/{id}`               Update (admin)
  PATCH    `/api/movies/{id}/deactivate`    Deactivate (admin)
  GET      `/api/movies/search?title=leo`   Search

Test POST, GET, GET by ID, PUT, and search in Postman.

**Checkpoint:** Movie CRUD works.

## PHASE 3 --- City, theatre, screen, and physical seats

### City

Fields: `id`, `name`, `state`, `active`.

Endpoints: `POST /api/admin/cities`, `GET /api/cities`,
`PUT /api/admin/cities/{id}`.

### Theatre

Fields: `id`, `name`, `address`, city relationship, `active`.

A theatre belongs to one city; a city has many theatres.

Endpoints: `POST /api/admin/theatres`, `GET /api/theatres?cityId=1`,
`GET /api/theatres/{id}`, `PUT /api/admin/theatres/{id}`.

### Screen

Fields: `id`, `screenName`, theatre relationship, `totalSeats`,
`active`.

A theatre has many screens; a screen belongs to one theatre.

Endpoints: `POST /api/admin/theatres/{theatreId}/screens`,
`GET /api/theatres/{theatreId}/screens`,
`PUT /api/admin/screens/{screenId}`.

### Seat

Fields: `id`, screen relationship, `rowLabel` (A/B/C), `seatNumber`,
`seatType` (REGULAR/PREMIUM/RECLINER), `active`.

Add a unique constraint on `(screen_id, row_label, seat_number)`.

Example seat layout:

``` text
       1   2   3   4   5   6
A      A1  A2  A3  A4  A5  A6
B      B1  B2  B3  B4  B5  B6
C      C1  C2  C3  C4  C5  C6
```

Admin endpoint: `POST /api/admin/screens/{screenId}/seats`.

Configure physical seats once; do not recreate them for every show.

**Checkpoint:** Admin can create a city, theatre, screen, and seat
layout.

## PHASE 4 --- Showtimes and inventory

Create `Show` entity with: - id - movie (many-to-one) - screen
(many-to-one) - `startTime` and `endTime` (`LocalDateTime`) -
`basePrice` (`BigDecimal`) - status (`SCHEDULED`, `CANCELLED`,
`COMPLETED`)

Create repository and service methods to create/list/find/cancel shows.
Reject overlapping shows on the same screen. Consider a cleaning/buffer
interval.

Endpoints: - `POST /api/admin/shows` -
`GET /api/shows?movieId=1&date=2026-11-20&cityId=1` -
`GET /api/shows/{id}` - `PATCH /api/admin/shows/{id}/cancel`

When creating a show, create one `show_seat_inventory` row per active
seat on that screen, all initially `AVAILABLE`. Do this in a transaction
and enforce unique `(show_id, seat_id)`.

**Checkpoint:** Showtimes appear and each show has its own seat
inventory.

## PHASE 5 --- Registration, login, and roles

Create `User` entity with: - id - fullName - email (unique) -
passwordHash - phone (optional) - role (`CUSTOMER` or `ADMIN`) -
active - createdAt

Registration endpoint: `POST /api/auth/register`.

Example request:

``` json
{
  "fullName": "Arun Kumar",
  "email": "arun@example.com",
  "password": "ChooseAStrongPassword123"
}
```

Validate fields, reject duplicate emails, hash passwords with BCrypt,
and save the user. Never store plain-text passwords.

Add Spring Security: - Public: registration, movie list/details, show
search. - Customer: booking and own booking history. - Admin: movie,
theatre, screen, seat, and show management.

For a beginner, server-side session authentication is a good first
approach. JWT can be an advanced extension. Do not trust a role or user
ID supplied by the browser.

Endpoints: `POST /api/auth/register`, `POST /api/auth/login`,
`POST /api/auth/logout`, `GET /api/auth/me`.

**Checkpoint:** Login works and admin routes are protected.

## PHASE 6 --- Seat layout API and frontend

Create `GET /api/shows/{showId}/seats`.

Return each seat's `seatId`, `rowLabel`, `seatNumber`, `seatType`,
price, and current status.

Example:

``` json
[
  {"seatId":1,"rowLabel":"A","seatNumber":1,"seatType":"REGULAR","price":150.00,"status":"AVAILABLE"},
  {"seatId":2,"rowLabel":"A","seatNumber":2,"seatType":"REGULAR","price":150.00,"status":"BOOKED"}
]
```

JavaScript should: 1. Fetch seat data. 2. Group seats by row. 3. Render
selectable seats. 4. Disable booked or unavailable seats. 5. Highlight
selected seats. 6. Show a preliminary total. 7. Send selected seat IDs
to the backend.

The backend recalculates the final price using trusted database values.

**Checkpoint:** Customer can see and select available seats.

## PHASE 7 --- Temporary seat holds (critical)

Two customers may select the same seat at almost the same time. The
backend must atomically hold seats; disabling a browser button is not
enough.

Create `POST /api/bookings/hold`.

Request:

``` json
{
  "showId": 12,
  "seatIds": [1, 2]
}
```

Identify the user from the authenticated security context, not from a
trusted user ID in the request.

Service validation: 1. Show exists, is active, and has not started. 2.
At least one seat is requested. 3. No duplicate seat IDs in the request.
4. Every seat belongs to the screen for that show. 5. No requested seat
is booked. 6. A seat held by another user with an unexpired hold is
rejected. 7. All requested seats are held together. 8. Holding user and
expiry time are stored. 9. A hold reference and expiry are returned.

Use a configurable short hold duration, for example five minutes.

### Concurrency protection

A simple `find available → update` without locking can let two requests
claim the same seat. Use a database-safe approach: - Pessimistic row
locking with a repository query using
`@Lock(LockModeType.PESSIMISTIC_WRITE)`, or - An atomic conditional
update that succeeds only when status is `AVAILABLE` or the prior hold
has expired.

Wrap the multi-seat operation in a transaction. If any requested seat
cannot be held, roll back the entire operation. Return a conflict such
as `409 Conflict`.

Add a scheduled cleanup task for expired holds, but also check expiry
during booking/payment requests. Correctness must not depend only on the
scheduled task.

**Checkpoint:** Two users cannot successfully hold the same show-seat
simultaneously.

## PHASE 8 --- Booking creation

Create `Booking` entity: - id - unique `bookingReference` - user -
show - server-calculated `totalAmount` - booking status - `createdAt` -
`expiresAt`

Create `BookingSeat` entity: - id - booking - seat -
`ticketPriceAtBooking`

Store the price at booking time so later price changes do not alter old
receipts.

Create `POST /api/bookings/confirm`. It should identify a valid active
hold, not accept a trusted user ID or total price from the browser.

Service flow: 1. Load authenticated user. 2. Load hold and verify it
belongs to the user. 3. Verify hold has not expired. 4. Verify all seats
belong to the show. 5. Recalculate total from server-side prices. 6.
Create booking in `PENDING_PAYMENT`. 7. Create booking-seat rows. 8.
Return booking reference and amount due.

Do not mark seats permanently `BOOKED` until simulated payment succeeds.

Other endpoints: - `GET /api/bookings/my` -
`GET /api/bookings/{reference}` (owner/admin only) -
`POST /api/bookings/{reference}/cancel` (owner only)

**Checkpoint:** A valid hold creates a pending booking.

## PHASE 9 --- Simulated payment

Create `Payment` entity with: - id - booking - paymentReference -
amount - method label (`DEMO_CARD`, `DEMO_UPI`) - status - createdAt -
completedAt

Do not store actual card data, CVVs, UPI PINs, or banking credentials.

For the demo, allow a success/failure simulation: - Success → payment
status `SUCCESS`. - Failure → payment status `FAILED`.

### Success transaction

1.  Verify booking belongs to current user.
2.  Verify status is `PENDING_PAYMENT`.
3.  Verify payment deadline has not passed.
4.  Verify held seats still belong to this user/booking.
5.  Record payment reference and success.
6.  Mark booking `CONFIRMED`.
7.  Mark corresponding inventory rows `BOOKED`.
8.  Clear hold expiry/owner fields.
9.  Return confirmation.

### Failure transaction

1.  Record failed payment.
2.  Mark booking `PAYMENT_FAILED`.
3.  Release its held seats to `AVAILABLE`.
4.  Clear hold owner and expiry.
5.  Return failure response.

Prevent a repeated request from confirming an already confirmed booking
twice. Real gateways require provider idempotency and verified payment
callbacks/webhooks.

**Checkpoint:** Success confirms seats; failure releases them.

## PHASE 10 --- Ticket, history, and cancellation

`GET /api/bookings/my` returns only the authenticated customer's
bookings: - booking reference - movie - theatre/screen - show
date/time - seat labels - total - status

`GET /api/bookings/{reference}` must verify owner/admin access.

Ticket confirmation page displays booking reference, movie, theatre,
screen, date/time, seats, ticket count, amount, and status.

Optional later: QR code or PDF ticket. QR code should contain a
non-sensitive verification URL/token, not personal or payment
credentials.

Define a cancellation policy (for example, cancellation allowed until a
configured number of hours before the show). Cancellation must verify
ownership, confirmed status, and deadline; then update booking and seat
inventory consistently. A real refund requires the payment provider;
changing a database status does not return money.

**Checkpoint:** Customers can see tickets and cancel eligible bookings.

------------------------------------------------------------------------

# 11. Frontend Website Structure

Put frontend files in `src/main/resources/static/`. Spring Boot serves
them from the same application, so use relative API paths.

``` text
static/
├── index.html
├── movies.html
├── movie-details.html
├── shows.html
├── seats.html
├── checkout.html
├── booking-success.html
├── my-bookings.html
├── login.html
├── register.html
├── admin/
│   ├── dashboard.html
│   ├── movies.html
│   ├── theatres.html
│   ├── screens.html
│   └── shows.html
├── css/
│   ├── style.css
│   ├── auth.css
│   ├── seats.css
│   └── admin.css
└── js/
    ├── api.js
    ├── auth.js
    ├── movies.js
    ├── shows.js
    ├── seats.js
    ├── checkout.js
    ├── bookings.js
    └── admin.js
```

  Page                            Purpose
  ------------------------------- ------------------------------
  `index.html`                    Home and featured movies
  `movies.html`                   Movie listing and filters
  `movie-details.html`            Movie details
  `shows.html`                    Theatre/date/show selection
  `seats.html`                    Seat map
  `checkout.html`                 Booking summary/demo payment
  `booking-success.html`          Confirmed ticket
  `my-bookings.html`              Booking history
  `login.html`, `register.html`   Authentication
  `admin/*`                       Admin pages

Create `js/api.js` to centralize fetch requests, JSON parsing, and error
handling. Use:

``` javascript
const API_BASE = "/api";
```

For session authentication, send cookies where required. For JWT, attach
the token according to the security design. Do not put passwords or
secrets in public source code.

Seat legend: Available, Selected, Held/unavailable, Booked. The server
recalculates the price.

------------------------------------------------------------------------

# 12. API Endpoint Summary

## Authentication

  Method   Endpoint               Access
  -------- ---------------------- ---------------
  POST     `/api/auth/register`   Public
  POST     `/api/auth/login`      Public
  POST     `/api/auth/logout`     Authenticated
  GET      `/api/auth/me`         Authenticated

## Movies

  Method   Endpoint                        Access
  -------- ------------------------------- --------
  GET      `/api/movies`                   Public
  GET      `/api/movies/{id}`              Public
  GET      `/api/movies/search`            Public
  POST     `/api/movies`                   Admin
  PUT      `/api/movies/{id}`              Admin
  PATCH    `/api/movies/{id}/deactivate`   Admin

## Cities and theatres

  Method   Endpoint                             Access
  -------- ------------------------------------ --------
  GET      `/api/cities`                        Public
  GET      `/api/theatres?cityId=1`             Public
  GET      `/api/theatres/{id}`                 Public
  POST     `/api/admin/cities`                  Admin
  POST     `/api/admin/theatres`                Admin
  POST     `/api/admin/theatres/{id}/screens`   Admin
  POST     `/api/admin/screens/{id}/seats`      Admin

## Shows and seats

  Method   Endpoint                         Access
  -------- -------------------------------- --------
  GET      `/api/shows`                     Public
  GET      `/api/shows/{id}`                Public
  GET      `/api/shows/{id}/seats`          Public
  POST     `/api/admin/shows`               Admin
  PATCH    `/api/admin/shows/{id}/cancel`   Admin

## Bookings and payments

  Method   Endpoint                             Access
  -------- ------------------------------------ -------------
  POST     `/api/bookings/hold`                 Customer
  POST     `/api/bookings/confirm`              Customer
  GET      `/api/bookings/my`                   Customer
  GET      `/api/bookings/{reference}`          Owner/Admin
  POST     `/api/bookings/{reference}/cancel`   Owner
  POST     `/api/payments/demo`                 Customer

------------------------------------------------------------------------

# 13. Testing Plan (Postman First)

## Catalogue

-   [ ] Add movie as admin.
-   [ ] List active movies.
-   [ ] View one movie.
-   [ ] Search by title.
-   [ ] Update movie.
-   [ ] Verify customer cannot call admin endpoints.

## Theatre and show setup

-   [ ] Create city.
-   [ ] Create theatre.
-   [ ] Create screen.
-   [ ] Add seats.
-   [ ] Create show.
-   [ ] Verify show-seat inventory is generated.
-   [ ] Try overlapping show on same screen; verify rejection.

## Booking

-   [ ] Register two customers.
-   [ ] Log in.
-   [ ] List shows and seats.
-   [ ] Hold one and multiple seats.
-   [ ] Try holding another user's held seat.
-   [ ] Try holding booked seat.
-   [ ] Try a seat from another screen.
-   [ ] Try confirming an expired hold.

## Payment and cancellation

-   [ ] Simulate successful payment; booking becomes `CONFIRMED`.
-   [ ] Verify inventory becomes `BOOKED`.
-   [ ] Simulate failed payment; seats return to `AVAILABLE`.
-   [ ] Retry payment for confirmed booking; duplicate confirmation is
    rejected.
-   [ ] View booking history.
-   [ ] Try viewing another user's booking; access is denied.
-   [ ] Cancel eligible booking.
-   [ ] Try cancellation after the deadline.

## Concurrency test

1.  Use two customer accounts/Postman sessions.
2.  Both view the same show.
3.  Both try to hold the same seat at nearly the same time.
4.  Only one succeeds.
5.  The other receives `409 Conflict`.
6.  Verify only one active hold/booked state exists for that show-seat
    pair.

------------------------------------------------------------------------

# 14. HTTP Status Codes and Error Handling

  Status   Meaning                 Example
  -------- ----------------------- -----------------------------------------------
  200      OK                      Movie/booking details returned
  201      Created                 Movie or booking created
  204      No Content              Successful deletion/cancellation with no body
  400      Bad Request             Invalid input
  401      Unauthorized            Not logged in
  403      Forbidden               Customer attempts admin action
  404      Not Found               Movie/show/booking missing
  409      Conflict                Seat unavailable or duplicate attempt
  500      Internal Server Error   Unexpected server issue

Add `@RestControllerAdvice` and `@ExceptionHandler` for consistent error
responses. Do not expose stack traces, SQL credentials, password hashes,
or internal exception details to customers.

------------------------------------------------------------------------

# 15. Security Checklist

-   [ ] Hash passwords with BCrypt.
-   [ ] Never store plain-text passwords.
-   [ ] Never commit database passwords or API secrets.
-   [ ] Protect admin endpoints with role-based authorization.
-   [ ] Derive current user from the authenticated session/token.
-   [ ] Do not trust user ID, booking owner, seat price, or total sent
    by browser.
-   [ ] Validate request fields on backend.
-   [ ] Verify booking ownership before viewing/cancelling.
-   [ ] Use transactions and locking/conditional updates for seat holds.
-   [ ] Expire abandoned holds.
-   [ ] Avoid logging passwords/tokens/payment credentials.
-   [ ] Use HTTPS in deployment.
-   [ ] Configure CORS narrowly if origins differ.
-   [ ] Configure CSRF protection appropriately for session-based
    browser authentication.
-   [ ] Store production secrets in hosting environment variables.

------------------------------------------------------------------------

# 16. Common Errors and Fixes

  ------------------------------------------------------------------------------------
  Error                            Possible reason         Check
  -------------------------------- ----------------------- ---------------------------
  `Communications link failure`    MySQL stopped/wrong URL MySQL service, host, port
                                                           3306

  `Access denied for user`         Wrong credentials       Username/password and
                                                           permissions

  `Unknown database`               Database missing        Create
                                                           `movie_ticket_booking_db`

  `Port 8080 was already in use`   Another app uses port   Stop it or change local
                                                           port

  `Not a managed type`             Entity scan issue       `@Entity`, imports, root
                                                           package

  `No qualifying bean`             Component not           Package structure and
                                   discovered              annotations

  `404 Not Found`                  Wrong route or ID       URL, HTTP method, mapping

  `400 Bad Request`                Invalid JSON/validation Request body and DTO

  `401 Unauthorized`               Missing/expired login   Session/token config

  `403 Forbidden`                  Insufficient role       Security rules

  `409 Conflict`                   Seat unavailable        Inventory status and
                                                           transaction

  Duplicate key                    Unique value exists     Email, seat coordinate,
                                                           show-seat pair

  Seat looked available but hold   Another customer got it Refresh; backend check is
  fails                            first                   authoritative

  Frontend API fails               Wrong path/server error Browser Network tab and
                                                           backend logs
  ------------------------------------------------------------------------------------

Read the first meaningful `Caused by` section in the backend console.
Fix one error at a time.

------------------------------------------------------------------------

# 17. Git and GitHub

Create repository: `spring-boot-movie-ticket-booking`.

From the folder containing `pom.xml`:

``` bash
git init
git add .
git commit -m "Initial movie ticket booking project"
git branch -M main
git remote add origin YOUR_GITHUB_REPOSITORY_URL
git push -u origin main
```

Replace the URL with your actual repository URL.

Suggested `.gitignore`:

``` gitignore
target/
.idea/
.vscode/
*.iml
.env
application-local.properties
```

Keep Maven Wrapper files generated by Initializr. Never upload passwords
or API secrets.

README sections: 1. Project overview and features. 2. Technology stack.
3. Architecture and database diagrams. 4. Screenshots. 5. API endpoints.
6. Setup instructions. 7. Environment variables. 8. Testing steps. 9.
Future enhancements.

------------------------------------------------------------------------

# 18. Deployment Flow

Deploy only after local APIs, frontend, and database operations work.

## Prepare and build

-   [ ] Test the app locally.
-   [ ] Test APIs and frontend.
-   [ ] Check relationships and booking flows.
-   [ ] Move secrets out of source code.
-   [ ] Ensure app uses the host-provided `PORT`.

On Windows, in the folder containing `pom.xml`:

``` powershell
.\mvnw.cmd clean package
```

The JAR is created in `target/`. Run locally (the exact filename depends
on project version):

``` powershell
java -jar target\movie-ticket-booking-0.0.1-SNAPSHOT.jar
```

## Deploy

1.  Create a hosting project.
2.  Connect GitHub repository.
3.  Add a MySQL database service or compatible managed MySQL database.
4.  Add DB variables to the Spring Boot application service.
5.  Set build/start commands if not auto-detected.
6.  Deploy and inspect logs.
7.  Generate a public domain.
8.  Test registration, movies, shows, seat holds, and demo booking.

This guide expects environment variable names:

``` text
DB_URL
DB_USERNAME
DB_PASSWORD
PORT
```

Use the exact host, port, database, username, and password supplied by
the platform. Variable-reference syntax differs between providers.

For production, do not rely on `ddl-auto=update`. Use Flyway or
Liquibase, backups, restricted DB access, HTTPS, health checks, and
concurrency tests. Review hosting charges and free-tier limits.

------------------------------------------------------------------------

# 19. Suggested Development Schedule

  Stage   Work                            Completion
  ------- ------------------------------- -----------------------------
  1       Project setup and MySQL         App starts
  2       Movie CRUD                      Catalogue works
  3       City/theatre/screen/seat CRUD   Theatre structure works
  4       Shows and inventory             Showtimes/seats exist
  5       Registration/login/roles        User access works
  6       Seat map API/frontend           Seats visible
  7       Temporary seat holds            Conflict prevention works
  8       Booking creation                Pending booking exists
  9       Simulated payment               Confirmation works
  10      History/ticket/cancellation     Customer flow complete
  11      Admin frontend                  Admin can manage data
  12      Testing/GitHub/deployment       Project demonstrable online

Spend extra time on seat inventory and concurrency; they are central to
ticket booking.

------------------------------------------------------------------------

# 20. Advanced Features --- Only After Core Booking Works

1.  Movie poster upload using cloud/object storage.
2.  Ratings and reviews.
3.  Search by language, genre, and certification.
4.  Dynamic seat-category pricing.
5.  Coupons and promotional codes.
6.  Email booking confirmation.
7.  QR-code ticket verification.
8.  PDF ticket download.
9.  Real payment gateway using hosted checkout.
10. Refund integration.
11. Admin sales/occupancy dashboard.
12. Pagination and sorting.
13. Redis caching where appropriate.
14. Scheduled cleanup of expired holds.
15. Admin audit logs.
16. Automated integration tests.
17. Docker and CI/CD.

Do not start with microservices. Build a correct modular monolith and
reliable booking transaction first.

------------------------------------------------------------------------

# 21. Final Project Checklist

## Setup

-   [ ] JDK 17 and IDE installed.
-   [ ] MySQL Server running.
-   [ ] Database created.
-   [ ] Spring Boot project generated.
-   [ ] Maven dependencies downloaded.
-   [ ] Properties configured without committed secrets.
-   [ ] Main application starts.

## Backend

-   [ ] Movie entity/repository/service/controller.
-   [ ] City, theatre, screen, and seat entities/APIs.
-   [ ] Show entity and APIs.
-   [ ] Show-seat inventory generated for every show.
-   [ ] User registration/login and roles.
-   [ ] Transactional seat hold and expiry.
-   [ ] Booking and booking-seat records.
-   [ ] Simulated payment.
-   [ ] Booking confirmation/history/cancellation.
-   [ ] Global exception handling.

## Frontend

-   [ ] Home and movie listing.
-   [ ] Movie details.
-   [ ] Theatre and showtime selection.
-   [ ] Seat layout.
-   [ ] Checkout and demo payment.
-   [ ] Booking confirmation.
-   [ ] Login and registration.
-   [ ] My bookings.
-   [ ] Admin pages.

## Testing and delivery

-   [ ] APIs tested in Postman.
-   [ ] Frontend connected to APIs.
-   [ ] Invalid inputs tested.
-   [ ] Customer/admin permissions tested.
-   [ ] Two-user same-seat concurrency test passed.
-   [ ] Payment success/failure tested.
-   [ ] Expired holds tested.
-   [ ] Cancellation tested.
-   [ ] MySQL records verified.
-   [ ] GitHub repository and README completed.
-   [ ] Deployment and public URL tested.

------------------------------------------------------------------------

# 22. First Coding Session --- Start Here

Do not create everything at once. First session tasks:

1.  Generate the Spring Boot project.
2.  Add Spring Web, Spring Data JPA, MySQL Driver, Validation, and
    DevTools.
3.  Create the MySQL database.
4.  Configure `application.properties`.
5.  Run the empty application.
6.  Create `Movie` entity.
7.  Create `MovieRepository`.
8.  Create `MovieService`.
9.  Create `MovieController`.
10. Test `POST /api/movies` and `GET /api/movies` in Postman.

After movie CRUD works, continue with city/theatre, then screen/seats,
then showtimes. Implement authentication before customer booking. Do not
begin booking until show-specific seat inventory works.

------------------------------------------------------------------------

# Official References

-   Spring Boot: https://spring.io/projects/spring-boot/
-   Spring Boot requirements:
    https://docs.spring.io/spring-boot/system-requirements.html
-   Spring Boot 4.1 docs: https://docs.spring.io/spring-boot/4.1/
-   Spring Initializr: https://start.spring.io/
-   Spring Data JPA: https://docs.spring.io/spring-data/jpa/reference/
-   Spring Security: https://docs.spring.io/spring-security/reference/
-   Hibernate ORM: https://hibernate.org/orm/documentation/
-   MySQL: https://dev.mysql.com/doc/
-   Postman: https://learning.postman.com/

**Project goal:** Build a complete movie ticket booking website with
Spring Boot and MySQL, progressing from CRUD APIs to secure,
concurrency-safe seat booking and deployment.
