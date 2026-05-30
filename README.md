# Mind

**Your second brain for TheItemApp.** Mind is a personal-memory app: a persistent knowledge vault that survives across AI sessions. People, projects, decisions, incidents, wins, competencies, and more — every record links to the others, so a knowledge graph builds itself as you (or an agent) work.

Mind is **seed-only and fully meta-driven**: it ships no application code. Everything — the 16 collections, their schemas, dashboards, and windows — is defined as platform data records under `dbseed/` and installed by the core seeder. Behavior is steered by the schema registry (`items`) and the live records, not by a server.

- **App id:** `900000000000000000000001` (`apps.key = "mind"`)
- **Icon:** `lucide:brain`
- **Collections:** 16 `memory_*` models (see below)

## The graph

The whole point of Mind is the links. `memory_people` is the central node — almost every other collection references it. Work is anchored under `memory_projects`; provable accomplishment runs through `memory_wins` → `memory_competencies` / `memory_evidence`; and the catch-all `memory_notes` feeds the more structured collections as raw thoughts solidify. A record without links is considered a bug.

```
                         memory_teams
                              ▲
                  team        │ team / parentTeam
        ┌──────────────── memory_people ───────────────┐
        │ owner/manager      ▲  ▲  ▲                    │ people
        │              people│  │  │people              ▼
 memory_action_items   memory_meetings           memory_decisions
        │ project            │ projects                │ projects
        ▼                    ▼                         ▼
                         memory_projects ◄──── memory_incidents
                          ▲  ▲  ▲  ▲                    │ competencies
              projects    │  │  │  │ projects           ▼
   memory_goals ──────────┘  │  │  └────────── memory_gotchas
        (parentGoal)         │  │
                    memory_wins  memory_topics / memory_patterns / memory_notes
                       │  │            │ competencies / evidence
            competencies│  │evidence   ▼
                        ▼  ▼      memory_competencies ◄── memory_evidence
                  memory_competencies          ▲
                        ▲                       │ wins/incidents/evidence/competencies
                        └──────────────── memory_reviews
```

## Collections

| Collection | What it holds | Key links |
|---|---|---|
| `memory_people` | Humans you interact with — the central node | `teamId`, `managerId` |
| `memory_teams` | Org units; people belong, projects are owned | `parentTeamId` |
| `memory_projects` | Time-boxed work; the anchor for most memory | `teamId`, `leadPersonId`, `collaboratorIds` |
| `memory_meetings` | 1:1s, standups, planning, customer calls | `peopleIds`, `projectIds` |
| `memory_decisions` | ADRs and durable choices (context/decision/rationale) | `projectIds`, `peopleIds`, `supersedesId` |
| `memory_incidents` | Outages and postmortems (root cause + remediation) | `projectIds`, `peopleIds`, `competencyIds` |
| `memory_action_items` | Discrete TODOs with one owner | `ownerId`, `meetingId`, `projectId` |
| `memory_wins` | Brag-doc entries — review-prep fuel | `projectIds`, `peopleIds`, `competencyIds`, `evidenceIds` |
| `memory_competencies` | Skill-rubric dimensions you're evaluated on | (linked from wins/incidents/evidence) |
| `memory_evidence` | Hard artifacts (PRs, docs, screenshots, quotes) | `personId`, `projectIds`, `competencyIds` |
| `memory_reviews` | Performance-review cycles — the aggregated brief | `personId`, `reviewerId`, `winIds`, `incidentIds`, `evidenceIds`, `competencyIds` |
| `memory_goals` | North Star + quarterly objectives | `parentGoalId`, `projectIds` |
| `memory_topics` | Durable knowledge notes — the wiki | `peopleIds`, `projectIds`, `competencyIds` |
| `memory_patterns` | Things that work — reusable playbook | `competencyIds`, `evidenceIds` |
| `memory_gotchas` | Things that don't work — foot-guns | `projectIds`, `incidentIds` |
| `memory_notes` | Scratch / inbox; promote into the structured kinds | `peopleIds`, `projectIds`, `competencyIds` |

Every collection also carries a polymorphic attachment pair — `attachmentModelKey` + `attachmentId` (`x-refPath`) — to point at any record in any collection (a file, a repo, etc.).

Full per-field reference: [`docs/data-model.md`](docs/data-model.md).

## How agents use it

Mind is designed for an AI agent to write into as the user talks (see `apps.ai.systemPrompt`). When the user mentions a person, decision, win, or anything a `memory_*` collection can hold, the agent creates or updates the relevant record **and** populates every x-ref that applies. The markdown body field on each record (`notes` / `body` / `description` / `narrative`) is the primary content.

## Layout

```
dbseed/
  apps.json        app catalog record (shared)
  items.json       the 16 memory_* model definitions (schema registry)
  users.json       functional app user (shared)
  dashboards.json  install-config dashboards
  windows.json     install-config windows
  manifest.json    seed manifest (collections, matchBy, exportFields)
docker-compose.yml / Dockerfile.seeds   one-shot seed server for install
```

## Local commands

```bash
npm start       # docker compose up -d --build (seed server) — alias: npm run up
npm run logs    # follow seed logs
npm stop        # docker compose down — alias: npm run down
```

The seed server registers the app against core on boot. The `mind-seeds` service reads an
optional `APP_REGISTRATION_KEY` from the environment (passed through by `docker-compose.yml`);
set it if core requires a registration key.

Re-export the seed from the live DB after editing records via MCP/UI:
`repo_export_seed` against the Mind repo, then commit the `dbseed/` diff.
