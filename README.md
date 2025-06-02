# NigūḍhaLoka: The Anonymous Discussion Platform

## 1. Vision

NigūḍhaLoka is an evolution of the legacy "Anonymous" application, aiming to create a rich, secure, and engaging platform for anonymous, thread-based discussions. Our core philosophy is to prioritize the **message over the source**, fostering open conversations where ideas can be explored freely and respectfully.

This platform is designed for users who wish to discuss topics, ask questions, and share replies without their registered identity being publicly tied to their contributions.

## 2. Platforms

NigūḍhaLoka will be accessible via:

* **Web Application:** A modern, responsive web interface.
* **Mobile Applications:** Dedicated apps for Android and iOS.

## 3. Core Features

* **Authenticated Anonymity:** Users register and authenticate, but their interactions (posts, replies) within discussions are anonymous.
* **Topic Creation:** Users can create new topics or questions that serve as the starting point for discussions.
* **Threaded Interactions:** Discussions are organized into threads, allowing replies to original topics and replies to other replies, creating nested conversations.
* **Tiered Moderation:**
    * **Admin:** Overall platform moderation capabilities.
    * **Topic Creator:** Moderation rights over the topic they initiated (e.g., disabling the topic).
    * **Subthread Creator:** Potential for moderation rights within specific reply chains they start (details TBD).
* **Content Management:** Ability to disable or delete threads/topics as per moderation rules.
* **Captcha Verification:** To prevent spam and bot activity during sensitive actions.
* **Shareable Snapshots:** Users can generate images or GIFs of discussion snippets for easy sharing on social media or elsewhere.
* **User Notifications:** A system to notify users of relevant activity (e.g., replies to their posts, moderation actions).

## 4. Architecture Overview

NigūḍhaLoka employs a multi-repository, microservices-inspired architecture to enhance modularity, scalability, and maintainability.

### 4.1. Repository Structure

The project is organized into the following dedicated repositories within the 'NigudhaLoka' GitHub organization:

* **`frontend-web` (Public):** Houses the Next.js and Tailwind CSS codebase for the web application.
* **`frontend-mobile` (Public):** Will contain the chosen mobile framework (e.g., React Native or Flutter) for both Android and iOS applications.
* **`gateway-php` (Private):** A PHP-based intermediary layer.
* **`backend-api` (Private):** The core Node.js/Express.js API (the "Actual API").
* **`documentation` (Public):** Contains all project documentation, including this README, architectural diagrams, API specifications, and style guides.

### 4.2. Key Architectural Components & Flow

1.  **Client Applications (`frontend-web`, `frontend-mobile`):** Users interact with these applications. All requests are directed to the `gateway-php` layer.
2.  **PHP Intermediary Layer (`gateway-php`):**
    * Acts as a front-facing API gateway, potentially fronted by services like Cloudflare for enhanced security (DDoS protection, WAF).
    * Performs initial request validation, including captcha verification.
    * Manages specific tasks like the snapshot generation queue (interacting with MySQL for queue status).
    * Forwards legitimate requests to the `backend-api`. It is the only service authorized to communicate directly with the `backend-api` (firewall rules based on IP).
    * Handles JWTs, passing them to the `backend-api` for validation and user context.
3.  **Core API (`backend-api` - Node.js with Express.js):**
    * Contains the primary business logic for NigūḍhaLoka.
    * Manages user authentication (JWT validation, password hashing).
    * Interacts with the various databases (MySQL, NoSQL, Thread DB) for data persistence and retrieval.
    * Handles asynchronous processes (e.g., notification delivery, actual snapshot generation).

### 4.3. Databases

* **MySQL:**
    * Stores core authentication details (e.g., user emails, hashed passwords).
    * Manages the snapshot generation queue and status (primarily via `gateway-php`).
* **NoSQL (e.g., MongoDB/Firebase):**
    * Stores user profiles, application metadata, and other non-relational data.
* **Thread Database:**
    * A specialized database solution chosen for its efficiency in handling deeply threaded content and complex read/write patterns typical of discussion forums.

## 5. Technology Stack Summary

* **Frontend Web:** Next.js, Tailwind CSS
* **Frontend Mobile:** To be decided (e.g., React Native, Flutter)
* **Gateway Layer:** PHP
* **Backend API:** Node.js, Express.js
* **Databases:**
    * MySQL
    * MongoDB (or similar NoSQL like Firebase Firestore)
    * Specialized Thread Database (e.g., a graph database, or a relational/NoSQL DB optimized for hierarchical data)
* **Authentication:** JSON Web Tokens (JWTs), with `backend-api` as the issuing and validation authority.
* **Containerization (Recommended):** Docker, Kubernetes for deployment and scaling.

## 6. Project Status

**In Development.**

Active development is underway across the new repository structure. Early drafts and foundational code are being established.

## 7. Evolution from "Anonymous" (Legacy)

NigūḍhaLoka represents a significant architectural and functional leap from the legacy "Anonymous" PHP application. Key improvements include:

* **Rich Threaded Discussions:** Moving beyond a simple question/reply model to complex, nested conversations.
* **Tiered Moderation:** Introducing more granular control over content.
* **Modern Technology Stack:** Leveraging Next.js, Node.js, and specialized databases for better performance, scalability, and developer experience.
* **Enhanced Security Model:** A dedicated gateway, JWT-based authentication, and a clear separation of concerns aim to provide a more secure platform.
* **Scalability:** The new architecture is designed with scalability in mind to handle a growing user base and content volume.

## 8. Getting Started & Contribution

* **Explore the Repositories:** Each repository (`frontend-web`, `frontend-mobile`, `gateway-php`, `backend-api`, `documentation`) has its own specific README and contribution guidelines.
* **Documentation:** For detailed architectural decisions, API contracts, and setup guides, refer to the relevant documents within the `documentation` repository.
