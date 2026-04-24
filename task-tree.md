# Task Tree & Context Management

Loaded when the Split Decision in SKILL.md evaluates to `SPLIT = true`.

## 1. Task Decomposition Tree

Decompose the current phase's work into a tree:

```
<Phase N>/
├── index.md          ← shared contracts, dependency map, summaries (written FIRST)
├── module-a/
│   └── <deliverable>
├── module-b/
│   └── <deliverable>
└── module-c/
    └── <deliverable>
```

**Directory structure** (project-root relative):

```
.tdd-pipeline/<feature-name>/
├── tree.md                  ← status tracker
├── phase-1/
│   ├── index.md
│   └── modules/
│       ├── user-auth.md
│       └── user-profile.md
├── phase-2/
│   ├── index.md
│   └── modules/
│       ├── auth-service.md
│       └── profile-service.md
├── phase-3/  (same pattern)
├── phase-4/  (test files organized by module)
└── phase-5/  (business code organized by module)
```

## 2. Execution Protocol

```
// Intra-phase: index-first → parallel modules → merged review
function execute_phase(phase):
    if SPLIT:
        // Step 1: index.md defines shared contracts FIRST
        write("index.md", {
            shared_contracts,    // interfaces, data models, cross-cutting constraints
            dependency_map,      // key=dependant, value=list of deps (e.g., {module-b: [module-a]})
            execution_order,     // ordered list of sub-tasks for sequential traversal
            module_summaries,    // 1–3 sentences per module
            cross_cutting_concerns,  // shared constraints, open questions
        })

        // Step 2: modules in parallel
        // Each module aligns goals with parent, reads needed context, then writes
        parallel for module in modules:
            align_goals(module)     // read parent to understand full scope
            read_context(module)    // incrementally read siblings/ancestors as needed
            write(module)

        // Step 3: merged single Ralph loop
        ralph_review(all modules + index.md as ONE deliverable)
    else:
        write(single_document)
        ralph_review(single_document)
```

**Dependency rule**: Modules depend on each other ONLY through shared contracts in `index.md`. If a module's internals depend on another module's internals, the module boundary is wrong — redesign the split.

**Recursive decomposition**: If a sub-task itself triggers the split threshold, decompose it recursively following the same index-first pattern.

## 3. Context Reading Protocol

When a sub-task executes, read incrementally. Start minimal, expand only as needed.

```
// Called by execute_phase before read_context
function align_goals(current_task):
    read(parent.index_md)
    if scope_unclear:
        read(parent.full_document)    // read complete parent BEFORE starting work
    // parent.index_md is now in context; read_context skips Level 1

function read_context(current_task):
    // Level 1: Skip if align_goals already loaded parent.index_md
    if NOT already_read(parent.index_md):
        read(parent.index_md)

    // Level 2: Dependent siblings
    if questions_remain:
        for sibling in parent.dependency_map[current_task]:
            read(sibling.full_document)

    // Level 3: All upstream siblings
    if questions_remain:
        for sibling in parent.execution_order:
            read(sibling.full_document)

    // Level 4: Full parent (may already be loaded by align_goals)
    if questions_remain AND NOT already_read(parent.full_document):
        read(parent.full_document)

    // Level 5: Grandparent (2 generations up max; deeper trees stop here)
    if questions_remain AND parent.parent EXISTS:
        read(grandparent.index_md)
        if questions_remain:
            read(grandparent.full_document)

    // If still unresolved — context exhausted, escalate to user
    if questions_remain:
        write_open_question(current_task, question, blocker=true)
        report_to_user(question)
        HALT
```

**`index.md` MUST contain**:
- **Dependency map**: map from each sub-task to its dependencies (key = dependant, value = list of deps it requires). Example: `{module-b: [module-a]}` means module-b depends on module-a
- **Execution order**: ordered list of sub-tasks for sequential traversal (used by Level 3 context reading)
- **Module summaries**: 1–3 sentences per sub-task (sufficient for sibling awareness)
- **Shared contracts**: interfaces, data models, constraints all modules must conform to
- **Cross-cutting concerns**: shared constraints, open questions

## 4. Document Versioning

Every intermediate document must include a version header:

```yaml
---
version: 2
date: 2026-04-24
previous_version: 1
change: "Phase 2 review: restructured module split — separated PaymentProcessor into Validator + Executor"
---
```

| Trigger | Action |
|---------|--------|
| Re-run after review feedback | Increment version, describe what changed |
| Rollback to prior phase + re-execute | New output starts at version 1; `change` references discarded version |
| Minor edit (typos, formatting) | No version bump |

## 5. Status Tracking

`tree.md` tracks every node's status:

```markdown
# Task Tree: <Feature Name>

## Phase 1 — Product Design [COMPLETE]
- [x] index.md
- [x] modules/user-auth.md
- [x] modules/user-profile.md

## Phase 2 — Technical Solution [IN PROGRESS]
- [x] index.md
- [x] modules/auth-service.md
- [ ] modules/profile-service.md  ← current

## Phase 3 — Test Plan [PENDING]
## Phase 4 — Test Code [PENDING]
## Phase 5 — Business Code [PENDING]
```

**Rollback handling**: Mark discarded nodes as `[DISCARDED — rollback to Phase N]`. Do not delete — preserve history.
