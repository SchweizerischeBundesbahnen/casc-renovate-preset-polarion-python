# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains a **Renovate bot preset configuration** for Python projects at SBB (Schweizerische Bundesbahnen).

**Version Compatibility:** Renovate v41+ (tested with v42.17.0 as of November 2025)

It is consumed by other repositories via:

```json
{
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-python"]
}
```

The preset is **not a Python codebase** - it's a Renovate configuration file published via GitHub.

## Architecture

### Core Files

**`default.json`** - The main Renovate preset configuration
- This is the **single source of truth** consumed by downstream repositories
- Published directly from the `main` branch (no build/release process)
- Configuration structure:
  - `enabledManagers`: Explicitly restricts to Python ecosystem managers only
  - `extends`: Base presets (`config:best-practices`, `:semanticCommits`)
  - `packageRules`: Automerge rules and manager-specific configuration
  - Migration strategy: Poetry → uv (pep621 manager)

**`renovate.json`** - Self-testing configuration
- Tests this preset against itself (dogfooding)
- Uses `"ignoreTests": true` to allow automerge without CI

**`TODO.md`** - Session work tracking (optional, not committed by default)
- Contains current session changes and notes
- Can be used to track work in progress

### Configuration Philosophy

**Python-Specific Preset:**
- `enabledManagers` restricts scanning to: `pep621` (uv), `poetry`, `pre-commit`, `github-actions`, `custom.regex`
- This prevents accidental detection of npm, docker, terraform, etc. in Python repos
- Explicit intent: "This is a Python-only preset"
- Includes custom regex support (`custom.regex`) for flexibility

**Poetry → uv Migration:**
- Poetry manager: Marked DEPRECATED with clear comment in `default.json:37-45`
- Remove Poetry from `enabledManagers` after migration complete
- uv support: Via `pep621` manager (auto-detected by `uv.lock` presence)

**Automerge Strategy:**
- `automergeType: "branch"` - Silent automerge (PR only created if tests fail)
- `platformAutomerge: true` - Uses GitHub native auto-merge
- `branchConcurrentLimit: 3` - Maximum 3 concurrent update branches
- `minimumReleaseAge: "3 days"` - **Global 3-day stabilization** for all automerged updates (no exceptions)
  - Applies to: pre-commit, github-actions, non-major updates (minor/patch/pin/digest)
  - Rationale: Community has time to find issues before automerge
  - Can be overridden per packageRule if needed (e.g., internal SBB packages)
- Pre-commit: `ignoreTests: true` - automerges even if CI fails
- GitHub Actions: Requires CI to pass before automerge
- Major updates: Require manual review (`automerge: false`)
- `lockFileMaintenance`: Enabled, Monday 4am schedule for transitive dependencies

**Security & Constraints:**
- `vulnerabilityAlerts`: Enabled with `security` label, manual review required
- Security vulnerabilities: Subject to 3-day stabilization like all other updates
- `constraints.python: ">=3.12"` - Enforces minimum Python version compatibility

## Development Commands

### Validation

**JSON syntax validation:**
```bash
jq empty default.json
```
- Validates JSON syntax (required)
- Fast and reliable
- No external dependencies

**Renovate config validation:**
```bash
# Schema validation via IDE/editor
# VSCode/IDEs validate against "$schema" automatically
# Check for squiggly lines/diagnostics in editor

# CLI validation requires GitHub token (not practical for local use)
npx --yes renovate validate-config default.json
# Note: Requires GITHUB_TOKEN environment variable
# Not recommended for routine validation
```
- **Primary method**: Use IDE/editor with JSON schema support
  - Schema: `https://docs.renovatebot.com/renovate-schema.json`
  - VSCode, IntelliJ, etc. provide real-time validation
- **CLI method**: Requires GitHub authentication, not suitable for local development
- **Alternative**: Test changes in a downstream repo with actual Renovate bot

**Pre-commit checks (MANDATORY before commit):**
```bash
pre-commit run --all-files
```
- Includes `check-json` hook for syntax validation
- Runs all quality checks (formatting, secrets detection, etc.)

### Pre-commit Hooks

Configuration: `.pre-commit-config.yaml`

**Hooks run automatically on commit:**
- `yamlfix` - YAML formatting (config: `.yamlfix.toml`)
- `check-merge-conflict` - Detects merge conflict markers
- `trailing-whitespace`, `mixed-line-ending` - Whitespace cleanup
- `check-json`, `check-toml`, `check-yaml` - Syntax validation
- `gitleaks` - Secrets detection
- `check-git-config-user-email` - Enforces `@sbb.ch` email domain
- `commitizen` - Conventional commit message validation (commit-msg hook)

**CRITICAL:** Never skip pre-commit (`--no-verify`) without user approval.

## Configuration Guidelines

### Renovate Preset Best Practices

1. **enabledManagers behavior:**
   - When present: Disables ALL managers except those listed (5 enabled: pep621, poetry, pre-commit, github-actions, custom.regex)
   - When absent: All 70+ managers enabled with auto-detection
   - **Keep it for this preset** - documents Python-only intent
   - See ARCHITECTURE.md for complete comparison table

2. **Adding new managers:**
   - Standard managers: Add to `enabledManagers` array (e.g., `"pep621"`, `"pre-commit"`)
   - Custom regex: Already enabled via `"custom.regex"` - define patterns in `customManagers`
   - Note: Custom managers require `custom.` prefix (e.g., `"custom.regex"`, not `"regex"`)

3. **Deprecating features:**
   - Add `"description": "DEPRECATED: ..."` to packageRules
   - Include removal timeline/conditions
   - Keep in config during transition period

4. **Testing changes:**
   - Validate JSON syntax
   - Run `renovate-config-validator`
   - Changes take effect immediately (main branch is live)

## Important Constraints

### Email Domain Enforcement
- Git commits MUST use `@sbb.ch` email addresses
- Enforced by pre-commit hook: `check-git-config-user-email`
- Configure: `git config user.email "name@sbb.ch"`

### Deployment Model
- **No CI/CD pipeline** - main branch is directly consumed
- Breaking changes affect all downstream repos immediately
- Test thoroughly before pushing to main

### Semantic Commits
- Conventional commits enforced by commitizen hook
- Format: `type(scope): description`
- Types: `feat`, `fix`, `chore`, `docs`, etc.
- Examples:
  - `chore: remove deprecated Poetry configuration`
  - `feat: add support for uv lock file maintenance`

## Internal Documentation

**[ARCHITECTURE.md](ARCHITECTURE.md)** - Complete technical documentation
- Configuration analysis and design decisions
- enabledManagers behavioral specifications
- Package manager support matrix
- Automerge strategy deep dive
- Migration timeline (Poetry → uv)
- All external documentation references

**[README.md](README.md)** - User-facing documentation
- How to consume this preset
- Feature overview
- Migration guide for Poetry users

**[TODO.md](TODO.md)** - Session work tracking (optional, not committed by default)
- Current session changes and notes
- Work-in-progress documentation

## Key Documentation Resources

**Renovate Documentation:**
- Config best practices: https://docs.renovatebot.com/presets-config/#configbest-practices
- pep621 manager (uv): https://docs.renovatebot.com/modules/manager/pep621/
- enabledManagers: https://docs.renovatebot.com/configuration-options/#enabledmanagers

**uv Integration:**
- Official uv guide: https://docs.astral.sh/uv/guides/integration/dependency-bots/
- Renovate detects uv via `uv.lock` presence
- Lock file maintenance keeps transitive dependencies current

**For detailed technical analysis:** See [ARCHITECTURE.md](ARCHITECTURE.md)
