# TODO

## Before Next Build

- [ ] Rename Go module path: `github.com/julb/blueprint-monitor` → `github.com/JuliusBrussee/blueprint-monitor` (go.mod + all .go files + blueprint-cli.md R1)
- [ ] Rename frontier → site across all blueprint specs and Go code (align with commit 4ffa421 rebrand)
  - `blueprint-frontier.md` → `blueprint-site.md`
  - `context/frontiers/` → `context/sites/` references
  - Go package names, struct fields, function names

## Spec Gaps

- [ ] Add error/crash recovery UX (no `Error` or `Crashed` status in session model, no TUI recovery flow)
- [ ] Add attention/notification mechanism (visual signal when an agent needs input while viewing another)
- [ ] Add "send command to instance" flow in TUI (currently only full-screen attach or auto-yes)
- [ ] Define instance list ordering (creation time? alphabetical? status?)
- [ ] Add merge conflict visibility (indicator when branches will conflict)
- [ ] Clarify Terminal tab value vs Preview tab (both show captured pane content)
- [ ] Add WSL requirement note to CLI spec R1 for Windows users
