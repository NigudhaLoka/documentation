# Legacy "Anonymous" Application - API Endpoints

This document lists the API endpoints for the legacy "Anonymous" application, primarily based on the provided Postman collection (`ANONYMOUS MESSAGE.postman_collection.json`). URLs may vary (e.g., `localhost`, `anonymous.developerswork.online`).

## Authentication & User Management

Endpoints related to user accounts, login, registration, and profile.

* **POST** `/api/v2/login`
    * **Purpose:** User login.
    * **Key Parameters:** `username`, `password`, `captcha`
* **POST** `/api/v1/logout` (Path seen as `/developerswork/anonymous_message/api/v1/logout` in collection)
    * **Purpose:** User logout.
* **POST** `/api/v2/updateprofile`
    * **Purpose:** Update user profile information.
    * **Key Parameters:** `username`, `password` (likely for confirmation or not primary update field), `phoneNumber`, `email`, `displayName`, `captcha`
* **POST** `/api/v1/getProfile` (Path seen as `/developerswork/anonymous_message/api/v1/getProfile` in collection)
    * **Purpose:** Retrieve user profile information.
* **POST** `/api/v2/getcaptcha`
    * **Purpose:** Request a new captcha.
* **POST** `/api/v1/changepassword` (Path seen as `/developerswork/anonymous_message/api/v1/changepassword` in collection)
    * **Purpose:** Change user password.
    * **Key Parameters:** `password` (new), `old_password`, `captcha`
* **POST** `/api/v2/resetpassword`
    * **Purpose:** Initiate password reset process.
    * **Key Parameters:** `email`, `username`, `captcha`
* **POST** `/api/v2/verify`
    * **Purpose:** Verify email address using a code.
    * **Key Parameters:** `id` (verification identifier), `code` (verification code)
* **POST** `/api/v2/resend_verification_email`
    * **Purpose:** Resend email verification link/code.
    * **Key Parameters:** `username`, `email`, `captcha` (collection shows `captacha`)

## Event Management

Endpoints for creating, viewing, and managing events (questions/topics) and their replies.

* **POST** `/api/v1/event/create`
    * **Purpose:** Create a new event.
    * **Key Parameters:** `context` (event content), `captcha`
* **GET** `/api/v1/event/:EVENTLINK` (Path seen as `/developerswork/anonymous_message/api/v1/event/:EVENTLINK` in collection)
    * **Purpose:** Get details of a specific event by its link.
* **POST** `/api/v1/event/comment` (Path seen as `/developerswork/anonymous_message/api/v1/event/comment` in collection)
    * **Purpose:** Add a comment/reply to an event.
    * **Key Parameters:** `message`, `captcha`, `link` (event link), `name` (display name for comment)
* **POST** `/api/v1/event/update` (Path seen as `/developerswork/anonymous_message/api/v1/event/update` in collection)
    * **Purpose:** Update an existing event.
    * **Key Parameters:** `context` (updated content), `captcha`, `link` (event link)
* **POST** `/api/v2/event/get`
    * **Purpose:** Get event details (seems to be an alternative to the GET by link, possibly by ID).
    * **Key Parameters:** `id` (event identifier)
* **POST** `/api/v1/event/disable` (Path seen as `/developerswork/anonymous_message/api/v1/event/disable` in collection)
    * **Purpose:** Disable an event.
    * **Key Parameters:** `link` (event link)
* **POST** `/api/v1/event/delete` (Path seen as `/developerswork/anonymous_message/api/v1/event/delete` in collection)
    * **Purpose:** Delete an event.
    * **Key Parameters:** `link` (event link)
* **POST** `/api/v1/event/list` (Path seen as `/developerswork/anonymous_message/api/v1/event/list` in collection)
    * **Purpose:** List events with pagination.
    * **Key Parameters:** `page`, `items` (per page)

## Mail Management

Endpoints related to mail/notifications.

* **POST** `/api/v2/mail/get`
    * **Purpose:** Retrieve mail/notification content (likely using a mail ID).
    * **Key Parameters:** `id` (mail identifier)

## Feedback

Endpoints for submitting feedback.

* **POST** `/api/v3/feedback/comment`
    * **Purpose:** Submit feedback or a comment through the feedback system.
    * **(Note: Parameters not detailed in collection for this specific entry, but would likely include message and captcha)**

## Service Worker / Notifications (External)

These endpoints point to an external Heroku application for push notifications.

* **POST** `https://anonymous-service-worker.herokuapp.com/sendNotification`
    * **Purpose:** Send a push notification via the service worker.
    * **Body (raw JSON):** `uid`, `title`, `message`, `link`
* **POST** `https://anonymous-service-worker.herokuapp.com/subscribe`
    * **Purpose:** Subscribe a user/device for push notifications.
    * **Body (raw JSON):** `user` (likely UID), `subscription` (push subscription object)

*Note: The Postman collection includes requests with varying base URLs (e.g., `http://localhost/developerswork/anonymous/`, `http://localhost/developerswork/anonymous_message/`, `https://anonymous.developerswork.online/`). The API versioning (v1, v2, v3) is indicated in the paths.*
