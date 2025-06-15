
# NigūḍhaLoka - Backend API Service

## 1. Overview

This document outlines the API endpoints for the `backend-api` service of the NigūḍhaLoka platform. The `backend-api` is the core **Python/FastAPI** application that serves as the central logic engine for the entire platform. It is responsible for all business logic, data persistence with Firebase, and management of asynchronous tasks.

This service is the internal "Actual API" and is designed to only accept requests from the `gateway-php` layer, which handles public-facing traffic and initial validation like Captcha.

## 2. Technology Stack

* **Framework**: **Python 3** with **FastAPI**.
* **Primary Database**: **Google Firestore** is used for storing user profiles, all discussion content (threads and replies), and push notification subscription tokens.
* **Authentication**: **Firebase Authentication** for user identity and **JSON Web Tokens (JWTs)** for securing API endpoints.
* **File Storage**: **Google Cloud Storage for Firebase** for user-uploaded media and generated shareable images.
* **Asynchronous Tasks**: **Google Cloud Functions** for background processing.
* **Queue & Mail Database**: **MySQL** is used for two specific tasks: as a job queue for image generation requests and to store and manage notification jobs.

## 3. API Endpoint Classification

All requests to these endpoints must be authenticated using a JWT, unless otherwise specified.

### 3.1. Authentication (`/auth`)

*Manages user accounts, sessions, and credentials.*

* **`POST /auth/signup`**: Register a new user.
* **`POST /auth/signin`**: Authenticate a user and issue JWTs.
* **`POST /auth/refresh-token`**: Issue a new access token using a refresh token.
* **`POST /auth/logout`**: Log out a user.
* **`POST /auth/change-password`**: Updates the user's password. This endpoint is used after the `gateway-php` layer has handled the necessary user verification steps.
* **`POST /auth/generate-verification-email`**: Generates the content for a verification email and saves it to the MySQL database. The `gateway-php` service is responsible for actually sending the email.

### 3.2. User Management (`/users`)

*Manages user-specific data.*

* **`GET /users/me`**: Retrieve the profile of the currently authenticated user.
* **`PUT /users/me`**: Update the profile of the currently authenticated user.

### 3.3. Thread Management (`/threads`)

*Handles the core functionality of creating and retrieving discussion threads and their replies. A thread with a `parentId` is a reply.*

* **`POST /threads`**: Create a new thread (either a top-level topic or a reply). This action also triggers the creation of a notification job in the MySQL queue.
* **`GET /threads/topics`**: List all initial threads (topics). Supports pagination and sorting.
* **`GET /threads/{threadId}`**: Retrieve a single thread's data.
* **`GET /threads/{threadId}/thread`**: Retrieve a full discussion with nested replies.
* **`PUT /threads/{threadId}`**: Update a thread's content (creator or moderator only).
* **`DELETE /threads/{threadId}`**: Delete a thread (creator or moderator only).

### 3.4. Social Shareables

*Manages the creation of shareable images from discussion threads for social media.*

* **`POST /threads/{threadId}/create-shareable`**: A user-facing endpoint to request the generation of an image from a specific thread. This action creates a job in the MySQL queue.

### 3.5. Notifications (`/notifications`)

*Handles retrieving a user's notification history. Note: The sending of notifications is handled by the `gateway-php` layer.*

* **`GET /notifications`**: Get the notification history for the authenticated user for display within the application.

## 4. Notification Architecture

To ensure scalability and cost-effectiveness, the notification system is decoupled across multiple services.

1.  **Client (Browser/Mobile)**: The client application is responsible for requesting a push subscription from the end user's device.
2.  **`gateway-php` (Dispatcher & Sender)**:
    * Exposes **`POST /notifications/subscribe`** and **`POST /notifications/unsubscribe`** endpoints.
    * It receives subscription tokens from the client and stores them in the user's profile in **Firestore**.
    * It runs a dedicated **Notification Worker** script that polls the MySQL `notifications` queue, retrieves jobs, and sends the actual push notifications via a service like Firebase Cloud Messaging (FCM).
3.  **`backend-api` (Trigger)**:
    * This service is responsible for business logic that **triggers** a notification.
    * When a relevant event occurs (e.g., a new reply is posted), this API creates a "notification job" in the **MySQL queue**. This job contains the `userId` of the recipient and the notification payload.

## 5. Asynchronous Tasks & Cron Jobs

Several critical processes are handled asynchronously by scheduled workers.

1.  **Shareable Image Worker**: A Cloud Function that checks the MySQL queue for pending image generation jobs, creates the images, and saves them to Cloud Storage.
2.  **Reminder Notification Trigger**: A scheduled cron job within the `backend-api` that identifies inactive users and adds notification jobs for them to the MySQL queue. The legacy application had a similar cron for sending reminder notifications.
3.  **Stale Data Cleanup**: A daily cron job that cleans up old data, such as completed jobs in the MySQL queues. The legacy application also had cron jobs for cleaning up old data like captcha entries and notification subscriptions.

## 6. Future Enhancements

* **Moderation**: A comprehensive suite of moderation tools is planned. This includes endpoints for admins and topic creators to manage content and users. Due to the complexity of handling permissions (admin-only vs. no-auth access for reporting), this functionality has been deferred to a later stage.
* **Automated Social Posting**: The "Social Shareables" feature may be enhanced to automatically post highly-rated threads to social media platforms like Reddit based on community engagement metrics.

## 7. Developer Notes

*This section is for capturing in-progress thoughts, decisions, and tasks. As the project develops, update the notes here. This information will be used as a reference to inform future updates to this README and the overall architecture.*

* *_(Example) Decision 2025-06-16: Decided against using WebSockets for real-time updates in v1 due to complexity. Will rely on client-side polling for now.*
* *_(Example) To-Do: Need to finalize the exact payload structure for the MySQL notification jobs.*
* *_(Example) To-Do: Benchmark Firestore query performance for deeply nested threads.*
