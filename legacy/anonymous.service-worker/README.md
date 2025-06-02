# Legacy: Anonymous Service Worker

**Status: Deprecated**

This document describes the `anonymous.service-worker`, a component of the legacy "Anonymous" application (anonymous.developerswork.online).

It was responsible for handling web push notifications. This system is being replaced by the new "NigūḍhaLoka" platform.

## Overview

The `anonymous.service-worker` is a Node.js application built with Express.js and MongoDB.

Its primary role was to manage subscriptions for web push notifications and send notifications to users of the "Anonymous" application.


## Core Functionality

* **Push Notification Subscription**:
    * Users could subscribe to push notifications. These subscriptions were stored in a MongoDB database, linked to a user identifier (`uid`).
       
* **Push Notification Sending**:
    * The system could send push notifications to specific users based on their `uid`.
       
    * Notifications included a title, message, and an optional link.
       
* **Scheduled Tasks**:
    * **Reminder Notifications**: A cron job sent reminder notifications to users who had been inactive for a defined period (1-3 days).
       
    * **Database Cleanup**: A cron job regularly cleaned up old data:
        * Notification subscriptions (`device_notifications`) older than 30 days were removed.
           
        * Entries in a `mails` collection were deleted if marked as `used` or older than 30 days.
           

## API Endpoints

The service exposed the following main HTTP POST endpoints:

* `POST /subscribe`:
    * **Purpose**: To register a new device for push notifications.
    * **Request Body**: Expected a `subscription` object and a `user` (uid).
       
    * **Response**: HTTP 201 on success.
       
* `POST /sendNotification`:
    * **Purpose**: To send a push notification to a subscribed user.
    * **Request Body**: Expected `uid`, `message`, and `title`. An optional `link` could also be provided.
       
    * **Response**: Empty object on success or an error object.
       

## Data Model (MongoDB)

The service utilized a MongoDB database named "anonymous" with the following key collections:


* **`device_notifications`**:
    * `_id`: A composite key generated from `subscription.endpoint + "_" + uid`.
       
    * `subscription`: The push subscription object provided by the client's browser.
       
    * `uid`: The user identifier.
       
    * `lastActive`: Timestamp indicating the last activity associated with the subscription.
       
* **`mails`**:
    * This collection was used for purposes not entirely detailed in the service worker code but was subject to cleanup based on a `used` flag (boolean) and a `timestamp`.
       

## Key Dependencies

The service relied on the following main npm packages:


* `express`
* `body-parser`
* `cors`
* `web-push`
* `mongodb`
* `node-cron`

## Setup and Running

The service could be started using the npm script:


```bash
npm start
```

This would execute `node index.js`.


## Security Considerations (Important Notes)

During the analysis of this legacy service, the following critical security concerns were identified:

* **Hardcoded MongoDB Credentials**: The MongoDB connection URI, including username and password, was hardcoded directly into the `index.js` file. This is a severe security vulnerability.
   
* **Hardcoded VAPID Keys**: Public and private VAPID keys for web push notifications were also hardcoded in `index.js`.
   
* **Endpoint Security**: The `/sendNotification` endpoint lacked robust authentication/authorization mechanisms beyond expecting a `uid`.

These practices are highly discouraged and should be addressed with secure alternatives (e.g., environment variables, secrets management services) in any new development.
