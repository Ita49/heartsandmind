# V4S Implementation Plan
## Hearts and Mind — Phase 2: Volunteer Management
### Step-by-Step Gated Implementation Guide

**Document Version:** 1.0  
**Date:** April 2026  
**Audience:** Implementation Team (BAs, Developers, Client)  
**Approach:** Gated — each gate must be confirmed complete before proceeding

---

## Guiding Principles

1. **Gated progress** — no phase begins until the previous gate is signed off
2. **Build smallest working slice first**, then extend
3. **Test in sandbox**, confirm with client, then move to production
4. **Student volunteers are the priority** (they are the majority)
5. **Do not break Phase 1** (support request / case intake is live)

---

## High-Level Phases Overview

| Phase | Name | Jira Epics | Gate |
|-------|------|-----------|------|
| **0** | Foundation & Data Model | HMP-38, HMP-28 | Data model confirmed by client |
| **1** | Student Intake (School → Record) | HMP-38, HMP-70, HMP-72 | Student record created end-to-end |
| **2** | Onboarding & Compliance | HMP-73, HMP-74, HMP-64 | Documents verified, agreement signed in system |
| **3** | General Volunteer Intake | HMP-113, HMP-28 | Walk-in volunteer record + confirmation email |
| **4** | Volunteer Matching & Assignment | HMP-43, HMP-71, HMP-76, HMP-81 | Volunteer assigned to case with rules applied |
| **5** | Hours & Feedback Tracking | HMP-77, HMP-67 | Hours logged; visit report captured |
| **6** | Exit & Lifecycle Management | HMP-79, HMP-116 | Exit decision recorded; access revoked |
| **7** | Reporting & Dashboards | HMP-32, HMP-95 | Dashboards live |

---

## Phase 0 — Foundation & Data Model

### Objective
Set up the Salesforce data model so all subsequent phases have a stable foundation.

### Steps

#### Step 0.1 — Create Contact Record Types
**Where:** Setup → Object Manager → Contact → Record Types

Create the following record types on Contact:
- `Student_Volunteer` — for students on placement
- `General_Volunteer` — for walk-in / community volunteers

> 💡 Rookie Tip: Go to Setup → type "Object Manager" in the search bar → click Contact → click "Record Types" from the left panel → click New.

**Gate 0.1:** Both record types visible on Contact. ✅

---

#### Step 0.2 — Create Custom Fields on Contact

**Where:** Setup → Object Manager → Contact → Fields & Relationships

Create these fields (all for both record types unless noted):

| Field Label | API Name | Type | Notes |
|-------------|----------|------|-------|
| Volunteer Type | Volunteer_Type__c | Picklist | Student / General |
| Drives | Drives__c | Checkbox | For proximity/driving matching |
| Tech Savvy | Tech_Savvy__c | Checkbox | For system access decisions |
| Volunteer Status | Volunteer_Status__c | Picklist | Active / Inactive / Pending / Exited |
| Gender | Gender__c | Picklist | Male / Female / Non-Binary / Prefer Not to Say |

> 💡 Rookie Tip: Fields & Relationships → New → choose type → fill in label → API name auto-fills → add to page layouts at the end.

**Gate 0.2:** All fields appear on Contact record in sandbox. ✅

---

#### Step 0.3 — Create Student_Placement__c Custom Object

**Where:** Setup → Object Manager → Create → Custom Object

| Property | Value |
|----------|-------|
| Object Label | Student Placement |
| Object API Name | Student_Placement__c |
| Record Name | Student Name (Text) |
| Allow Activities | Yes |
| Allow Reports | Yes |

**Fields to create on Student_Placement__c:**

| Field Label | API Name | Type | Notes |
|-------------|----------|------|-------|
| Student | Student__c | Lookup (Contact) | Master lookup |
| School | School__c | Text (255) | School name |
| Programme | Programme__c | Picklist | PSW / CDSW / DSW / BSW / MSW / Addictions & Mental Health |
| Volunteer Category | Volunteer_Category__c | Picklist | Basic / Advanced |
| Placement Start Date | Placement_Start_Date__c | Date | |
| Placement End Date | Placement_End_Date__c | Date | |
| Required Placement Hours | Required_Hours__c | Number | Hours required by programme |
| Placement Status | Placement_Status__c | Picklist | Applied / Interview Scheduled / Verified – Ready for Onboarding / Active / Completed / Rejected |
| Orientation Completed | Orientation_Completed__c | Checkbox | |
| Agreement Status | Agreement_Status__c | Picklist | Not Sent / Sent / Signed |
| Documents Verified | Documents_Verified__c | Checkbox | All 5 docs received and checked |
| VSC Received | VSC_Received__c | Checkbox | Vulnerable Sector Check |
| Immunisation Received | Immunisation_Received__c | Checkbox | |
| CPR Received | CPR_Received__c | Checkbox | |
| CPI Received | CPI_Received__c | Checkbox | |
| Interview Date | Interview_Date__c | DateTime | |
| Interview Notes | Interview_Notes__c | Long Text | |
| Exit Decision | Exit_Decision__c | Picklist | Stay as Volunteer / End Placement |
| Remote Hours Total | Remote_Hours_Total__c | Number | Running total |
| In-Person Hours Total | In_Person_Hours_Total__c | Number | Running total |

**Gate 0.3:** Student Placement object created with all fields. Record can be created and saved in sandbox. ✅

---

#### Step 0.4 — Add Student Placement to Contact Page Layout

**Where:** Setup → Object Manager → Contact → Page Layouts

Add the Student Placement related list to the Student_Volunteer record type page layout.

**Gate 0.4:** Student Placement related list visible on a Contact with record type = Student Volunteer. ✅

---

## Phase 1 — Student Intake (School → Salesforce Record)

### Objective
When a school submits a student, a Salesforce record is created and the coordinator is notified. This replaces the manual email + Drive folder creation process.

### Steps

#### Step 1.1 — Build the School Submission Web-to-Lead Form (or Experience Cloud Form)

> ⚠️ Important: Per HMP-68 (Document Upload Without Website), the public website form must NOT include document upload fields. Documents are submitted through a separate secure channel.

**Option A (Simplest — Web-to-Lead):**
Go to Setup → Web-to-Lead → Create Web-to-Lead Form

Include these fields:
- First Name, Last Name (student)
- Email
- Phone
- School Name (map to custom field)
- Programme (picklist)
- Placement Start Date (requested)
- Cover Letter submitted: Yes/No checkbox
- Resume submitted: Yes/No checkbox

The form creates a **Lead** in Salesforce. A flow then converts this to a Contact with Record Type = Student_Volunteer + creates a linked Student_Placement__c record.

**Option B (Better — Salesforce Experience Cloud form):**  
Build a guest-accessible form page that creates a Contact + Student_Placement__c directly. Use this if Experience Cloud is already partially configured.

> 💡 Rookie Decision: Start with Option A (Web-to-Lead) for speed, then migrate to Option B in a later phase when Experience Cloud is set up.

**Gate 1.1:** School submits form → Lead (or Contact) record appears in Salesforce. ✅

---

#### Step 1.2 — Lead-to-Contact Conversion Flow (for Web-to-Lead path)

**Where:** Setup → Flows → New Flow → Record-Triggered Flow

Trigger: When a Lead is created AND Lead Source = "Student Placement"

Actions:
1. Convert Lead → create Contact with Record Type = Student_Volunteer
2. Create Student_Placement__c record linked to new Contact
3. Populate fields from Lead (Programme, School, Start Date)
4. Send email notification to coordinator: "New student submission from [School] — [Student Name]"

**Gate 1.2:** Lead created → Contact + Student Placement created automatically → Coordinator receives email. ✅

---

#### Step 1.3 — Interview Scheduling

This step is manual (coordinator schedules via calendar), but the system must log it.

**What to build:**
- Add an "Interview Scheduled" button on the Student Placement record
- Clicking this triggers a flow that:
  1. Updates Placement_Status__c to "Interview Scheduled"
  2. Creates a Salesforce Task/Event linked to the record
  3. Stores Google Meet link in Interview_Date__c (or a text field)

**Gate 1.3:** Coordinator can mark interview as scheduled; date/status stored on Student Placement record. ✅

---

#### Step 1.4 — Document Verification Checklist

**Where:** Student Placement record, Document Verification section

Coordinator manually checks off each document as it's received from the school:
- VSC Received ✅
- Immunisation Received ✅
- CPR Received ✅
- CPI Received ✅
- Resume + Cover Letter ✅

**Automation:** Build a Flow that:
- When all 5 checkboxes = true
- Updates Documents_Verified__c = true
- Updates Placement_Status__c = "Verified – Ready for Onboarding"
- Sends confirmation email to coordinator

**Gate 1.4:** All docs checked → status auto-updates → email sent. ✅

---

## Phase 2 — Onboarding & Compliance

### Objective
Issue and track service agreements, orientation, and onboarding checklist.

#### Step 2.1 — Service Agreement Generation (HMP-73, HMP-66)

**Approach:** Use Salesforce native document generation (or DocuSign if available)

Simple approach (no DocuSign):
1. Create an email template: "Hearts and Mind Student Placement Agreement"
2. Create a button on Student Placement: "Send Service Agreement"
3. Button triggers a Flow → sends templated email to student Contact
4. Coordinator manually updates Agreement_Status__c = Signed when returned

Advanced approach (with DocuSign / Conga):
- Generate PDF agreement from template
- Send via DocuSign
- On signing, webhook updates Agreement_Status__c = Signed automatically

> 💡 Rookie Default: Use the simple email approach first. Upgrade to DocuSign in a later sprint.

**Gate 2.1:** "Send Agreement" button works → student receives email → coordinator can mark as Signed. ✅

---

#### Step 2.2 — Onboarding Orientation Tracking (HMP-74)

Add a section to the Student Placement page layout: "Onboarding Checklist"

Checklist items (checkbox fields):
- [ ] Placement handbook sent
- [ ] Orientation session completed
- [ ] Task calendar shared
- [ ] Added to volunteer communication group
- [ ] First weekly meeting held

Build a formula field: `Onboarding_Completion_Pct__c` = (checked items / total) × 100

**Gate 2.2:** Checklist visible and functional; % completion calculates correctly. ✅

---

#### Step 2.3 — Secure Document Storage (HMP-64)

**Where:** Student Placement record → Files related list

Configure:
1. All uploaded files on Student Placement default to "Private" sharing (not public)
2. Create a Permission Set: "Placement Coordinator" — grants access to Files on Student Placement
3. Volunteers/students do NOT have access to Files unless explicitly granted

> 💡 Rookie Tip: In Salesforce, go to the Files related list on a record → each file has a sharing setting. Set the default org-wide setting for Files to "Private" in Setup → Sharing Settings.

**Gate 2.3:** Coordinator can upload a document; a non-coordinator user cannot see it. ✅

---

## Phase 3 — General Volunteer Intake

### Objective
Build the standard V4S web intake for walk-in / community volunteers.

#### Step 3.1 — Volunteer Website Form (HMP-113)

V4S ships with a default volunteer signup page (Visualforce page: `GW_Volunteers__VolunteersJobListing`). This can be embedded on the Hearts and Mind website.

**Customise the form to capture:**
- Name, Email, Phone
- Interested volunteer role (picklist)
- Drives: Yes/No
- Relevant experience
- Consent checkbox

**Automation (V4S standard + custom):**
When volunteer submits form:
1. V4S creates a Contact with Record Type = General_Volunteer
2. Auto-sends acknowledgement email (configure in V4S Settings)
3. Creates a Task for coordinator: "Review new volunteer application"

**Gate 3.1:** Walk-in volunteer submits form → Contact created → confirmation email received → coordinator notified. ✅

---

## Phase 4 — Volunteer Matching & Assignment

### Objective
Coordinators can search, filter, and assign volunteers to a Case (support request) with safety rules enforced.

#### Step 4.1 — Add Matching Fields to Contact

Ensure these fields exist and are populated on all volunteer Contacts:
- Gender__c
- Drives__c
- Skills__c (multi-select picklist: Meal Prep, Laundry, Light Housekeeping, Errands, Mental Health Support, Outreach, Admin)
- Address / Geolocation (for proximity — use standard Address field + enable Geocoding in Setup)
- Availability (text or picklist for now)

**Gate 4.1:** Fields exist and are populated on test volunteer records. ✅

---

#### Step 4.2 — Proximity Matching (HMP-71)

**Approach:**
1. Enable Data Integration Rules for address geocoding: Setup → Data Integration Rules → Geocodes for Contacts
2. This auto-populates Latitude/Longitude on Contact address
3. On the Case record, caregiver address is also geocoded
4. Build a Flow or use a custom formula to calculate distance between volunteer and caregiver

> 💡 Note: Full automatic proximity ranking requires Apex code or a matching tool. The simplest Phase 4 approach is to display volunteer distance as a field on a list view so coordinators can sort manually.

**Gate 4.2:** Volunteer and caregiver addresses are geocoded. Distance can be viewed on a list report. ✅

---

#### Step 4.3 — Gender Pairing Validation (HMP-81)

**Build a Validation Rule on Volunteer Hours (assignment):**
- If the assignment is for a home visit AND caregiver gender includes Female (or children present)
- AND both assigned volunteers are Male
- THEN show error: "Gender safety rule violation — at least one female volunteer required."

This requires knowing the gender of both assigned volunteers. If using V4S Shifts, add a custom field to the Shift record to capture team composition.

**Gate 4.3:** System blocks a male-only pairing for a female-present household with a clear error message. ✅

---

#### Step 4.4 — Driving Capability Filter (HMP-76)

**On the V4S Volunteer Job / matching screen:**
- Filter volunteers by Drives__c = true when caregiver is beyond transit distance
- Ensure at least 1 volunteer in every 2-person team has Drives = Yes

Build as a validation rule on Volunteer Shift assignment:
- If team size = 2 AND no volunteer has Drives = Yes → block with error

**Gate 4.4:** System blocks assignment of 2 non-driving volunteers to a far-distance case. ✅

---

#### Step 4.5 — Assign Volunteers to Case (HMP-85)

**Where:** Case record → related list → Volunteer Jobs

Steps:
1. On the Case record, a "Assign Volunteers" button triggers a Flow
2. Flow opens a modal to select 1–3 volunteers
3. Flow creates a V4S Volunteer Job linked to the Case
4. Flow creates a V4S Volunteer Shift with date/time
5. Flow creates Volunteer Hours records for each assigned volunteer
6. Flow sends email confirmation to each assigned volunteer
7. Updates Case Status to "Volunteers Assigned"

**Gate 4.5:** Coordinator clicks Assign Volunteers → selects volunteers → shift created → emails sent → case updated. ✅

---

## Phase 5 — Hours & Feedback Tracking

#### Step 5.1 — Hours Logging (HMP-77)

Add to Student Placement record:
- Weekly Hours Log (custom related object or V4S Hours): Remote hours + In-person hours per week
- Flow: When week ends (scheduled Flow), if no hours logged → send reminder email to student
- Coordinator approval: Hours__c.Status = Pending → Verified

**Gate 5.1:** Student can have hours logged; coordinator can verify; running total updates. ✅

---

#### Step 5.2 — Post-Visit Feedback (HMP-67)

Create a custom object: `Visit_Report__c`

Fields: Date, Volunteer, Case, Home Visit Notes (long text), Coordinator Follow-Up Needed (checkbox), Referral Required (checkbox, for Advanced students)

After each shift, an automated email is sent to the volunteer with a link to a Salesforce Experience Cloud form (or simple web form) to submit their visit report.

**Gate 5.2:** Volunteer receives post-visit email → submits report → record appears in Salesforce linked to Case and Volunteer. ✅

---

## Phase 6 — Exit & Access Management

#### Step 6.1 — Student Exit Process (HMP-79)

When Placement_End_Date__c is reached:
- Trigger a Flow: coordinator receives prompt — "Placement ending for [Student]. Stay as volunteer: Yes/No?"
- If Yes: Copy Contact to General_Volunteer record type, retain history
- If No: Update Volunteer_Status__c = Exited

**Gate 6.1:** Exit decision captured; student either transitions to General Volunteer or is marked Exited. ✅

---

#### Step 6.2 — Access Revocation (HMP-116)

When a volunteer's status changes to Exited or Inactive:
- Flow deactivates their Salesforce User record (if they had one)
- Sends notification to admin: "[Volunteer Name] system access has been revoked"
- Coordinator manually changes shared outreach email password (documented procedure)

**Gate 6.2:** Volunteer status → Exited triggers access revocation notification and User deactivation. ✅

---

## Phase 7 — Reporting & Dashboards

#### Step 7.1 — Volunteer Utilization Dashboard (HMP-95)

Reports to build:
1. Volunteers by status (Active / Pending / Exited)
2. Student volunteers by programme
3. Hours logged this month (remote vs in-person)
4. Volunteers assigned to active cases
5. Students nearing placement end date (within 30 days)

**Gate 7.1:** Dashboard live with 5 reports; data is accurate against test records. ✅

---

## Overall Implementation Timeline (Estimate)

| Phase | Duration | Dependencies |
|-------|----------|-------------|
| Phase 0 — Data Model | 1 week | Sandbox access confirmed |
| Phase 1 — Student Intake | 1–2 weeks | Phase 0 gate cleared |
| Phase 2 — Onboarding | 1 week | Phase 1 gate cleared |
| Phase 3 — General Volunteer | 0.5 weeks | Phase 0 gate cleared (can run parallel to Phase 2) |
| Phase 4 — Matching & Assignment | 2 weeks | Phase 2 + 3 gates cleared |
| Phase 5 — Hours & Feedback | 1 week | Phase 4 gate cleared |
| Phase 6 — Exit & Access | 0.5 weeks | Phase 5 gate cleared |
| Phase 7 — Reporting | 1 week | All prior gates cleared |
| **Total** | **~9 weeks** | |

---

## Roles & Responsibilities

| Role | Responsibilities |
|------|----------------|
| BA (Taiwo, BC) | User story refinement, UAT, client communication |
| Developer (Ital + team) | Flow builds, custom objects, validation rules, automation |
| Client (Abigail / Hearts and Mind) | UAT sign-off at each gate, content for email templates |
| Project Manager | Gate tracking, sprint planning, risk management |

---

## Definition of Done (per Story)

A story is complete when:
- [ ] Built in sandbox
- [ ] BA has tested against all acceptance criteria
- [ ] Client (Abigail) has reviewed and confirmed
- [ ] No regression in Phase 1 (support request intake still works)
- [ ] Moved to production

---

*End of Implementation Plan*
