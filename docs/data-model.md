# Mind — Data Model Reference

Per-field reference for all 16 `memory_*` collections, generated from the live schema registry (`items`). Each collection installs with full CRUD + realtime and the standard list/edit/show prefabs. The polymorphic attachment pair (`attachmentModelKey` + `attachmentId`) appears on most collections and is omitted from the per-field tables below for brevity — it lets any record point at any other record in any collection.

## `memory_people` — People

Humans you interact with — coworkers, managers, reports, mentors, customers. The central node of the Mind graph; nearly every other memory_* record links here.

| Field | Type | Req | Description |
|---|---|---|---|
| `name` | string | ✓ | Display name. |
| `role` | string |  | Job title or relationship (e.g. "Engineering Manager", "Mentor"). |
| `teamId` | ref → memory_teams |  | Team this person belongs to. |
| `managerId` | ref → memory_people |  | This person's manager (another person record). |
| `email` | string (email) |  | Email address. |
| `tagIds` | string[] |  | Loose labels — mentor, peer, customer, etc. |
| `notes` | string |  | Running markdown notes about this person. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_teams` — Teams

Org units — teams, squads, departments. People belong to teams, projects are owned by teams.

| Field | Type | Req | Description |
|---|---|---|---|
| `name` | string | ✓ | Team name — keep it short and stable. |
| `description` | string |  | What this team does (markdown). |
| `parentTeamId` | ref → memory_teams |  | Containing team (for sub-team hierarchy). |

## `memory_projects` — Projects

Time-boxed pieces of work. Decisions, incidents, wins, meetings all reference the project they pertain to.

| Field | Type | Req | Description |
|---|---|---|---|
| `name` | string | ✓ | Project name. |
| `description` | string |  | What the project is and its scope (markdown). |
| `status` | enum | ✓ | Lifecycle stage: planning → active → paused → done → archived. Values: planning, active, paused, done, archived. |
| `teamId` | ref → memory_teams |  | Team that owns this project. |
| `leadPersonId` | ref → memory_people |  | Directly responsible person. |
| `collaboratorIds` | ref[] → memory_people |  | Other people contributing to the project. |
| `startedAt` | string (date) |  | Date work began. |
| `endedAt` | string (date) |  | Date the project finished or was archived. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_meetings` — Meetings

1:1s, standups, planning sessions, customer calls. Captures attendees, agenda, and notes.

| Field | Type | Req | Description |
|---|---|---|---|
| `title` | string | ✓ | Meeting title. |
| `type` | enum | ✓ | Meeting kind: 1:1, standup, planning, review, customer, interview, or other. Values: 1:1, standup, planning, review, customer, interview, other. |
| `occurredAt` | string (date-time) |  | When the meeting took place. |
| `peopleIds` | ref[] → memory_people |  | Attendees. |
| `projectIds` | ref[] → memory_projects |  | Projects discussed. |
| `notes` | string |  | Summary of what was discussed (markdown). The primary content of the record. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_decisions` — Decisions

Architecture Decision Records (ADRs) and other durable choices. Captures context, options, and rationale so the choice doesn't have to be re-argued.

| Field | Type | Req | Description |
|---|---|---|---|
| `title` | string | ✓ | The decision stated in active voice (e.g. "Use Redis for session cache"). |
| `status` | enum | ✓ | Lifecycle: proposed → accepted → superseded → deprecated. Values: proposed, accepted, superseded, deprecated. |
| `decidedAt` | string (date) |  | Date the decision was made. |
| `context` | string |  | The problem or situation that prompted the decision (markdown). |
| `decision` | string |  | The actual choice that was made (markdown). |
| `rationale` | string |  | Why this choice over the alternatives (markdown). |
| `consequences` | string |  | Tradeoffs and follow-on effects (markdown). |
| `projectIds` | ref[] → memory_projects |  | Projects this decision pertains to. |
| `peopleIds` | ref[] → memory_people |  | People who made or owned the decision. |
| `supersedesId` | ref → memory_decisions |  | Earlier decision this one replaces. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_incidents` — Incidents

Outages, regressions, security events. Captures timeline, responders, root cause, and remediation.

| Field | Type | Req | Description |
|---|---|---|---|
| `title` | string | ✓ | Short incident name. |
| `severity` | integer | ✓ | 1 = critical, 4 = minor. |
| `status` | enum | ✓ | Lifecycle: active → mitigated → resolved → postmortem. Values: active, mitigated, resolved, postmortem. |
| `occurredAt` | string (date-time) |  | When the incident started. |
| `resolvedAt` | string (date-time) |  | When the incident was resolved. |
| `summary` | string |  | What happened, in brief (markdown). |
| `rootCause` | string |  | The underlying systemic cause (markdown). |
| `remediation` | string |  | What was done to fix it and prevent recurrence (markdown). |
| `projectIds` | ref[] → memory_projects |  | Projects affected. |
| `peopleIds` | ref[] → memory_people |  | Responders. |
| `competencyIds` | ref[] → memory_competencies |  | Competencies demonstrated during the response. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_action_items` — Action Items

Discrete TODOs with an owner and (optionally) a due date. Often spawned from meetings or decisions.

| Field | Type | Req | Description |
|---|---|---|---|
| `title` | string | ✓ | Verb-first description of the task (e.g. "Email Sarah the spec"). |
| `status` | enum | ✓ | todo → in_progress → done (or cancelled). Values: todo, in_progress, done, cancelled. |
| `ownerId` | ref → memory_people | ✓ | The single person responsible for completing this. |
| `dueAt` | string (date) |  | Optional due date. |
| `meetingId` | ref → memory_meetings |  | Meeting this action item was spawned from. |
| `projectId` | ref → memory_projects |  | Project this action item advances. |
| `notes` | string |  | Extra detail or context (markdown). |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_wins` — Wins

Brag-doc entries. Things you shipped, problems you solved, people you helped. The core of performance-review prep.

| Field | Type | Req | Description |
|---|---|---|---|
| `title` | string | ✓ | Short headline for the accomplishment. |
| `occurredAt` | string (date) |  | When it happened. |
| `description` | string |  | The story — what you did and how (markdown). |
| `impact` | string |  | Measurable outcomes — metrics, $, headcount-time saved, etc. |
| `projectIds` | ref[] → memory_projects |  | Projects this win relates to. |
| `peopleIds` | ref[] → memory_people |  | Collaborators who contributed. |
| `competencyIds` | ref[] → memory_competencies |  | Competencies this win demonstrates. |
| `evidenceIds` | ref[] → memory_evidence |  | Hard artifacts backing the win. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_competencies` — Competencies

Skill rubric entries — the dimensions you're evaluated on (technical depth, leadership, communication, etc.). Wins, incidents, and evidence link here.

| Field | Type | Req | Description |
|---|---|---|---|
| `name` | string | ✓ | Competency name (e.g. "Technical depth"). |
| `category` | string |  | e.g. Technical, Leadership, Craft, Impact. |
| `level` | string |  | Self-assessed level — e.g. "Senior" or "3 / 5". |
| `description` | string |  | What this competency means at your company (markdown). |

## `memory_evidence` — Evidence

Hard artifacts — PRs, dashboards, screenshots, customer quotes. Pointer-with-context, not the artifact itself.

| Field | Type | Req | Description |
|---|---|---|---|
| `title` | string | ✓ | Short label for the artifact. |
| `kind` | enum | ✓ | Artifact type: pr, doc, screenshot, metric, quote, or other. Values: pr, doc, screenshot, metric, quote, other. |
| `url` | string (uri) |  | Canonical link to the artifact. |
| `description` | string |  | Why this evidence matters (markdown). |
| `occurredAt` | string (date) |  | When the artifact was created or captured. |
| `personId` | ref → memory_people |  | Who said it / made it (for quotes, PR authors). |
| `projectIds` | ref[] → memory_projects |  | Projects this evidence relates to. |
| `competencyIds` | ref[] → memory_competencies |  | Competencies this evidence supports. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_reviews` — Reviews

Performance review cycles. The aggregated brief — narrative pulled from wins, incidents, evidence over a period.

| Field | Type | Req | Description |
|---|---|---|---|
| `period` | string | ✓ | Cycle label, e.g. "H1 2026". |
| `kind` | enum | ✓ | Review type: self, peer, manager, or summary. Values: self, peer, manager, summary. |
| `personId` | ref → memory_people |  | The person being reviewed (often the user themselves). |
| `reviewerId` | ref → memory_people |  | Who is giving the review. |
| `narrative` | string |  | The review body — the written assessment (markdown). |
| `winIds` | ref[] → memory_wins |  | Wins cited in this review. |
| `incidentIds` | ref[] → memory_incidents |  | Incidents cited in this review. |
| `evidenceIds` | ref[] → memory_evidence |  | Evidence cited in this review. |
| `competencyIds` | ref[] → memory_competencies |  | Competencies covered by this review. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_goals` — Goals

North Star + quarterly objectives. The 'why' anchor — wins and projects can reference the goals they advance.

| Field | Type | Req | Description |
|---|---|---|---|
| `title` | string | ✓ | The goal stated as an outcome. |
| `horizon` | enum | ✓ | Time scale: north_star, annual, quarterly, or monthly. Values: north_star, annual, quarterly, monthly. |
| `status` | enum | ✓ | active → achieved / missed / dropped. Values: active, achieved, missed, dropped. |
| `description` | string |  | What the goal is and why it matters (markdown). |
| `metric` | string |  | How you'll know — the measurable target. |
| `startedAt` | string (date) |  | When the goal period begins. |
| `endedAt` | string (date) |  | When the goal period ends. |
| `parentGoalId` | ref → memory_goals |  | Higher-horizon goal this one supports. |
| `projectIds` | ref[] → memory_projects |  | Projects that advance this goal. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_topics` — Topics

Durable knowledge notes — "things I know about X". Conceptual, not time-bound. The wiki of your second brain.

| Field | Type | Req | Description |
|---|---|---|---|
| `title` | string | ✓ | The topic name (e.g. "Distributed consensus"). |
| `body` | string |  | The knowledge content (markdown). |
| `tagIds` | string[] |  | Loose labels for grouping topics. |
| `peopleIds` | ref[] → memory_people |  | People related to this topic. |
| `projectIds` | ref[] → memory_projects |  | Projects related to this topic. |
| `competencyIds` | ref[] → memory_competencies |  | Competencies related to this topic. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_patterns` — Patterns

Things that work. Reusable approaches the user has validated — "when X, do Y".

| Field | Type | Req | Description |
|---|---|---|---|
| `name` | string | ✓ | The pattern, named as a reusable approach (e.g. "Two-phase rollout for risky migrations"). |
| `whenToUse` | string |  | The trigger or situation where this pattern applies (markdown). |
| `approach` | string |  | The steps to follow (markdown). |
| `worksBecause` | string |  | Why the approach works (markdown). |
| `competencyIds` | ref[] → memory_competencies |  | Competencies this pattern exercises. |
| `evidenceIds` | ref[] → memory_evidence |  | Evidence that the pattern works. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_gotchas` — Gotchas

Things that don't work, edge cases, foot-guns. The 'don't do that again' notebook.

| Field | Type | Req | Description |
|---|---|---|---|
| `name` | string | ✓ | The gotcha, stated tightly (e.g. "Mongo $or doesn't use compound indexes"). |
| `symptom` | string |  | How the problem shows up (markdown). |
| `cause` | string |  | The root cause (markdown). |
| `avoidance` | string |  | The workaround or how to avoid it (markdown). |
| `projectIds` | ref[] → memory_projects |  | Projects bitten by this gotcha. |
| `incidentIds` | ref[] → memory_incidents |  | Incidents related to this gotcha. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

## `memory_notes` — Notes

Scratch / thinking. The catch-all for stuff that doesn't (yet) fit any other collection. Promote to a more specific kind once it solidifies.

| Field | Type | Req | Description |
|---|---|---|---|
| `title` | string | ✓ | Short label for the note. |
| `body` | string |  | The note content (markdown). |
| `tagIds` | string[] |  | Loose labels for triage. |
| `peopleIds` | ref[] → memory_people |  | People this note touches. |
| `projectIds` | ref[] → memory_projects |  | Projects this note touches. |
| `competencyIds` | ref[] → memory_competencies |  | Competencies this note touches. |
| `attachmentModelKey` | string |  | Collection key of the attached record (e.g. files, coding_agent_repos). Pair with attachmentId. |
| `attachmentId` | string |  | Optional polyref to any record in any collection. Resolves via attachmentModelKey. |

