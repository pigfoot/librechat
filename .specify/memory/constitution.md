<!--
Sync Impact Report:
Version change: Template → 1.0.0 (Initial ratification)
Modified principles: N/A (Initial creation)
Added sections:
  - Core Principles (5 principles)
  - Fork Management Guidelines
  - Development Philosophy
  - Governance
Templates requiring updates: ✅ All templates aligned with Brownfield-first approach
Follow-up TODOs: None
-->

# LibreChat Fork Constitution

## Core Principles

### I. Brownfield-First Development

Every change MUST work with and modify existing behavior rather than creating new
systems from scratch. When updates span multiple specifications, prioritize
integration over isolation. This approach minimizes maintenance burden when
rebasing against upstream changes.

**Rationale**: As a fork of a rapidly evolving upstream project, minimizing
divergence through modification of existing systems (1→n) rather than new
implementations (0→n) ensures sustainable maintenance.

### II. Minimal Patch Philosophy

All modifications MUST be consolidated into a single, atomic patch that can be
easily rebased. Use squash rebasing to maintain ONE clean patch against the
upstream main branch. This ensures clear visibility of divergence and simplified
conflict resolution.

**Rationale**: Upstream updates frequently. A single patch reduces cognitive
load during rebases and makes it trivial to see exactly what has been changed
from the original LibreChat implementation.

### III. Simplicity Over Perfection

Proof of concept takes absolute priority over performance optimization and
comprehensive testing. Features MUST work first, optimize later (if ever).
Avoid premature optimization and complex abstractions unless absolutely necessary
for functionality.

**Rationale**: As an experimental fork, the goal is to validate ideas quickly
rather than build production-ready systems. Working code that demonstrates
concepts has more value than perfect code that takes longer to implement.

### IV. Upstream Compatibility

Changes MUST NOT break core LibreChat functionality unless that breakage is
the explicit purpose of the fork. Maintain API compatibility where possible.
Document any intentional divergences clearly in commit messages.

**Rationale**: Users switching between upstream and this fork should experience
predictable behavior. Unintentional breakages create confusion and reduce the
fork's utility as a testing ground for new ideas.

### V. Manual Rebase Workflow

The rebase process as defined in `run.sh` is the canonical workflow:
1. Fetch upstream tags
2. Rebase when divergence exceeds one commit
3. Force push after successful rebase
4. Rebuild containers after workflow completion

This workflow MUST NOT be automated beyond the existing script.

**Rationale**: Manual oversight of rebases ensures conscious decisions about
conflict resolution and prevents automated processes from introducing subtle bugs.

## Fork Management Guidelines

### Patch Maintenance

- **Single Commit Rule**: All fork changes exist in ONE commit
- **Squash Strategy**: New changes are squashed into the existing patch
- **Rebase Trigger**: Only rebase when `git rev-list HEAD...upstream/main --count` != 1
- **Force Push Policy**: Acceptable and expected after rebases

### Upstream Tracking

- **Branch**: Always rebase against `upstream/main`
- **Frequency**: Check upstream changes via `run.sh` regularly
- **Conflicts**: Resolve in favor of maintaining fork functionality over upstream patterns

## Development Philosophy

### When to Modify vs. When to Extend

**Modify existing code when:**
- The change enhances or alters existing LibreChat features
- Integration with existing systems is required
- The modification can be contained within existing file structures

**Extend with new files only when:**
- The feature is completely orthogonal to existing functionality
- Upstream is unlikely to implement similar functionality
- The extension can be cleanly separated from core code

### Documentation Standards

- Changes need minimal documentation - the code should be self-evident
- Complex modifications require inline comments explaining divergence
- Maintain a single PATCH_NOTES.md if changes become numerous

## Governance

### Amendment Process

1. Propose changes via issue or direct modification
2. Test amendments against current patch
3. Update version following semantic versioning
4. Squash amendment into the main patch

### Version Policy

- MAJOR: Fundamental shift in fork philosophy
- MINOR: New principles or significant workflow changes
- PATCH: Clarifications and minor adjustments

### Compliance

- All changes MUST align with these principles
- Violations should trigger patch refactoring, not principle changes
- Use global instructions in `~/.claude/CLAUDE.md` for runtime development guidance

**Version**: 1.0.0 | **Ratified**: 2025-10-22 | **Last Amended**: 2025-10-22