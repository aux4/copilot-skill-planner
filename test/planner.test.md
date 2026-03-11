# Copilot Skill Planner Tests

Tests for the `aux4 copilot skills planner` commands.

## Test 1: Prompt command

### should output planner instructions

```execute
aux4 copilot skills planner prompt
```

```expect:partial
# Task Planner Skill
```

### should include phase descriptions

```execute
aux4 copilot skills planner prompt
```

```expect:partial
### Phase 1: Understand
```

### should include critical rules

```execute
aux4 copilot skills planner prompt
```

```expect:partial
## CRITICAL RULES
```

### should include todo command reference

```execute
aux4 copilot skills planner prompt
```

```expect:partial
## Todo Command Reference
```

## Test 2: Status command

```beforeEach
rm -f .todo.json
```

```afterAll
rm -f .todo.json
```

### should show status command parameters

```execute
aux4 copilot skills planner status --help
```

```expect:partial
Name of the plan to view
```

### should show file parameter

```execute
aux4 copilot skills planner status --help
```

```expect:partial
Todo file path
```

### should show todo list status

```execute
aux4 todo new "test-plan" --item "Step one" && aux4 copilot skills planner status test-plan
```

```expect
Todo 'test-plan' created.
## test-plan
  0: [ ] Step one
```

### should show completed items

```execute
aux4 todo new "status-plan" --item "First step" && aux4 todo add "status-plan" --item "Second step" && aux4 todo complete "status-plan" --index 0 && aux4 copilot skills planner status status-plan
```

```expect:regex
Todo 'status-plan' created.
Item added to 'status-plan'.
Item 0 in 'status-plan' marked as completed.
## status-plan
  0: \[x\] .*̶.*
  1: \[ \] Second step
```

## Test 3: Plan command help

### should show plan command parameters

```execute
aux4 copilot skills planner plan --help
```

```expect:partial
The task to plan
```

### should show name parameter

```execute
aux4 copilot skills planner plan --help
```

```expect:partial
Name for the plan and todo list
```

## Test 4: Execute command help

### should show execute command parameters

```execute
aux4 copilot skills planner execute --help
```

```expect:partial
Name of the plan to execute
```

## Test 5: Planner skill registration

### should appear in copilot skills

```execute
aux4 copilot skills --help
```

```expect:partial
planner
```
