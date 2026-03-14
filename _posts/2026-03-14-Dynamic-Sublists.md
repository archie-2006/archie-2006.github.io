# GSoC 2026 Proposal: Dynamic Sublists (Conversations) 

<!-- ## 1. Applicant Information
* **Name:** Archie Singh
* **University:** Indian Institute of Technology (IIT) Jodhpur
* **Major:** Computer Science and Engineering (Expected Graduation: 2028)
* **GitHub:** [Your GitHub Link]
* **Email:** [Your Email Address]
* **Timezone:** IST (UTC +5:30) -->

## Project Synopsis
Mailing lists are a powerful communication tool, but highly active lists often suffer from "inbox fatigue." When a niche topic generates dozens of replies, every subscriber receives every message, even if they have no interest in that specific thread. 

This project implements **"Dynamic Sublists" (Conversations)** into Mailman Core—a feature originally pioneered by the Systers (AnitaOrg, which I read in a 2018 article itself wished to migrate to Mailman under gsoc). It transitions Mailman from a strict broadcast model to an opt-in thread model. Under this system, only the initial "Thread Root" email is broadcast to the entire list. Subsequent replies are routed exclusively to the original sender and users who have explicitly opted in to follow that thread.

## 3. Technical Architecture & Implementation Plan
To implement Dynamic Sublists, I will modify four key areas of Mailman Core:

### A. Database Schema & State Management (SQLAlchemy)
I will introduce a new SQLAlchemy model to persist thread opt-ins.
* **New Model (`ThreadSubscription`):** This model will map a list subscriber to a specific thread. It will contain fields for `id` (Primary Key), `mailing_list_id` (Foreign Key), `address_id` (Foreign Key mapping to the subscriber's verified address), and `thread_id` (The hashed `Message-ID` of the root email).
* **Manager Interface:** A new `IThreadManager` interface will handle CRUD operations, such as `subscribe_to_thread()` and `unsubscribe_from_thread()`.

### B. Recipient Filtering (Pipeline Handlers)
Currently, Mailman populates `msgdata['recipients']` with all list members. I will create a new Pipeline Handler (e.g., `DynamicSublistHandler`).
* **Header Parsing:** The handler will inspect incoming emails for `In-Reply-To` and `References` headers using Python's `email.message` API.
* **Set Intersection:** If a thread match is found in the database, the handler will perform a set intersection between the total list subscribers and the thread subscribers, overriding `msgdata['recipients']` so that only opted-in users receive the delivery.

### C. Email Command Interface
I will implement new email commands (e.g., `-follow` and `-unfollow`).
* Leveraging the command parsing logic in `mailman.commands`, these commands will extract the user's `Address` and the target `Message-ID`.
* The command runner will then pass these arguments to the `IThreadManager` to update the database state.

### D. REST API Endpoints
To allow web frontends (like HyperKitty) to interact with this feature, I will expose it via the Core REST API.
* `POST /lists/<list_id>/threads/<thread_id>/subscribers`
* `DELETE /lists/<list_id>/threads/<thread_id>/subscribers/<address>`

## 4. Project Timeline (12 Weeks)

* **Community Bonding Period (May):** * Finalize the database schema design with my mentors.
  * **Warm-up Task:** Complete **Issue #1226** (Routing mail to list-owners only) to familiarize myself with safely modifying `msgdata['recipients']` in the delivery pipeline.

* **Phase 1: Database & Core Models (Weeks 1 - 3)**
  * Define the `ThreadSubscription` SQLAlchemy models and implement the `IThreadManager` interface. Write comprehensive unit tests for database queries.

* **Phase 2: Pipeline Handler Logic (Weeks 4 - 6)**
  * Develop the `DynamicSublistHandler` to intercept `In-Reply-To` headers and implement the recipient set-intersection logic. Test against missing headers and malformed `Message-IDs`.

* **Phase 3: User Interfaces (Weeks 7 - 9)**
  * Implement the `-follow` and `-unfollow` email commands. Write `.rst` doctests to ensure the command runner properly processes the arguments.

* **Phase 4: API & Wrap-up (Weeks 10 - 12)**
  * Build out the REST API endpoints in Core.
  * Finalize documentation, clean up code to meet PEP8 standards, and submit the final evaluation.

## 5. About Me & Prior Contributions
I am a Computer Science Engineering student with a deep interest in backend system architecture, data structures, C++, and Python development. I am highly comfortable navigating complex database routing, writing unit tests, and adhering to strict CI/CD pipelines.

**Mailman Contributions:**
* **Issue #1105 (Merge Request Submitted):** Fixed the `leave` email command to properly process the `address=` argument. This required navigating the scopes between `User` and `Address` objects, refining Python matching logic in `eml_membership.py`, and writing precise `.rst` doctests. *(Note: Encountered and diagnosed the known `#1258` flaky test bug involving `_io.BufferedWriter` signal handlers during the CI pipeline).*