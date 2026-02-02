---
description: Creates an implementation plan with team orchestration — named team members, task dependencies, and parallel/sequential execution
argument-hint: <user-prompt> [orchestration-prompt]
model: opus
disallowed-tools: Task, EnterPlanMode
---

# Plan With Team

Create a detailed implementation plan based on the user's requirements provided through the `USER_PROMPT` variable. Analyze the request, think through the implementation approach, and save a comprehensive specification document to `PLAN_OUTPUT_DIRECTORY/<name-of-plan>.md` that can be used as a blueprint for team-orchestrated development. Follow the `Instructions` and work through the `Workflow` to create the plan.

## Variables

USER_PROMPT: $1
ORCHESTRATION_PROMPT: $2 - (Optional) Guidance for team assembly, task structure, and execution strategy
PLAN_OUTPUT_DIRECTORY: `.planning/phases/`
TEAM_MEMBERS: `agents/team/*.md` (from ark-ai-agent plugin)
GENERAL_PURPOSE_AGENT: `general-purpose`

## Instructions

- **PLANNING ONLY**: Do NOT build, write code, or deploy agents. Your only output is a plan document saved to `PLAN_OUTPUT_DIRECTORY`.
- If no `USER_PROMPT` is provided, stop and ask the user to provide it.
- If `ORCHESTRATION_PROMPT` is provided, use it to guide team composition, task granularity, dependency structure, and parallel/sequential decisions.
- Carefully analyze the user's requirements provided in the USER_PROMPT variable
- Determine the task type (chore|feature|refactor|fix|enhancement) and complexity (simple|medium|complex)
- Think deeply (ultrathink) about the best approach to implement the requested functionality or solve the problem
- Understand the codebase directly without subagents to understand existing patterns and architecture
- Follow the Plan Format below to create a comprehensive implementation plan
- Include all required sections and conditional sections based on task type and complexity
- Generate a descriptive, kebab-case filename based on the main topic of the plan
- Save the complete implementation plan to `PLAN_OUTPUT_DIRECTORY/<descriptive-name>.md`
- Ensure the plan is detailed enough that another developer could follow it to implement the solution
- **Scope rule**: 2-3 tasks per plan, 50% context max. Claude degrades at ~40-50% context usage.
- Understand your role as the team lead. Refer to the `Team Orchestration` section for more details.

### Team Orchestration

As the team lead, you design the team and task structure in the plan. The `/build` command will later orchestrate execution using these tools — you document HOW they should be used, not use them yourself.

#### Task Management Tools (reference for plan design)

**TaskCreate** - Create tasks in the shared task list:
```typescript
TaskCreate({
  subject: "Implement user authentication",
  description: "Create login/logout endpoints with JWT tokens.",
  activeForm: "Implementing authentication"
})
```

**TaskUpdate** - Update task status, assignment, or dependencies:
```typescript
TaskUpdate({
  taskId: "1",
  status: "in_progress",
  addBlockedBy: ["0"]  // Task 1 blocked until Task 0 completes
})
```

**Task** - Deploy an agent to do work:
```typescript
Task({
  description: "Implement auth endpoints",
  prompt: "...",
  subagent_type: "team:builder",  // or specific agent type
  model: "opus",
  run_in_background: false  // true for parallel execution
})
```

#### Agent Types Available

- **team:builder** — Default. Focused execution agent from `agents/team/builder.md`
- **team:validator** — Read-only verification agent from `agents/team/validator.md`
- **general-purpose** — General-purpose agent for tasks not fitting builder/validator
- Any registered subagent (e.g., `database-engineer`, `training-agent`, `deployment-coordinator`)

## Workflow

IMPORTANT: **PLANNING ONLY** - Do not execute, build, or deploy. Output is a plan document.

1. Analyze Requirements - Parse the USER_PROMPT to understand the core problem and desired outcome
2. Understand Codebase - Without subagents, directly understand existing patterns, architecture, and relevant files
3. Design Solution - Develop technical approach including architecture decisions and implementation strategy
4. Define Team Members - Use `ORCHESTRATION_PROMPT` (if provided) to guide team composition. Identify from `agents/team/*.md` or use `general-purpose`. Document in plan.
5. Define Step by Step Tasks - Use `ORCHESTRATION_PROMPT` (if provided) to guide task granularity and parallel/sequential structure. Write out tasks with IDs, dependencies, assignments. Document in plan.
6. Generate Filename - Create a descriptive kebab-case filename based on the plan's main topic
7. Save Plan - Write the plan to `PLAN_OUTPUT_DIRECTORY/<filename>.md`
8. Report - Follow the `Report` section

## Plan Format

- IMPORTANT: Replace `<requested content>` with the requested content.
- IMPORTANT: Anything NOT in `<requested content>` should be written EXACTLY as it appears below.
- IMPORTANT: Follow this EXACT format:

```md
# Plan: <task name>

## Task Description
<describe the task in detail based on the prompt>

## Objective
<clearly state what will be accomplished when this plan is complete>

<if task_type is feature or complexity is medium/complex, include these sections:>
## Problem Statement
<clearly define the specific problem or opportunity this task addresses>

## Solution Approach
<describe the proposed solution approach and how it addresses the objective>
</if>

## Relevant Files
Use these files to complete the task:

<list files relevant to the task with bullet points explaining why. Include new files to be created under an h3 'New Files' section if needed>

<if complexity is medium/complex, include this section:>
## Implementation Phases
### Phase 1: Foundation
<describe any foundational work needed>

### Phase 2: Core Implementation
<describe the main implementation work>

### Phase 3: Integration & Polish
<describe integration, testing, and final touches>
</if>

## Team Orchestration

- You operate as the team lead and orchestrate the team to execute the plan.
- You're responsible for deploying the right team members with the right context.
- IMPORTANT: You NEVER operate directly on the codebase. You use `Task` and `Task*` tools to deploy team members for building, validating, testing, and other tasks.
- You validate all work is progressing correctly and ensure the team is on track.
- You use Task* tools to manage coordination between team members.
- Take note of the session id of each team member for reference.

### Team Members
<list the team members needed to execute the plan>

- Builder
  - Name: <unique name for this builder>
  - Role: <the single role and focus of this builder>
  - Agent Type: <subagent type — team:builder, general-purpose, or a specific registered agent>
  - Resume: <default true. Pass false to start fresh.>
- <continue with additional team members as needed>

## Step by Step Tasks

- IMPORTANT: Execute every step in order, top to bottom. Each task maps directly to a `TaskCreate` call.
- Before starting, run `TaskCreate` to create the initial task list that all team members can see and execute.

<list step by step tasks as h3 headers>

### 1. <First Task Name>
- **Task ID**: <unique kebab-case identifier>
- **Depends On**: <Task ID(s) this depends on, or "none">
- **Assigned To**: <team member name from Team Members section>
- **Agent Type**: <subagent type>
- **Parallel**: <true if can run alongside other tasks, false if sequential>
- <specific action to complete>
- <specific action to complete>

### 2. <Second Task Name>
- **Task ID**: <unique-id>
- **Depends On**: <previous Task ID>
- **Assigned To**: <team member name>
- **Agent Type**: <subagent type>
- **Parallel**: <true/false>
- <specific action>
- <specific action>

### N. <Final Validation Task>
- **Task ID**: validate-all
- **Depends On**: <all previous Task IDs>
- **Assigned To**: <validator team member>
- **Agent Type**: team:validator
- **Parallel**: false
- Run all validation commands
- Verify acceptance criteria met

## Acceptance Criteria
<list specific, measurable criteria that must be met>

## Validation Commands
Execute these commands to validate the task is complete:

<list specific commands to validate the work>
```

## Report

After creating and saving the implementation plan, provide a concise report:

```
Implementation Plan Created

File: PLAN_OUTPUT_DIRECTORY/<filename>.md
Topic: <brief description of what the plan covers>
Key Components:
- <main component 1>
- <main component 2>
- <main component 3>

Team Task List:
- <list of tasks and owner (concise)>

Team members:
- <list of team members and their roles (concise)>

When ready, execute with:
/build <path to plan>
```
