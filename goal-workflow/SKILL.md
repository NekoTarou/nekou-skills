---
name: goal-workflow
description: Turn a user's task into a reviewed Pi Goal mission and ordered, verifiable objectives, then configure pi-goal-tool and start its Autopilot only after explicit human approval. Use this whenever the user asks to plan and automatically execute a task with Goal, split work into Goal steps, set a mission and goals, or start a controlled Goal/Autopilot workflow—even if they do not name this skill. Also use it when revising a not-yet-approved Goal plan. Do not use it merely to inspect or manually edit existing goals.
compatibility: Requires pi-goal-tool and its goal_list, goal_set_mission, goal_add, and goal_auto_on tools.
---

# Goal Workflow

Convert one user task into a safe, reviewable Goal plan. Keep planning separate from execution: the user should see and approve the exact mission, goals, and round limit before any Goal state changes.

## Non-negotiable boundary

Do not call `goal_set_mission`, `goal_add`, `goal_auto_on`, or any other state-changing Goal tool before the user explicitly approves the displayed proposal.

Planning approval authorizes only the approved Goal configuration and its Autopilot start. It does not bypass environment permission prompts or authorize destructive actions beyond the user's original request.

## Required tools

Use these tools from `pi-goal-tool`:

- `goal_list({})` reads current state.
- `goal_set_mission({ text })` writes the global mission.
- `goal_add({ text })` appends one goal and returns its number.
- `goal_auto_on({ maxRounds })` starts Autopilot immediately.

If any required tool is unavailable, stop and explain that `pi-goal-tool` must be installed and Pi restarted or `/reload` run. Do not pretend that plain text or slash commands changed Goal state.

## Phase 1: Preflight

1. Call `goal_list({})` before drafting.
2. Inspect structured details when available. Treat every goal with `done: false` as unfinished; when only text is available, treat `[ ]` entries as unfinished.
3. If any unfinished goal exists, stop without changing anything. Show the unfinished goal numbers and tell the user to handle them with the relevant commands:

```text
/goal
/goal done <编号>
/goal rm <编号>
/goal auto off
```

Say that they can invoke this workflow again after the old unfinished goals are resolved. Do not append, replace, delete, complete, or start existing goals on the user's behalf.

Completed historical goals do not block a new proposal.

## Phase 2: Understand and decompose

Use the user's request plus safe, read-only inspection of the current project when it materially improves the plan. Ask a concise clarifying question only when an unresolved choice would change the deliverables or acceptance criteria.

Draft:

- One mission: one to three sentences that preserve the requested outcome, important constraints, and definition of success.
- Normally three to seven ordered goals. Use fewer for a small task and more only when genuinely necessary.
- A positive integer round limit. Default to `10` unless the user explicitly requests another value.

Each goal should:

- produce an observable result rather than describe ongoing activity;
- include its own completion evidence or acceptance condition;
- be executable in sequence without silently expanding scope;
- be large enough to represent meaningful progress but small enough to verify;
- preserve testing, validation, and final review as explicit work when relevant.

Avoid goals such as “continue working,” “research the problem,” or “finish everything.” Prefer formulations such as “Implement X while preserving Y; verify with Z.”

## Phase 3: Present the approval proposal

Use this format:

```markdown
## Goal 执行草案（尚未写入）

任务总纲：
<mission>

具体目标：
1. <goal with completion evidence>
2. <goal with completion evidence>
3. <goal with completion evidence>

Autopilot：最多 <N> 轮

请回复“确认执行”开始设置；也可以指出要修改的总纲、目标、顺序或轮数。
```

After presenting it, wait. Do not continue into configuration in the same turn unless the user's current message already contains an unambiguous approval of that exact displayed proposal, which normally cannot happen on its first display.

Interpret responses as follows:

- Clear approval such as “确认执行” or “按这个开始”：continue to Phase 4.
- Requested change, including “确认，但是……”：revise and display the full proposal again, then wait for fresh approval.
- Question or uncertainty: answer it and keep the proposal uncommitted.
- Cancellation: acknowledge it and leave Goal state unchanged.

## Phase 4: Recheck and configure

The review may take time, so protect against state changes before writing:

1. Call `goal_list({})` again.
2. If an unfinished goal now exists, stop without writing the proposal and report the conflict.
3. Call `goal_set_mission({ text: <approved mission> })` exactly once.
4. Call `goal_add({ text: <approved goal> })` once per goal, in approved order. Preserve each goal's acceptance condition in its text.
5. Call `goal_list({})` and verify:
   - the mission matches the approved mission;
   - every approved goal exists exactly once and is unfinished;
   - their order matches the proposal.
6. Only after successful verification, call `goal_auto_on({ maxRounds: <approved positive integer> })`.

Always pass `maxRounds` explicitly. For the default, the exact call is:

```text
goal_auto_on({ maxRounds: 10 })
```

An omitted, zero, invalid, or negative value can mean unlimited execution in the extension and must not be used.

## Failure handling

If setting the mission, adding any goal, or final verification fails:

1. Do not call `goal_auto_on`.
2. Call `goal_list({})` when possible to capture the actual state.
3. Report which approved items were written and which failed or differ.
4. Give exact recovery commands such as `/goal rm <编号>` or ask the user to inspect with `/goal`.
5. Wait for user direction; do not retry mutations automatically.

After a successful start, report the mission, created goal numbers, and maximum rounds. Autopilot itself is responsible for execution and for calling `goal_complete` only when each goal is genuinely achieved.

## Example

User task:

```text
帮我修复登录超时问题，拆好目标让我确认，然后自动跑。
```

Suitable proposal:

```markdown
## Goal 执行草案（尚未写入）

任务总纲：
定位并修复登录流程的超时问题，保持现有登录接口兼容，并用自动化测试证明正常路径和超时路径均符合预期。

具体目标：
1. 复现登录超时并记录可重复的触发条件；以失败日志或测试作为证据。
2. 定位根因并实现最小修复；不改变现有登录接口契约。
3. 补充覆盖正常登录和超时处理的自动化测试；相关测试全部通过。
4. 运行完整验证并检查变更范围；无新增失败且没有无关改动。

Autopilot：最多 10 轮

请回复“确认执行”开始设置；也可以指出要修改的总纲、目标、顺序或轮数。
```
