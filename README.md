# eng-review

A Claude Code plugin that adds `/eng-review` — a thorough engineering best-practices code review covering 20 categories: input validation, auth/IDOR, SQL performance, N+1 queries, pagination, rate limiting, password storage, race conditions, file uploads, production architecture, monitoring, scale readiness, and more.

## Prerequisites

- [Claude Code](https://claude.com/claude-code) installed and signed in
- Network access to github.com (the repo is public — no auth required)

## Install (run inside Claude Code)

Three commands, in order:

```
/plugin marketplace add Jobairshi/eng-review-plugin
/plugin install eng-review@jobairshi-marketplace
/reload-plugins
```

That's it. The plugin is now available on this device.

> After `/plugin install` you'll see `✓ Installed eng-review. Run /reload-plugins to apply.` — that's why step 3 is required. (Quitting and reopening Claude Code also works.)

## Use

```
/eng-review
/eng-review src/auth/
/eng-review the changes on this branch
```

The command reviews the project (or the files/scope you specify) against the embedded checklist and reports each issue with:

1. The rule violated
2. File and line number
3. Why it matters (real-world consequence)
4. The bad code
5. The corrected version

It ends with a summary of total issues, severity breakdown, and the top 3 priorities to fix first.

## Updating to the latest version

When the plugin is updated in this repo, pull the changes on each device with:

```
/plugin marketplace update jobairshi-marketplace
/reload-plugins
```

## Uninstall

```
/plugin uninstall eng-review@jobairshi-marketplace
/plugin marketplace remove jobairshi-marketplace
```

## Troubleshooting

**`/eng-review` not found after install**
You skipped `/reload-plugins`. Run it, or restart Claude Code.

**An old personal version is shadowing the plugin**
If `/eng-review` was previously installed as a personal slash command, the file at `~/.claude/commands/eng-review.md` will override the plugin. Remove it:

```
rm ~/.claude/commands/eng-review.md
```

**Want to verify the plugin is installed**

```
ls ~/.claude/plugins/cache/jobairshi-marketplace/eng-review/
```

## Repo layout

```
.
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest
│   └── marketplace.json     # marketplace catalog (same repo serves both)
├── commands/
│   └── eng-review.md        # the slash command
└── README.md
```
