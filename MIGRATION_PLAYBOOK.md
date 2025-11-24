# Migration Playbook: Docker and Java Renovate Presets

**Date:** 2025-11-22
**Context:** Applying Python preset optimizations to parallel repositories

## Decision: Separate Sessions vs Single Session

**Recommendation: Use separate Claude Code sessions for each repository**

## Reasons for Separate Sessions

### 1. Context Window Management (Critical)

- Docker and Java ecosystems require extensive Context7 research (different managers, different best practices)
- Each repo needs dedicated focus without polluting context
- We just did deep research for Python - adding Docker + Java would exhaust context quickly

### 2. Ecosystem-Specific Requirements

**Docker preset needs:**
- `dockerfile` manager - Dockerfile base image updates
- `docker-compose` manager - Docker Compose service versions
- Container digest pinning - Security best practices
- Base image policies - Which base images to track/automerge

**Java preset needs:**
- `maven` manager - Maven pom.xml dependencies
- `gradle` manager - Gradle build files
- Java version management - JDK/JRE version tracking
- Spring Boot specifics - Spring Boot dependency management

### 3. Clean, Focused Work

- No risk of mixing files between repos
- Each repository gets proper analysis and documentation
- Better quality results per repo

## Template/Playbook for Docker and Java Repos

Apply the same optimization approach as `casc-renovate-preset-polarion-python`:

### 1. Configuration Structure

**Use `enabledManagers` to restrict to ecosystem-specific managers only:**

```json
{
  "enabledManagers": [
    "dockerfile",              // Docker: Dockerfile updates
    "docker-compose",          // Docker: docker-compose.yml updates
    "github-actions",          // Both: GitHub workflow actions
    "renovate-config-presets", // Both: Preset self-updates
    "custom.regex"             // Both: Custom dependency patterns
  ]
}
```

OR for Java:

```json
{
  "enabledManagers": [
    "maven",                   // Java: pom.xml dependencies
    "gradle",                  // Java: build.gradle dependencies (if used)
    "github-actions",          // Both: GitHub workflow actions
    "renovate-config-presets", // Both: Preset self-updates
    "custom.regex"             // Both: Custom dependency patterns
  ]
}
```

**Document which managers are enabled and why:**
- Add clear comments explaining each manager's purpose
- Create comparison table showing "with vs without enabledManagers"

### 2. Documentation Organization

**Create ARCHITECTURE.md:**
- Technical analysis and design decisions
- Package manager support matrix
- `enabledManagers` behavioral analysis with comparison tables
- Automerge strategy and rules
- Version compatibility notes
- Documentation references

**Enhance README.md:**
- User-facing documentation for preset consumers
- Features overview
- How to use the preset
- Migration guides (if applicable)
- Developer setup instructions
- Deployment warnings

**Create CLAUDE.md:**
- AI assistant development guidance
- Repository purpose and architecture
- Development commands (validation, pre-commit)
- Configuration guidelines
- Important constraints (email domain, deployment model, semantic commits)
- Internal documentation references

### 3. Research Requirements

**Use Context7 to verify all configuration:**
- Research each manager's capabilities and options
- Check for deprecations and inconsistencies
- Verify automerge best practices for the ecosystem
- Validate security options (digest pinning, etc.)

**Version Compatibility:**
- Document tested Renovate version
- Note forward compatibility (v41+/v42+)
- Verify all configuration options are stable

### 4. Validation Commands

**JSON syntax check:**
```bash
jq empty default.json
```

**Renovate config validation:**
```bash
npx renovate-config-validator default.json
```

**Pre-commit hooks (before commit):**
```bash
pre-commit run --all-files
```

### 5. Configuration Philosophy

**Ecosystem-Specific Focus:**
- Use `enabledManagers` to make intent explicit
- Prevents accidental detection of other ecosystems
- Clear documentation of scope

**Automerge Strategy:**
- Define what auto-merges (minor/patch, specific managers)
- Define what requires manual review (major updates)
- Enable lock file maintenance if applicable

**Security by Default:**
- Digest pinning for critical dependencies
- Lock file maintenance for transitive dependencies
- Gitleaks in pre-commit hooks

## Reference Implementation

**Python preset:** `github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-python`

**Key files to review:**
- `default.json` - Main configuration structure
- `ARCHITECTURE.md` - Complete technical documentation
- `CLAUDE.md` - AI assistant guidance
- `README.md` - User-facing documentation

## Docker Preset Specific Considerations

### Recommended enabledManagers

```json
"enabledManagers": [
  "dockerfile",              // Base image updates (FROM lines)
  "docker-compose",          // Service image versions
  "github-actions",          // Workflow dependencies
  "renovate-config-presets", // Self-updates
  "custom.regex"             // Custom patterns
]
```

### Docker-Specific Package Rules

**Digest pinning:**
```json
{
  "description": "Enable digest pinning for Docker images",
  "matchDatasources": ["docker"],
  "pinDigests": true
}
```

**Base image automerge:**
```json
{
  "description": "Automerge official base images (minor/patch)",
  "matchDatasources": ["docker"],
  "matchPackagePatterns": ["^alpine$", "^debian$", "^ubuntu$"],
  "matchUpdateTypes": ["minor", "patch", "digest"],
  "automerge": true
}
```

### Research Topics (Context7)

- Dockerfile manager capabilities
- Docker Compose manager features
- Digest pinning best practices
- Base image update strategies
- Multi-stage build considerations

## Java Preset Specific Considerations

### Recommended enabledManagers

```json
"enabledManagers": [
  "maven",                   // pom.xml dependencies
  "gradle",                  // build.gradle dependencies (if used)
  "github-actions",          // Workflow dependencies
  "renovate-config-presets", // Self-updates
  "custom.regex"             // Custom patterns
]
```

### Java-Specific Package Rules

**Spring Boot automerge:**
```json
{
  "description": "Automerge Spring Boot minor/patch updates",
  "matchPackagePatterns": ["^org.springframework.boot:"],
  "matchUpdateTypes": ["minor", "patch"],
  "automerge": true
}
```

**Test dependencies automerge:**
```json
{
  "description": "Automerge test dependencies",
  "matchDepTypes": ["test"],
  "automerge": true
}
```

### Research Topics (Context7)

- Maven manager capabilities
- Gradle manager features (if applicable)
- Spring Boot dependency management
- Java version tracking
- BOM (Bill of Materials) handling

## Deployment Notes

**Critical Reminders:**
- Changes to `default.json` are **live immediately** when merged to main
- No CI/CD pipeline - all consuming repositories receive updates directly
- Test thoroughly before pushing to main
- Use `renovate.json` in each preset repo for self-testing

## Next Steps

1. **Navigate to casc-renovate-preset-polarion-docker:**
   - Open new Claude Code session
   - Reference this playbook
   - Apply Docker-specific optimizations

2. **Navigate to casc-renovate-preset-polarion-java:**
   - Open new Claude Code session
   - Reference this playbook
   - Apply Java-specific optimizations

3. **Post-Migration:**
   - Verify all three presets follow consistent documentation structure
   - Cross-reference between presets for shared patterns
   - Update consuming repositories as needed
