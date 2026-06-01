---
name: release
description: Universal release workflow. Auto-detects version files and changelogs. Supports Node.js, Python, Rust, Claude Plugin, and generic projects. Use when user says "release", "发布", "new version", "bump version", "push", "推送".
---

# Release Skills

Universal release workflow supporting any project type with multi-language changelog.

## Quick Start

```bash
/release              # Auto-detect version bump
/release 1.1.0        # Specify version directly
/release --dry-run    # Preview only
/release --minor      # Force minor bump
/release --patch      # Force patch bump
/release --major      # Force major bump (with confirmation)
```

## Supported Projects

| Project Type | Version File | Auto-Detected |
|--------------|--------------|---------------|
| Node.js | package.json | ✓ |
| Python | pyproject.toml / setup.py | ✓ |
| Rust | Cargo.toml | ✓ |
| Claude Plugin | marketplace.json | ✓ |
| Browser Extension | manifest.json | ✓ |
| uTools Plugin | plugin.json | ✓ |
| Generic | VERSION / version.txt | ✓ |

## Options

| Flag | Description |
|------|-------------|
| `--dry-run` | Preview changes without executing |
| `--major` | Force major version bump |
| `--minor` | Force minor version bump |
| `--patch` | Force patch version bump |

## Workflow

### Step 1: Validate Version / Detect Project Configuration

**If user provides version directly** (e.g., `/release 1.2.0`):
1. Validate version format matches SemVer (e.g., 1.0.0, 1.2.3, 2.0.0-rc.1)
2. Use provided version instead of auto-detecting bump

**Otherwise, auto-detect**:
1. Check for `.releaserc.yml` (optional config override)
2. Auto-detect version file by scanning (priority order):
   - `package.json` (Node.js)
   - `pyproject.toml` (Python)
   - `setup.py` (Python)
   - `Cargo.toml` (Rust)
   - `manifest.json` (Browser Extension)
   - `plugin.json` (uTools)
   - `marketplace.json` or `.claude-plugin/marketplace.json` (Claude Plugin)
   - `VERSION` or `version.txt` (Generic)
3. Scan for changelog files using glob patterns:
   - `CHANGELOG*.md`
   - `HISTORY*.md`
   - `CHANGES*.md`
   - `documents/updates/*.md`
   - `docs/releases/*.md`
4. Identify language of each changelog by filename suffix
5. Display detected configuration

**Language Detection Rules**:

| Filename Pattern | Language |
|------------------|----------|
| `CHANGELOG.md` (no suffix) | en (default) |
| `CHANGELOG.zh.md` / `CHANGELOG_CN.md` / `CHANGELOG.zh-CN.md` | zh |
| `CHANGELOG.ja.md` / `CHANGELOG_JP.md` | ja |
| `CHANGELOG.ko.md` / `CHANGELOG_KR.md` | ko |
| `CHANGELOG.de.md` / `CHANGELOG_DE.md` | de |
| `CHANGELOG.fr.md` / `CHANGELOG_FR.md` | fr |
| `CHANGELOG.es.md` / `CHANGELOG_ES.md` | es |
| `CHANGELOG.{lang}.md` | Corresponding language code |

**Output Example**:
```
Project detected:
  Version file: package.json (1.2.3)
  Changelogs:
    - CHANGELOG.md (en)
    - CHANGELOG.zh.md (zh)
    - CHANGELOG.ja.md (ja)
```

### Step 2: Read Project Information

Read project files to understand current state:

**Required (if exists)**:
- `package.json` - Project name, version, description, dependencies

**Required for README sync** (read and audit):
- `README.md` / `README.zh.md` - Project introduction, features
- `SKILLS-INDEX.md` - Skill registry (if exists)

**Optional** (based on project type):
- `CLAUDE.md` - Architecture, tech stack, development guide
- `CHANGELOG.md` - Historical version updates
- `pyproject.toml` / `setup.py` - Python project config
- `Cargo.toml` - Rust project config
- `manifest.json` - Browser extension config
- `public/plugin.json` - uTools plugin config

**Extract**:
- Project name
- Project type (npm package, plugin, app, library)
- Current version
- Description
- Core features

### Step 3: Analyze Changes Since Last Tag

```bash
LAST_TAG=$(git tag --sort=-v:refname | head -1)
git log ${LAST_TAG}..HEAD --oneline
git diff ${LAST_TAG}..HEAD --stat
```

Categorize by conventional commit types:

| Type | Description |
|------|-------------|
| feat | New features |
| fix | Bug fixes |
| docs | Documentation |
| refactor | Code refactoring |
| perf | Performance improvements |
| test | Test changes |
| style | Formatting, styling |
| chore | Maintenance (skip in changelog) |

**Breaking Change Detection**:
- Commit message starts with `BREAKING CHANGE`
- Commit body/footer contains `BREAKING CHANGE:`
- Removed public APIs, renamed exports, changed interfaces

If breaking changes detected, warn user: "Breaking changes detected. Consider major version bump (--major flag)."

### Step 4: Determine Version Bump

Rules (in priority order):
1. User explicitly provided version → Use that version
2. User flag `--major/--minor/--patch` → Use specified
3. BREAKING CHANGE detected → Major bump (1.x.x → 2.0.0)
4. `feat:` commits present → Minor bump (1.2.x → 1.3.0)
5. Otherwise → Patch bump (1.2.3 → 1.2.4)

Display version change: `1.2.3 → 1.3.0`

### Step 5: Generate Multi-language Changelogs

For each detected changelog file:

1. **Identify language** from filename suffix
2. **Detect third-party contributors**:
   - Check merge commits: `git log ${LAST_TAG}..HEAD --merges --pretty=format:"%H %s"`
   - For each merged PR, identify the PR author via `gh pr view <number> --json author --jq '.author.login'`
   - Compare against repo owner (`gh repo view --json owner --jq '.owner.login'`)
   - If PR author ≠ repo owner → third-party contributor
3. **Generate content in that language**:
   - Section titles in target language
   - Change descriptions written naturally in target language (not translated)
   - Date format: YYYY-MM-DD (universal)
   - **Third-party contributions**: Append contributor attribution `(by @username)` to the changelog entry
4. **Insert at file head** (preserve existing content)

**Section Title Translations** (built-in):

| Type | en | zh | ja | ko | de | fr | es |
|------|----|----|----|----|----|----|-----|
| feat | Features | 新功能 | 新機能 | 새로운 기능 | Funktionen | Fonctionnalités | Características |
| fix | Fixes | 修复 | 修正 | 수정 | Fehlerbehebungen | Corrections | Correcciones |
| docs | Documentation | 文档 | ドキュメント | 문서 | Dokumentation | Documentation | Documentación |
| refactor | Refactor | 重构 | リファクタリング | 리팩토링 | Refactoring | Refactorisation | Refactorización |
| perf | Performance | 性能优化 | パフォーマンス | 성능 | Leistung | Performance | Rendimiento |
| breaking | Breaking Changes | 破坏性变更 | 破壊的変更 | 주요 변경사항 | Breaking Changes | Changements majeurs | Cambios importantes |

**Changelog Format (Minimal)**:

```markdown
## {VERSION} - {YYYY-MM-DD}

### Features
- Description of new feature
- Description of third-party contribution (by @username)

### Fixes
- Description of fix
```

Only include sections that have changes. Omit empty sections.

### Step 6: Generate Detailed Release Notes (Optional)

For projects that need detailed release documentation, generate standalone release notes:

**Document Path Options**:
- `CHANGELOG.md` - If project uses unified changelog
- `documents/updates/{version}.md` - If project uses separate version docs
- `docs/releases/{version}.md` - Alternative docs directory

**Detailed Release Note Template**:
```markdown
# {Project Name} {version} Release Notes

Release Date: {YYYY-MM-DD}

## 🎉 Overview
[Describe: major features, breaking changes, important changes]

## ✨ New Features
- Feature 1 description
- Feature 2 description

## 🔧 Improvements
- Improvement 1 description
- Performance optimization details

## 🐛 Bug Fixes
- Fixed issue 1
- Fixed issue 2

## 💥 Breaking Changes
- Change 1 (migration guide)
- Change 2

## 📦 Dependency Updates
- Updated package-A to version X
- Added package-B

## 🚀 Performance
- Performance improvement details

## ⚠️ Known Issues
- Issue description (if any)

## 🔮 Next Steps
- Planned features for next release

---
Thanks for using {Project Name}!
```

### Step 7: Synchronize README.md (MANDATORY)

**Purpose**: Ensure README.md accurately reflects the current project code and structure. This is NOT optional — stale documentation erodes user trust and onboarding experience.

#### 7.1 Audit README.md Against Project Reality

Run the following checks and identify ALL discrepancies:

**A. Directory Structure Check**
- Read README.md's directory tree section
- Compare against actual project directories (`find . -maxdepth 2 -type d ! -path './.git/*' ! -path './.idea/*' | sort`)
- Flag: directories listed in README but missing from project, and vice versa

**B. Skill/Module List Check**
- If `SKILLS-INDEX.md` exists, extract the current skill list from it
- Compare against README.md's skill/module descriptions
- Flag: skills missing from README, stale skill references, wrong paths

**C. Metadata Check**
- Author/maintainer name: match git config `user.name` or repo owner
- Repository URL: match `git remote get-url origin`
- License: match actual LICENSE file content
- Fork source: match git remote or repo metadata

**D. Feature/Description Check**
- Each feature/module described in README must have corresponding code
- Project tagline/description should match package.json or project config

**E. Quick Start / Usage Instructions**
- Verify commands shown are actually runnable
- Verify file paths referenced exist in project

#### 7.2 Generate Updated README.md

For each discrepancy found, update README.md to reflect reality:

1. **Update directory tree** to match actual project structure
2. **Sync skill list** with SKILLS-INDEX.md (if exists) and actual directories
3. **Fix metadata**: author, repo URL, license, fork info
4. **Remove stale sections** referencing deleted features/modules
5. **Add missing sections** for new top-level modules discovered
6. **Update "last updated" date** to release date

**README.md Content Rules**:
- Directory tree must use actual directory names (not planned/future ones)
- Only list skills that have SKILL.md files in their directories
- Keep the structure consistent with the project's README style
- Preserve any badges, shields, or status indicators

#### 7.3 Update CHANGELOG.md (if exists)

```markdown
## [{VERSION}] - {YYYY-MM-DD}

### Added
- New feature descriptions

### Changed
- Changed functionality descriptions

### Fixed
- Fixed bug descriptions

### Deprecated
- Deprecated features (if any)

### Removed
- Removed features (if any)

### Security
- Security fixes (if any)
```

#### 7.4 README Sync Verification

After updating README.md, verify:
- [ ] No stale directory references remain
- [ ] All mentioned skills exist as SKILL.md files
- [ ] Author and repo URL are correct
- [ ] Directory tree is accurate
- [ ] No placeholder/future content without corresponding code

### Step 8: Update Version Number

Ask user: "Update version number in config files?" Then update:

| File Type | Update Path |
|-----------|-------------|
| package.json | `$.version` |
| pyproject.toml | `project.version` |
| setup.py | `version=` parameter |
| Cargo.toml | `package.version` |
| manifest.json | `$.version` |
| plugin.json | `$.version` |
| marketplace.json | `$.metadata.version` |
| VERSION / version.txt | Direct content |

### Step 9: Group Changes by Skill/Module

Analyze commits since last tag and group by affected skill/module:

1. **Identify changed files** per commit
2. **Group by skill/module**:
   - `skills/<skill-name>/*` → Group under that skill
   - Root files (CLAUDE.md, etc.) → Group as "project"
   - Multiple skills in one commit → Split into multiple groups
3. **For each group**, identify related README updates needed

### Step 10: Commit Each Skill/Module Separately

For each skill/module group (in order of changes):

1. **Check README updates needed** (from Step 7 audit)
2. **Stage and commit**:
   ```bash
   git add skills/<skill-name>/*
   git add README.md README.zh.md
   git commit -m "<type>(<skill-name>): <meaningful description>"
   ```

**Important**: README.md updates from Step 7 synchronization MUST be included. If the sync revealed discrepancies, update README.md before committing.

### Step 11: User Confirmation

Before creating the release commit, ask user to confirm:

**Use AskUserQuestion with two questions**:

1. **Version bump** (single select):
   - Show recommended version: `1.2.3 → 1.3.0 (Recommended)`
   - Other options: `1.2.3 → 1.2.4`, `1.2.3 → 2.0.0`

2. **Push to remote** (single select):
   - Options: "Yes, push after commit", "No, keep local only"

### Step 12: Create Release Commit and Tag

After user confirmation:

1. **Stage version, changelog, and README files**:
   ```bash
   git add <version-file>
   git add CHANGELOG*.md
   git add README.md README.zh.md
   ```

2. **Create release commit**:
   ```bash
   git commit -m "chore: release v{VERSION}"
   ```

3. **Create tag**:
   ```bash
   git tag v{VERSION}
   ```

4. **Push if user confirmed**:
   ```bash
   git push origin main
   git tag v{VERSION} && git push origin v{VERSION}
   ```

**Note**: Do NOT add Co-Authored-By line. This is a release commit, not a code contribution.

### Step 13: Generate Release Checklist

After release, output a checklist for the user:

```markdown
## 📋 Release Checklist

### Documentation
- [x] Release notes generated
- [x] CHANGELOG.md updated
- [x] README.md synchronized with project structure
- [x] README.md discrepancies fixed (if any found)
- [x] Version number updated

### Version Control
- [x] Changes committed
- [x] Tag created: v{VERSION}
- [x] Pushed to remote (if applicable)

### Post-Release (Manual)
- [ ] Publish to npm/pypi/crates.io (if package)
- [ ] Create GitHub Release (if applicable)
- [ ] Update project website (if applicable)
- [ ] Notify users/community (if applicable)

### For Specific Project Types:
**npm package**: Run `npm publish` or `pnpm publish`
**Python package**: Run `python -m build && twine upload dist/*`
**Rust crate**: Run `cargo publish`
**Browser extension**: Submit to Chrome Web Store / Firefox Add-ons
**uTools plugin**: Submit to uTools plugin center
```

## Configuration (.releaserc.yml)

Optional config file in project root to override defaults:

```yaml
# .releaserc.yml - Optional configuration

version:
  file: package.json
  path: $.version

changelog:
  files:
    - path: CHANGELOG.md
      lang: en
    - path: CHANGELOG.zh.md
      lang: zh

  sections:
    feat: Features
    fix: Fixes
    docs: Documentation
    refactor: Refactor
    perf: Performance
    test: Tests
    chore: null

commit:
  message: "chore: release v{version}"

tag:
  prefix: v
  sign: false

# Files always included in release commit for documentation sync
include:
  - README.md       # Always sync README with project reality
  - README.zh.md    # Chinese README (if exists)
  - package.json    # Version file
```

## Dry-Run Mode

When `--dry-run` is specified:

```
=== DRY RUN MODE ===

Project detected:
  Version file: package.json (1.2.3)

Proposed version: v1.3.0

README sync audit:
  ✓ Directory structure: OK
  ✗ Stale sections: "productivity", "communication"
  ✗ Missing modules: "harness/", "release/"
  ✗ Metadata: maintainer mismatch

Changes grouped:
  - feat: add new feature
  - fix: fix bug

Changelog preview:
  ## 1.3.0 - 2026-01-22
  ### Features
  - Add new feature

No changes made. Run without --dry-run to execute.
```

## Documentation Quality Requirements

### Content Accuracy
- **README.md MUST match project reality**: Every directory, skill, feature, and command mentioned must exist
- No stale/planned/future content in README without corresponding code
- Keep release notes consistent with main documentation
- Verify technical stack info matches config files
- Release process MUST audit and fix README.md discrepancies

### Markdown Format
- Empty lines before/after headings (MD022)
- Empty lines before/after lists (MD032)
- Consistent list markers (- or *)
- Code blocks with language identifiers
- Correct link formatting

### Writing Style
- Clear headings and structure
- Use emoji icons appropriately (optional)
- Professional yet friendly tone
- Detailed but not verbose
-面向目标用户群体 (developer, end user, etc.)

## Version Number Specification

Follow Semantic Versioning 2.0.0:
- **Major** (X.0.0): Incompatible API changes
- **Minor** (1.X.0): Backwards-compatible new features
- **Patch** (1.2.X): Backwards-compatible bug fixes

**Format**: `MAJOR.MINOR.PATCH` (e.g., `1.0.0`, `1.2.3`, `2.0.0`)

**Pre-release identifiers**:
- Alpha: `1.0.0-alpha.1`
- Beta: `1.0.0-beta.1`
- RC: `1.0.0-rc.1`

## When to Use

Trigger this skill when user requests:
- "release", "发布", "create release", "new version", "新版本"
- "bump version", "update version", "更新版本"
- "prepare release"
- "push to remote" (with uncommitted changes)

**Important**: If user says "just push" or "直接 push" with uncommitted changes, STILL follow all steps above first.

## Supported Project Types

This skill supports but is not limited to:
- **Web Apps**: React, Vue, Angular, Next.js, etc.
- **Backend Services**: Node.js, Python, Rust, Go, etc.
- **npm Packages**: JavaScript/TypeScript libraries
- **Python Packages**: PyPI packages
- **Rust Crates**: crates.io packages
- **Browser Extensions**: Chrome, Firefox, Edge extensions
- **Editor Plugins**: VS Code, Sublime Text, Vim plugins
- **Desktop Apps**: Electron, Tauri apps
- **Mobile Apps**: React Native, Flutter apps
- **uTools Plugins**: uTools platform plugins
- **Claude Plugins**: Claude Code extensions