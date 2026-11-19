# The canonical install block

One install story, one wording. `README.md`, `.changeset/*`, and every page under `docs/` must say **this** and nothing else. Change it here first, then propagate.

## Managed installs first

Lead with installs that update themselves. Fall back to skills.sh, labelled "manual updates", only for agents that have no such install. Call a route managed only if it updates without the user running a command. Put any one-time opt-in beside its install command. Never promise "instant".

Every managed route for this personal fork installs from this repo's own marketplace, `.claude-plugin/marketplace.json` and `plugin.json`. Those routes reinstall only when the `version` in `plugin.json` changes. The Release workflow runs `npm run version`, which copies that field from `package.json`, so users get a release when the "chore: version skills" PR merges.

| Agent         | Route                            | Updates itself?                                                  |
| ------------- | -------------------------------- | ---------------------------------------------------------------- |
| Claude Code   | `@hung` marketplace       | Yes, by default                                                  |
| Codex         | `@hung` marketplace        | Yes, at startup, by default                                      |
| Copilot CLI   | `@hung` marketplace        | Yes, after a one-time `autoUpdate: true`                         |
| VS Code       | Chat: Install Plugin From Source | Yes, daily, by default                                           |
| Gemini CLI    | `gemini skills install` (copies) | No. A Gemini extension would need a flat `skills/<name>/` layout |
| Everyone else | skills.sh                        | No. Run `npx skills update`, and re-run `add` for new skills     |

Sources (2026-10-08): Codex `core-plugins/src/manager.rs`; Copilot [cli-config-dir-reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference); VS Code `pluginAutoUpdate.ts`; Gemini `skillLoader.ts`.

Nobody has run Auggie, Droid, Qwen or Goose against this repo, so they stay on skills.sh until someone does.

## Claude Code

<canonical-block name="claude-code">

```bash
claude plugin marketplace add tqhung04/skills
claude plugin install hung-skills@hung
```

Updates itself by default.

</canonical-block>

## Codex

<canonical-block name="codex">

```bash
codex plugin marketplace add tqhung04/skills
codex plugin add hung-skills@hung
```

Updates itself at startup.

</canonical-block>

## GitHub Copilot (CLI and VS Code)

Use the marketplace, not `copilot plugin install tqhung04/skills`. Copilot has deprecated direct repo installs, and they can't auto-update. The `extraKnownMarketplaces` entry follows the documented schema, but nobody has run it yet.

<canonical-block name="copilot">

```bash
copilot plugin marketplace add tqhung04/skills
copilot plugin install hung-skills@hung
```

Then, once, add to `~/.copilot/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "hung": { "source": { "source": "github", "repo": "tqhung04/skills" }, "autoUpdate": true }
  }
}
```

In VS Code, run **Chat: Install Plugin From Source** and enter `https://github.com/tqhung04/skills`. It updates daily.

</canonical-block>

## Gemini CLI

Gemini doesn't read `.claude-plugin`, and `--path` takes one bucket. One install per promoted bucket gives exactly the promoted set (27 skills, verified 2026-10-07).

<canonical-block name="gemini">

```bash
gemini skills install https://github.com/tqhung04/skills.git --path skills/engineering
gemini skills install https://github.com/tqhung04/skills.git --path skills/productivity
```

Re-run both commands to update.

</canonical-block>

## Every other agent, or editable files (skills.sh)

`-a` preselects the agent. Each agent in the comment was run on 2026-10-07. Prefer skills.sh to an agent's own repo installer, such as `amp skill add` or `pi install git:…`, because those also pull in `misc/` and `in-progress/`. `npx skills update` skips skills added upstream since `add`, so the block tells users to re-run `add`.

<canonical-block name="other-agents">

```bash
npx skills@latest add tqhung04/skills -a <agent>  # cursor, opencode, devin, windsurf, amp, pi; omit -a to choose
```

When the installer asks which skills to take, include `setup-hung-skills`. To update, run `npx skills@latest update`, and re-run `add` to pick up new skills.

</canonical-block>

Use the single-skill form wherever one skill is named on its own. `docs/` pages don't use this block, because ai-hero renders the install widget above the page body. See [writing-docs.md](./writing-docs.md).

<canonical-block name="skills-sh-one-skill">

```bash
npx skills@latest add tqhung04/skills --skill=<name>
```

```bash
npx skills@latest update <name>
```

</canonical-block>

`skills@latest` is the pinned spelling everywhere.

## The two routes are exclusive

The plugin is a managed, read-only bundle you subscribe to. skills.sh writes files you own and edit. Installing both leaves the user with every skill twice: always say "pick one".

## Local Codex development

<canonical-block name="codex-local-development">

### Codex: develop from this checkout

To load unpublished changes from this machine, use the local marketplace instead
of the GitHub marketplace. Pick one source for the `hung` marketplace.

```bash
codex plugin marketplace add /Users/hung/skills
codex plugin add hung-skills@hung
```

After editing skills or manifests, reinstall the plugin to replace its cached
copy, then start a new Codex chat:

```bash
codex plugin remove hung-skills@hung
codex plugin add hung-skills@hung
```

When adding a promoted skill, also add its path to the `skills` array in
`.claude-plugin/plugin.json`, update the bucket and root READMEs, create its docs
page, and update `ask-hung` as required by `AGENTS.md`. Skills in `in-progress/`
are excluded from the plugin.

For a GitHub marketplace install, commit and push first, run
`codex plugin marketplace upgrade hung`, then reinstall with the commands above.
For versioned releases, keep `package.json` and the plugin manifest versions in
sync using `scripts/sync-plugin-version.mjs`.

</canonical-block>
