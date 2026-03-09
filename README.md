# casc-renovate-preset-polarion-python

Renovate bot preset configuration for Python projects at SBB (Schweizerische Bundesbahnen).

**Compatible with:** Renovate v41+ (tested with v42.17.0)

## Features

### Python Ecosystem Support
- ✅ **uv** - Modern Python package manager (via pep621 manager)
- ✅ **Poetry** - Legacy support (deprecated, will be removed after migration)
- ✅ **Pre-commit** - Automated hook updates
- ✅ **GitHub Actions** - Workflow dependency updates

### Automerge Strategy
- 🔀 **Branch-based automerge** - Silent merges when tests pass
- 🤖 **Platform automerge** - Uses GitHub's native auto-merge feature
- ⏱️ **3-day stabilization** - All updates wait 3 days after release before automerge (strictly enforced)
- 🔢 **Concurrent limit** - Maximum 3 update branches to avoid overwhelming CI
- 📊 **Dependency Dashboard** - Monitor pending updates waiting for stabilization
- 📦 **Lock file maintenance** - Monday 4am transitive dependency updates
- 🔒 **Safe boundaries** - Manual review required for major updates and security alerts
- 🔐 **Security alerts** - Manual review with `security` label
- 🐍 **Python >=3.12** - Enforces minimum Python version compatibility
- ⚡ **Auto-updates (after 3 days):**
  - Pre-commit hooks (automerges even if tests fail)
  - GitHub Actions (requires CI to pass)
  - Minor/patch updates (requires CI to pass)

## How to Use

Add this preset to your `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-python"
  ]
}
```

## What This Preset Does

### Enabled Package Managers

Only Python ecosystem and CI/CD managers are enabled:
- `pep621` - PEP 621 standard (uv, pyproject.toml, uv.lock)
- `poetry` - Poetry (deprecated, transitional)
- `pre-commit` - Pre-commit framework hooks
- `github-actions` - GitHub Actions workflows
- `custom.regex` - Custom regex dependency extraction (flexible for consumers)

**All other managers are disabled** (npm, docker, terraform, etc.)

#### Why Restrict Managers?

| Aspect | Without Restriction | **With Restriction (This Preset)** |
|--------|-------------------|-----------------------------------|
| Python dependencies | ✅ Detected | ✅ Detected |
| npm/Node.js | ✅ Detected if present | ❌ Disabled |
| Docker | ✅ Detected if present | ❌ Disabled |
| Terraform | ✅ Detected if present | ❌ Disabled |
| Intent | Generic preset | **Python-specific preset** |

This restriction ensures predictable, Python-only behavior across all repositories using this preset.

### Automerge Rules

| Update Type | Automerged? | Stabilization Period | Special Handling |
|-------------|-------------|---------------------|------------------|
| Security vulnerabilities | ❌ No (manual review) | 0 days | `security` label, `prPriority: 99` to jump rate limit queue |
| Pre-commit hooks (any incl. major) | ✅ Yes | 3 days | **ignoreTests: true** - automerges even if CI fails |
| GitHub Actions (any incl. major) | ✅ Yes | 3 days | Requires CI to pass |
| Minor/patch updates | ✅ Yes | 3 days | Requires CI to pass |
| Major updates | ❌ No (manual review) | N/A | Breaking changes possible |

**Stabilization Period:**
- All updates wait 3 days after release before automerge (no exceptions)
- Allows community time to discover issues before automerge
- Can be overridden per-repository if needed
- **Concurrent limit:** Maximum 3 update branches to avoid overwhelming CI

### Lock File Maintenance

Transitive dependencies in `uv.lock` or `poetry.lock` are updated automatically:
- **Schedule:** Monday mornings at 4am
- **Automerge:** Enabled (respects 3-day stabilization)
- **Purpose:** Keep indirect dependencies current with security patches

## Migration Guide: Poetry → uv

### Current Behavior

This preset supports **both Poetry and uv** during the transition period.

**If your project uses Poetry:**
- Poetry dependencies continue to work
- `poetry.lock` is automatically updated
- ⚠️ **Note:** Poetry support is deprecated and will be removed in the future

**If your project uses uv:**
- uv is auto-detected via `uv.lock` file
- Both `pyproject.toml` and `uv.lock` are updated
- Lock file maintenance keeps transitive dependencies current

### Migrating from Poetry to uv

1. **Migrate your project to uv** (see [uv documentation](https://docs.astral.sh/uv/))
2. **No changes needed to Renovate config** - uv is automatically detected
3. Remove `poetry.lock` from your repository
4. Renovate will detect `uv.lock` and use the pep621 manager

## Developer Setup (This Repository)

### Prerequisites
- Text editor
- Git configured with `@sbb.ch` email address

### Testing Changes

**Validate configuration:**
```bash
# JSON syntax validation (required)
jq empty default.json

# Schema validation via IDE (recommended)
# VSCode/IDEs validate against $schema automatically

# CLI validation (requires GitHub token, not recommended for local use)
npx --yes renovate validate-config default.json
```

**Run pre-commit hooks (MANDATORY before commit):**
```bash
pre-commit run --all-files
```

## Deployment

Changes to `default.json` are **live immediately** when merged to `main` branch.

⚠️ **No CI/CD pipeline** - all consuming repositories will receive updates directly.

## Documentation

- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Technical analysis and design decisions
- **[CLAUDE.md](CLAUDE.md)** - AI assistant development guidance

## Support

For issues or questions about this preset:
- Open an issue in this repository
- Contact the SBB Polarion Python team