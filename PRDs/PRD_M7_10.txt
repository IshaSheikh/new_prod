

# Module 7.10 Communication Hub

The Communication Hub module is responsible for creating, scheduling, delivering, tracking, and auditing school communications across multiple channels. It centralizes notices, alerts, reminders, and targeted messages for parents, students, teachers, and staff while maintaining strict tenant isolation and delivery history. 

This module enables schools to:
- Publish school-wide and targeted notices
- Send fee reminders and payment follow-ups
- Deliver attendance alerts and absence notifications
- Notify users about timetable changes, results publication, and operational updates
- Maintain searchable communication history
- Track delivery outcomes for important alerts
- Manage channel preferences and templates centrally

The module must support:
- In-app notifications
- Email
- SMS
- WhatsApp
- Push notifications

The Communication Hub is a cross-cutting platform service. It should consume audience, student, guardian, class, section, and role relationships from existing source modules rather than creating its own parallel identity model. 

***

## 1. Module Objective

Schools communicate constantly with parents, students, teachers, and staff. Without a centralized communication system:
- Notices are sent inconsistently
- Fee reminders are delayed or missed
- Attendance alerts depend on manual calls
- Parents receive incomplete or duplicate information
- Communication history becomes impossible to audit
- Staff spend excessive time sending repetitive messages

Schools need this module to:
- Standardize communication delivery across all school operations
- Reduce manual communication work for admins and office staff
- Improve parent transparency through timely updates
- Support fee collection through automated reminders
- Ensure critical alerts are delivered and traceable
- Make school communication measurable and auditable

***

## 2. Why Schools Need It

### 2.1 Centralized Communication Control
Schools need one communication system instead of separate WhatsApp groups, manual SMS tools, email threads, and paper notices.

### 2.2 Reduced Administrative Work
Routine communication such as attendance alerts, fee reminders, timetable changes, and result publication should be automated rather than manually coordinated. 

### 2.3 Improved Parent Transparency
Parents need timely, reliable communication about absences, payments, academic updates, and school announcements. 

### 2.4 Better Fee Collection
Automated due-date reminders, overdue notices, and receipt notifications help improve collection behavior and reduce finance follow-up workload.

### 2.5 Searchable and Auditable History
Schools need a permanent, searchable record of what was sent, to whom, when, through which channel, and with what delivery result.

### 2.6 Multi-Channel Reach
Different schools and families rely on different channels. A commercial K-12 platform must support multi-channel communication with policy-based routing.

### 2.7 Measurable Operations
Delivery success, read rates, failed notifications, and response turnaround must become operational metrics rather than invisible activities.

***

## 3. Primary Users

### 3.1 School Admin
Primary communication owner inside the school. Responsibilities:
- Create notices
- Configure templates
- Define audiences
- Schedule communications
- Monitor delivery status
- Manage communication settings

### 3.2 Principal
Oversight user. Responsibilities:
- Review major announcements
- Monitor communication effectiveness
- Approve high-priority or sensitive school-wide messages
- Track school communication health

### 3.3 Teacher
Instructional sender. Responsibilities:
- Send class-level notices where permitted
- Trigger parent-facing updates related to classroom activity
- Review communication history for assigned classes

### 3.4 Accountant
Finance communication user. Responsibilities:
- Trigger fee reminders
- Send payment confirmations
- Monitor fee reminder outcomes
- Review finance-related message history

### 3.5 Parent
Recipient user. Responsibilities:
- Receive notices and alerts
- View communication history in portal
- Acknowledge messages where required
- Follow fee reminders and attendance alerts

### 3.6 Student
Recipient user. Responsibilities:
- Receive student-targeted notices
- View timetable, event, and academic notifications
- Acknowledge messages where required

### 3.7 Super Admin
Platform oversight user. Responsibilities:
- Monitor tenant-safe communication activity
- Investigate delivery failures
- Review audit logs
- Support school configuration issues

***

## 4. User Stories

### 4.1 User Stories - Notice Creation
- **US-001** As a School Admin, I want to create a notice, so the school can communicate announcements centrally.
- **US-002** As a School Admin, I want to target a notice by school, grade, section, role, or individual, so messages reach only relevant users.
- **US-003** As a School Admin, I want to schedule a notice for future delivery, so communication can be planned in advance.
- **US-004** As a School Admin, I want to attach files or links to a notice, so recipients receive complete information.
- **US-005** As a Principal, I want high-impact school-wide messages reviewed before delivery, so communication quality remains controlled.

### 4.2 User Stories - Attendance and Safety Alerts
- **US-006** As a School Admin, I want attendance alerts triggered after official attendance submission, so parents are informed reliably. 
- **US-007** As a Parent, I want to receive absence and lateness alerts promptly, so I can act quickly. 
- **US-008** As a Teacher, I want attendance-triggered parent notifications to be automatic, so I do not need to contact families manually. 

### 4.3 User Stories - Fee Reminders
- **US-009** As an Accountant, I want to send fee due and overdue reminders, so payment collection improves.
- **US-010** As a Parent, I want fee reminders to include due amount, due date, and payment action, so I know exactly what to do.
- **US-011** As an Accountant, I want payment confirmation messages sent automatically after successful payment, so parents receive proof and reassurance.

### 4.4 User Stories - Academic and Timetable Notifications
- **US-012** As a School Admin, I want result publication notifications sent automatically, so parents and students know when report cards are available. 
- **US-013** As a Parent, I want timetable change alerts, so I stay aware of operational changes affecting my child. 
- **US-014** As a Student, I want school notices and academic notifications in one place, so I do not miss important updates.

### 4.5 User Stories - Templates and Automation
- **US-015** As a School Admin, I want reusable message templates, so routine communication becomes faster and more consistent.
- **US-016** As a School Admin, I want variable placeholders in templates, so messages can include student name, class, due date, amount, and attendance status.
- **US-017** As a School Admin, I want event-based automation rules, so notices can be triggered by attendance submission, fee due dates, payment success, and publication events.
- **US-018** As a Principal, I want approval workflows for selected communication types, so sensitive messaging is reviewed before being sent.

### 4.6 User Stories - Delivery Tracking
- **US-019** As a School Admin, I want delivery status per channel, so I know whether a message was sent, delivered, failed, or read.
- **US-020** As a School Admin, I want high-priority alerts to require delivery tracking, so critical communication is auditable.
- **US-021** As a Super Admin, I want communication delivery failures visible by tenant, so platform issues can be investigated safely.

### 4.7 User Stories - History and Search
- **US-022** As a School Admin, I want to search historical communication by audience, channel, date, and content, so school records are easy to retrieve.
- **US-023** As a Teacher, I want to view prior notices sent to my assigned classes, so I can maintain continuity.
- **US-024** As a Parent, I want to view my past notices and alerts in the portal, so I can refer back to important information.

### 4.8 User Stories - Preferences and Consent
- **US-025** As a Parent, I want notification preferences managed according to school policy, so communication reaches me through allowed channels.
- **US-026** As a School Admin, I want channel preferences configurable by message type, so critical alerts can override optional preferences when necessary.
- **US-027** As a School Admin, I want invalid phone numbers or email addresses flagged before sending, so delivery quality improves.

***

## 5. Business Rules

### 5.1 Business Rules - Tenant Isolation
- **BR-001** Every communication record must belong to exactly one tenant.
- **BR-002** Messages, templates, audiences, and delivery logs must never be accessible across tenants.
- **BR-003** Audience resolution must enforce tenant boundaries before message generation. 

### 5.2 Business Rules - Audience Targeting
- **BR-004** Audience may be school-wide, grade-wise, section-wise, role-wise, or individual.
- **BR-005** Parent audiences must be resolved through guardian-student relationships from platform source data. 
- **BR-006** Student audiences must be resolved from active enrollment and portal-accessible relationships. 
- **BR-007** Teacher audiences must be resolved from assigned staff and subject-section relationships where applicable. 
- **BR-008** Audience snapshots must be persisted at send time for auditability.
- **BR-009** A communication must not be sent if audience resolution returns zero valid recipients unless explicitly confirmed by an authorized user.

### 5.3 Business Rules - Message Composition
- **BR-010** Messages may be created from templates or freeform content.
- **BR-011** Templates may include placeholders for dynamic values.
- **BR-012** Placeholder values must be validated before send.
- **BR-013** Unsupported or unresolved template variables must block final dispatch.
- **BR-014** File attachments must follow allowed file-type and size restrictions configured by the platform.
- **BR-015** Message content must be stored in a normalized form for audit and search.

### 5.4 Business Rules - Channel Routing
- **BR-016** Supported channels are in-app, email, SMS, WhatsApp, and push notification.
- **BR-017** One message may be sent through one or multiple channels.
- **BR-018** Channel eligibility must depend on recipient contact availability and channel enablement.
- **BR-019** Channel fallback rules may be configured per message type.
- **BR-020** High-priority alerts require delivery tracking.
- **BR-021** If a high-priority alert fails on the primary channel, configured fallback channels may be attempted automatically.
- **BR-022** Optional informational notices do not require delivery confirmation on every channel.
- **BR-023** Communication delivery history must be searchable.

### 5.5 Business Rules - Event-Triggered Messaging
- **BR-024** Attendance alerts may only trigger after official attendance submission or approval, not during draft entry. 
- **BR-025** Result publication messages may only trigger after published result status exists in the gradebook module.
- **BR-026** Fee reminders may only use official dues and payment states from the finance module.
- **BR-027** Timetable change notifications may only use active timetable changes from the timetable module. 
- **BR-028** The Communication Hub must not independently modify attendance, fees, grades, or timetable records.

### 5.6 Business Rules - Approval and Priority
- **BR-029** Schools may configure which communication categories require approval before send.
- **BR-030** High-priority messages must include delivery status visibility.
- **BR-031** Emergency or safety alerts may bypass certain optional preference settings if school policy allows.
- **BR-032** Approval decisions must be audited with actor, timestamp, and reason when rejected.

### 5.7 Business Rules - Preferences and Consent
- **BR-033** Notification preferences may be stored per user and channel.
- **BR-034** Mandatory operational alerts may override optional channel preferences when configured by school policy.
- **BR-035** Marketing or non-essential communication categories must respect opt-out rules if such categories are introduced later.
- **BR-036** Invalid or missing contact data must be recorded as a delivery eligibility issue, not silently ignored.

### 5.8 Business Rules - Search and History
- **BR-037** Sent communication history must be retained and searchable by sender, recipient, role, audience type, channel, date range, and status.
- **BR-038** Communication records must preserve message body snapshot, resolved audience snapshot, and channel outcome snapshot.
- **BR-039** Searchable history must not expose other tenants’ data.
- **BR-040** Message deletion is prohibited after dispatch; corrections require cancellation before send or follow-up communication after send.

### 5.9 Business Rules - Audit and Reliability
- **BR-041** Every create, edit, schedule, approve, cancel, send, retry, and fail event must be auditable.
- **BR-042** Delivery provider callbacks must be processed idempotently.
- **BR-043** Retrying failed messages must not create duplicate successful sends without explicit retry tracking.
- **BR-044** Communication queues must preserve message integrity even during provider outages.

***

## 6. State Machine

### 6.1 State Machine - Notice Lifecycle
```text
Draft
  v
Scheduled
  v
Queued
  v
Sent
  |------ Partially Delivered
  |         v
  |------ Delivered
  |         v
  |------ Read
  v
Archived
```

### 6.2 State Machine - Approval Path
```text
Draft
  v
Pending Approval
  |------ Rejected
  |         v
  |------ Draft
  v
Approved
  v
Scheduled
```

### 6.3 State Machine - Outbound Message Lifecycle
```text
Queued
  v
Sending
  |------ Failed
  |         v
  |------ Retrying
  |         v
  |------ Sending
  v
Sent
  |------ Delivered
  |         v
  |------ Read
```

### 6.4 State Machine - Template Lifecycle
```text
Draft
  v
Active
  |------ Inactive
  |         v
  |------ Active
  v
Archived
```

### 6.5 State Machine - Notification Preference Lifecycle
```text
Default
  v
Customized
  |------ Suspended
  |         v
  |------ Customized
```

***

## 7. Required Data Entities

### 7.1 Required Data Entities - notices
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
title | VARCHAR
body | TEXT
category | ENUM
priority | ENUM
status | ENUM
createdby | UUID
approvedby | UUID NULL
scheduledat | TIMESTAMP NULL
sentat | TIMESTAMP NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.2 Required Data Entities - notice_audiences
Field | Type
--- | ---
id | UUID
tenantid | UUID
noticeid | UUID
audiencetype | ENUM
audienceref | VARCHAR
recipientcount | INTEGER
resolvedsnapshot | JSONB
createdat | TIMESTAMP

### 7.3 Required Data Entities - message_templates
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
templatename | VARCHAR
templatecode | VARCHAR
category | ENUM
channel | ENUM
subjecttemplate | TEXT NULL
bodytemplate | TEXT
variableschema | JSONB
status | ENUM
createdby | UUID
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.4 Required Data Entities - outbound_messages
Field | Type
--- | ---
id | UUID
tenantid | UUID
noticeid | UUID NULL
templateid | UUID NULL
recipientuserid | UUID NULL
recipientcontact | VARCHAR
channel | ENUM
priority | ENUM
status | ENUM
providerref | VARCHAR NULL
failurereason | TEXT NULL
queuedat | TIMESTAMP
sentat | TIMESTAMP NULL
deliveredat | TIMESTAMP NULL
readat | TIMESTAMP NULL
retrycount | INTEGER
payloadsnapshot | JSONB

### 7.5 Required Data Entities - notification_preferences
Field | Type
--- | ---
id | UUID
tenantid | UUID
userid | UUID
channel | ENUM
messagecategory | ENUM
enabled | BOOLEAN
overridepolicy | ENUM NULL
updatedat | TIMESTAMP

### 7.6 Required Data Entities - channels
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
channelcode | ENUM
providername | VARCHAR
enabled | BOOLEAN
configstatus | ENUM
priorityorder | INTEGER
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.7 Additional Supporting Entities
- `communication_approvals`
- `communication_attachments`
- `communication_events`
- `communication_delivery_logs`
- `communication_search_index`
- `automation_rules`
- `recipient_resolution_jobs`

### 7.8 Required Data Entities - communication_approvals
Field | Type
--- | ---
id | UUID
tenantid | UUID
noticeid | UUID
status | ENUM
requestedby | UUID
reviewedby | UUID NULL
reviewreason | TEXT NULL
reviewedat | TIMESTAMP NULL
createdat | TIMESTAMP

### 7.9 Required Data Entities - communication_delivery_logs
Field | Type
--- | ---
id | UUID
tenantid | UUID
outboundmessageid | UUID
eventtype | ENUM
eventtime | TIMESTAMP
providerpayload | JSONB
normalizedstatus | ENUM

***

## 8. Permissions Matrix

| Capability | Super Admin | School Admin | Principal | Teacher | Accountant | Parent | Student |
|---|---|---|---|---|---|---|---|
| View communication dashboard | Yes | Yes | Yes | Limited | Limited | No | No |
| Create school-wide notices | Limited Support | Yes | Yes if allowed | No | No | No | No |
| Create class-wise notices | No | Yes | Yes if allowed | Yes for assigned classes | No | No | No |
| Create fee reminders | No | Yes | Yes if allowed | No | Yes | No | No |
| Approve messages | Limited Audit | Yes | Yes | No | No | No | No |
| Schedule messages | No | Yes | Yes if allowed | Limited | Limited finance only | No | No |
| Send messages immediately | No | Yes | Yes if allowed | Limited assigned scope | Limited finance scope | No | No |
| View delivery tracking | Limited Audit | Yes | Yes | Limited own scope | Limited finance scope | Own received only | Own received only |
| Search communication history | Limited Audit | Yes | Yes | Limited own scope | Limited finance scope | Own received only | Own received only |
| Configure templates | Limited Support | Yes | Review only | No | Limited finance templates | No | No |
| Configure channels | Limited Support | Yes | Review only | No | No | No | No |
| Configure preferences | No | Yes | No | No | No | Limited own if policy allows | Limited own if policy allows |

***

## 9. APIs Needed

### 9.1 APIs Needed - Notice APIs
- `POST /communications/notices`
- `GET /communications/notices`
- `GET /communications/notices/{id}`
- `PATCH /communications/notices/{id}`
- `POST /communications/notices/{id}/schedule`
- `POST /communications/notices/{id}/submit-approval`
- `POST /communications/notices/{id}/approve`
- `POST /communications/notices/{id}/reject`
- `POST /communications/notices/{id}/cancel`
- `POST /communications/notices/{id}/send`

### 9.2 APIs Needed - Audience APIs
- `POST /communications/audiences/resolve`
- `GET /communications/notices/{id}/audiences`
- `GET /communications/notices/{id}/recipients`

### 9.3 APIs Needed - Template APIs
- `POST /communications/templates`
- `GET /communications/templates`
- `GET /communications/templates/{id}`
- `PATCH /communications/templates/{id}`
- `POST /communications/templates/{id}/activate`
- `POST /communications/templates/{id}/archive`
- `POST /communications/templates/{id}/preview`

### 9.4 APIs Needed - Outbound Messaging APIs
- `GET /communications/outbound-messages`
- `GET /communications/outbound-messages/{id}`
- `POST /communications/outbound-messages/{id}/retry`
- `GET /communications/delivery-logs/{id}`

### 9.5 APIs Needed - Preference APIs
- `GET /communications/preferences`
- `PATCH /communications/preferences`
- `GET /communications/channels`
- `PATCH /communications/channels/{id}`

### 9.6 APIs Needed - Automation APIs
- `POST /communications/automation-rules`
- `GET /communications/automation-rules`
- `PATCH /communications/automation-rules/{id}`
- `POST /communications/automation-rules/{id}/activate`
- `POST /communications/automation-rules/{id}/deactivate`

### 9.7 APIs Needed - Portal Recipient APIs
- `GET /parent/my-notices`
- `GET /student/my-notices`
- `GET /parent/my-alerts`
- `GET /student/my-alerts`

### 9.8 APIs Needed - Analytics APIs
- `GET /communications/analytics/summary`
- `GET /communications/analytics/channel-performance`
- `GET /communications/analytics/failures`
- `GET /communications/search`

***

## 10. Screens Needed

### 10.1 Screens Needed - Communication Dashboard
Features:
- Delivery summary
- Pending approvals
- Failed messages
- Channel performance
- Recent activity

### 10.2 Screens Needed - Notice Composer Screen
Features:
- Title and body editor
- Template selection
- Audience builder
- Channel selection
- Attachment support
- Priority selection
- Preview before send

### 10.3 Screens Needed - Audience Selection Screen
Features:
- School-wide targeting
- Grade and section targeting
- Role targeting
- Individual targeting
- Recipient preview
- Duplicate-recipient handling

### 10.4 Screens Needed - Template Management Screen
Features:
- Create templates
- Edit templates
- Placeholder mapping
- Activate or archive templates
- Preview rendered output

### 10.5 Screens Needed - Approval Queue Screen
Features:
- Pending items
- Approve or reject actions
- Reviewer notes
- Priority indicators

### 10.6 Screens Needed - Delivery Tracking Screen
Features:
- Sent, delivered, failed, read counts
- Channel-wise performance
- Retry actions
- Failure reasons
- Provider reference IDs

### 10.7 Screens Needed - Communication History Screen
Features:
- Search by keyword
- Filter by channel
- Filter by audience
- Filter by sender
- Filter by date range
- View full communication log

### 10.8 Screens Needed - Channel Configuration Screen
Features:
- Enable or disable channels
- Provider configuration status
- Priority order
- Test send
- Fallback rules

### 10.9 Screens Needed - Notification Preferences Screen
Features:
- User or role defaults
- Channel toggles
- Category preferences
- Mandatory alert rules

### 10.10 Screens Needed - Parent and Student Communication Inbox
Features:
- Notice list
- Alert badges
- Read status
- Search and filter
- Download attachments
- Acknowledge important messages

***

## 11. Edge Cases

- **EC-001** Audience resolves to recipients with duplicate phone numbers or email addresses. System should deduplicate according to configured rules.
- **EC-002** A parent is linked to multiple children and qualifies for the same message twice. System should avoid duplicate delivery unless policy requires child-specific duplication. 
- **EC-003** A teacher tries to send a notice to an unassigned section. Operation blocked by scope validation. 
- **EC-004** Attendance alert trigger runs before attendance submission. Communication must not be sent. 
- **EC-005** Fee reminder is scheduled for a student whose dues are cleared before send time. System should skip or recalculate message eligibility.
- **EC-006** Invalid phone number or email address detected during dispatch. Message marked failed with validation reason.
- **EC-007** External provider callback arrives twice. Duplicate callback processed idempotently.
- **EC-008** Scheduled message remains pending while channel provider is down. System retries according to retry policy.
- **EC-009** Notice approved after scheduled time has already passed. System should either queue immediately or require rescheduling based on policy.
- **EC-010** Parent opts out of non-critical notifications. Mandatory attendance or safety alerts may still be sent if policy allows.
- **EC-011** Student is transferred or withdrawn after audience resolution but before scheduled send. Audience snapshot and current eligibility rules must be applied consistently.
- **EC-012** Attachment removed from source storage after notice creation. Send blocked or attachment flagged invalid.
- **EC-013** Message template updated while scheduled messages already exist. Scheduled messages must use stored snapshot or explicit re-render rules.
- **EC-014** Same school message uses multiple channels and one succeeds while another fails. Delivery status must show partial success.
- **EC-015** School-wide urgent alert exceeds SMS length limits. System should segment, truncate by rule, or reroute according to channel policy.

***

## 12. Risks

### 12.1 Risks - Security Risk
Unauthorized users access communication history or recipient lists.  
Mitigation: Tenant-scoped access control, RBAC enforcement, audited access logs. 

### 12.2 Risks - Privacy Risk
A message is delivered to the wrong audience because of incorrect guardian or enrollment relationships.  
Mitigation: Source-of-truth relationship validation, audience preview, recipient snapshots. 
### 12.3 Risks - Delivery Risk
External providers fail or degrade, causing delayed or missed messages.  
Mitigation: Retry queues, fallback channels, provider monitoring, failure dashboards.

### 12.4 Risks - Operational Risk
Staff send duplicate or incorrect messages.  
Mitigation: Preview step, approval workflows, deduplication controls, draft and schedule states.

### 12.5 Risks - Compliance Risk
Communication history cannot be reconstructed during disputes.  
Mitigation: Immutable sent-message snapshots, searchable logs, audit retention.

### 12.6 Risks - Scalability Risk
Large schools generate very high outbound volume during attendance or fee cycles.  
Mitigation: Queue-based dispatch, partitioned delivery logs, indexed search, async processing. 

### 12.7 Risks - Data Quality Risk
Recipient contact data is missing or outdated.  
Mitigation: Contact validation rules, failed-delivery reporting, upstream profile accuracy monitoring. 

### 12.8 Risks - Trust Risk
Parents receive too many low-value notifications and start ignoring important alerts.  
Mitigation: Priority categories, preference policies, template discipline, notification governance.

***

## 13. Success Metrics

### 13.1 Success Metrics - Product Metrics
- Message composer save success rate 99 percent
- Scheduled message execution success 98 percent
- Delivery tracking dashboard load time 2 seconds
- Communication history search response 2 seconds
- Template preview generation success 99 percent

### 13.2 Success Metrics - Delivery Metrics
- Email delivery success 95 percent
- SMS accepted-by-provider success 98 percent
- High-priority alert delivery tracking coverage 100 percent
- Failed message retry recovery rate 80 percent
- Duplicate-send rate below 0.5 percent

### 13.3 Success Metrics - Parent Transparency Metrics
- Parent notice open rate 75 percent
- Attendance alert reach rate 95 percent for valid contacts. 
- Fee reminder engagement rate improved after rollout
- Result publication notification open rate 80 percent. 
### 13.4 Success Metrics - Operational Metrics
- Reduction in manual notice distribution effort 80 percent
- Reduction in office calls for routine updates 40 percent
- Reduction in manual fee reminder work 70 percent
- Approval turnaround for sensitive communications 4 hours

### 13.5 Success Metrics - Business Metrics
- Improved parent engagement with school communications
- Improved fee collection follow-through after reminder automation
- Increased value perception of the platform as a unified communication system
- Reduced operational overhead across school administration

### 13.6 Success Metrics - Reliability and Compliance Metrics
- Cross-tenant communication leakage incidents 0
- Untracked high-priority alert sends 0
- Searchable communication retention coverage 100 percent
- Audit event coverage for send lifecycle 100 percent

***

## 14. Out of Scope

The following are excluded from the Communication Hub MVP:
- Two-way live chat between parents and teachers
- Community discussion forums
- Social feed style announcements
- Voice calling integration
- AI-generated message writing
- Translation engine with real-time multilingual generation
- Full campaign marketing automation
- External CRM integration
- Public newsletter management
- Advanced sentiment analysis on message replies
- Student-to-student messaging
- Teacher-to-teacher collaboration chat
- Video broadcasting and webinar hosting
- Government notification gateway integration
- Advanced contact-center ticketing workflows beyond basic support routing

The MVP should focus on:
1. Notice creation and targeting
2. Multi-channel outbound delivery
3. Templates and automation
4. Delivery tracking
5. Searchable communication history
6. Parent and student inbox visibility
7. Channel preferences and governance

This keeps the module commercially strong while remaining implementable for a solo founder using a document-first, modular architecture.