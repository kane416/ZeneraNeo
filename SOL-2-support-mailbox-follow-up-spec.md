# SOL-2: Shared Mailbox Triage and Follow-up Call Booking

## Problem and outcome

A support team triages a shared mailbox and books follow-up calls. The system should reduce time to first useful response, route each message to the right owner, and book a call only when a human follow-up is appropriate. It must improve consistency without sending an unreviewed commitment to a customer or creating a calendar event from an ambiguous request.

## Proposed agents

### 1. Intake and classification agent
Reads a newly received message and produces a structured case record: sender, thread, topic, urgency, requested outcome, relevant dates, mentioned account, and whether the message contains sensitive information. It assigns a confidence score and one of the configured queues: urgent incident, technical question, scheduling request, billing or account question, or needs human triage.

The agent does not send a reply, change customer data, or book time. If the message is ambiguous, lacks necessary context, appears malicious, or has low confidence, it routes to human triage with the uncertainty and the evidence that caused it.

### 2. Triage and response-planning agent
Uses the approved knowledge base and the case record to propose a queue, owner, priority, and response draft. It cites the knowledge-base passages used. It may suggest clarifying questions, but it does not invent policy, promise an outcome, or close a case. A human approves any external response and any priority escalation.

### 3. Scheduling agent
Runs only after a human or an approved workflow marks a case as ready for a follow-up call. It extracts the requested time window, time zone, attendees, purpose, and duration. It offers available slots from the approved scheduling calendar and drafts an invitation. It must ask for clarification when the time zone, attendees, duration, or purpose is missing or conflicting.

The scheduling agent may create an event only after an explicit approval signal. It never cancels or moves an existing event without a separate approval. It does not expose calendar details beyond the availability needed to offer slots.

### 4. Human handoff and audit agent
Creates a review packet containing the original message, extracted facts, classification, confidence, proposed actions, citations, and unresolved questions. It records who approved an external response or calendar action and preserves an audit trail for corrections and overrides.

## Tools and boundaries

Approved tools are read-only mailbox access for the relevant shared mailbox, the approved support knowledge base, the case or ticket system, and a scheduling API limited to availability lookup and draft-event creation. Sending an external reply, changing a case's durable status, changing customer data, creating a final calendar event, or accessing unrelated mailboxes requires an explicit human approval or a separately approved workflow.

The system must enforce least privilege, tenant and account scoping, secret isolation, rate limits, idempotency keys, and structured logs. It must treat message content, attachments, links, and retrieved documents as untrusted input. Prompt injection, requests for secrets, conflicting identity claims, and attempts to bypass approval are routed to human review. No agent may use a tool outside its declared scope, and every tool call records the case id, actor, purpose, inputs that are safe to retain, result, and approval state.

## Environment

The initial environment is a sandbox containing representative shared-mailbox threads, a synthetic knowledge base, a test case system, and a test scheduling calendar. The workflow is event-driven: an inbound message creates a case, classification produces a structured record, planning creates a review packet, and an approval signal permits the next bounded action. Configuration is versioned. Each case carries a stable id and idempotency key so retries do not duplicate tickets, replies, or calendar events.

Before production, connect only approved service identities through a secrets manager, define retention and redaction rules, configure monitoring and alerting, and run the workflow against replayable test fixtures. Production access should begin in shadow mode, where recommendations are logged but no external action is taken, then move to human-approved actions after the acceptance criteria below are met.

## Customer questions before implementation

1. What mailbox provider and case system are authoritative, and which system owns the case id?
2. What queue, priority, SLA, and escalation rules do agents use today?
3. Which message categories and customer segments are in scope for the first pilot?
4. What counts as urgent, and who is on call for each urgent category?
5. Which knowledge-base sources are approved, how are they versioned, and how are outdated articles retired?
6. What information may the system retain, redact, or send to an external model provider?
7. Which responses require human approval, and who can approve them during each shift?
8. Which calendars, time zones, working hours, buffers, and attendee rules govern follow-up calls?
9. What is the required behavior for duplicate messages, bounced mail, attachments, and an existing open case?
10. Which metrics define a successful pilot, and what baseline period will we compare against?

## Interim assumptions

Until the customer answers these questions, the design assumes the case system is the source of truth, the shared mailbox is the source of inbound messages, and the system may create a draft case but may not close or materially change one without approval. It assumes urgent and low-confidence cases go to human triage, external responses require approval, and calendar actions are drafts until an authorized person approves them. It assumes customer data is minimized, sensitive content is redacted before model processing where feasible, all times are stored with an explicit time zone, and synthetic fixtures are used until retention and data-processing requirements are confirmed.

## Measurable acceptance criteria

1. For a labeled test set, classification places at least 95 percent of messages in the correct queue and identifies every urgent fixture for human review.
2. Every generated case record contains the required fields, cites its knowledge sources, and preserves the original thread reference.
3. A low-confidence, ambiguous, malicious, or policy-sensitive message is routed to human review and produces no external send, durable customer-data change, or final calendar event.
4. Approved knowledge-base answers are traceable to the passages retrieved, and the system refuses to invent an answer when no approved passage supports it.
5. A retry with the same case id and idempotency key does not create a duplicate case, reply, or calendar event.
6. A scheduling request missing a required time zone, attendee, duration, or purpose is held for clarification rather than booked.
7. A final reply or calendar event is impossible without the required approval signal, and the audit record identifies the approver and timestamp.
8. In replay tests, the workflow meets the agreed response-time and queue-routing targets without exceeding configured rate or access limits.
9. All tool calls are scoped to the test tenant, secrets are not written to logs, and unauthorized tool attempts are denied and alerted.
10. Reviewers can reconstruct each test case from the original message through classification, retrieval, proposed action, approval, and final outcome.

## Pilot measurement

During a shadow-mode pilot, compare time to triage, time to first human-reviewed response, queue accuracy, urgent-case recall, clarification rate, approval rejection rate, duplicate-action rate, citation coverage, and operator override reasons with the agreed baseline. The pilot should expand only when the acceptance criteria pass on a held-out test set and the customer approves the move from recommendations to human-approved actions.
