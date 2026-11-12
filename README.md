# El Llar Hub: Smart cohousing onboarding and governance pipeline

### 📌 Project Overview

**The problem:** Handling an admission funnel by hand is a heavy load for a volunteer Steering Committee in social economy organizations, such as *El Llar Cohousing* in Asturias, Spain.

**A possible solution:** El Llar Hub is a web application built on OutSystems Developer Cloud (ODC). It automates the intake, validation, and qualification of applicants for *El Llar Cohousing*. The app gives candidates a public portal guided by AI, queues and sends emails asynchronously, and provides the board with a secured backoffice where people make the final decisions.

---

### 🚀 Core Funnel & Application Logic

#### 1. Public intake and eligibility filter
* **Llarina001 (FAQ agent):** A chat that answers general questions about El Llar.
* **Eligibility check:** Captures the applicant's basic data and age. If the applicant is over 70, the contact is set to `IsActive = False`, the process stops politely, and no emails are sent.
* **Contact reference:** Every contact gets a public reference in the format `LLAR-000123`. Candidates use it to identify themselves in the chat, and the board uses it in notifications.
* **Questionnaire:** Eligible applicants get an application and a 16-question values form. They can submit it online, or download a `.txt` template that carries their reference and upload it later. The upload is validated against the contact and email before it is filed.

#### 2. Llarina002: Agentic evaluation
When a questionnaire is received, Llarina002 compares the answers with El Llar's Manifesto. It sends its verdict and justification through an exposed service action, which stores the response and triggers the business rules:
* **Suitable:** Queues an interview invitation for the applicant and a notification to a board member.
* **Archived:** For nonsense or non-aligned answers. The contact is set to inactive and a courtesy letter is queued.
* **Under review:** For vague or ambiguous answers. An expectant letter is queued.

#### 3. Asynchronous email architecture (Mailtrap sandbox)
Emails are never sent during a user's screen action, so the app avoids timeouts and SMTP throttling:
* **Queue management:** Every decision writes an entry in an `EmailQueue` entity with status `Pending`. The entry has a type: expectant letter, board notification, interview invitation, review digest, or closure letter.
* **ProcessEmailQueueTimer:** Runs every 5 minutes and sends at most `EmailsPerRun` emails per run, oldest first. This setting can be changed without republishing.
* **Retry policy:** Each failed send keeps the error in `LastError` and counts as an attempt. After 3 attempts, the email becomes `Failed`. The same rule closes emails that got stuck after a timeout or an interrupted run.
* **WeeklyReviewDigestTimer:** Runs every Monday at 08:00. It gathers the applications still `UnderReview` that the board has not yet been told about, and lists their names, dates, application IDs, and `LLAR-xxxxxx` references in a single digest email. It then stamps `BoardReviewNotifiedOn` so nothing is reported twice.

#### 4. Secured administrative backoffice
* **Access control:** Needs a private security code. The code is a secret setting managed from the ODC Portal, not stored in the screens.
* **Candidate list:** Search by name or email, sortable columns, pagination, status pills, and a switch to show or hide archived candidates. Admins can also create a new candidate.
* **Candidate detail:** Displays general data, Llarina002's full justification, the original questionnaire as a `.txt` download, and the decision history.
* **Human-in-the-loop decisions:** Four buttons move a candidate to **Waiting List**, **Partner**, **Archived**, or **Under Review**. Each change asks for a written justification, which is saved in a history log with date and status (newest first). The matching email is queued, and archiving is logical so the record is kept.

---

### 🛠️ Hackathon Implementation Note

This is a working prototype that focuses on business rules and architecture inside ODC:
* **Asynchronous email system:** Emails are stored in an internal queue and a timer sends them gradually. All of them go to a **Mailtrap sandbox**, so no real email reaches anyone.
* **Secured backoffice:** A simple access code keeps status changes in the hands of the cooperative's managers.
* **Mocked AI components:** The responses of Llarina001 and Llarina002 are **simulated with internal test datasets and visual simulations**. Judges can evaluate the full flow and the business rules without live connections to paid APIs or third-party databases. The agent's verdict enters through a single service action that stores the decision, updates the candidate, and queues the right emails. Connecting a real agent only requires calling that action, with no changes to the business logic.

## 3 SETTINGS IN ODC Portal

## 1. 📧 Email & Mailtrap Sandbox Configuration (ODC Portal)

This application uses ODC's built-in **Send Email** functionality combined with a **5-minute Timer** that processes pending records from the `EmailQueue` entity. 

Because SMTP configurations are managed at the stage level in ODC (and are not exported inside `.oml` files), please configure your stage's SMTP settings in the **ODC Portal** after importing the modules:

1. Go to **ODC Portal** -> **Configure** -> **Email** (select your target stage, e.g., *Development*).
2. Enter the following **Mailtrap Sandbox SMTP** CREDENTIALS:
   * **Server / Host:** `sandbox.smtp.mailtrap.io`
   * **Port:** `2525` *(or `587`)*
   * **Username:** `<6c413a2aa262ed>` *(provided privately to the judges)*
   * **Password:** `<09eba20d18131e>` *(provided privately to the judges)*
   * **Default Sender Email:** Any valid email format (e.g., `noreply@hackathon-app.com`)
   * **Default Sender Name:** `Hackathon App`
3. Click **Save** and deploy/publish the app.

### How to test the 5-Minute Email Timer
1. Trigger an action in the application that queues a new email in `EmailQueue`.
2. Wait up to **5 minutes** for the scheduled Timer to run automatically (or trigger it manually via **ODC Portal -> Operations -> Timers**).
3. Open the **Mailtrap Sandbox Inbox** to verify the delivered email and its rendered template.

### 2. BoardMemberEmail (App Setting):
Set its value to: **`javier@el-llar.com`**

### 3. Backoffice Access Code (App Setting):
1. Go to **ODC Portal** -> **Apps** -> select **`El Llar Hub`** -> **Configuration** -> **Settings**.
2. Locate the setting **`BackofficeAccessCode`** (used to validate access to the Backoffice).
3. Set its value to: **`2026`** *(e.g., `1234` or whatever code you want to use)* and save/apply changes.


## 2 VIDEOS:
[**Demo**](https://youtu.be/-rAHNDC-m4c)

[**Technical**](https://youtu.be/yk8uPOCQWvg)
