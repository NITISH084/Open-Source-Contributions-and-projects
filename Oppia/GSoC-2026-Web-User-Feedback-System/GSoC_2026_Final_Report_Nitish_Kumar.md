<div align="center">
  <img width="1774" height="887" alt="Oppia X GSoC" src="https://github.com/user-attachments/assets/23e60e50-f106-4296-8bd9-103537e707ec" />

  <h1>Google Summer of Code 2026 - Final Report</h1>
</div>


| Project | [Web User Feedback System for Oppia](https://summerofcode.withgoogle.com/programs/2026/projects/DLicszJv) |
|---|---|
| **Mentors** | [Mohit Ruwatia](https://github.com/mon4our), [Brian Rodriguez](https://github.com/brianrodri) |
| **Organization** | [Oppia Foundation](https://github.com/oppia/oppia) |
| **Tracking Issue** | [oppia/oppia#24716](https://github.com/oppia/oppia/issues/24716) |
| **Proposal** | [Technical Design Document](https://docs.google.com/document/d/1hKv4D84m52iq9mFOxbyABXKNjbx-J91ywwq4FsihJ60/edit?tab=t.0) |
| **GSoC Journals** | [medium.com/@imnitishkumar04](https://medium.com/@imnitishkumar04) |


## The Idea Behind the Project 

My GSoC project began with a simple question: **Could a user report an issue or share feedback from anywhere on Oppia without having to leave the platform or figure out where to send it?**

The difficult part was not just adding another feedback form. Feedback can come from different kinds of users and can be related to a lesson, a specific part of the platform, or a general suggestion. The feedback also needs to reach the right people, while giving teams enough context to actually act on it. I wanted the system to fit naturally into Oppia and provide a useful workflow for both users and the teams responding to their feedback.

Over these fourteen weeks, I built **Web User Feedback System**, a native feedback system for Oppia. Users can submit **lesson feedback, report lesson issues, or report site issues** directly from the platform. Depending on the type of feedback, the system can capture **screenshots and session information**, route the feedback to the appropriate team, and provide internal interfaces for reviewing and managing it. Learners can also **track their suggestions and see responses from lesson creators**.

The project also includes the migration of existing lesson feedback from the legacy `GeneralFeedbackThreadModel` to the new `LessonFeedbackModel`, so that **historical feedback is preserved** as the old feedback system is deprecated.

<img width="1511" height="661" alt="image" src="https://github.com/user-attachments/assets/4122cfb2-d579-4e50-baa7-dbe35f5c19ee" />

---

## Why a New Feedback System Mattered

> *"If a user encounters a problem, reporting it should be part of the experience, not something that requires them to leave the platform."*

Feedback is one of the ways Oppia learns what is working well for learners and where the platform needs improvement. As Oppia grows, having an easy and consistent way for users to report issues and share suggestions becomes increasingly important.

Before this project, feedback was spread across disconnected channels. Lesson feedback lived inside the lesson player but was only visible to the creators of that specific lesson. Site feedback was collected through a footer link that was easy to miss. Technical issues often required users to leave Oppia and file a GitHub issue themselves — a flow most learners would never complete.

The result was that a lot of feedback simply never got submitted, and the feedback that did arrive lacked the **context needed to act on it**. The right teams were not always getting notified, and learners had **no visibility** into what happened after they reported something.

| **Area** | **Existing limitation** | **Why it mattered** |
|---|---|---|
| **Site feedback** | The existing feedback form was only linked from the footer and was not very discoverable | Users who encountered problems outside the lesson player had no obvious way to report them from where they encountered them |
| **Issue reporting** | There was no single in-platform flow for reporting technical issues across Oppia | Users could end up using different channels or leaving Oppia entirely to report a problem |
| **Actionable context** | Feedback did not consistently include enough context for teams to understand what happened | Reproducing and investigating an issue can be difficult when the report does not contain information about the user's experience |
| **Spam and low-quality submissions** | The existing site feedback form received spammy and non-actionable submissions | Feedback that cannot be acted upon adds noise and makes it harder for teams to identify useful reports |
| **Routing** | Feedback from different parts of the platform did not have a unified routing workflow | The right team needs to receive feedback without relying on users to know who should handle it |
| **Learner visibility** | Learners did not have a central place to see the feedback they had submitted | Users had limited visibility into what happened after they reported an issue or suggestion |

---
## A Walk Through the New Feedback Experience

The new feedback system is designed to cover the complete journey from submitting feedback to seeing what happens afterwards. Here is a walkthrough of the main experiences added during my GSoC period.

### 1. Changes in the "Add a new classroom" Modal

In Oppia, most lessons belong to a classroom, and each classroom has a dedicated Lessons Team responsible for preparing and maintaining those lessons. However, the classroom model did not previously store which Lessons Team was responsible for it, which made it difficult to reliably route learner feedback to the appropriate team.

To support the new feedback system, I introduced a required **Feedback Recipient Email** field in the "Add a new classroom" modal. This email identifies the Lessons Team responsible for the classroom and is used to **notify the team whenever a learner submits lesson feedback or reports an issue related to one of their lessons**. The same address is also used for the corresponding email notifications, so the team can be alerted without having to continuously check the feedback interface.

<img width="1920" height="892" alt="Screenshot from 2026-09-06 17-05-00" src="https://github.com/user-attachments/assets/2ae940b5-eb2b-44a0-ab7f-db2cc5f5ac86" />


> Since this field was added to the classroom model, I also had to migrated the existing classroom models to populate the new field.

### 2. Lesson Feedback Modal

While playing a lesson, learners can share suggestions to improve it by writing their feedback in the modal's textarea and submitting the form. The submission is directly sent to the **Lessons Team responsible for the classroom** to which the lesson belongs.

The system also captures important lesson context such as the **card name and index, exploration ID, and lesson version** at the time the feedback was submitted. This gives lesson creators the context they need to understand exactly where the feedback came from and review it against the appropriate version of the lesson.  

Logged-Out Learner View:
<img width="1920" height="892" alt="Screenshot from 2026-09-06 16-58-43" src="https://github.com/user-attachments/assets/8438ccbb-1d22-4fa5-bc05-f3eceacbb65a" />


Logged-In Learner View:
<img width="1920" height="892" alt="Screenshot from 2026-09-06 16-54-44" src="https://github.com/user-attachments/assets/7be8dd58-1245-40ed-81f8-a85c08edad86" />



### 3. Report a Lesson Issue

When a learner encounters a problem while playing a lesson, they can report it directly from the lesson player. The modal allows them to **select an issue category** such as **Typo, Broken layout / image, Confusing or incorrect answer, or Other / not sure**, making it easier to understand the type of problem being reported.

The report can also include a **description, screenshot, and technical error logs** to provide additional context. Based on the selected issue category, the feedback is routed to the appropriate workflow: reports such as **typos and confusing/incorrect answers are routed to the Creator Feedback Tab**, while other issue types are routed to the **Technical Feedback Dashboard** for the technical teams to triage.

Logged-Out Learner View (Has *Captcha integrated* for spam protection): 
<img width="1288" height="777" alt="Screenshot from 2026-09-06 17-24-16" src="https://github.com/user-attachments/assets/950a32a2-fcb8-42e9-a326-674c75629fe6" />


Logged-In Learner View:
<img width="1920" height="892" alt="Screenshot from 2026-09-06 17-18-58" src="https://github.com/user-attachments/assets/7fe0e0d5-b7e3-4e0c-becf-b40a53f67024" />



### 4. Report a Site Issue

Users can report platform-level issues from **anywhere on Oppia**, without needing to find a separate feedback page. The report form provides a dedicated space to describe what went wrong and optionally attach a **screenshot** to make the issue easier to understand.

The system also captures the **page URL** from which the report was submitted and can include **session logs** when the user chooses to share them. This gives the technical team useful context about where the issue occurred and what happened during the user's session, without requiring the user to manually provide all of this information.

Unlike lesson issue reports, site issues are not routed based on lesson or classroom ownership. Instead, the system uses the **URL path of the page where the report was submitted** to determine whether the issue belongs to the **LEAP or CORE team**. The report is then routed to the appropriate technical team for triage, and the team can use the **Technical Feedback Dashboard** to review the report and create a GitHub issue when further engineering work is required.

Logged-Out Learner View (Has *Captcha integrated* for spam protection): 
<img width="1920" height="896" alt="Screenshot from 2026-09-06 17-29-15" src="https://github.com/user-attachments/assets/60cd7541-cb4a-4957-8193-90cbed63c69b" />



Logged-In Learner View:
<img width="1920" height="896" alt="Screenshot from 2026-09-06 17-28-43" src="https://github.com/user-attachments/assets/cce891a1-1d83-4d35-8e49-4c900b39df06" />



### 5. Feedback Review for Lesson Creators

Once a lesson feedback or relevant lesson issue is submitted, it becomes available to the **Lessons Team** through the Creator Feedback Tab. Creators can open an individual feedback entry, review the learner's message along with the lesson context captured at submission time, and take action on it.

For lesson feedback, creators can **reply directly to the learner** and update the status of the feedback. The response is then shared back with the learner through the feedback system, keeping the interaction within Oppia instead of requiring a separate communication channel.

The video below walks through the complete workflow from the creator's side — opening a feedback entry, reviewing its details, responding to the learner, and updating its status.

**Creator Feedback Workflow:**

https://gist.github.com/user-attachments/assets/e7507f73-db6f-410a-a4d9-4b1a524c1da2


### 6. Technical Feedback Dashboard

Technical issue reports submitted by learners are collected in the **Technical Feedback Dashboard**, giving the LEAP and CORE teams a central place to review and triage issues. The dashboard provides the relevant details captured with each report, including the learner's description, page information, and available session context.

From the dashboard, team members can review an issue, update its status, and create a **GitHub issue** when the report requires further engineering work. This provides a structured workflow for taking a report from the initial learner submission to technical investigation and resolution.

**Technical Feedback dashboard Workflow:**


https://gist.github.com/user-attachments/assets/a0a9fdb9-de80-42e6-9819-f570c9fc9ac5


### 7. My Suggestions

The **My Suggestions** tab gives learners a place to view the feedback and suggestions they have submitted through Oppia. Each submission shows its current status, so learners can follow what happened after they reported something.

For lesson feedback, learners can also see **responses from lesson creators**, allowing the conversation to continue within Oppia. This closes the loop between submitting feedback and seeing how the lesson team responded to it.

List View of all submitted Suggestions by User.
<img width="1920" height="896" alt="Screenshot from 2026-09-06 17-51-26" src="https://github.com/user-attachments/assets/51398938-9f44-4841-9099-f24fa82bfd46" />

Detailed View of a particular suggestion.
<img width="1920" height="896" alt="Screenshot from 2026-09-06 17-51-43" src="https://github.com/user-attachments/assets/75d1e813-bd35-4907-9371-8f0c975a3edc" />


Video demo:

https://gist.github.com/user-attachments/assets/e7507f73-db6f-410a-a4d9-4b1a524c1da2

---

## System Architecture and Data Flow

The feedback system is built as a set of connected flows rather than a single feedback form. The main design goal was to keep the learner-facing experience simple while giving Oppia enough information to route, review, and eventually act on each submission.

### 1. End-to-End Feedback Flow

<img width="1177" height="1337" alt="End-to-End Feedback Flow" src="https://github.com/user-attachments/assets/b8276b10-187a-4398-9add-2c9d52e5997e" />


At a high level, every feedback submission follows the same pipeline:

<img width="6392" height="780" alt="Oppia Feedback Submission-2026-09-06-131230" src="https://github.com/user-attachments/assets/280cfe08-3732-46fe-acaf-6db3a843ec07" />


There are three entry points into the system:

- **Lesson player:** for lesson feedback and lesson issue reports.
- **Global feedback entry point:** for platform/site issue reports.
- **Internal feedback interfaces:** for lesson teams and technical teams to review submitted feedback.

Although the three submission types have different requirements, they share common infrastructure for validation, storage, screenshot handling, session information, routing, and data retention.

This separation was important because it keeps the learner-facing flows specific to the problem they are trying to report, while avoiding three completely independent implementations underneath.

---

### 2. Feedback Storage and Submission Pipeline

A submitted feedback item is first validated by the backend before being persisted.

The storage layer is built around a common `BaseFeedbackModel`, with concrete models for the two major categories of stored feedback:

- `LessonFeedbackModel` — lesson feedback and lesson issue reports.
- `PlatformFeedbackModel` — platform/site issue reports.

Additional information that does not belong directly to the feedback record is stored separately. In particular, session telemetry is represented through `FeedbackSessionLogModel`, while screenshots are stored in **Google Cloud Storage** rather than directly inside the feedback entity.

This gives the system a clear separation between:

**Feedback metadata** → what the user reported  
**Context** → where and when it was reported  
**Session information** → what happened during the user's session  
**Screenshot data** → visual evidence attached to the report

The backend service layer is responsible for validating these inputs, creating the appropriate feedback entity, and coordinating any additional storage or processing required for the submission.

---

### 3. Email Routing Logic

<img width="4751" height="5404" alt="Oppia Feedback Submission-2026-09-06-124759" src="https://github.com/user-attachments/assets/c1888cf2-34a3-4c1e-bb32-6cc88b242414" />


Routing was one of the important parts of the architecture because the person submitting feedback should not have to know which team is responsible for it.

For **lesson-related feedback**, routing is based on the classroom containing the lesson. Each classroom now stores a **Feedback Recipient Email**, which identifies the Lessons Team responsible for that classroom. This value is used both to determine the recipient and to send notification emails when new lesson feedback is submitted.

For **platform issue reports**, routing follows a different model. Since there is no classroom owner for a site-wide issue, the system uses the **URL path of the page from which the report was submitted** to determine whether it should be handled by the **LEAP** or **CORE** team.

This means routing is derived from information the system already knows about the user's context instead of asking the learner to select or identify the responsible team.

---

### 4. Session Information and Screenshot Capture

<img width="7992" height="3837" alt="Oppia Feedback Submission-2026-09-06-125047" src="https://github.com/user-attachments/assets/9ff501b1-14a1-4e31-90d4-90f7363629cf" />


One of the main design goals was to make reports useful for investigation without requiring learners to manually collect technical information.

When the user chooses to provide additional context, the frontend collects session information such as browser activity and navigation history. The feedback flow can also capture a screenshot of the state in which the issue occurred.

The session information and screenshot follow separate paths through the system:

**Session information → validation → `FeedbackSessionLogModel`**

**Screenshot → staging/upload flow → Google Cloud Storage**

Keeping these pieces separate prevents large binary data from being stored directly with the feedback entity while still allowing the feedback record to reference the relevant evidence.

The frontend also performs the collection of session information through a dedicated `FeedbackSessionInfoService`, which is responsible for capturing the required browser-side context before submission.

---

### 5. Feedback Review and Downstream Actions

After persistence and routing, the feedback enters one of the internal review workflows.

**Lesson feedback and lesson issue reports** are exposed through the **Creator Feedback Tab**, where lesson teams can review the report. Lesson feedback supports a response workflow, allowing creators to reply to the learner and update the status.

**Platform issue reports** are exposed through the **Technical Feedback Dashboard**, where the LEAP and CORE teams can triage reports and create GitHub issues when engineering work is required.

For the learner, submitted feedback is surfaced through **My Suggestions**, which provides visibility into the feedback they have submitted and, for applicable lesson feedback, the response from the lesson team.

This creates a complete feedback loop:

**Submit → Route → Review → Act → Communicate back to the learner**

---

### 6. Data Retention and Migration

#### Data retention

<img width="5536" height="5004" alt="Oppia Feedback Submission-2026-09-06-125147" src="https://github.com/user-attachments/assets/87fc2a5c-9ee1-424c-80c7-f8858075573c" />


Feedback data is subject to automated retention rules so that information is not stored indefinitely.

Platform feedback is deleted after **3 months**. Lesson feedback is retained for **6 months after resolution** or **12 months while still open**, with the retention jobs preserving the required metadata while clearing the feedback text according to the retention policy.

These rules are implemented through scheduled CRON jobs rather than requiring manual cleanup.

#### Legacy feedback migration

The new system also needed to coexist with Oppia's existing feedback data.

Historically, lesson feedback was stored using `GeneralFeedbackThreadModel`. Since the new system uses `LessonFeedbackModel`, I added a Beam migration job to move the existing records into the new representation.

The migration is designed to preserve historical learner feedback while allowing the legacy feedback implementation to be deprecated safely.

This gives the new architecture a clean storage model without losing the history accumulated by the previous system.

---

## Work Completed

### Milestone 1 — Learner-Facing Submission System

The full feedback entry point and submission pipeline for learners.

| PR | Description | Status |
|----|-------------|--------|
| [#26179](https://github.com/oppia/oppia/pull/26179) | Initial storage models, domain objects, and GCS screenshot entity support | ✅ Merged |
| [#26183](https://github.com/oppia/oppia/pull/26183) | Centralized session\_info validators in domain\_objects\_validator.py | ✅ Merged |
| [#26194](https://github.com/oppia/oppia/pull/26194) | Feedback service layer and Cloudflare Turnstile CAPTCHA verification service | ✅ Merged |
| [#26273](https://github.com/oppia/oppia/pull/26273) | Backend endpoints: POST /feedback, POST /report, GET /feedback\_captcha\_config | ✅ Merged |
| [#26195](https://github.com/oppia/oppia/pull/26195) | Global floating feedback entry button and feature flags | ✅ Merged |
| [#26223](https://github.com/oppia/oppia/pull/26223) | FeedbackBackendApiService and screenshot staging service | ✅ Merged |
| [#26260](https://github.com/oppia/oppia/pull/26260) | FeedbackSessionInfoService — console patching, HTTP interception, router-history tracking | ✅ Merged |
| [#26397](https://github.com/oppia/oppia/pull/26397) | Three feedback modal components (SiteFeedback, LessonFeedback, ReportAnIssue), CAPTCHA integration, revert of superseded PRs | ✅ Merged |
| [#26459](https://github.com/oppia/oppia/pull/26459) | Redesigned backend models and endpoints aligned with updated CUJs, revert of superseded backend PRs | ✅ Merged |
| [#27133](https://github.com/oppia/oppia/pull/27133) | Learner-side acceptance tests for CUJs LO.15, LO.16, LI.7 | ✅ Merged |
| [#26651](https://github.com/oppia/oppia/pull/26651) | Promoted exploration\_id to top-level indexed field in BaseFeedbackModel; fixed description validation | ✅ Merged |
| [#26786](https://github.com/oppia/oppia/pull/26786) | Configurable dismiss button for Oppia toast messages; enabled for feedback submission success toast | ✅ Merged |

**Milestone 1 is complete.** All user-facing feedback submission journeys — floating entry button, three modal types, screenshot upload, CAPTCHA for logged-out users, session telemetry, wipeout/takeout compliance — are implemented and tested.

---

### Milestone 2 — Internal Dashboards and Moderation Infrastructure

The review and moderation surfaces for internal teams.

| PR | Description | Status |
|----|-------------|--------|
| [#26617](https://github.com/oppia/oppia/pull/26617) | Technical Feedback Dashboard backend: TECH\_TEAM\_LEAD role, handlers, domain objects, services | ✅ Merged |
| [#26732](https://github.com/oppia/oppia/pull/26732) | Stable id fields on GitHub Issue Form for URL-based pre-population of issue templates | ✅ Merged |
| [#26734](https://github.com/oppia/oppia/pull/26734) | Feedback recipient email field on Classroom model + Beam migration job for existing classrooms | ✅ Merged |
| [#26749](https://github.com/oppia/oppia/pull/26749) | Reusable feedback UI components Part 1: feedback-table, feedback-filter-bar, feedback-status-chip, feedback-empty-state | ✅ Merged |
| [#26776](https://github.com/oppia/oppia/pull/26776) | Reusable UI components Part 2 + backend APIs for Creator Feedback Tab and My Suggestions | ✅ Merged |
| [#26824](https://github.com/oppia/oppia/pull/26824) | CRON jobs for automated data retention (3-month platform, 6/12-month lesson) | ✅ Merged |
| [#27004](https://github.com/oppia/oppia/pull/27004) | Creator Feedback Tab assembly — lesson feedback, issue reports, reply workflow, status updates | ✅ Merged |
| [#27081](https://github.com/oppia/oppia/pull/27081) | Analytics event tracking (modal impression, submission) + email notification pipeline | ✅ Merged |
| [#27112](https://github.com/oppia/oppia/pull/27112) | MigrateLegacyFeedbackJob — Beam job to migrate GeneralFeedbackThreadModel to new LessonFeedbackModel | 🔄 In Review |
| [#27152](https://github.com/oppia/oppia/pull/27152) | My Suggestions tab — learner dashboard view for submitted feedback, creator replies, status tracking |✅ Merged  |
| [#27133](https://github.com/oppia/oppia/pull/27133) | Acceptance tests for full Milestone 2 scope | 🔄 In Progress |

---
## Quality Assurance

The feedback system was tested across the different user roles and layers of the application.

- **Acceptance Tests:** Covered the complete CUJs for **logged-in learners, logged-out learners, lesson creators, technical team leads, and admins**.  [CUJs for Web user Feedback](https://docs.google.com/spreadsheets/d/17Y5yOGuy0y5YFPPCUF5fZAZjEyY60bq6J0lik6yH3KE)
- **Frontend Unit Tests:** Added and updated **Karma/Jasmine tests** for the feedback components and services.
- **Backend Unit Tests:** Added **Python unit tests** covering the feedback models, services, handlers, validation, routing, and related backend logic.

The acceptance tests ensure the end-to-end user journeys work as defined in the CUJs, while the frontend and backend unit tests provide coverage for the individual components that make up those flows.

---

## Challenges and Design Decisions

### Mid-Project Architectural Pivot

The most significant challenge of this project was not technical — it was a full requirements change midway through Milestone 1. The CUJs for the learner-facing feedback system were redesigned after several PRs had already merged. The old architecture used a thread-and-message model with a back-and-forth conversation flow. The new CUJs described a single-submission model with no thread structure.

This required reverting multiple merged PRs, redesigning the storage models, endpoints, services, and frontend flows from scratch, and coordinating approvals across PMs, UXR, design, and senior maintainers before implementation could continue. The revised architecture is cleaner and better aligned with the actual product requirements, but getting there took about two weeks of planning before any code could be written.

### CI-Only Acceptance Test Failure

The learner feedback acceptance tests passed locally in non-headless Chromium but failed in CI headless mode. After several days of debugging — including help from peer contributors Tanmay and Mohak — the cause was traced to the `--disable-site-isolation-trials` Chromium flag, which had been added as a workaround for a separate Puppeteer issue. Disabling site isolation affected how Cloudflare Turnstile's cross-origin iframe initialized in headless mode. The fix was making the browser launcher conditionally skip the flag only for the two affected feedback acceptance test specs while preserving the workaround everywhere else.

---

## Future Scope

**My Suggestions notifications**: the notification system for My Suggestions (alert dot on profile icon, notification micro-cards in profile dropdown) has known open design questions around notification lifecycle and read state that were still being discussed with the product and design teams at the end of the program. The tab itself is built; the notification layer is partially implemented. [Issue filed: #27159](https://github.com/oppia/oppia/issues/27159)

**Legacy Feedback system deprecation**: removing the old `GeneralFeedbackThreadModel` endpoints, frontend components, and storage models is blocked on the migration job being validated on the production environment.

---

## Key Links

| Resource | Link |
|----------|------|
| **Pre-Launch Docs** | [M1-Doc](https://docs.google.com/document/d/13DhR2uTVHOSw5aEdXIR34h53vF0-RJOdTLNJQ5WuNa8/edit?tab=t.4t336fwwj9ly#heading=h.fp3tr0o80ub), [M2-Doc](https://docs.google.com/document/d/1ny4mBjmFlkJ8dj05dkrJZSjrdorTBpdgMvCDSs2O1OA/edit?usp=drive_web&ouid=103353750459402371711) |
| **PM Demo Video** | [Drive video-link](https://drive.google.com/drive/folders/145HNlBSGKJXSIZwP1ZTazs8xp0sB4EPK) |
| **GSoC 1.1 Web User Feedback Approval Docs** | [Technical Feedback Dashboard](https://docs.google.com/document/d/1ezAr4NfqEhnip-HNXw7fOVvvLu1qt7Q-4D4jFP4VVlU/edit?tab=t.qrfqgsnmsdcj), [Creator Feedback Dashboard](https://docs.google.com/document/d/1ezAr4NfqEhnip-HNXw7fOVvvLu1qt7Q-4D4jFP4VVlU/edit?tab=t.8m1ajq1ycib0) |
| **UI Mocks** | [Mocks Document](https://docs.google.com/document/d/1ITlUe5c6khtH18ErIG8fJIWI05s_6fzM5-E_5sXAikY/edit?tab=t.0) |
| **Project Google Drive |[Drive Link](https://drive.google.com/drive/folders/145HNlBSGKJXSIZwP1ZTazs8xp0sB4EPK) |
| **CUJs** | [Critical User Journeys](https://docs.google.com/spreadsheets/d/17Y5yOGuy0y5YFPPCUF5fZAZjEyY60bq6J0lik6yH3KE) |
| **GSoC Journals** | [medium.com/@imnitishkumar04](https://medium.com/@imnitishkumar04) |
| **GSoC Project Page** | [Google Summer of Code](https://summerofcode.withgoogle.com/programs/2026/projects/DLicszJv) |

---
## Learnings

This summer taught me that building software is a lot more than writing code.

I got to work with PMs, designers, senior maintainers, and contributors across multiple time zones. I learned what it actually takes to ship a feature end to end on a real product used by real learners the planning, the coordination, the decisions, the reviews, all of it.

The system I built looks different from what I proposed in April. It is better. And that is the thing I will carry longest from this summer **that good software is shaped by the people around you, not just the person writing the code**.

---

## Acknowledgements

Thank you to Mohit Ruwatia for the mentorship throughout this project.He gave me the freedom to work at my own pace while making sure I always had the right guidance whenever I got stuck. There were weeks where the entire scope changed and having someone who helped me think through the problem rather than just telling me what to do made the difference.

Thank you to Sean Lip, Chris, Aanu Adeoti, Janee Vue, Hardik Goyal, and the rest of the Oppia community for the design reviews, technical feedback, and product discussions that shaped this system. Thank you also to Tanmay and Mohak for helping debug the CI acceptance test failure.

And thank you to Google Summer of Code and the Oppia Foundation for the opportunity to build something that will be used by real learners.
