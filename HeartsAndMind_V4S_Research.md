# V4S Implementation Research Document
## Hearts and Mind — Salesforce Volunteer Management Phase 2

**Document Version:** 1.0  
**Date:** April 2026  
**Project:** Hearts and Mind Salesforce Integration — Phase 2: Volunteer Management  
**Prepared By:** Cloud Deck Inc. / BA Team

---

## 1. Project Context

Hearts and Mind is a nonprofit organisation supporting foster families (caregivers) by coordinating volunteer support. **Phase 1** (already live) covers caregiver support requests — a web-to-lead intake form, 48-hour SLA tracking, and pod-based routing. **Phase 2** (this document) covers the volunteer supply side: intake, onboarding, matching, and assignment.

---

## 2. Technology Stack in Scope

| Component | Description | Status |
|-----------|-------------|--------|
| Salesforce NPSP | Core CRM foundation | Installed |
| Volunteers for Salesforce (V4S) | Volunteer job/shift management | Installed |
| Program Management Module (PMM) | Program/activity tracking | Installed |
| Salesforce Files | Secure document storage | Available |
| Flow / Process Builder | Automation layer | Available |
| Experience Cloud (future) | Volunteer portal | Planned |

---

## 3. Two Classes of Volunteers — Critical Distinction

This is the most architecturally significant finding from the discovery session.

### 3A. Student Volunteers (Primary / Majority)

**Source:** Colleges and universities via formal placement agreements  
**Entry Path:** School contacts Hearts and Mind → Student is redirected back to school → School formally submits student to Hearts and Mind  
**Programs:** PSW (Personal Support Worker), CDSW, DSW, Addictions & Mental Health, BSW (Bachelor of Social Work), MSW (Master of Social Work)

**Sub-categories:**
- **Basic Students** (PSW, CSW, CDSW, DSW): Perform day-to-day in-home tasks — laundry, meal prep, light housekeeping, errands
- **Advanced Students** (Addictions & Mental Health, BSW, MSW): Provide emotional/mental health oversight, supervise basic students on home visits, escalate referrals

**Required Documents (from school):**
1. Vulnerable Sector Check (VSC) / Criminal Record Check
2. Immunisation records
3. First Aid & CPR certificate
4. CPI (Crisis Prevention Intervention) certificate
5. Resume + Cover Letter

**Placement Hours:** Vary by programme (e.g., PSW ≠ CDSW ≠ BSW hours). Target ~40 hrs/week equivalent measured by task execution, not time-tracking alone.

**Work Split:** ~30% in-person home visits, ~70% virtual (research, outreach, event support, weekly reports)

**Exit Path:** Placement ends → option to stay on as regular volunteer or depart

### 3B. General / Walk-In Volunteers

**Source:** Charity Village posts, website volunteer form, referrals  
**Entry Path:** Submit interest via website form → coordinator reviews → informal interview → onboarding  
**Volume:** Low (approx. 3 recorded to date)  
**V4S Fit:** Standard V4S Web-to-Lead → Volunteer Contact flow works well for this group

---

## 4. Current Process (As-Is) — Detailed

### 4A. Student Intake Flow
```
School reaches out OR student expresses interest
→ Student redirected back to school
→ School formally submits student (resume + cover letter)
→ Hearts and Mind reviews → schedule interview
→ Interview conducted
→ Accept: proceed | Reject: regret email to school
→ Documents requested (VSC, immunisation, CPR, CPI, etc.)
→ Service Agreement issued → signed
→ Orientation & Onboarding (2–3 weeks / first month)
→ Task assignment begins (tied to events/programmes)
→ Weekly task reports submitted via Google Doc (in student's folder)
→ Weekly team meeting (review progress, challenges)
→ Home visit assignments with 48-hour notice
→ Post-visit feedback documented in folder
→ End of placement → exit decision (stay or leave)
```

### 4B. General Volunteer Flow
```
Volunteer submits interest (website form or direct email)
→ Coordinator reviews
→ Informal interview (phone/video)
→ Document check (varies by role — VSC for respite, simpler for event support)
→ Onboarding
→ Task assignment
```

### 4C. Home Visit Coordination
```
Support request raised by caregiver (Phase 1 case)
→ Coordinator identifies suitable volunteers
→ Team assigned (min 2 per visit)
→ Gender pairing rules applied (no 2 males if female present)
→ 48-hour notice sent to volunteers (email)
→ Meet & Greet for first visit (introduction to caregiver)
→ New team members introduced on subsequent visits
→ Visit occurs
→ Volunteer submits post-visit feedback (Google Doc)
→ Caregiver feedback collected (Google Form → Drive)
```

---

## 5. Current Pain Points (As-Is)

| # | Pain Point | Source |
|---|-----------|--------|
| 1 | Student intake entirely manual — Google Drive folders per student | Discovery session |
| 2 | Document tracking (VSC, CPR, etc.) done manually in Drive | Discovery session |
| 3 | No formal hours tracking — relies on task output as proxy | Discovery session |
| 4 | WhatsApp used for informal coordination (not auditable) | Discovery session |
| 5 | Feedback from volunteers stored as freeform Google Docs | Discovery session |
| 6 | Caregiver feedback via Google Form — manual follow-up | Discovery session |
| 7 | No system-enforced gender pairing rules | Jira HMP-81 |
| 8 | No proximity-based volunteer matching | Jira HMP-71 |
| 9 | V4S standard flow doesn't accommodate school-mediated student intake | Discovery session |
| 10 | No Salesforce access for volunteers (all on Drive/WhatsApp/Email) | Discovery session |
| 11 | Post-placement student transitions tracked manually | Jira HMP-79 |
| 12 | No centralised task calendar in CRM | Discovery session |

---

## 6. V4S Architecture — How It Works (Primer for Rookies)

Volunteers for Salesforce (V4S) is a free Salesforce Labs managed package. It extends Salesforce with:

| Object | Purpose |
|--------|---------|
| **GW_Volunteers__Volunteer_Job__c** | Defines a role/job for a campaign (e.g., "Home Visit Support") |
| **GW_Volunteers__Volunteer_Shift__c** | A specific time/date slot under a Job |
| **GW_Volunteers__Volunteer_Hours__c** | Records a specific volunteer's hours on a shift |
| **GW_Volunteers__Volunteer_Recurrence_Schedule__c** | Recurring availability |

V4S ties volunteers to **Campaigns** in Salesforce. Each campaign represents a programme or event. Volunteers sign up for Jobs within a campaign, then get assigned to Shifts.

**Key insight from discovery:** V4S works natively for general volunteers (web form → Contact → Volunteer Hours). For student volunteers, custom fields, record types, and a modified intake path are needed.

---

## 7. PMM Architecture — How It Works

The Program Management Module (PMM) tracks:
- **Programs** (e.g., Heart & Home Support, Reflect & Renew Workshop)
- **Services** (activities within a program)
- **Program Engagements** (a specific Contact's participation in a Program)
- **Service Deliveries** (instances of service delivered)

**Overlap with V4S:** Both PMM and V4S can track volunteer involvement in programmes. The recommended approach (from discovery session) is to use them together:
- PMM = programme structure + participant tracking (caregivers enrolled, activities, outcomes)
- V4S = volunteer supply side (jobs, shifts, hours, assignments)

---

## 8. Data Model — Proposed Objects & Relationships

```
Contact (Volunteer)
  ├── Record Type: Student Volunteer
  ├── Record Type: General Volunteer
  ├── Custom Fields: Programme, School, Placement Hours Required, 
  │   Drives (Y/N), Gender, Placement Status, Tech Savvy (Y/N)
  │
  ├── Student_Placement__c (custom object)
  │   ├── School__c (lookup)
  │   ├── Programme__c (picklist: PSW, CDSW, DSW, BSW, MSW, AMH)
  │   ├── Volunteer_Category__c (Basic / Advanced)
  │   ├── Placement_Start_Date__c
  │   ├── Placement_End_Date__c
  │   ├── Required_Hours__c
  │   ├── Orientation_Completed__c
  │   ├── Agreement_Status__c
  │   ├── Documents_Verified__c
  │   └── Exit_Decision__c (Stay / End)
  │
  ├── V4S Volunteer Hours (standard V4S)
  │   ├── Linked to: Volunteer Job / Shift
  │   └── Remote vs In-Person Hours (custom)
  │
  └── PMM Program Engagement
      └── Linked to: Programme record

Case (Support Request — Phase 1, already live)
  └── Volunteer Job__c (V4S) linked to Case

Campaign (Programme/Event)
  ├── Volunteer Job__c (V4S)
  │   └── Volunteer Shift__c
  │       └── Volunteer Hours__c (who was assigned)
  └── PMM Program (parallel tracking)
```

---

## 9. Jira Story Mapping — Phase 2 Scope

### Epic: Students - Placement & Onboarding (HMP-38)

| Jira ID | Story | Priority |
|---------|-------|----------|
| HMP-70 | Capture Student Submission From School | P1 |
| HMP-72 | Student Interview & School Verification Workflow | P1 |
| HMP-73 | Student Service Agreement Management | P1 |
| HMP-74 | Student Onboarding & Orientation Management | P1 |
| HMP-75 | Placement Assignment to Support Requests | P1 |
| HMP-77 | Placement Hours Tracking (Remote + In-Person) | P2 |
| HMP-78 | School Timesheet & Evaluation Submission | P2 |
| HMP-79 | Student Exit Process (Stay-On or End) | P2 |

### Epic: Volunteer Intake & Routing (HMP-28)

| Jira ID | Story | Priority |
|---------|-------|----------|
| HMP-113 | Volunteer Signup via Website | P1 |
| HMP-11 | Volunteer Participation & Engagement History | P2 |

### Epic: Volunteer Matching (HMP-43)

| Jira ID | Story | Priority |
|---------|-------|----------|
| HMP-71 | Recommend Volunteers Based on Proximity | P1 |
| HMP-76 | Ensure Driving Capability Matching | P1 |
| HMP-81 | Gender-Safe Pairing Logic | P1 |
| HMP-82 | Match Based on Skills & Sensitivities | P2 |
| HMP-84 | Auto-Recommend Top Volunteer Pairings | P2 |
| HMP-85 | Assign Selected Volunteers to Support Request | P1 |
| HMP-86 | Track Cancellations and Reassignments | P2 |

### Epic: Document & Agreement Handling (HMP-39)

| Jira ID | Story | Priority |
|---------|-------|----------|
| HMP-64 | Secure Upload of Sensitive Documents | P1 |
| HMP-66 | Generate & Send Student Volunteer Service Agreements | P1 |
| HMP-67 | Capture & Store Parent/Volunteer Feedback Forms | P2 |
| HMP-68 | Document Upload Without Using the Website | P2 |
| HMP-69 | Download Files for School Submission | P2 |

### Epic: Security & Access Control (HMP-33)

| Jira ID | Story | Priority |
|---------|-------|----------|
| HMP-115 | Flexible Volunteer Interaction Model | P1 |
| HMP-116 | Controlled Volunteer System Access | P1 |
| HMP-117 | Shared Email Access for Outreach | P2 |

---

## 10. Key Design Decisions

1. **Student Volunteer = Custom Record Type on Contact**, not a separate object. This preserves the native V4S Contact relationship while allowing programme-specific fields.

2. **Student Placement = Custom Object** linked to Contact. This captures all placement metadata (programme, hours, documents, status) without cluttering the Contact record.

3. **V4S + PMM Used Together** — V4S manages volunteer supply (shifts, hours, assignments). PMM manages programme demand (caregivers enrolled, activities). They are linked via Campaign (V4S) ↔ Program (PMM).

4. **No direct Salesforce access for most volunteers** — most interactions remain email-based with the option to grant limited Experience Cloud access to tech-savvy, trusted volunteers in future.

5. **Google Drive folder structure is replaced** by Salesforce Files attached to the Student Placement record, with FLS (Field-Level Security) restricting access.

6. **Feedback forms** — caregiver feedback (currently Google Form) will be replaced by Salesforce Survey or a Flow-triggered form. Volunteer post-visit reports will be structured as custom Activities or a Visit Report custom object.

---

## 11. Constraints & Risks

| Risk | Mitigation |
|------|-----------|
| V4S not designed for school-mediated intake | Custom student record type + modified flow |
| Volunteer hours difficult to verify | Shift execution logging + task completion tracking |
| Tech literacy varies among volunteers | Role-based access; email-only path for non-tech volunteers |
| WhatsApp coordination (not auditable) | Email automation + Salesforce Chatter for logged volunteers |
| Phase 1 case object must be linked to volunteer assignments | V4S Volunteer Hours linked to Case via lookup |
| Data sensitivity (VSC, medical docs) | Salesforce Files + FLS + restricted sharing settings |

---

## 12. Out of Scope for Phase 2

- Stripe/donation integration (HMP-31 epic)
- Eventbrite event sync (HMP-30 epic)
- Mailchimp newsletter sync (HMP-40 epic)
- Full Experience Cloud volunteer portal (future phase)
- Reporting & Dashboards (HMP-32 epic — will be built after data model is live)

---

*End of Research Document*
