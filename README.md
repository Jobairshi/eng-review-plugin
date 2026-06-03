# eng-review

A Claude Code plugin that adds `/eng-review` — a thorough engineering best-practices code review covering 20 categories (security, performance, auth, scale, architecture).

## Install

```
/plugin marketplace add Jobairshi/eng-review-plugin
/plugin install eng-review@jobairshi-marketplace
```

(Replace `Jobairshi/eng-review-plugin` with your actual GitHub `owner/repo`.)

## Use

```
/eng-review
/eng-review src/auth/
/eng-review the changes on this branch
```

The command reviews the project (or the files/scope you specify) against the embedded checklist and reports each issue with the rule violated, why it matters, the bad code, and a fix.
