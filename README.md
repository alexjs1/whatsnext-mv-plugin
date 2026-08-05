# whatsnext-mv-plugin

A single-plugin Claude Code marketplace for **What's Next MV**, the Martha's Vineyard
visitor guide built by [Island Analytics](https://island-analytics.com).

The plugin, **`whatsnext`**, carries the skills that maintain the app: adding places and
events, refreshing Google ratings, auditing descriptions for accuracy, selling featured
listings, setting up a hotel's private-label Pocket Concierge edition, and publishing.

- **The plugin and its full skill list:** [`plugins/whatsnext/README.md`](plugins/whatsnext/README.md)
- **App repo:** [`alexjs1/whats-next-mv`](https://github.com/alexjs1/whats-next-mv)

## Install

```
/plugin marketplace add alexjs1/whatsnext-mv-plugin
/plugin install whatsnext@whatsnext-mv
```

In Cowork (desktop), enable it from Customize → Workspace.

## Updating

`plugin.json` declares no `version` on purpose, so **every commit is a new version** and
pushing is all it takes to ship a change. Pull it with `/plugin marketplace update` then
`/plugin update whatsnext`, or refresh from Customize → Workspace in Cowork.

Do not reintroduce a pinned `version` unless you intend to bump it on every release: a
pinned version that stops moving leaves everyone on a cached copy, with no sign that
anything is wrong.

## Layout

```
.claude-plugin/marketplace.json   the marketplace catalog (one entry)
plugins/whatsnext/
  .claude-plugin/plugin.json      the plugin manifest
  skills/<name>/SKILL.md          one directory per skill
  README.md                       skills, install, conventions
```
