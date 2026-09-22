# Campus Pool
## Student Carpool & Ride-Share Matching Platform

Campus Pool is a medium-sized full-stack web application that connects verified students who have similar commuting routes and compatible travel times. Students can publish routes, discover compatible rides, send or respond to requests, communicate after acceptance, rate ride partners, and report conduct or safety concerns. Administrators can review reports and manage user accounts.

This project is based on the **Campus Pool SRS v1.0 (September 2026)**. The SRS defines route posting, automatic route matching, and ride-request management as the MVP, with chat, ratings, moderation, recurring rides, and notifications as additional priorities.

> **Database note:** The SRS recommends PostgreSQL/PostGIS in its proposed architecture. This implementation uses **Oracle Database 21c XE** with the `oracledb` Node.js driver because the development environment uses Oracle and SQL*Plus. The application layer remains React → Express → database.

---

# 1. Project Overview

### Name
**Campus Pool**

### Type
Responsive full-stack web application.

### Problem
Students often travel from nearby areas to the same campus at similar times but may not know other students with compatible routes. This can increase individual commute cost, traffic congestion, and campus parking demand.

### Solution
Campus Pool provides a trusted student commuter network where users can:

- Register with an institutional email
- Maintain a student profile
- Post a one-time commute
- Find compatible rides
- Compare route proximity and time overlap
- Send and manage ride requests
- Chat after acceptance
- Rate ride partners
- Report users

Admins can moderate users and reports.

---

# 2. Objectives

1. Provide institutional-email-based student registration.
2. Prevent unverified students from using protected ride functions.
3. Let students publish commute routes using approximate pickup areas.
4. Match routes using distance and time compatibility.
5. Allow riders to request seats.
6. Allow route owners to accept or decline requests.
7. Support confirmed-ride communication.
8. Support ratings, reviews, reports, and moderation.
9. Protect sensitive location information.
10. Provide a minimalistic, responsive interface suitable for academic demonstration.

---

# 3. Scope

## Core MVP

- Registration and login
- Demo account verification
- Profile management
- Route creation
- Route editing/cancellation
- Route browsing
- Matching engine
- Match ranking
- Join requests
- Accept/decline/cancel request actions
- Confirmed rides

## Additional implemented features

- Basic in-app chat
- Ratings/reviews API and database
- Reporting
- Admin moderation
- In-app notifications

## Simplified features

The following are intentionally simplified so the project remains easy to run as a medium-sized college project:

- Map functionality uses preset approximate campus zones instead of requiring a paid Maps API.
- Verification uses a development/demo endpoint instead of real email delivery.
- Notifications are in-app instead of SMS/email/push-provider based.
- Chat uses API refresh/polling rather than production WebSockets.
- Geographic matching uses the Haversine formula in application logic instead of PostGIS.

## Future / not production-grade in this build

- Full Google Maps/Mapbox integration
- Redis
- PostGIS
- Background job queues
- Real email/SMS/push delivery
- Full recurring-ride automation
- College SSO
- Native mobile applications
- Multi-campus support
- Fuel/cost splitting
- Carbon-footprint leaderboard
- Campus event calendar integration

---

# 4. User Roles

## Student

Students can register, log in, maintain their profile, post routes, find rides, send join requests, manage requests, chat after acceptance, rate other participants, report users, and view notifications.

## Administrator

Administrators can view platform statistics, inspect users, view reports, and change account status between active, suspended, and banned.

---

# 5. Main Modules

## Authentication

- Registration
- College email validation
- Password hashing with bcrypt
- Login
- JWT authentication
- Demo verification

## Profile

- Name
- College email
- Phone
- Department
- Year
- Default pickup zone
- Approximate location
- Rating summary

## Route Management

- Create route
- Edit open route
- Cancel route
- View own routes
- View public/open routes

A route contains an origin zone, coordinates, destination, date, time, flexibility, seat capacity, and status.

## Matching

The matching engine considers:

- Origin distance
- Time-window overlap
- Combined compatibility score

Default matching radius: **2 km**.

## Requests

Lifecycle:

```text
PENDING
  ├── ACCEPTED
  ├── DECLINED
  └── CANCELLED
```

Duplicate pending requests are prevented.

## Chat

Only members of an accepted ride can access its conversation.

## Ratings

- 1–5 star rating
- Optional review text
- Average rating
- Review count
- Duplicate-review prevention

## Reports

Users can report another user with a reason and optional description. Admins can inspect reports and moderate accounts.

## Notifications

In-app notifications are generated for relevant ride/request events.

## Admin

- User statistics
- Open routes
- Confirmed rides
- Open reports
- User status control
- Report list

---

# 6. User Interface

The UI follows the SRS's clean and trustworthy design direction:

- Minimalistic layout
- One clear primary action per major screen
- Card-based student UI
- Generous whitespace
- Consistent typography
- Subtle borders/shadows
- Rounded corners
- Clear status badges
- Simple icons
- Responsive design
- Approximate location disclosure

### Main screens

Public:

- Login
- Registration

Student:

- Dashboard
- Post a Route
- Find a Ride
- Requests
- Chat
- Profile
- Notifications

Admin:

- Admin Dashboard
- User management
- Report management

---

# 7. Technology Stack

## Frontend

### React
Component-based user interface.

### Vite
Frontend development/build tool.

### Tailwind CSS
Responsive styling and visual consistency.

### React Router
Client-side navigation.

### Axios
HTTP communication with the backend REST API.

### Lucide React
Interface icons.

## Backend

### Node.js
Server-side JavaScript runtime.

### Express
REST API framework.

### bcryptjs
Password hashing and verification.

### JSON Web Token
Authentication token generation and verification.

### oracledb
Node.js driver used to communicate with Oracle Database.

## Database

### Oracle Database 21c XE
Relational database used by Campus Pool.

### SQL*Plus
Command-line administration/query tool used for schema setup, seed data, and database maintenance.

**SQL*Plus is not required to remain open while the web application is running.** Node connects directly to Oracle using `oracledb`.

---

# 8. System Architecture

```text
                    Campus Pool
                         |
              +----------+----------+
              |                     |
           Frontend              Backend
        React + Vite        Node.js + Express
              |                     |
              | Axios               | oracledb
              +---------->----------+
                                    |
                             Oracle Database
                                Oracle XE 21c
                                    ^
                                    |
                                 SQL*Plus
                             (setup/admin tool)
```

The backend is the application layer between React and Oracle. React never connects directly to Oracle.

---

# 9. Database Design

Main entities:

```text
USERS
  |
  +---- ROUTE_POSTS
  |         |
  |         +---- MATCH_SUGGESTIONS
  |
  +---- JOIN_REQUESTS
  |         |
  |         +---- MESSAGES
  |         +---- REVIEWS
  |
  +---- REPORTS
  |
  +---- NOTIFICATIONS
```

## USERS

Stores student and administrator accounts.

Key fields:

- ID
- NAME
- COLLEGE_EMAIL
- PASSWORD_HASH
- PHONE
- DEPARTMENT
- YEAR
- DEFAULT_ZONE
- DEFAULT_LAT
- DEFAULT_LNG
- RATING_AVG
- RATING_COUNT
- ROLE
- IS_VERIFIED
- STATUS
- CREATED_AT

## ROUTE_POSTS

Stores posted commute routes.

Key fields:

- ID
- USER_ID
- ORIGIN_ZONE
- ORIGIN_LAT
- ORIGIN_LNG
- DESTINATION
- ROUTE_DATE
- TIME_START
- TIME_FLEX_MINUTES
- STATUS
- SEATS_AVAILABLE
- CREATED_AT

## MATCH_SUGGESTIONS

Stores route compatibility values:

- Distance
- Time-overlap score
- Combined score
- Generation time

## JOIN_REQUESTS

Stores rider requests and their statuses.

## MESSAGES

Stores chat messages associated with accepted ride requests.

## REVIEWS

Stores ratings and review text.

## REPORTS

Stores conduct/safety reports.

## NOTIFICATIONS

Stores in-app user notifications.

---

# 10. Matching Algorithm

For two route posts, the system calculates:

### Geographic distance

Coordinates are compared using the Haversine formula.

```text
latitude/longitude A
        +
latitude/longitude B
        ↓
Haversine calculation
        ↓
distance in km
```

Routes beyond the 2 km matching radius are excluded in the current build.

### Time compatibility

Each route has:

```text
Start time ± flexibility
```

Example:

```text
A: 08:30 ± 20 min
B: 08:40 ± 15 min
```

The overlapping portion of the two windows is calculated.

### Combined score

The current implementation uses:

```text
60% distance compatibility
40% time compatibility
```

The resulting score is represented approximately from 0–100 and used for ranking.

> This weighting is an implementation choice for the academic build. The SRS requires ranking using proximity and time overlap but does not prescribe this exact 60/40 formula.

---

# 11. Authentication Flow

```text
Registration/Login form
        ↓
Express API
        ↓
Oracle USER lookup
        ↓
bcrypt verification
        ↓
JWT issued
        ↓
Frontend stores token
        ↓
Authorization: Bearer <token>
        ↓
Protected API route
```

Passwords are never stored as plain text.

---

# 12. Privacy and Security

Current controls include:

- bcrypt password hashing
- JWT authentication
- Protected API endpoints
- Admin role authorization
- Parameterized Oracle queries
- Environment-based secrets
- Database constraints
- Duplicate-request prevention
- Approximate location use
- No exact home addresses in public route responses

For production deployment, additional hardening would still be required, including TLS, production secret management, rate limiting, secure token storage, monitoring, backups, and database hardening.

---

# 13. API Overview

## Health

```http
GET /api/health
```

## Authentication

```http
POST /api/auth/register
POST /api/auth/demo-verify
POST /api/auth/login
GET  /api/auth/me
```

## Profile

```http
PUT /api/users/me
```

## Routes

```http
GET    /api/routes
GET    /api/routes/mine
POST   /api/routes
PUT    /api/routes/:id
DELETE /api/routes/:id
```

## Matching

```http
GET /api/matches/:routeId
```

## Requests

```http
POST  /api/requests
GET   /api/requests
PATCH /api/requests/:id
```

## Chat

```http
GET  /api/chat/:id
POST /api/chat/:id
```

## Reviews

```http
POST /api/reviews
```

## Reports

```http
POST /api/reports
```

## Notifications

```http
GET   /api/notifications
PATCH /api/notifications/:id/read
```

## Admin

```http
GET   /api/admin/stats
GET   /api/admin/users
GET   /api/admin/reports
PATCH /api/admin/users/:id/status
```

---

# 14. Project Structure

```text
CampusPool/
|
+-- client/
|   +-- src/
|   |   +-- api.js
|   |   +-- index.css
|   |   +-- main.jsx
|   +-- index.html
|   +-- package.json
|   +-- package-lock.json
|   +-- tailwind.config.js
|   +-- postcss.config.js
|
+-- server/
|   +-- database/
|   |   +-- schema.sql
|   |   +-- seed.sql
|   +-- src/
|   |   +-- auth.js
|   |   +-- db.js
|   |   +-- server.js
|   |   +-- utils.js
|   +-- .env
|   +-- .env.example
|   +-- package.json
|   +-- package-lock.json
|
+-- README.md
+-- .gitignore
```

---

# 15. Oracle Database Setup

## Start Oracle XE

Ensure the Windows service is running:

```cmd
net start OracleServiceXE
```

## Enter SQL*Plus as administrator

```cmd
set ORACLE_SID=XE
sqlplus / as sysdba
```

Switch to the application PDB:

```sql
ALTER SESSION SET CONTAINER = XEPDB1;
```

Create the application schema if needed:

```sql
CREATE USER CAMPUS_POOL IDENTIFIED BY CampusPool123;
```

Grant privileges:

```sql
GRANT CREATE SESSION TO CAMPUS_POOL;
GRANT CREATE TABLE TO CAMPUS_POOL;
GRANT CREATE SEQUENCE TO CAMPUS_POOL;
GRANT CREATE VIEW TO CAMPUS_POOL;
GRANT CREATE PROCEDURE TO CAMPUS_POOL;
GRANT CREATE TRIGGER TO CAMPUS_POOL;
ALTER USER CAMPUS_POOL QUOTA UNLIMITED ON USERS;
```

Connect:

```sql
CONNECT CAMPUS_POOL/CampusPool123@localhost:1521/XEPDB1
```

Create tables using the Oracle schema script:

```sql
@C:\Users\YOUR_USERNAME\Downloads\CampusPool\server\database\schema.sql
```

Verify:

```sql
SELECT table_name FROM user_tables ORDER BY table_name;
```

Expected tables:

```text
JOIN_REQUESTS
MATCH_SUGGESTIONS
MESSAGES
NOTIFICATIONS
REPORTS
REVIEWS
ROUTE_POSTS
USERS
```

---

# 16. Environment Variables

Create:

```text
server/.env
```

Example:

```env
PORT=5000
ORACLE_USER=CAMPUS_POOL
ORACLE_PASSWORD=CampusPool123
ORACLE_CONNECT_STRING=localhost:1521/XEPDB1
JWT_SECRET=replace_with_a_long_random_secret
CLIENT_URL=http://localhost:5173
```

Never commit the real `.env` file.

Share `.env.example` instead.

---

# 17. Installation

## Backend

Open a VS Code terminal:

```powershell
cd C:\Users\YOUR_USERNAME\Downloads\CampusPool\server
npm install
npm run dev
```

Expected:

```text
Campus Pool API running on http://localhost:5000
```

## Frontend

Open a second terminal:

```powershell
cd C:\Users\YOUR_USERNAME\Downloads\CampusPool\client
npm install
npm run dev
```

Open:

```text
http://localhost:5173
```

---

# 18. Health Check

To verify the backend and Oracle database connection, open:

```text
http://localhost:5000/api/health
```

Expected response:

```json
{
  "ok": true,
  "database": 1
}
```

This confirms:

```text
Express
   ↓
oracledb
   ↓
Oracle XE
```

---

# 19. Demo Accounts

Development seed accounts:

```text
Student:
student1@campus.edu
student2@campus.edu
student3@campus.edu

Admin:
admin@campus.edu
```

Demo password:

```text
Password123!
```

Do not reuse this password in production.

---

# 20. Demonstration Flow

A strong college-project demo can follow this sequence:

### Demo A — Student

Login as Student 1.

Show:

- Dashboard
- Profile
- Post a Route

### Demo B — Another Student

Login as Student 2.

Show:

- Find a Ride
- View a compatible route
- Send a request

### Demo C — Route Owner

Return to Student 1.

Show:

- Received request
- Accept request
- Confirmed ride
- Chat

### Demo D — Admin

Login as Admin.

Show:

- Platform statistics
- Users
- Reports
- User moderation

This demonstrates the main end-to-end system workflow.

---

# 21. Common Errors

## `ECONNREFUSED 127.0.0.1:1521`

Oracle listener/database is not available. Check the Oracle XE service and listener.

## `ORA-12560`

The local Oracle instance/SID is not correctly selected or the Oracle service is unavailable.

Try:

```cmd
set ORACLE_SID=XE
sqlplus / as sysdba
```

## `Cannot GET /`

The Express backend does not define a root webpage. Test:

```text
http://localhost:5000/api/health
```

## Frontend cannot connect to backend

Ensure both are running:

```text
Backend  → http://localhost:5000
Frontend → http://localhost:5173
```

## Invalid login

Check that the user exists in Oracle and that the password hash in the seed data matches the password used for login.

---

# 22. Testing Checklist

### Authentication

- [ ] Register user
- [ ] Verify user
- [ ] Login
- [ ] Protected route rejects unauthenticated requests

### Routes

- [ ] Create route
- [ ] View routes
- [ ] Edit open route
- [ ] Cancel route

### Matching

- [ ] Compatible routes appear
- [ ] Distance is calculated
- [ ] Time overlap is calculated
- [ ] Results are ranked
- [ ] Routes outside the matching radius are excluded

### Requests

- [ ] Send request
- [ ] Duplicate pending request blocked
- [ ] Accept request
- [ ] Decline request
- [ ] Cancel request
- [ ] Seat count updates

### Chat

- [ ] Chat blocked before acceptance
- [ ] Chat available after acceptance
- [ ] Messages persist

### Reviews

- [ ] Valid 1–5 rating accepted
- [ ] Duplicate rating blocked
- [ ] Rating average updates

### Moderation

- [ ] Report created
- [ ] Admin sees report
- [ ] Admin can suspend/ban user

### UI

- [ ] Desktop layout
- [ ] Mobile layout
- [ ] Loading states
- [ ] Error states
- [ ] Empty states

---

# 23. SRS Requirement Mapping

| Requirement | Current status |
|---|---|
| FR-1 Registration & Verification | Implemented with development/demo verification |
| FR-2 Profile Management | Implemented |
| FR-3 Post a Commute Route | Implemented |
| FR-4 Route Similarity Matching | Implemented using Haversine + time overlap |
| FR-5 Send Join Request | Implemented |
| FR-6 Accept / Decline Request | Implemented |
| FR-7 Cancel Confirmed Ride | Implemented |
| FR-8 Chat / Contact Exchange | Basic in-app chat implemented |
| FR-9 Rating & Review | API/database implemented |
| FR-10 Reporting & Moderation | Implemented |
| FR-11 Recurring Ride Scheduling | Not implemented as full automation |
| FR-12 Ride Notifications | In-app notification implementation |

The SRS classifies FR-1 through FR-7 as the MVP; FR-8, FR-9, FR-10 and FR-12 are Should-Have items; FR-11 is a Could-Have/stretch item.

---

# 24. Academic Review Questions

### Why React?
React supports reusable components and state-driven UI updates.

### Why Node.js and Express?
Node.js provides the backend runtime and Express simplifies REST API development.

### Why Oracle?
This implementation uses Oracle Database 21c XE to match the available development database environment.

### Why `oracledb`?
It provides the Node.js application with direct connectivity to Oracle Database.

### Why SQL*Plus?
SQL*Plus is used to create the project schema, run SQL scripts, inspect data, and administer Oracle. It is not needed to remain open during normal website use.

### Why bcrypt?
Passwords should not be stored in plain text. bcrypt stores a one-way password hash.

### Why JWT?
JWT provides a stateless mechanism for identifying authenticated users on protected API calls.

### Why Haversine?
It gives a practical straight-line distance between two geographic coordinate points.

### Why foreign keys?
Foreign keys protect relational integrity between users, routes, requests, messages, reviews, and reports.

### Why approximate locations?
The SRS requires protection against exposing exact home addresses to other users.

---

# 25. Future Enhancements

The SRS proposes future functionality including:

- Cost/fuel-split calculator
- Carbon footprint tracking
- Multi-campus/inter-city support
- Native mobile applications
- Background location for smarter matching
- Campus event-calendar integrations
- Full recurring rides
- Real push/email/SMS notifications
- SSO integration
- Full map/geocoding integration
- Production geospatial indexing
- Redis caching and background workers

---

# 26. Project Status

```text
Frontend                    ✅
Backend                     ✅
Oracle connection           ✅
Oracle schema               ✅
Demo users/routes           ✅
Authentication              ✅
Route management            ✅
Matching engine             ✅
Join requests               ✅
Chat                        ✅
Ratings/reviews API         ✅
Reports                     ✅
Notifications               ✅
Admin dashboard             ✅
Responsive UI               ✅
```

This is a **medium-sized academic implementation**, not a production ride-sharing system.

---

# 27. Quick Start

### Start Oracle

```cmd
net start OracleServiceXE
```

### Terminal 1 — Backend

```powershell
cd C:\Users\YOUR_USERNAME\Downloads\CampusPool\server
npm run dev
```

### Terminal 2 — Frontend

```powershell
cd C:\Users\YOUR_USERNAME\Downloads\CampusPool\client
npm run dev
```

### Open

```text
http://localhost:5173
```

### Verify API

```text
http://localhost:5000/api/health
```

---

# 28. SRS Reference

Source basis: **Campus Pool — Software Requirements Specification v1.0, September 2026**.

The SRS defines the platform scope, functional requirements, user classes, interface requirements, non-functional requirements, proposed architecture, data entities, use cases, and UI/UX guidelines used as the basis for this project.
