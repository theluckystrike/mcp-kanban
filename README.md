# mcp-kanban

<!-- mirror-seo:start -->

**MCP server for a kanban task board and project board.** A task board per project that hands off to the time tracker.

Works with Claude Desktop, Claude Code, Cursor and any Model Context Protocol client. Runs on your own machine, or hosted with no install.

## Install

**Hosted, nothing to install.** Get a token from <https://mcp.zovo.one/mcp/connect> (the connect page) or <https://mcp.zovo.one/mcp/token> (the same token as JSON); a free anonymous one is issued on the spot and a Pro key works the same way. Then point an MCP client at `https://mcp.zovo.one/mcp/kanban` over streamable-http and send the token as `Authorization: Bearer <token>`.

If your client cannot set headers, put the token in the path instead: `https://mcp.zovo.one/mcp/kanban/t/<token>`. Both forms work. The bare URL with no token answers 401 on `tools/call`, so the token is not optional.

**Claude Desktop, one click.** Download `kanban.mcpb` from the [latest release](https://github.com/theluckystrike/mcp-servers/releases/latest) and double-click it.

**From source.** The mirror is self-contained: every `@theluckystrike/*` dependency is vendored, so a fresh clone builds with no extra setup.

```sh
git clone https://github.com/theluckystrike/mcp-kanban.git
cd mcp-kanban
npm install && npm run build
```

Then point your client at the built entry point:

```json
{
  "mcpServers": {
    "kanban": {
      "command": "node",
      "args": ["/absolute/path/to/mcp-kanban/dist/index.js"]
    }
  }
}
```

> `@theluckystrike/mcp-kanban` is **not published on npm yet**, so an `npx -y @theluckystrike/mcp-kanban` command will fail. The three paths above are the working ones and each is exercised by CI.

![kanban demo](https://raw.githubusercontent.com/theluckystrike/mcp-servers/main/assets/demo-kanban.gif)

Read-only mirror of [mcp-servers/servers/kanban](https://github.com/theluckystrike/mcp-servers/tree/main/servers/kanban). See [MIRROR.md](MIRROR.md).

<!-- mirror-seo:end -->

A task board for each of your projects, driven from your AI chat. Say "add a task to the nova site board: write the
launch email, due Friday, 90 minutes", then ask "what's on the nova board?", "what's overdue?" or "plan my week".
Tasks carry a title, notes, tags, a due date, an estimate, a priority and a column (backlog, todo, doing, review,
done by default), each one gets a short stable id like `NS-12`, and a task can hand its exact arguments to the
time-tracker server so the time you spend on it lands against the same project name. Everything is stored as plain
JSON on your own machine.

Built by [theluckystrike](https://github.com/theluckystrike).


**A local kanban board per project -- columns, due dates, estimates, weekly review -- zero setup, all local.**

## 60-second install

npm publish for `@theluckystrike/mcp-kanban` is pending. Until then, the `.mcpb` one-click bundle or a clone+build
is the working path -- both are verified below.

**One-click (.mcpb):** download `kanban.mcpb` from the latest release and double-click it in Claude Desktop:
https://github.com/theluckystrike/mcp-servers/releases/latest

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "kanban": {
      "command": "npx",
      "args": ["-y", "@theluckystrike/mcp-kanban"]
    }
  }
}
```

**Claude Code:**

```sh
claude mcp add kanban -- npx -y @theluckystrike/mcp-kanban
```

**Cursor** (`.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "kanban": {
      "command": "npx",
      "args": ["-y", "@theluckystrike/mcp-kanban"]
    }
  }
}
```

The `npx` form above starts working the moment the package is published. Until then, use the .mcpb bundle above, or
build from source with exactly these three commands:

```sh
git clone https://github.com/theluckystrike/mcp-servers.git && cd mcp-servers
npm install
npm run build -w packages/mcp-license -w servers/kanban
```

Then point your client's `command` at `node` with one arg: the absolute path to `servers/kanban/dist/index.js`.

To run in Pro mode set `MCP_LICENSE_KEY` in the same config block, or call `license_activate` once with your key.

## Tools

| Tool | What it does |
| --- | --- |
| `task_add` | Add a task: title plus optional project, column, due date, estimate, priority, tags and notes. A new project name creates its board; a partial name that matches exactly one existing project is used as that project. |
| | Due dates accept `YYYY-MM-DD`, `today`, `tomorrow`, `yesterday`, `+3d`, a weekday (`friday`, the nearest one from today) and `next friday` (the one after that). |
| `task_list` | Table of tasks, filtered by project, column, tag, `due_before` or `overdue`. Finished tasks are hidden unless you ask for them. Prints 200 rows by default and says how many more there are; raise it with `limit` (max 2,000). |
| `task_move` | Move a task to another column on its board. |
| `task_update` | Change any field: title, notes, due (`none` clears it), estimate, priority, tags, column or project. |
| `task_done` | Mark a task done: it moves to the done column and is stamped with the time. |
| `task_delete` | Delete a task. The id is never reused. |
| | An identical `task_add` (same project, title, column, due date, estimate, priority, tags and notes as a task already open) is refused and names the task that already holds the slot, rather than writing a second copy. |
| `task_search` | Find tasks by text in the title, notes, tags or id. |
| `board` | Column-by-column summary of one board: tasks, estimate total, actual total, overdue count. |
| `task_start_timer` | Return the exact arguments to pass to the time-tracker server's `timer_start` for this task, and record the link on the task. |
| `task_log_time` | Add real minutes worked to a task, so estimate and actual can be compared. |
| `project_list` | Every board with open and done counts, remaining estimate and overdue count. |
| `project_delete` | Delete an empty board and free its project slot. A board that still holds tasks, open or done, is refused with a task named; clear them with `task_delete` first. |
| `overdue` | Everything past its due date, across all boards; `as_of` measures against another day. |
| `weekly_review` | Done versus planned for a week, with estimate against actual per project. This week is free; past weeks are Pro. |
| `columns_set` | Replace one board's columns (Pro). Tasks in a removed column move to the first column. |
| `license_status` | Free or Pro, and where to upgrade. |
| `license_activate` | Activate a Pro key (verified offline). |

Also exposed: the resource `kanban://today` (due today plus everything overdue) and the prompt `plan_week`
(turns the open board into a day-by-day plan that fits the hours you actually have).

## What you can say

- "Add a task to the nova site board: write the launch email, due Friday, 90 minutes, high priority."
- "Move NS-3 to doing."
- "What does the nova board look like?"
- "What's overdue?"
- "Start a timer on NS-3." -- then pass the arguments it returns to the time-tracker server.
- "I spent 40 minutes on NS-3."
- "Weekly review."

## Free vs Pro

| | Free | Pro |
| --- | --- | --- |
| Project boards | 3 | unlimited |
| Open tasks | 200 | unlimited |
| Columns | the default five (backlog, todo, doing, review, done) | custom columns per board (`columns_set`) |
| Weekly review | the current week | any week, full history |
| Estimate vs actual reports | current week only | every week |
| Listing output | 200 rows per call, `limit` up to 2,000 | same |
| Everything else | included | included |

Pro is a one-time $19 for this server, or $39 for every server, lifetime.

**Get Pro:** https://mcp.zovo.one/buy/kanban

## Pairs with

- [mcp-time-tracker](../time-tracker) -- `task_start_timer` hands it the project and task name, so hours land against the same project.
- [mcp-calendar](../calendar) -- due dates on the board next to what is actually in your week.
- [mcp-invoice](../invoice) -- turn the tracked hours behind those tasks into invoice line items.

## Privacy

All data stays local, in `${XDG_DATA_HOME:-~/.local/share}/mcp-servers/kanban/`. Nothing is uploaded, no account is
needed, and licence keys are verified offline.

## License

MIT. Support: support@zovo.one
