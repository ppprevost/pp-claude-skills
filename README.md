# pp-claude-skills

Claude Code skills marketplace, packaged the same way as [obra/superpowers](https://github.com/obra/superpowers): a single plugin at repo root (`.claude-plugin/marketplace.json` + `.claude-plugin/plugin.json`), skills under `skills/<name>/SKILL.md`.

## Install

```
/plugin marketplace add ppprevost/pp-claude-skills
/plugin install pp-claude-skills@pp-claude-skills
```

## Skills

| Skill | Origin |
|---|---|
| `pp-backend-developer` | Original — NestJS/Next.js/Astro backend stack opinions |
| `pp-frontend-developer` | Original — React/Next.js frontend stack opinions, duplication detection |
| `mastering-typescript` | Fork of [spillwavesolutions/mastering-typescript-skill](https://github.com/spillwavesolutions/mastering-typescript-skill), locally tuned |
| `clean-ddd-hexagonal` | Fork of [ccheney/robust-skills](https://github.com/ccheney/robust-skills), locally tuned |
| `requesting-code-review` | Fork of [obra/superpowers](https://github.com/obra/superpowers), locally tuned |

Each forked skill carries a "Fork notice" at the top of its `SKILL.md` pointing back to the upstream source. Check the upstream repo's license before redistributing those three.
