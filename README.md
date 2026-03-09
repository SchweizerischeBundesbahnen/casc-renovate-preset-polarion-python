# casc-renovate-preset-polarion-python

**Renovate preset for Polarion Python repositories on GitHub**

Extends [casc-renovate-preset-polarion](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion) (shared base) and adds Python-specific configuration.

---

## Quick Start

Add this to your repository's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-python"
  ]
}
```

---

## Supported Ecosystems

| Ecosystem | Manager | Auto-Update |
|-----------|---------|-------------|
| **Python** | pep621 (uv/pyproject.toml) | Minor/patch (3-day wait) |
| **Python** | Poetry (pyproject.toml) | Minor/patch (3-day wait) |
| **Pre-commit** | pre-commit | All updates incl. major (grouped) |
| **GitHub Actions** | github-actions | All updates incl. major (grouped) |

**All other managers are disabled** — this is a Python-specific preset.

---

## Automerge Strategy

Inherited from the [shared base preset](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion). See base preset for full details.

| Update Type | Automerged? | Stabilization | Notes |
|-------------|-------------|---------------|-------|
| Minor/patch | ✅ Yes | 3 days | Requires CI to pass |
| Major | ❌ No | — | Manual review required |
| GitHub Actions (any incl. major) | ✅ Yes | 3 days | Grouped into one branch/PR |
| Pre-commit hooks (any incl. major) | ✅ Yes | 3 days | Grouped into one branch/PR, ignoreTests: true |
| Security vulnerabilities | ❌ No | 0 days | Labeled `security`, priority queue |
| Lock file maintenance | ✅ Yes | — | Monday before 4am |

---

## Configuration Examples

### Disable Poetry (uv-only repos)

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-python"],
  "enabledManagers": ["pep621", "pre-commit", "github-actions", "custom.regex"]
}
```

### Override Python Version Constraint

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-python"],
  "constraints": {
    "python": ">=3.13"
  }
}
```

---

## Developer Setup

```bash
# JSON syntax check
jq empty default.json

# Renovate schema validation
npx -p renovate renovate-config-validator default.json

# Run all pre-commit hooks
pre-commit run --all-files
```

---

## Deployment

Changes to `default.json` are **live immediately** when merged to `main`. All consuming repositories receive updates on the next Renovate run — there is no staging environment.

Changes to the [shared base preset](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion) also affect all repos using this preset.

---

**Maintained by:** SBB Polarion Team
