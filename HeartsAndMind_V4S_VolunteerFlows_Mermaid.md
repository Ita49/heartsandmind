# Hearts and Mind — Volunteer User Flow Diagrams
## Mermaid JS Format — Extractable for Presentation

---

## Flow 1: Student Volunteer Intake (School → Salesforce)

```mermaid
flowchart TD
    A([🏫 School / Student Reaches Out]) --> B{Who made contact?}
    B -- Student contacts H&M directly --> C[Redirect student back to school\nInform school of interest]
    B -- School contacts H&M --> D[School submits student details\nResume + Cover Letter]
    C --> D
    D --> E[(Salesforce: Lead / Contact Created\nRecord Type: Student Volunteer\n+ Student Placement record)]
    E --> F{📧 Coordinator Notified\nReview submission}
    F -- Suitable? No --> G[Send Regret Email to School]
    F -- Suitable? Yes --> H[Schedule Interview\nStore date + Google Meet link]
    H --> I[Conduct Interview]
    I --> J{Interview outcome?}
    J -- Not a fit --> G
    J -- Proceed --> K[Request 5 Compliance Documents\nVSC, Immunisation, CPR, CPI, Resume]
    K --> L{All 5 docs received?}
    L -- Incomplete --> M[Chase school / student\nDoc checklist tracked in Salesforce]
    M --> L
    L -- Complete --> N[✅ Documents_Verified = TRUE\nStatus → Verified – Ready for Onboarding]
    N --> O([➡ Go to Onboarding Flow])

    style A fill:#4A90D9,color:#fff
    style E fill:#27AE60,color:#fff
    style G fill:#E74C3C,color:#fff
    style N fill:#27AE60,color:#fff
    style O fill:#F39C12,color:#fff
```

---

## Flow 2: Student Volunteer Onboarding

```mermaid
flowchart TD
    A([▶ Student Verified – Ready for Onboarding]) --> B[Coordinator sends Service Agreement\nvia Salesforce email template]
    B --> C[Student receives agreement email]
    C --> D{Agreement signed and returned?}
    D -- No – follow up --> D
    D -- Yes --> E[Agreement_Status = Signed\nStored in Salesforce Files\nRestricted access]
    E --> F[Onboarding begins\nPlacement Handbook sent\nTask Calendar shared]
    F --> G[Orientation session held\n2–3 weeks / first month]
    G --> H[Orientation_Completed = Yes]
    H --> I[Added to volunteer communication group\nActivity logged]
    I --> J[First weekly meeting\nIntroductions, process walkthrough]
    J --> K[✅ Student Placement Status → Active]
    K --> L([➡ Go to Task Assignment Flow])

    style A fill:#4A90D9,color:#fff
    style E fill:#27AE60,color:#fff
    style K fill:#27AE60,color:#fff
    style L fill:#F39C12,color:#fff
```

---

## Flow 3: General Volunteer Intake (Walk-In)

```mermaid
flowchart TD
    A([🌐 Potential Volunteer\nVisits Website or Charity Village]) --> B[Submits Volunteer Interest Form\nName, Email, Role Interest, Drives Y/N]
    B --> C[(Salesforce: Contact Created\nRecord Type: General Volunteer\nAuto-acknowledgement email sent)]
    C --> D[📧 Coordinator Notified\nTask created: Review application]
    D --> E{Role requires\nhome visit / respite?}
    E -- Yes --> F[Request Vulnerable Sector Check\n+ relevant compliance docs]
    E -- No event support only --> G[Proceed to informal interview]
    F --> G
    G --> H[Informal interview held\nExperience, availability, skills captured]
    H --> I{Suitable?}
    I -- No --> J[Regret email sent\nRecord marked: Not Proceeding]
    I -- Yes --> K[Volunteer onboarded\nVolunteer_Status = Active]
    K --> L([➡ Go to Matching & Assignment Flow])

    style A fill:#4A90D9,color:#fff
    style C fill:#27AE60,color:#fff
    style J fill:#E74C3C,color:#fff
    style K fill:#27AE60,color:#fff
    style L fill:#F39C12,color:#fff
```

---

## Flow 4: Volunteer Matching & Assignment to Support Request

```mermaid
flowchart TD
    A([📋 Case: Support Request Active\nPhase 1 — caregiver case live]) --> B[Coordinator opens Case\nClicks Assign Volunteers]
    B --> C[System filters available volunteers\nby: Status=Active, Availability, Proximity]
    C --> D{Caregiver beyond\ntransit distance?}
    D -- Yes --> E[Filter: Drives = Yes required\nfor at least 1 volunteer]
    D -- No --> F[All available volunteers shown]
    E --> F
    F --> G{Female caregiver\nor children present?}
    G -- Yes --> H[Gender Rule Applied:\nNo 2-male pairings allowed]
    G -- No --> I[Standard pairing options]
    H --> I
    I --> J[System shows top\nrecommended pairings]
    J --> K[Coordinator selects\n1–3 volunteers]
    K --> L{Skills match\ncaregiver needs?}
    L -- Gap detected --> M[System flags warning\nCoordinator can override + note]
    L -- Match OK --> N[Create V4S Volunteer Job\n+ Volunteer Shift linked to Case]
    M --> N
    N --> O[📧 48-hour notice emails sent\nto assigned volunteers]
    O --> P{First visit to\nthis household?}
    P -- Yes --> Q[Schedule Meet & Greet\nIntroduce team to caregiver]
    P -- No --> R[Confirm visit details]
    Q --> R
    R --> S([➡ Go to Home Visit & Feedback Flow])

    style A fill:#4A90D9,color:#fff
    style H fill:#E67E22,color:#fff
    style M fill:#E67E22,color:#fff
    style N fill:#27AE60,color:#fff
    style S fill:#F39C12,color:#fff
```

---

## Flow 5: Home Visit Execution & Feedback

```mermaid
flowchart TD
    A([🏠 Home Visit Scheduled]) --> B{Cancellation or change\nrequested?}
    B -- Yes < 24 hrs notice --> C[Coordinator notified\nReschedule attempted\nShift updated in Salesforce]
    B -- No change --> D[Volunteers travel to home\n48-hr notice confirmed]
    C --> D
    D --> E{New volunteer\njoining existing team?}
    E -- Yes --> F[Brief introduction to caregiver\nParent notified in advance]
    E -- No --> G[Visit proceeds]
    F --> G
    G --> H{Student type?}
    H -- Basic Student --> I[Performs in-home tasks:\nLaundry, Meal Prep, Housekeeping, Errands]
    H -- Advanced Student --> J[Observes caregiver wellbeing\nSupervises basic students\nDocuments emotional state]
    I --> K[Post-visit: Complete Visit Report\nIn Salesforce or secure link]
    J --> L{Referral needed?}
    L -- Yes --> M[Escalate to supervisor\nReferral logged on Case]
    L -- No --> K
    M --> K
    K --> N[📧 Caregiver feedback email sent\nGoogle Form → Salesforce]
    N --> O[Coordinator reviews feedback\nCase notes updated]
    O --> P{Weekly meeting\nthis week?}
    P -- Yes --> Q[Review: tasks, challenges, progress\nHours logged for week]
    P -- No --> R[Continue next cycle]
    Q --> R
    R --> S([🔄 Next Support Request Cycle])

    style A fill:#4A90D9,color:#fff
    style I fill:#3498DB,color:#fff
    style J fill:#9B59B6,color:#fff
    style M fill:#E74C3C,color:#fff
    style S fill:#F39C12,color:#fff
```

---

## Flow 6: Student Exit & Transition

```mermaid
flowchart TD
    A([📅 Placement End Date Approaching\nSystem triggers alert]) --> B[Coordinator receives notification:\nPlacement ending for Student X]
    B --> C[School timesheet submitted\nCoordinator signs evaluation]
    C --> D[Evaluation stored in Salesforce\nPlacement Status → Completed]
    D --> E{Student wishes to\ncontinue volunteering?}
    E -- Yes --> F[Exit Decision = Stay as Volunteer\nRecord type converted:\nStudent Volunteer → General Volunteer\nFull history retained]
    E -- No --> G[Exit Decision = End Placement\nVolunteer Status → Exited]
    F --> H[System access reviewed\nIf Salesforce user: permissions updated]
    G --> I[System access revoked\nUser deactivated if applicable\nShared email password changed]
    H --> J[Student added to alumni record\nEngagement history preserved]
    I --> K([End of Placement Record])
    J --> L([Continue as General Volunteer\nGo to General Volunteer Flow])

    style A fill:#4A90D9,color:#fff
    style F fill:#27AE60,color:#fff
    style G fill:#E74C3C,color:#fff
    style I fill:#E74C3C,color:#fff
    style L fill:#F39C12,color:#fff
```

---

## Flow 7: Cancellation & Reassignment

```mermaid
flowchart TD
    A([❌ Cancellation Received]) --> B{Who cancelled?}
    B -- Caregiver cancels --> C{How much notice?}
    C -- ≥ 24 hours --> D[Mark shift Cancelled\nNotify volunteer team\nCase rescheduled]
    C -- < 24 hours --> E[Mark as Last-Minute Cancel\nLog on Case\nNotify team ASAP]
    B -- Volunteer cancels --> F[Mark volunteer shift Cancelled\nFind replacement]
    D --> G[Schedule new visit\nNew shift created in V4S]
    E --> G
    F --> H{Replacement\nvolunteer available?}
    H -- Yes --> I[New assignment created\nEmail sent to replacement]
    H -- No --> J[Notify caregiver of delay\nReschedule]
    I --> G
    J --> G
    G --> K([🔄 Matching & Assignment Flow])

    style A fill:#E74C3C,color:#fff
    style E fill:#E67E22,color:#fff
    style I fill:#27AE60,color:#fff
    style K fill:#F39C12,color:#fff
```

---

## Flow 8: Full End-to-End Volunteer Lifecycle (Summary View)

```mermaid
flowchart LR
    A([Apply]) --> B([Verify])
    B --> C([Onboard])
    C --> D([Assign])
    D --> E([Visit])
    E --> F([Report])
    F --> G([Review])
    G --> D
    G --> H{Placement\nending?}
    H -- No --> D
    H -- Yes --> I([Exit Decision])
    I --> J([Stay: General\nVolunteer])
    I --> K([End:\nExited])

    style A fill:#4A90D9,color:#fff
    style B fill:#27AE60,color:#fff
    style C fill:#27AE60,color:#fff
    style D fill:#F39C12,color:#fff
    style E fill:#3498DB,color:#fff
    style F fill:#9B59B6,color:#fff
    style G fill:#3498DB,color:#fff
    style I fill:#E67E22,color:#fff
    style J fill:#27AE60,color:#fff
    style K fill:#E74C3C,color:#fff
```

---

*These diagrams can be rendered at https://mermaid.live or embedded in any Mermaid-compatible tool.*
