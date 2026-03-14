# Archie Singh

<!-- ## 1. Applicant Information
* **Name:** Archie Singh
* **University:** Indian Institute of Technology (IIT) Jodhpur
* **Major:** Computer Science and Engineering (Expected Graduation: 2028)
* **GitHub:** 
* **Email:** 
* **Timezone:** IST (UTC +5:30) -->

## Project Synopsis

Having familiarised with mailman-developers mailing list, it is obvious that mailings lists are powerful tools for communication when you don't want to reach out to only one person but get feedback etc. from multiple people at once/ or whoever decides to help. 

The issue that occurs is lists like these are very active and there is a chance of inboxes getting flooded even for mails that are of no interest to the receiver. When a particular topic receives multiple replies, every subscriber will receive every message in that thread even if they have no interest in that particular topic, and there is no option to opt out of it, as currently noted. (like for example, during gsoc the mails are flooded with people sending their proposals on the mailing list and others who are working on their project ideas are receiving the mails for other project ideas that they are not working on)

This project idea is to implement **"Dynamic Sublists" (Conversations)** into Mailman Core (on gitlab). The main focus is on how currently mailman is following a broadcast model, it will change into an opt-in thread model. Under this system, only the main thread (root) email will be broadcast to the entire mailing list and the replies that fall under this thread topic will be routed only to the original poster and to the other members/users who have explicitly opted in to follow that thread/topic.

Systers being the largest private email community for women in technical computing roles happened to solve this exact same issue by writing their own fork of Mailman for taking care of conversations and dynamic sublists (although that was for Mailman 2.1.0), their features were also changed and redefined to cater to syster's logic. The tasks at hand therefore are,

- Due to the architectural changes in Mailman 3, we cannot directly use the systers logic to define dynamic sublists and have to come up with something else.
- Therefore, the focus is shifted to bring the same Systers conversation model (dynamic sublists) into the modern Mailman 3 Core architecture (from scratch although, as mailman 2 (python2) and mailman 3 (python3, sqlalchemy) have vastly different architectures (monolithic and REST API respectively)

## Technical Architecture & Implementation Plan & Initial Research
Following are the key areas to define (but not limited to):

### A. Database Schema & State Management (SQLAlchemy)
A new SQLAlchemy model for thread opt-ins.
* **New Model (`ThreadSubscription`):** A list subscribe is mapped to a particular thread using this model. Fields: `id` (Primary Key), `mailing_list_id` (Foreign Key), `address_id` (Foreign Key mapping to the subscriber's verified address), and `thread_id` (The hashed `Message-ID` of the root email).
* **Manager Interface:** `IThreadManager` interface will handle all the CRUD operations, such as `subscribe_to_thread()` and `unsubscribe_from_thread()`.

### B. Recipient Filtering (Pipeline Handlers)
Currently, Mailman adds `msgdata['recipients']` with all list members. Creating a new Pipeline Handler (like `DynamicSublistHandler`).
* **Header Parsing:** Inspects incoming emails for `In-Reply-To` and `References` headers using Python's `email.message` API.
* **Set Intersection:** If a thread match is found in the database, the handler will perform a set intersection between the total list subscribers and the thread subscribers, overriding `msgdata['recipients']` so that only the users that have opted in receive the delivery of those mails.

### C. Email Command Interface
I will implement new email commands (e.g., `-follow` and `-unfollow`).
* Using the command parsing logic in `mailman.commands`, these will extract the user's `Address` and the target `Message-ID`.
* The command runner will then pass these arguments to the `IThreadManager` (see manager interface in sqlalchemy) to update the DB state.

### D. REST API Endpoints
To allow HyperKitty (web frontend) to interact with this feature. This ensures people can subscribe to these threads via the web interface in addition to email cmds.
* `POST /lists/<list_id>/threads/<thread_id>/subscribers`
* `DELETE /lists/<list_id>/threads/<thread_id>/subscribers/<address>`

## Project Timeline (Currently working on it)
*(To be updated based on mentor feedback regarding 175 vs 350-hour scope).*

## Prior Open Source Contributions

I have previously worked on the OTW (Organization for Transformative Works) archive to fix bugs and suggest some changes and features for the Archive of Our Own. I have also contributed to other open-source projects, which you can find on my [GitHub profile](https://github.com/archie-2006).

**Mailman Contributions:**
* **Issue #1105 (Merge Request Submitted):** Fixed the `leave` email command to properly process the `address=` argument. This required navigating the scopes between `User` and `Address` objects, refining Python matching logic in `eml_membership.py`, and writing precise `.rst` doctests. *(Note: Encountered and diagnosed the known `#1258` flaky test bug involving `_io.BufferedWriter` signal handlers during the CI pipeline in this issue itself).*