# StellarAdmin agent skills

[Claude Code](https://claude.com/claude-code) skills that teach an AI coding agent how to build UIs with [StellarAdmin](https://www.stellaradmin.com) — the component catalog, the library's conventions, and task workflows for forms, layout, and theming.

With the skills installed, an agent knows the `<sa-*>` tag names, their attributes and enum values, and the composition rules, so it writes correct markup instead of guessing.

This repository doubles as a Claude Code **plugin marketplace**.

## Installation

From within Claude Code:

```text
/plugin marketplace add stellar-admin/skills
/plugin install stellar-admin@stellar-admin
/reload-skills
```

## What's included

The `stellar-admin` plugin covers the free, open-source [`StellarAdmin.TagHelpers`](https://www.nuget.org/packages/StellarAdmin.TagHelpers) package:

| Skill | What it does |
|-------|--------------|
| `stellar-admin:tag-helpers` | The anchor skill. Setup, the full component catalog, and the library's conventions. **Auto-activates** when you edit `.cshtml` or `.razor` files. |
| `stellar-admin:forms` | Fields, validation, model binding, and the input family. |
| `stellar-admin:layout` | Page shells — app header, page header, page container, sidebar dashboards, cards. |
| `stellar-admin:theming` | Theme stylesheets, dark mode, design tokens, and menu appearance. |

A `stellar-admin-pro` plugin covering the paid tier will ship from this same marketplace.

## Layout

```
.claude-plugin/marketplace.json     the marketplace manifest
plugins/stellar-admin/              the OSS plugin
    .claude-plugin/plugin.json
    skills/
        tag-helpers/                anchor skill + references/
        forms/  layout/  theming/   task skills
plugins/stellar-admin-pro/          the paid-tier plugin (skills to come)
```

## Regenerating the component reference

Everything under `plugins/stellar-admin/skills/tag-helpers/references/components/` and `components-index.md` is **generated** — do not hand-edit it. The source of truth is the tag helper C# in the `stellar-admin` repo (`[HtmlTargetElement]` attributes, bound properties, XML doc comments, enum members) plus curated example snippets from the DocsSamples app.

The generator lives in the [workspace](https://github.com/stellar-admin/workspace) repo and expects `stellar-admin` and `skills` checked out beside it:

```bash
cd ~/projects/stellar-admin/workspace
dotnet run --project src/SkillsGenerator             # regenerate
dotnet run --project src/SkillsGenerator -- --check  # fail on drift
```

Generated files carry `generated: true` in their frontmatter. One escape hatch exists: a region delimited by `<!-- structure:begin -->` / `<!-- structure:end -->` in a component file is hand-authored and preserved verbatim across regeneration (used for the required-structure trees on composite components such as Sidebar and Sheet).

Every other file in this repository is hand-written and kept in sync with the documentation at <https://www.stellaradmin.com/docs/tag-helpers>.

## License

MIT — see [LICENSE](LICENSE).
