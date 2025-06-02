# NigūḍhaLoka - Backend API Service

## Overview

This document outlines the API endpoints for the `backend-api` service of the NigūḍhaLoka platform. The `backend-api` is the core Node.js/Express.js application responsible for all business logic, user authentication (JWT-based), database interactions (MySQL for authentication, NoSQL for profiles/metadata, and a specialized Thread Database for discussions), and asynchronous processes.

It serves as the "Actual API" that the `gateway-php` layer communicates with.

## API Endpoints

All endpoints are relative to the base URL of the `backend-api` service.

### I. Authentication (`/auth`)

Handles user registration, login, JWT issuance and validation, and password management.

* **`POST /auth/signup`**
    * **Purpose:** Register a new user.
    * **Request Body:**
        ```json
        {
          "email": "user@example.com",
          "password": "securePassword123",
          "username": "newuser",
          "displayName": "New User"
        }
        ```
    * **Response (Success 201):** User information (excluding sensitive data).
    * **Notes:** Handles password hashing.

* **`POST /auth/signin`**
    * **Purpose:** Log in an existing user.
    * **Request Body:**
        ```json
        {
          "loginIdentifier": "user@example.com", // or "username"
          "password": "securePassword123"
        }
        ```
    * **Response (Success 200):**
        ```json
        {
          "accessToken": "your_access_token",
          "refreshToken": "your_refresh_token"
        }
        ```

* **`POST /auth/refresh-token`**
    * **Purpose:** Obtain a new access token using a refresh token.
    * **Request Body:**
        ```json
        {
          "refreshToken": "your_refresh_token"
        }
        ```
    * **Response (Success 200):**
        ```json
        {
          "accessToken": "new_access_token"
        }
        ```

* **`POST /auth/logout`**
    * **Purpose:** Log out a user (e.g., invalidate refresh token).
    * **Response (Success 200):** Success message.
    * **Notes:** Requires `accessToken` for identifying the session to invalidate.

* **`POST /auth/forgot-password`**
    * **Purpose:** Initiate the password reset process.
    * **Request Body:**
        ```json
        {
          "email": "user@example.com"
        }
        ```
    * **Response (Success 200):** Success message.

* **`POST /auth/reset-password`**
    * **Purpose:** Set a new password using a reset token.
    * **Request Body:**
        ```json
        {
          "token": "password_reset_token_from_email",
          "newPassword": "newSecurePassword456"
        }
        ```
    * **Response (Success 200):** Success message.

* **`POST /auth/verify-email`**
    * **Purpose:** Verify a user's email address using a verification token/code.
    * **Request Body:**
        ```json
        {
          "token": "email_verification_token_from_email"
        }
        ```
    * **Response (Success 200):** Success message.

* **`POST /auth/resend-verification-email`**
    * **Purpose:** Resend the email verification link/code.
    * **Request Body:**
        ```json
        {
          "email": "user@example.com"
        }
        ```
    * **Response (Success 200):** Success message.

### II. User Profile (`/profile`)

Manages data for the authenticated user.

* **`GET /profile/me`**
    * **Purpose:** Retrieve the profile of the currently authenticated user.
    * **Response (Success 200):** User profile data (e.g., `userId`, `username`, `displayName`, `email`, custom profile fields).

* **`PUT /profile/me`**
    * **Purpose:** Update the profile of the currently authenticated user.
    * **Request Body:** Updatable profile fields (e.g., `displayName`, custom fields).
        ```json
        {
          "displayName": "Updated Name"
        }
        ```
    * **Response (Success 200):** Updated user profile data.

### III. Posts (Discussions & Replies) (`/posts`)

Manages content creation and retrieval. A "post" can be an initial topic starter or a reply to another post, differentiated by the presence or absence of a `parentId`.

* **`POST /posts`**
    * **Purpose:** Create a new post (topic or reply).
    * **Request Body:**
        ```json
        {
          "content": "This is the main text of the post.",
          "parentId": "optional_parent_post_id_if_reply" // Omit or null for a new topic
        }
        ```
    * **Response (Success 201):** Created post data (including `postId`, `content`, `parentId`, `authorId`, `createdAt`).

* **`GET /posts`**
    * **Purpose:** List initial posts (topic starters).
    * **Query Parameters:** `page` (number), `limit` (number), `sortBy` (e.g., `createdAt`, `lastActivity`), `order` (e.g., `asc`, `desc`).
    * **Response (Success 200):** Paginated list of initial posts.

* **`GET /posts/{postId}`**
    * **Purpose:** Retrieve a specific post.
    * **Response (Success 200):** Post data.

* **`GET /posts/{postId}/thread`**
    * **Purpose:** Retrieve a specific post and its reply thread.
    * **Query Parameters:** `page` (number, for paginating replies), `limit` (number, replies per page), `depth` (number, how many levels of replies to fetch).
    * **Response (Success 200):** Post data along with its nested replies.

* **`PUT /posts/{postId}`**
    * **Purpose:** Update a post's content (by its creator or a moderator).
    * **Request Body:**
        ```json
        {
          "content": "Updated content of the post."
        }
        ```
    * **Response (Success 200):** Updated post data.

* **`DELETE /posts/{postId}`**
    * **Purpose:** Delete a post (by its creator or a moderator).
    * **Response (Success 200 or 204):** Success message or no content.

* **`POST /posts/{postId}/disable`**
    * **Purpose:** Disable a post (moderation action by creator or moderator).
    * **Response (Success 200):** Success message.

* **`POST /posts/{postId}/enable`**
    * **Purpose:** Re-enable a disabled post (moderation action by creator or moderator).
    * **Response (Success 200):** Success message.

### IV. Moderation (`/admin` and `/reports`)

Handles content reporting and administrative moderation actions. User-level moderation (editing/deleting own posts, topic creator moderating their thread) is handled via the `/posts` endpoints with appropriate authorization logic.

* **`POST /reports`**
    * **Purpose:** Allow users to report a post.
    * **Request Body:**
        ```json
        {
          "postId": "reported_post_id",
          "reason": "Reason for reporting."
        }
        ```
    * **Response (Success 201):** Report submission confirmation.

* **`GET /admin/reports`** (Admin Only)
    * **Purpose:** List reported posts for admin review.
    * **Query Parameters:** `page`, `limit`, `status` (e.g., `pending`, `resolved`).
    * **Response (Success 200):** Paginated list of reports.

* **`POST /admin/reports/{reportId}/action`** (Admin Only)
    * **Purpose:** Admin takes action on a report.
    * **Request Body:**
        ```json
        {
          "action": "dismiss_report | disable_post | warn_user", // etc.
          "remarks": "Admin comments."
        }
        ```
    * **Response (Success 200):** Action confirmation.

* **`GET /admin/users`** (Admin Only)
    * **Purpose:** List users for administrative purposes.
    * **Query Parameters:** `page`, `limit`, `searchQuery`.
    * **Response (Success 200):** Paginated list of users.

* **`PUT /admin/users/{userId}/status`** (Admin Only)
    * **Purpose:** Admin updates a user's status (e.g., ban, suspend, verify).
    * **Request Body:**
        ```json
        {
          "status": "active | banned | suspended",
          "reason": "Reason for status change."
        }
        ```
    * **Response (Success 200):** Confirmation.

### V. Snapshot Generation (`/snapshots` and `/posts`)

Manages requests for and status of shareable snapshots of posts/threads.

* **`POST /posts/{postId}/request-snapshot`**
    * **Purpose:** User requests a snapshot of a specific post or its thread.
    * **Request Body:** (Optional parameters for snapshot customization)
        ```json
        {
          "format": "image" // or "gif"
        }
        ```
    * **Response (Success 202):**
        ```json
        {
          "jobId": "snapshot_job_id"
        }
        ```
    * **Notes:** This initiates an asynchronous snapshot generation process.

* **`GET /snapshots/{jobId}/status`**
    * **Purpose:** Client polls for the status of a snapshot generation request.
    * **Response (Success 200):**
        ```json
        {
          "jobId": "snapshot_job_id",
          "status": "pending | processing | completed | failed",
          "snapshotUrl": "url_to_snapshot_if_completed_or_null"
        }
        ```

### VI. User Notifications (`/notifications`)

Manages user notifications and push notification subscriptions.

* **`GET /notifications`**
    * **Purpose:** Retrieve notifications for the authenticated user.
    * **Query Parameters:** `page`, `limit`, `unreadOnly` (boolean).
    * **Response (Success 200):** Paginated list of notifications.

* **`POST /notifications/mark-read`**
    * **Purpose:** Mark specific notifications as read.
    * **Request Body:**
        ```json
        {
          "notificationIds": ["id1", "id2"]
        }
        ```
    * **Response (Success 200):** Success message.

* **`POST /notifications/mark-all-read`**
    * **Purpose:** Mark all notifications for the user as read.
    * **Response (Success 200):** Success message.

* **`POST /notifications/subscribe`**
    * **Purpose:** Subscribe the user's device for push notifications.
    * **Request Body:** Push subscription object from the browser/client.
    * **Response (Success 201):** Success message.

* **`POST /notifications/unsubscribe`**
    * **Purpose:** Unsubscribe the user's device from push notifications.
    * **Request Body:** Push subscription endpoint or identifier.
    * **Response (Success 200):** Success message.

## Authentication

All requests to protected endpoints must include a JWT `accessToken` in the `Authorization` header using the `Bearer` scheme:

`Authorization: Bearer <your_access_token>`

## Error Handling

The API will use standard HTTP status codes to indicate success or failure. Error responses will typically be in JSON format:

```json
{
  "error": {
    "message": "A descriptive error message.",
    "code": "ERROR_CODE_IF_APPLICABLE" // Optional application-specific error code
  }
}
```

Common Status Codes:
* `200 OK`: Request successful.
* `201 Created`: Resource successfully created.
* `202 Accepted`: Request accepted for processing (e.g., async tasks).
* `204 No Content`: Request successful, no content to return.
* `400 Bad Request`: Invalid request payload or parameters.
* `401 Unauthorized`: Authentication failed or token missing/invalid.
* `403 Forbidden`: Authenticated user does not have permission to access the resource.
* `404 Not Found`: Resource not found.
* `500 Internal Server Error`: An unexpected error occurred on the server.

## Further Development

This document outlines the initial set of planned endpoints. As development progresses, new endpoints may be added, and existing ones may be refined. API versioning may be introduced in the future if significant breaking changes are required.
