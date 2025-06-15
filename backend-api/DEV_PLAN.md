# NigūḍhaLoka: Backend-API Development Plan

## Phase 1: Project Foundation & Authentication

-   [ ] Initialize FastAPI project structure.
-   [ ] Configure environment variables (e.g., for database credentials, JWT secrets).
-   [ ] Establish connection to Firebase/Firestore and MySQL.
-   [ ] Implement `/auth` endpoints:
    -   [ ] `POST /auth/signup`
    -   [ ] `POST /auth/signin`
    -   [ ] `POST /auth/refresh-token`
    -   [ ] `POST /auth/logout`
    -   [ ] `POST /auth/change-password`
    -   [ ] `POST /auth/generate-verification-email`

## Phase 2: User Management

-   [ ] Implement `/users` endpoints:
    -   [ ] `GET /users/me`
    -   [ ] `PUT /users/me`

## Phase 3: Core Thread Functionality (MVP)

-   [ ] Implement `/threads` endpoints for core discussion features:
    -   [ ] `POST /threads` (for both topics and replies)
    -   [ ] `GET /threads/topics` (list view)
    -   [ ] `GET /threads/{threadId}` (single thread view)
    -   [ ] `GET /threads/{threadId}/thread` (full nested discussion)
    -   [ ] `PUT /threads/{threadId}` (for updates)
    -   [ ] `DELETE /threads/{threadId}`

## Phase 4: Supporting Features

-   [ ] Implement Social Shareables endpoint:
    -   [ ] `POST /threads/{threadId}/create-shareable` (creates job in MySQL)
-   [ ] Implement Notifications endpoint:
    -   [ ] `GET /notifications` (history for the user)
-   [ ] Implement logic for triggering notification jobs in MySQL on new replies.

## Phase 5: Asynchronous Workers & Cron Jobs

-   [ ] Create Shareable Image Worker (Cloud Function).
-   [ ] Create Reminder Notification Trigger (Cron Job).
-   [ ] Create Stale Data Cleanup (Cron Job).

## Deferred for Future Versions (Do Not Start)

-   [ ] **Moderation:** Advanced tools for admins and creators.
-   [ ] **Automated Social Posting:** Automatically posting threads to Reddit.
