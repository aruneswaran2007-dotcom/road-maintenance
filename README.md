# Smart Municipal Road Maintenance and Resource Allocation

A full-stack, enterprise-grade web application developed with **Java (Spring Boot)**, **HTML5/CSS3/JavaScript (Bootstrap 5)**, and **MySQL**.

The system enables municipal corporations to collect road damage complaints from citizens, automatically prioritize repairs using a transparent mathematical scoring algorithm, dispatch work orders to authorized maintenance crews, and track field repairs to verified completion.

---

## 1. System Highlights & Features

- **Three Dedicated Portals**:
  1. **Public Citizen Portal**: Register, report road hazards with Leaflet GPS map picking, upload damage photos, monitor live repair progress, and exchange support queries with municipal engineers.
  2. **Municipal Officer Portal**: Executive command dashboard, review and filter complaints, transparent priority score breakdown, dispatch work orders to active crews, manage maintenance team accounts (strictly enforcing a maximum 3 active teams capacity), and respond to public inquiries.
  3. **Maintenance Crew Portal**: Field dashboard displaying assigned work orders, accept jobs into `IN_PROGRESS`, record repair field notes, and submit completed work with verified before-and-after photographic evidence.
- **Transparent Road Repair Priority Algorithm**:
  $$\text{Priority Score} = (\text{Damage Severity} \times 0.40) + (\text{Accident-Risk Rank} \times 0.30) + (\text{Traffic Rank} \times 0.20) + (\text{Road-Importance Rank} \times 0.10)$$
  - **High Priority**: Score $\ge 2.50$ (Urgent hazard, crater potholes, high-risk arterial roads)
  - **Medium Priority**: Score $1.75 \le \text{Score} < 2.50$ (Moderate pavement wear, secondary streets)
  - **Low Priority**: Score $< 1.75$ (Light surface raveling, minor residential alleys)
- **Role-Based Security & Audit History**:
  - Encrypted passwords with **BCrypt**.
  - Secure session token authorization.
  - Full audit logging in `task_status_history` preserving every transition and remark.
  - Active crew capacity enforcement (strictly $\le 3$ active crews).

---

## 2. Technology Stack

- **Backend**: Java 21, Spring Boot 3.3.4
- **ORM & Database**: Spring Data JPA, Hibernate, MySQL 8.0
- **Security**: Spring Security with role-based access control (RBAC) and BCrypt password hashing
- **Build Tool**: Apache Maven 3.9+
- **Frontend**: HTML5, Vanilla CSS3 Custom Design System, JavaScript (ES6+), Bootstrap 5.3, Bootstrap Icons, Leaflet.js

---

## 3. Database Architecture (MySQL 8)

The application utilizes a normalized schema with 8 relational tables:

1. `public_users`: Registered citizen profiles (`id`, `name`, `username`, `email`, `phone`, `password_hash`, timestamps).
2. `municipal_officer`: Single authorized municipal officer account (`id`, `name`, `username`, `email`, `phone`, `password_hash`, timestamps).
3. `maintenance_teams`: Crew accounts (`id`, `team_name`, `username`, `password_hash`, `contact_number`, `is_active`, `specialization`, timestamps).
4. `road_complaints`: Road damage complaints (`id`, `complaint_code`, `public_user_id`, `location_name`, `latitude`, `longitude`, `damage_description`, `damage_severity`, `accident_risk_rank`, `traffic_rank`, `road_importance_rank`, `priority_score`, `priority_category`, `priority_explanation`, `image_path`, `status`, `officer_remarks`, timestamps).
5. `maintenance_tasks`: Work orders assigned to crews (`id`, `task_code`, `complaint_id`, `assigned_team_id`, `task_description`, `repair_instructions`, `assigned_by_officer_id`, `assigned_date`, `due_date`, `status`, `progress_notes`, `completion_notes`, `completion_date`, `before_image_path`, `after_image_path`, timestamps).
6. `task_status_history`: Audit trail tracking every state change, timestamp, and responsible user.
7. `support_requests`: Citizen inquiries submitted to the municipal desk (`id`, `ticket_code`, `public_user_id`, `category`, `subject`, `message`, `status`, timestamps).
8. `support_replies`: Conversation replies between citizens and municipal officers (`id`, `support_request_id`, `sender_role`, `sender_name`, `message`, `created_at`).

---

## 4. Default Seeded Accounts & Credentials

The system seeds the initial accounts automatically on first startup via `DataInitializer`:

| Portal | Role | Username | Password | Additional Details |
| :--- | :--- | :--- | :--- | :--- |
| **Municipal Officer** | `ROLE_OFFICER` | `officer` | `Officer@123` | Provisioned setup account (Chief Road Engineer) |
| **Crew Alpha** | `ROLE_TEAM` | `team_alpha` | `TeamAlpha@123` | **Team Name**: `Rapid Response Team Alpha`<br>*Specialization*: Emergency Potholes |
| **Crew Beta** | `ROLE_TEAM` | `team_beta` | `TeamBeta@123` | **Team Name**: `Pothole Patch Squad Beta`<br>*Specialization*: Asphalt Patching |
| **Crew Gamma** | `ROLE_TEAM` | `team_gamma` | `TeamGamma@123` | **Team Name**: `Heavy Asphalt Crew Gamma`<br>*Specialization*: Road Resurfacing |
| **Demo Citizen** | `ROLE_PUBLIC` | `citizen_john` | `Citizen@123` | `John Resident` (Public Citizen) |

*Citizens can also register their own new accounts anytime directly on `/login.html`.*

---

## 5. Local Setup & Execution Guide

### Prerequisites
1. **Java JDK 21** or later installed (`java -version`).
2. **Apache Maven 3.8+** installed (`mvn -version`).
3. **MySQL Server 8.0** running locally on port 3306 (`MySQL80` service).

### Step 1: Prepare MySQL Database
Open your MySQL client or command prompt and run:
```sql
CREATE DATABASE IF NOT EXISTS smart_road_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
*(The schema DDL is in `src/main/resources/schema.sql` and Spring Data JPA automatically synchronizes tables upon startup).*

### Step 2: Configure Database Credentials
Verify or update credentials in `src/main/resources/application.properties`:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/smart_road_db?createDatabaseIfNotExist=true&useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=root
spring.datasource.password=root
```
*(Environment variables `DB_USERNAME` and `DB_PASSWORD` can also be supplied if preferred).*

### Step 3: Build & Run the Spring Boot Application
From the project root (`Road maintenance/`):
```bash
mvn clean compile
mvn spring-boot:run
```

### Step 4: Access the Application
Open your web browser and navigate to:
```
http://localhost:8080/
```

- **Home Page / Overview**: `http://localhost:8080/`
- **Login Portal (All Roles)**: `http://localhost:8080/login.html`
- **Public Citizen Portal**: `http://localhost:8080/public-dashboard.html`
- **Municipal Officer Portal**: `http://localhost:8080/officer-dashboard.html`
- **Maintenance Crew Portal**: `http://localhost:8080/team-dashboard.html`

---

## 6. End-to-End Workflow Demonstration

1. **Citizen Reports Damage**:
   - Log in as `citizen_john` (or register a new user).
   - Click **Report Road Damage**, drop a pin on the interactive Leaflet map to capture latitude and longitude, select severity and hazard ranks, optionally attach a photo, and submit.
   - The system automatically calculates and displays the transparent **Priority Score**.
2. **Municipal Officer Approves & Dispatches**:
   - Log in as `officer` / `Officer@123`.
   - View the prioritized complaint queue sorted by score.
   - Click **Review**, inspect the location on the map, review photographic evidence, set status to **Approved**, and click **Assign to Maintenance Crew**.
   - Select an active crew (viewing each crew's current active task workload) and set a due date.
3. **Maintenance Crew Executes Repair**:
   - Log in as `team_alpha` (Team Name: `Rapid Response Team Alpha`).
   - Click **Accept & Start** to mark order as `IN_PROGRESS` and log field notes.
   - Once completed, click **Complete**, record the repair work performed, upload before/after photos, and submit.
   - The associated road complaint is automatically transitioned to `COMPLETED`.
4. **Citizen Tracks Resolution**:
   - The reporting citizen refreshes their dashboard to view the completion status, assigned crew, and resolved timestamp.
