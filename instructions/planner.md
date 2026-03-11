# Task Planner Skill

## Commands

- `aux4 copilot skills planner plan` - Plan and execute a multi-step task
- `aux4 copilot skills planner execute` - Resume execution of an existing plan
- `aux4 copilot skills planner status` - View the status of a plan (no AI cost)

Run `aux4 copilot skills planner <command> --help` for parameter details.

## Task: Plan and Execute Multi-Step Tasks

Follow these phases in order. Do NOT skip any phase.

### Phase 1: Understand

1. Read the task description carefully
2. Identify any gaps or ambiguities in the requirements
3. Use `askUser` to clarify — ask one question at a time, wait for the answer before asking the next
4. If the task involves code, use `readFile` to examine relevant files
5. If the task involves an aux4 package, read the `.aux4` file to understand the command structure
6. Gather enough context to create a concrete, actionable plan

### Phase 2: Plan

Break the task into 3–10 actionable steps. Each step should:
- Start with a verb (e.g., "Create", "Update", "Add", "Configure", "Test")
- Be specific — include target file paths or command names
- Be independently verifiable

Derive a short plan name from the task (e.g., "add-export-command", "fix-auth-flow"). Use this as `<name>` for the todo list and plan file.

Create the task list using `executeAux4`:

```
todo new "<name>" --item "Step 1 description" --file <file>
```

Then add remaining items one at a time:

```
todo add "<name>" --item "Step 2 description" --file <file>
todo add "<name>" --item "Step 3 description" --file <file>
```

Use the `--file` parameter value from the command variables. If not set, default to `.todo.json`.

### Phase 3: Save Plan

Write a `<name>.plan.md` file (or use the `output` variable if provided) containing:

1. **Task** — the original task description
2. **Context** — rationale, approach, constraints gathered during Phase 1
3. **Steps** — the task list as a readable numbered checklist

Use `writeFile` to create this file. Example structure:

```
# Plan: <name>

## Task
<original task description>

## Context
<what was learned during clarification — approach, constraints, relevant files>

## Steps
1. [ ] Step 1 description
2. [ ] Step 2 description
3. [ ] Step 3 description
```

### Phase 4: Confirm

1. Present the plan to the user — show the content of the plan file
2. Use `askUser`: "Does this plan look good? You can say 'yes' to proceed, or suggest changes."
3. If the user requests changes:
   - Add/remove todo items as needed (`todo add`, `todo remove`)
   - Update the plan file with `editFile`
   - Ask for confirmation again
4. Do NOT proceed to execution until the user confirms

### Phase 5: Execute

For each incomplete task in the todo list, you MUST follow ALL of these steps in order:

1. Announce which step you are starting (e.g., "Starting step 1: Create the config file")
2. Do the work using available tools (`readFile`, `writeFile`, `editFile`, `executeAux4`, `listFiles`, `searchFiles`)
3. Verify the step was completed correctly (read the output, check the file, run a test)
4. **MANDATORY: Mark the step complete by calling `executeAux4` with `todo complete "<name>" --index <N> --file <file>`** (0-based index). You MUST call this for every successfully completed step. This is NOT optional — the todo list is the source of truth for progress tracking.
5. Use `askUser`: "Step N complete. Ready to continue with the next step?"
6. If the user says to stop, pause execution — they can resume later with `aux4 copilot skills planner execute <name>`

**Example execution of a step:**
```
# 1. Announce
"Starting step 0: Create .aux4 configuration file"

# 2. Do the work
executeAux4: (create/edit files as needed)

# 3. Verify
readFile: (check the file was created correctly)

# 4. MANDATORY: Mark complete
executeAux4: todo complete "my-plan" --index 0 --file .todo.json

# 5. Ask user
askUser: "Step 0 complete. Ready to continue with the next step?"
```

**On failure:**
- Do NOT mark the step as complete
- Explain what went wrong
- Use `askUser`: "This step failed. Would you like to retry, skip, or modify the plan?"
- Act on the user's response

### Phase 6: Summary

After all steps are complete:

1. Show the final todo list: `todo view "<name>" --file <file>`
2. Summarize what was accomplished
3. Note any steps that were skipped or modified

## Resuming Execution

When called with "Execute plan: <name>":

1. Read the plan file (`<name>.plan.md`) to understand the context
2. View the todo list via `executeAux4`: `todo view "<name>" --file <file>` to see which steps are incomplete
3. For EACH incomplete step (the index is the number shown before the `[ ]` in `todo view` output):
   a. Do the work
   b. Verify it succeeded
   c. Call `executeAux4` with: `todo complete "<name>" --index <INDEX> --file <file>` where `<INDEX>` is the number from the `todo view` output (e.g., `0:`, `1:`, `2:`)
4. After all steps are done, call `executeAux4` with `todo view "<name>" --file <file>` to show the final state
5. Summarize what was accomplished

**Example: todo view shows:**
```
## my-plan
  0: [ ] Create the config file
  1: [ ] Write tests
```

**To complete step 0, call:** `todo complete "my-plan" --index 0 --file .todo.json`
**To complete step 1, call:** `todo complete "my-plan" --index 1 --file .todo.json`

The index is ALWAYS the number shown at the beginning of each line in the `todo view` output.

## CRITICAL RULES

**ALWAYS:**
- Ask the user to confirm the plan before executing (Phase 4)
- Use `aux4 todo` commands via `executeAux4` to manage the task list
- Call `executeAux4` with `todo complete "<name>" --index <N> --file <file>` after EVERY successfully completed step — this is mandatory, not optional
- Ask for confirmation between steps during execution
- Include the `--file` parameter in all todo commands

**NEVER:**
- Start executing without user confirmation
- Write `.todo.json` directly — always use `aux4 todo` commands via `executeAux4`
- Call `askUser` in parallel with other tools — it must be called alone
- Mark a step as complete if it failed
- Skip Phase 4 (confirmation) even if the task seems straightforward

## Todo Command Reference

All commands are run via `executeAux4` (without the `aux4` prefix):

| Command | Description |
|---------|-------------|
| `todo new "<name>" --item "text" --file <f>` | Create a new list with first item |
| `todo add "<name>" --item "text" --file <f>` | Add an item to an existing list |
| `todo remove "<name>" --index <N> --file <f>` | Remove an item (0-based index) |
| `todo complete "<name>" --index <N> --file <f>` | Mark item complete (0-based index) |
| `todo view "<name>" --file <f>` | View a list with all items |
| `todo list --file <f>` | List all todo lists with progress |
| `todo delete "<name>" --file <f>` | Delete an entire list |

## Token Management

- Do NOT read every file in a project — focus on the files relevant to the current step
- Summarize findings in your own words — do not paste entire file contents
- When examining code, read only the sections you need to understand or modify
