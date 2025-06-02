# Legacy: Anonymous Service Worker - API Endpoints

**Status: Deprecated**

This document details the HTTP API endpoints exposed by the legacy `anonymous.service-worker`. This service was a component of the old "Anonymous" application and is being superseded by the "NigūḍhaLoka" platform. All endpoints were part of an Express.js application.


## Common Behaviors

* **CORS**: Enabled with `cors({ origin: true })`.
   
* **Body Parsing**: JSON body parser was used (`bodyParser.json()`).
   
* **Default 404 Response**: For undefined routes or missing required parameters in defined routes, a 404 status with a JSON message `{"message": "YOU ARE IN A WRONG SPOT!!"}` was returned.
   

## Endpoints

### 1. `POST /subscribe`

* **Description**: Registers a new device for web push notifications. It saves or updates the subscription details in the `device_notifications` MongoDB collection.
   
* **Request Body Parameters**:
    * `subscription`: `Object` (Required) - The push subscription object from the client. Must contain an `endpoint`.
       
    * `user`: `String` (Required) - The user identifier (`uid`) for whom to register the subscription.
       
* **Responses**:
    * `201 Created`: On successful creation or update of the subscription. Returns an empty JSON object `{}`.
       
    * `404 Not Found`: If `subscription`, `user`, or `subscription.endpoint` are not provided.
       
    * `500 Internal Server Error`: If there's an error during database operation. Returns an empty JSON object `{}`. (Note: The code attempts to return 500 but might still send an empty object without a specific error message to the client beyond console logs).
       

### 2. `POST /sendNotification`

* **Description**: Sends a push notification to all devices subscribed by a specific user.
   
* **Request Body Parameters**:
    * `uid`: `String` (Required) - The user identifier to send the notification to.
       
    * `message`: `String` (Required) - The body text of the notification.
       
    * `title`: `String` (Required) - The title of the notification.
       
    * `link`: `String` (Optional) - A URL to open when the notification is clicked. If provided, it's appended to `https://anonymous.developerswork.online/event/` and included in the notification's `data.url` field.
       
* **Responses**:
    * `200 OK`: Returns an empty JSON object `{}` after attempting to send notifications.
       
    * `200 OK` with `{"error": true}`: If an error occurs during the process of fetching subscriptions or sending notifications. (Note: The status code is still 200 despite the error flag).
       
    * `404 Not Found`: If `uid`, `message`, or `title` are not provided in the request body.
       

### 3. `GET /`
### 4. `POST /`
### 5. `GET /subscribe` (This is distinct from `POST /subscribe`)

* **Description**: These routes are explicitly defined to return a 404 error with the standard "YOU ARE IN A WRONG SPOT!!" message. They likely serve as catch-alls or to prevent unintended access via these methods/paths.
   
* **Responses**:
    * `404 Not Found`: Returns `{"message": "YOU ARE IN A WRONG SPOT!!"}`.
       

## VAPID Key Details (Hardcoded)

The service used hardcoded VAPID keys for web push notifications.

* **Public Key**
   
* **Private Key**
   
* **Mailto**
   

**Security Note**: Hardcoding these keys, especially the private VAPID key, is a security risk.
