# aux4/copilot-skill-planner

Copilot skill for planning and executing multi-step tasks.

This skill extends the aux4 copilot with the ability to break down complex tasks into structured plans, create task lists, and execute them step by step with user confirmation at each stage.

## Installation

```bash
aux4 aux4 pkger install aux4/copilot-skill-planner
```

## Usage

### With Copilot Chat

Start a copilot chat session and ask it to plan a task:

```bash
aux4 copilot chat
> Plan adding a new command to the todo package
```

The copilot will discover this skill, create a plan, and guide you through execution.

### Direct Commands

Plan a task:

```bash
aux4 copilot skills planner plan "add export command to the config package"
```

Resume execution of an existing plan:

```bash
aux4 copilot skills planner execute add-export-command
```

Check plan status:

```bash
aux4 copilot skills planner status add-export-command
```

### Get Skill Prompt

To see what instructions this skill provides to the copilot:

```bash
aux4 copilot skills planner prompt
```

## Commands

- `aux4 copilot skills planner prompt` - Get the skill instructions
- `aux4 copilot skills planner plan` - Plan and execute a multi-step task
- `aux4 copilot skills planner execute` - Resume execution of an existing plan
- `aux4 copilot skills planner status` - View the status of a plan's task list

## Workflow

1. **Understand** - Analyze the task, ask clarifying questions
2. **Plan** - Break into actionable steps, create a todo list
3. **Save** - Write a `.plan.md` file with context and steps
4. **Confirm** - Present the plan and get user approval
5. **Execute** - Work through each step with verification
6. **Summary** - Report what was accomplished

## License

This package is licensed under the Apache-2.0 License.

See [LICENSE](./LICENSE) for details.
