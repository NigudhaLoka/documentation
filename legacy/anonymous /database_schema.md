# Legacy "Anonymous" Application - Database Schema

This document outlines the MySQL database schema used by the legacy "Anonymous" application. The information is based on the provided SQL dump files.

## Table: `anonymous_users`

Stores user account information.

| Column            | Data Type     | Constraints                                                | Notes                                     |
|-------------------|---------------|------------------------------------------------------------|-------------------------------------------|
| `uid`             | `bigint(20)`  | PRIMARY KEY, NOT NULL, AUTO_INCREMENT                      | Unique user identifier                    |
| `username`        | `varchar(64)` | UNIQUE KEY, NOT NULL                                       | User's login name                         |
| `email`           | `varchar(128)`| UNIQUE KEY, DEFAULT NULL                                   | User's email address                      |
| `isEmailVerified` | `tinyint(1)`  | NOT NULL, DEFAULT 0                                        | Flag indicating if email is verified (0=No, 1=Yes) |
| `phoneNumber`     | `varchar(16)` | UNIQUE KEY, DEFAULT NULL                                   | User's phone number                       |
| `password`        | `varchar(512)`| NOT NULL                                                   | Hashed password (MD5 in the application)  |
| `displayName`     | `varchar(254)`| NOT NULL                                                   | Public display name for the user          |
| `isAnonymous`     | `tinyint(1)`  | NOT NULL                                                   | Default anonymity preference for the user |
| `timestamp`       | `timestamp`   | NOT NULL, DEFAULT current_timestamp()                      | Record creation/update timestamp          |

**Indexes:**
* PRIMARY KEY (`uid`)
* UNIQUE KEY `username` (`username`)
* UNIQUE KEY `email` (`email`)
* UNIQUE KEY `phoneNumber` (`phoneNumber`)

---

## Table: `anonymous_events`

Stores the "events" or topics/questions created by users.

| Column    | Data Type      | Constraints                                    | Notes                               |
|-----------|----------------|------------------------------------------------|-------------------------------------|
| `id`      | `bigint(20)`   | PRIMARY KEY, NOT NULL, AUTO_INCREMENT          | Unique event identifier             |
| `link`    | `varchar(512)` | UNIQUE KEY, NOT NULL                           | A unique link/slug for the event    |
| `uid`     | `bigint(20)`   | NOT NULL, FOREIGN KEY (`anonymous_users`.`uid`) | Creator's user ID                   |
| `context` | `varchar(1024)`| NOT NULL                                       | The content/text of the event       |
| `disabled`| `tinyint(1)`   | NOT NULL, DEFAULT 0                            | Flag if event is disabled (0=No, 1=Yes) |
| `deleted` | `tinyint(1)`   | NOT NULL, DEFAULT 0                            | Flag if event is soft-deleted (0=No, 1=Yes) |
| `timestamp`| `timestamp`    | NOT NULL, DEFAULT current_timestamp()          | Record creation/update timestamp    |

**Indexes:**
* PRIMARY KEY (`id`)
* UNIQUE KEY `id` (`id`, `uid`)
* UNIQUE KEY `link` (`link`)
* KEY `event uid maps users uid` (`uid`)

**Relationships:**
* `uid` references `anonymous_users`.`uid`.

---

## Table: `anonymous_replies`

Stores replies made to events.

| Column       | Data Type      | Constraints                                             | Notes                                     |
|--------------|----------------|---------------------------------------------------------|-------------------------------------------|
| `id`         | `bigint(20)`   | PRIMARY KEY, NOT NULL, AUTO_INCREMENT                   | Unique reply identifier                   |
| `event_id`   | `varchar(512)` | NOT NULL, FOREIGN KEY (`anonymous_events`.`link`)       | Link of the event this reply belongs to   |
| `uid`        | `bigint(20)`   | DEFAULT NULL, FOREIGN KEY (`anonymous_users`.`uid`)     | User ID of the replier (if not anonymous) |
| `name`       | `varchar(128)` | DEFAULT NULL                                            | Display name for the reply (if anonymous or overridden) |
| `message`    | `varchar(1024)`| NOT NULL                                                | The content/text of the reply           |
| `ip_address` | `varchar(128)` | NOT NULL                                                | IP address of the replier                 |
| `session_id` | `varchar(128)` | NOT NULL                                                | Session ID of the replier                 |
| `anonymous`  | `tinyint(1)`   | NOT NULL, DEFAULT 1                                     | Flag if reply is anonymous (0=No, 1=Yes)  |
| `deleted`    | `tinyint(1)`   | NOT NULL, DEFAULT 0                                     | Flag if reply is soft-deleted (0=No, 1=Yes) |
| `timestamp`  | `timestamp`    | NOT NULL, DEFAULT current_timestamp()                   | Record creation/update timestamp          |

**Indexes:**
* PRIMARY KEY (`id`)
* KEY `replies uid maps users uid` (`uid`)
* KEY `replies event_id maps event id` (`event_id`)

**Relationships:**
* `event_id` references `anonymous_events`.`link`.
* `uid` references `anonymous_users`.`uid`.

---

## Table: `anonymous_captcha`

Stores captcha codes for verification.

| Column     | Data Type    | Constraints                           | Notes                                     |
|------------|--------------|---------------------------------------|-------------------------------------------|
| `id`       | `bigint(20)` | PRIMARY KEY (`id`, `session_id`), NOT NULL, AUTO_INCREMENT | Unique captcha identifier               |
| `session_id`| `varchar(128)`| PRIMARY KEY (`id`, `session_id`), NOT NULL | Session ID associated with the captcha    |
| `code`     | `varchar(6)` | NOT NULL                              | The actual captcha code                   |
| `used`     | `tinyint(1)` | NOT NULL, DEFAULT 0                   | Flag if captcha has been used (0=No, 1=Yes) |
| `timestamp`| `timestamp`  | NOT NULL, DEFAULT current_timestamp() | Record creation timestamp                 |

**Indexes:**
* PRIMARY KEY (`id`, `session_id`)

---

## Table: `anonymous_login_information`

Tracks user login activity and notification counts.

| Column        | Data Type    | Constraints                                    | Notes                             |
|---------------|--------------|------------------------------------------------|-----------------------------------|
| `uid`         | `bigint(11)` | PRIMARY KEY, NOT NULL, FOREIGN KEY (`anonymous_users`.`uid`) | User identifier                   |
| `last_login`  | `timestamp`  | NOT NULL, DEFAULT current_timestamp()          | Timestamp of the last login       |
| `notifications`| `bigint(20)` | NOT NULL, DEFAULT 0                            | Count of notifications for the user |
| `timestamp`   | `timestamp`  | NOT NULL, DEFAULT current_timestamp()          | Record creation/update timestamp  |

**Indexes:**
* PRIMARY KEY (`uid`)

**Relationships:**
* `uid` references `anonymous_users`.`uid`.
