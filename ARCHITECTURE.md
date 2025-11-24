# Architecture & Design Documentation

This document contains technical analysis, design decisions, and behavioral specifications for the Renovate preset configuration.

## Version Compatibility

**Tested with:** Renovate v41.140.1
**Compatible with:** Renovate v42+ (latest: v42.17.0 as of November 2025)

All configuration options used in this preset are stable, core features with no breaking changes between v41 and v42. The preset is forward-compatible with current Renovate releases.

## Configuration Analysis

### Base Configuration

**Presets Extended:**
- `config:best-practices` - Renovate maintainers' recommended configuration (2025 current)
- `:semanticCommits` - Enables conventional commit message format

**Core Settings:**
- `automergeType: "branch"` - Silent automerge (PR only created if tests fail)
- `platformAutomerge: true` - Uses GitHub's native auto-merge feature
- `branchConcurrentLimit: 3` - Maximum 3 concurrent update branches
- `minimumReleaseAge: "3 days"` - Global stabilization period for all automerged updates
- `prCreation: "not-pending"` - Create PRs immediately, don't wait for status
- `rebaseWhen: "behind-base-branch"` - Keep branches up to date
- `lockFileMaintenance` - Monday 4am automated lock file updates for transitive dependencies
- `vulnerabilityAlerts` - Enabled with `security` label, manual review required
- `constraints.python: ">=3.12"` - Enforces minimum Python version compatibility

## Package Manager Support Matrix

### Enabled Managers (Python Ecosystem + CI/CD + Preset Management)

**`pep621`** - PEP 621 Standard
- Primary manager for modern Python projects
- Handles: `pyproject.toml`, `uv.lock`
- Auto-detects uv by presence of `uv.lock` file
- Supports: project dependencies, optional dependencies, dev-dependencies
- Manages both uv and standard PEP 621 projects

**`poetry`** - Poetry Package Manager
- **Status: DEPRECATED** - Will be removed after uv migration
- Handles: `pyproject.toml`, `poetry.lock`
- Config includes `postUpdateOptions: ["poetry-lock"]`
- Keep during transition period, remove when migration complete

**`pre-commit`** - Pre-commit Framework
- Handles: `.pre-commit-config.yaml`
- Updates hook repositories and versions
- **Note:** Disabled by default in Renovate, requires opt-in
- Enabled via `enabledManagers` (not via `:enablePreCommit` preset - redundant)

**`github-actions`** - GitHub Actions Workflows
- Handles: `.github/workflows/*.yml`
- Updates action versions (e.g., `actions/checkout@v4`)
- Includes digest pinning via `config:best-practices`

**`renovate-config-presets`** - Renovate Preset References
- Handles: `renovate.json`, `.renovaterc`, etc.
- Updates preset references in `extends` arrays
- **Critical for this repository** - keeps `renovate.json` updated
- Only updates pinned preset versions (e.g., `github>user/preset#v1.2.3`)
- Unpinned references (e.g., `github>user/preset`) are not updated
- Enabled by default in Renovate, but disabled when `enabledManagers` is used

**`custom.regex`** - Custom Regex Managers
- Enables custom dependency extraction via regex patterns
- Requires `customManagers` configuration block with `"customType": "regex"`
- **Note:** Currently no custom managers defined in this preset
- Enabled for flexibility - consuming projects can define custom patterns
- Useful for: non-standard version files, custom dependency formats
- **Name format:** Must use `custom.` prefix in `enabledManagers`

### Explicitly Disabled Managers

All other managers are disabled when `enabledManagers` is used:

**Other Python Managers:**
- `pip_requirements` - requirements.txt files (not used in uv/poetry projects)
- `pip-compile` - pip-tools compiled requirements (not used)
- `pipenv` - Pipenv lock files (not used)
- `setup-cfg` - Legacy setup.cfg (not used)

**Non-Python Managers:**
- `npm`, `dockerfile`, `terraform`, `ansible`, etc. - All disabled (70+ managers)

## enabledManagers Behavioral Analysis

### Why Use enabledManagers?

**Design Decision: Explicit Python-Only Preset**

This preset is named `casc-renovate-preset-polarion-python` - the Python scope is intentional.

### Comparison: With vs Without enabledManagers

| Aspect | **Without enabledManagers** (Default) | **With enabledManagers** (Current) |
|--------|--------------------------------------|-----------------------------------|
| Managers active | All 70+ managers scan automatically | Only 6 specified: pep621, poetry, pre-commit, github-actions, renovate-config-presets, custom.regex |
| Python deps | ✅ Detected | ✅ Detected |
| npm/Node.js | ✅ Detected if present | ❌ Disabled |
| Docker | ✅ Detected if present | ❌ Disabled |
| Terraform | ✅ Detected if present | ❌ Disabled |
| Renovate presets | ✅ Detected | ✅ Explicitly enabled |
| Custom regex | ✅ Enabled by default | ✅ Explicitly enabled |
| Intent | Generic - detects everything | Explicit - Python-only preset |
| Warnings | None | Warns if enabled manager finds no files |

### Advantages of Using enabledManagers ✅

- **Explicit control** - only Python ecosystem + preset management
- **Prevents accidental detection** (e.g., stray `package.json` in docs/)
- **Clear intent**: "This is a Python-only preset"
- **Self-documenting code** - manager list documents scope
- **Predictable behavior** across all consuming repositories

### Disadvantages ⚠️

- Must manually add new managers if needed
- Warning messages if enabled manager finds no files

**Verdict: Keep `enabledManagers`** for explicit Python-only scope.

### Manager Enablement Rules

**When `enabledManagers` is present:**
- Renovate ONLY runs managers listed in the array
- ALL other managers are completely disabled (not even file scanning)
- No exceptions - custom managers also need explicit inclusion

**Standard Managers:**
Add to array directly: `"pep621"`, `"poetry"`, `"pre-commit"`

**Custom Regex Managers:**
Must use prefix: `"custom.regex"`

**Example if custom regex needed:**
```json
"enabledManagers": ["pep621", "poetry", "pre-commit", "github-actions", "custom.regex"]
```

### Special Cases

**Renovate Config Validation:**
- `renovate-config-validator` is a **CLI tool**, not a manager
- Does NOT go in `enabledManagers`
- Usage: `npx renovate-config-validator default.json`
- Can be added to pre-commit hooks via `renovatebot/pre-commit-hooks`

**custom.regex Manager:**
- ✅ Enabled in this preset for flexibility
- Currently no `customManagers` defined in preset itself
- Allows consuming projects to define custom regex patterns without overriding `enabledManagers`
- Example use cases: Version files, non-standard dependency formats
- **Important:** Use `"custom.regex"` in `enabledManagers`, not `"regex"`

## Automerge Strategy

### Branch-Based Automerge

**Configuration:**
```json
"automergeType": "branch",
"platformAutomerge": true,
"branchConcurrentLimit": 3,
"minimumReleaseAge": "3 days"
```

**Behavior:**
1. Renovate creates a branch for updates
2. **Waits 3 days after release** (stabilization period)
3. CI runs on the branch after waiting period
4. **If tests pass:** Changes pushed directly to base branch (silent merge)
5. **If tests fail:** PR created for manual review
6. No notifications for successful automerges
7. Maximum 3 concurrent update branches to avoid overwhelming CI

**Global Stabilization Period:**
- `minimumReleaseAge: "3 days"` applies to **all automerged updates** without exception
- Rationale: Community has time to find issues before automerge
- Applies uniformly to pre-commit hooks, GitHub Actions, and dependency updates
- Can be overridden per packageRule if needed (e.g., `"minimumReleaseAge": "0 days"` for trusted internal packages)

**Automerge Rules:**

| Update Type | Automerge? | Stabilization | Special Handling |
|-------------|-----------|---------------|------------------|
| Security vulnerabilities | ❌ No | 3 days | Manual review with `security` label via `vulnerabilityAlerts` |
| pre-commit hooks | ✅ Yes | 3 days | **ignoreTests: true** - automerges even if CI fails |
| github-actions | ✅ Yes | 3 days | Requires CI to pass |
| Minor/patch updates | ✅ Yes | 3 days | Semantic versioning guarantees |
| Pin/digest updates | ✅ Yes | 3 days | Security, no version change |
| Major updates | ❌ No | N/A | Requires manual review |

### Lock File Maintenance

**Configuration:**
```json
"lockFileMaintenance": {
  "enabled": true,
  "automerge": true,
  "schedule": ["before 4am on monday"]
}
```

**Purpose:**
- Updates transitive dependencies in `uv.lock` / `poetry.lock`
- Runs Monday mornings at 4am (independent of direct dependency updates)
- Critical for security patches in sub-dependencies
- Auto-merges if tests pass (respects global 3-day stabilization)

## Dependency Type Reference

### Python Ecosystem (Hyphenated)

Format: `kebab-case` / hyphenated

**PEP 621 / Poetry:**
- `dev-dependencies` - Development dependencies
- `test` - Test dependencies
- `optional` - Optional feature dependencies

**Usage in packageRules:**
```json
{
  "matchDepTypes": ["dev-dependencies", "test"],
  "automerge": true
}
```

### Node.js / npm (camelCase)

Format: `camelCase`

**npm / package.json:**
- `devDependencies` - Development dependencies
- `dependencies` - Production dependencies
- `peerDependencies` - Peer dependencies

**NOT applicable to Python projects** - this preset disables npm manager.

## Migration Strategy: Poetry → uv

### Current State

**Poetry Support:**
- Marked `DEPRECATED` in configuration comments
- Still enabled in `enabledManagers` array
- Includes `postUpdateOptions: ["poetry-lock"]` for lock file updates

### Migration Timeline

**Phase 1: Transition (Current)**
- Keep Poetry in `enabledManagers`
- Mark as deprecated with clear comment
- Both Poetry and uv (pep621) supported simultaneously
- Projects can migrate at their own pace

**Phase 2: Complete Migration**
- All consuming projects migrated to uv
- Remove Poetry from `enabledManagers`:
  ```json
  "enabledManagers": ["pep621", "pre-commit", "github-actions"]
  ```
- Delete Poetry packageRule from `default.json`

**Phase 3: Post-Migration**
- Only pep621 manager for Python dependencies
- uv auto-detected via `uv.lock` presence
- Lock file maintenance handles transitive dependencies

### uv Support Details

**Manager:** `pep621` (NOT a separate "uv" manager)

**Detection:**
- Presence of `uv.lock` file
- PEP 621 compliant `pyproject.toml`

**Files Updated:**
- `pyproject.toml` - Direct dependency versions
- `uv.lock` - Full dependency graph with hashes

**Lock File Refresh:**
- Weekly via `lockFileMaintenance`
- After each dependency update
- Ensures transitive dependencies stay current

## Configuration Best Practices

### Design Principles Applied

✅ **Explicit Over Implicit**
- `enabledManagers` makes Python-only scope clear
- Descriptions on all packageRules
- Deprecation comments for transitional features

✅ **DRY (Don't Repeat Yourself)**
- Removed `:enablePreCommit` (redundant with `enabledManagers`)
- Single source of truth for manager enablement

✅ **Security by Default**
- Lock file maintenance enabled (transitive deps)
- Digest pinning for GitHub Actions (via `config:best-practices`)
- Gitleaks in pre-commit hooks

✅ **Aggressive Automerge, Safe Boundaries**
- Non-major updates auto-merge (semantic versioning trust)
- Major updates require review (breaking changes possible)
- CI must pass before any automerge

## Documentation References

**Renovate Documentation:**
- Best practices preset: https://docs.renovatebot.com/presets-config/#configbest-practices
- pep621 manager: https://docs.renovatebot.com/modules/manager/pep621/
- enabledManagers: https://docs.renovatebot.com/configuration-options/#enabledmanagers
- Custom managers: https://docs.renovatebot.com/modules/manager/regex/
- Automerge: https://docs.renovatebot.com/key-concepts/automerge/

**uv Integration:**
- Official uv guide: https://docs.astral.sh/uv/guides/integration/dependency-bots/
- uv detected via `uv.lock` presence
- Managed through pep621 manager

**Poetry Documentation:**
- Poetry manager: https://docs.renovatebot.com/modules/manager/poetry/
- postUpdateOptions: https://docs.renovatebot.com/configuration-options/#postupdateoptions
