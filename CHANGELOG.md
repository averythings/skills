# Changelog

## Unreleased

### Added
- `skills/ontology` — ontology skill installed from ClawHub
  (`@oswalpalash/ontology` v1.0.4, MIT-0). Provides a typed knowledge graph
  for structured agent memory: entity CRUD, relations, and constraint
  validation via `python3 scripts/ontology.py`.
- Registered `./skills/ontology` in `.claude-plugin/marketplace.json`
  alongside the existing VoltAgent skills.
- Optional starter `skills/ontology/memory/ontology/schema.yaml` covering the
  common types from `SKILL.md` (Person, Project, Task, Event, Credential)
  and a few relations (`has_owner`, `has_task`, `blocks`). The ontology skill
  auto-creates `memory/ontology/graph.jsonl` on first write; this schema is
  optional.

### Runtime notes
- The ontology skill requires Python 3 (standard library only; no
  `requirements.txt` ships with it). No install step is needed where Python 3
  is already available.
