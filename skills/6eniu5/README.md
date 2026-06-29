# 6eniu5 (personal)

My own Claude Code skills, layered on top of the upstream `mattpocock/skills` fork.

This bucket is **new to the fork** — upstream never writes here, so it stays
conflict-free across `git merge upstream/main`. Add a skill by creating
`skills/6eniu5/<skill-name>/SKILL.md` (frontmatter `name` + `description`), then
re-run the manager repo's `scripts/install-claude-skills.sh` to symlink it into
`~/.claude/skills`. See `docs/claude-skills/adding-a-skill.md` in the esetup repo
for the full authoring loop.
