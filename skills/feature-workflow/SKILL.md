---
name: feature-workflow
description: "Guided workflow for adding new features or fixing bugs in DockNimbus. Use this skill whenever the user asks to add a feature, fix a bug, implement a new API endpoint, add a CLI command, modify the UI, change agent behavior, or any task that involves changing functionality across the codebase layers (types, SDK, API handlers, repository, CLI, UI, agent). Trigger on phrases like 'add feature', 'fix bug', 'implement X', 'new command', 'new endpoint', 'agent not working', 'UI missing X', or any request that touches application logic. Even if the user mentions only one layer (e.g. 'fix the agent'), use this skill because the root cause often spans multiple layers."
---

# DockNimbus Feature Workflow

DockNimbus has a layered architecture where data flows through every layer. A bug in the agent might actually originate in the API handler not passing the right fields, which might originate in the CLI not sending them. A new feature that only touches the backend still needs its types exposed to the SDK so the CLI and UI can use them. This is why every change — whether a new feature or a bug fix — requires verifying all layers.

## Architecture Overview

The request flow is:

```
CLI (cmd/nimbus/main.go) ──┐
                           ├──► SDK (pkg/sdk/client.go) ──► API Handler (internal/handlers/) ──► Repository (internal/state/) ──► SQLite
UI  (ui/src/api.js)  ──────┘                                       │
                                                              Task Queue
                                                                   │
                                                              Agent (cmd/nimbus-agent/ + internal/agent/)
```

Every feature lives across these files:

| Layer | Location | Role |
|-------|----------|------|
| Types | `pkg/types/types.go` | Shared structs for requests, responses, models |
| SDK | `pkg/sdk/client.go` | Go HTTP client wrapping API calls — used by CLI and manifest system |
| Router | `internal/api/router.go` | Route registration and middleware |
| Handlers | `internal/handlers/<feature>.go` | Business logic, validation, task queuing |
| Repository | `internal/state/repository.go` | SQLite data access |
| Models | `internal/state/models.go` | DB row types and conversion helpers |
| CLI | `cmd/nimbus/main.go` | Cobra commands that call the SDK |
| Agent | `cmd/nimbus-agent/main.go` + `internal/agent/` | Executes tasks on nodes (Docker/K8s operations) |
| UI API | `ui/src/api.js` | JS wrapper for REST endpoints |
| UI Views | `ui/src/views/<Feature>.vue` | Vue 3 components |
| E2E Tests | `tests_e2e/` | End-to-end test scripts |
| Docs | `docs/` | User-facing documentation |

## Workflow

### Step 1: Understand the Change

Before writing any code, read the relevant existing code across all layers to understand the current state. Even for a "simple" bug fix, trace the data flow end-to-end:

1. Read the relevant type definitions in `pkg/types/types.go`
2. Read the SDK method in `pkg/sdk/client.go`
3. Read the handler in `internal/handlers/`
4. Read the repository method in `internal/state/repository.go`
5. Read the CLI command in `cmd/nimbus/main.go`
6. Read the agent task handler if it involves node-side execution
7. Read the UI api wrapper and view component
8. Read existing E2E tests for this feature

This upfront reading prevents the most common mistake: fixing one layer while breaking assumptions in another.

### Step 2: Start from Shared Types

Always begin changes at the type layer (`pkg/types/types.go`). This is the contract between all other layers. If you're adding a field, changing a struct, or creating a new request/response type, do it here first.

Follow the project's naming conventions:
- Request types: `Create<Resource>Request`, `Update<Resource>Request`
- Resources: exported structs with JSON tags
- Use consistent field names across the codebase

### Step 3: Implement Top-Down

Work through the layers in this order. Each layer builds on the previous one, so this order minimizes back-and-forth:

1. **Types** (`pkg/types/types.go`) — define or update structs
2. **Repository** (`internal/state/repository.go` + `models.go`) — add DB methods and model conversions
3. **Handler** (`internal/handlers/<feature>.go`) — implement business logic using the repository
4. **Router** (`internal/api/router.go`) — register new routes
5. **SDK** (`pkg/sdk/client.go`) — add client methods that call the new/changed endpoints
6. **Agent** (`internal/agent/`) — if the feature involves node-side execution, implement the task handler. Include clear error messages and debug logging so failures can be traced back through the layers
7. **CLI** (`cmd/nimbus/main.go`) — add Cobra commands that use the SDK
8. **UI API** (`ui/src/api.js`) — add JS API wrapper functions
9. **UI View** (`ui/src/views/<Feature>.vue`) — add or update the Vue component

### Step 4: Verify Data Consistency Across Layers

After implementing, verify that the same data flows correctly through every layer. Check for these common issues:

- **Field name mismatches**: JSON tags in Go types must match what the UI JavaScript expects. A field tagged `json:"instance_type"` in Go must be accessed as `instance_type` in JS, not `instanceType`.
- **Missing fields in SDK**: If the handler accepts a new field, the SDK method must pass it through. Check that the SDK struct or method signature includes the new field.
- **Missing fields in CLI**: If the SDK accepts a new parameter, the CLI command needs a flag for it. Check Cobra flag definitions.
- **Missing fields in UI**: If the API returns a new field, the Vue component should display it. If the API accepts a new field, the UI form should include it.
- **Agent task payload**: If the handler queues a task with new data, the agent must read and use that data. Log the received payload in the agent for debugging.
- **Missing API wrapper**: If a new endpoint was added, `ui/src/api.js` needs a corresponding function.

### Step 5: Error Handling and Debugging

For agent-related features, debugging is critical because the agent runs on remote nodes. Always:

- Log the full task payload when the agent receives it (helps trace what the API actually sent vs what was expected)
- Return descriptive error messages that identify which layer failed (e.g., "agent: failed to pull image X on node Y" rather than just "task failed")
- In handlers, validate inputs before queuing tasks — catching errors early at the API layer is much easier to debug than agent-side failures

### Step 6: Tests

Add or update E2E tests in `tests_e2e/`. Follow the existing test patterns in that directory. Tests should exercise the feature through the CLI or API (not by calling internal functions directly) to verify the full stack works together.

### Step 7: Documentation

Update documentation in `docs/` to reflect the changes. If a new command, endpoint, or UI feature was added, document it. If behavior changed, update the existing docs. Check that:

- CLI help text (Cobra `Short`/`Long` descriptions) is accurate
- API endpoint documentation matches the actual request/response format
- Any new configuration options are documented

### Step 8: Final Cross-Layer Checklist

Before considering the work done, go through this checklist:

- [ ] Types in `pkg/types/types.go` match what all layers expect
- [ ] SDK methods in `pkg/sdk/client.go` cover all new/changed endpoints
- [ ] Handler validates input and returns proper HTTP status codes
- [ ] Route is registered in `internal/api/router.go`
- [ ] Repository methods handle the new data correctly
- [ ] CLI flags and commands use the SDK (no direct HTTP calls)
- [ ] Agent handles the task and logs useful debug info (if applicable)
- [ ] `ui/src/api.js` has wrapper functions for new/changed endpoints
- [ ] Vue components display and submit all relevant fields
- [ ] E2E tests cover the new/changed functionality
- [ ] Documentation is updated
- [ ] No field name mismatches between Go JSON tags and JS property access
- [ ] No duplicated logic — shared code lives in `pkg/` or handler utilities

## Bug Fix Pattern

When fixing a bug, the instinct is to find the broken layer and fix it there. Resist this. Instead:

1. **Reproduce** the issue and identify the symptom
2. **Trace the data flow** from the entry point (CLI/UI) through SDK → API → handler → repo → agent
3. **Find where the data diverges** from what's expected — this is the actual bug location, which is often upstream from where the symptom appears
4. **Fix at the source**, then verify all downstream layers handle the corrected data properly
5. **Add a test** that would have caught this bug

A common example: "the agent isn't creating the container correctly" often turns out to be "the handler isn't including field X in the task payload" or "the CLI isn't sending parameter Y to the SDK."
