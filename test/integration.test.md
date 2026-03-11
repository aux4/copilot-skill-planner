# Copilot Planner Integration Test

End-to-end test: ask the copilot to plan a task, then execute it, and verify the planner skill is used.

```beforeAll
rm -f .todo.json history-plan.json history-exec.json
```

```afterAll
rm -f .todo.json history-plan.json history-exec.json .aux4
```

## Test 1: Plan a task

### should create a plan via copilot ask

```timeout
180000
```

```execute
rm -f .aux4 && aux4 copilot ask "plan creating a simple greet command that prints hello world" --history history-plan.json 2>&1; echo "DONE"
```

```expect:partial
DONE
```

### should have used the planner skill

```execute
grep -c "copilot skills planner" history-plan.json | xargs -I{} sh -c 'if [ {} -gt 0 ]; then echo "planner used"; else echo "planner not used"; fi'
```

```expect
planner used
```

### should have created a todo list

```execute
grep -c "todo new" history-plan.json | xargs -I{} sh -c 'if [ {} -gt 0 ]; then echo "todo created"; else echo "todo not created"; fi'
```

```expect
todo created
```

### should have written a plan file

```execute
grep -c "plan.md" history-plan.json | xargs -I{} sh -c 'if [ {} -gt 0 ]; then echo "plan saved"; else echo "plan not saved"; fi'
```

```expect
plan saved
```

### should have asked user for confirmation

```execute
grep -c "askUser" history-plan.json | xargs -I{} sh -c 'if [ {} -gt 0 ]; then echo "user asked"; else echo "user not asked"; fi'
```

```expect
user asked
```

### should have created .todo.json file

```execute
test -f .todo.json && echo "exists" || echo "missing"
```

```expect
exists
```

### should have pending tasks in todo list

```execute
name=$(jq -r 'keys[0]' .todo.json) && aux4 todo view "$name" | grep -c "\[ \]" | xargs -I{} sh -c 'if [ {} -gt 0 ]; then echo "has pending"; else echo "no pending"; fi'
```

```expect
has pending
```

## Test 2: Execute the plan

### should execute the plan via copilot ask

```timeout
180000
```

```execute
name=$(jq -r 'keys[0]' .todo.json) && rm -f .aux4 && aux4 copilot ask "execute plan $name" --history history-exec.json 2>&1; echo "DONE"
```

```expect:partial
DONE
```

### should have called todo complete

```execute
grep -c "todo complete" history-exec.json | xargs -I{} sh -c 'if [ {} -gt 0 ]; then echo "tasks completed"; else echo "tasks not completed"; fi'
```

```expect
tasks completed
```

### should have marked all tasks as done

```execute
name=$(jq -r 'keys[0]' .todo.json) && aux4 todo view "$name" | grep -c "\[x\]" | xargs -I{} sh -c 'if [ {} -gt 0 ]; then echo "all done"; else echo "incomplete"; fi'
```

```expect
all done
```
