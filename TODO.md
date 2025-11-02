# TODO: Convert to Reusable GitHub Action

## Project Goal

Convert the example workflow (`example.yml`) into a reusable GitHub Action that can be published to the GitHub Marketplace as `citation-sync-action`. This will allow other users to easily integrate CITATION.cff synchronization into their repositories without copying the entire workflow.

## 📋 Design Decisions Summary (All Confirmed)

### Core Settings
- **Action Name**: `citation-sync-action`
- **Author**: Adamtaranto
- **License**: MIT
- **Action Type**: Composite action (shell-based)
- **No backward compatibility required** - can improve and enhance

### Key Features
1. **Update Modes**:
   - `increment` (default): Increments patch version, creates new tag
   - `match`: Updates CITATION.cff to match current tag, no increment/new tag
   - Both modes exit successfully when no update needed

2. **Version Tag Support**:
   - Default format: `v0.0.0` (semantic versioning with 'v' prefix)
   - **Pre-release versions supported**: `v1.0.0-beta.1`, `v2.0.0-rc.2`, etc.
   - Configurable prefix (empty, single, or multi-character): `v`, `release-`, `ver-`, etc.
   - Configurable format validation via regex
   - Pre-release suffix preserved in CITATION.cff
   - Incrementing ignores pre-release (v1.0.0-beta → v1.0.1)

3. **Branch Management**:
   - Auto-detects default branch using git symbolic-ref (local, fast)
   - Fallback to git remote show origin
   - Last resort: 'main'
   - No GitHub API calls (avoids rate limits)

4. **Validation Strategy**:
   - **Before update**: Validate structure, fail fast if errors
   - **After update**: Validate changes, rollback on failure
   - Field uniqueness check (version, date-released must appear exactly once)
   - YAML syntax validation
   - Optional CFF schema validation

5. **Error Handling**:
   - Use GitHub Actions annotations (`::error::`, `::warning::`, `::notice::`)
   - Provide actionable suggestions in error messages
   - Automatic rollback if post-update validation fails

## Design Decisions (Confirmed)

- **Action Name**: `citation-sync-action`
- **Author**: Adamtaranto
- **License**: MIT
- **Action Type**: Composite action (shell-based)
- **Default Behavior**: Increment patch version and create new tag
- **Branch Detection**: Auto-detect default branch
- **Versioning**: Keep patch increment as default behavior
- **Backward Compatibility**: Not required (can improve and enhance)

## Action Type Decision: Composite Action

Based on the current implementation and GitHub Actions documentation, a **composite action** is the best choice because:

1. The workflow is entirely shell-based (bash scripts)
2. No custom JavaScript/TypeScript code is needed
3. Composite actions can run multiple steps using `run:` commands
4. They support all the features we need (inputs, outputs, using other actions)
5. Easier to maintain and test than Docker-based actions
6. Faster execution than Docker (no container build/pull overhead)

## Implementation Plan

### Phase 1: Repository Setup

#### 1.1 Create Core Files

- [ ] **Create `action.yml`** (or `action.yaml`)
  - This is the metadata file required for all GitHub Actions
  - Must be in the repository root
  - Defines the action's interface (inputs, outputs, branding)

- [ ] **Create comprehensive `README.md`**
  - Clear description of what the action does
  - Installation/usage instructions with examples
  - Required permissions and setup
  - Input/output documentation
  - Troubleshooting section
  - Contributing guidelines

- [ ] **Create `LICENSE`** file
  - Choose appropriate open-source license (MIT, Apache 2.0, etc.)
  - Required for marketplace publication

- [ ] **Create `.github/workflows/ci.yml`**
  - Automated testing workflow
  - Validates action.yml syntax
  - Tests the action in a real scenario
  - Runs on pull requests and pushes to main

- [ ] **Create community health files**
  - `CODE_OF_CONDUCT.md` - Community standards
  - `CONTRIBUTING.md` - How to contribute
  - `SECURITY.md` - Security policy and vulnerability reporting
  - `.github/ISSUE_TEMPLATE/` - Issue templates
  - `.github/PULL_REQUEST_TEMPLATE.md` - PR template

#### 1.2 Repository Configuration

- [ ] Ensure repository is public (required for Marketplace)
- [ ] Configure repository settings:
  - Enable issues
  - Enable wikis (optional)
  - Configure branch protection for main
- [ ] Add topics/tags for discoverability:
  - `github-actions`
  - `citation`
  - `cff`
  - `citation-file-format`
  - `semantic-versioning`
  - `automation`

### Phase 2: Action Implementation

#### 2.1 Define Action Metadata (`action.yml`)

```yaml
name: 'Citation Sync Action'
description: 'Automatically synchronize CITATION.cff version and date with Git tags'
author: 'Adamtaranto'

branding:
  icon: 'tag'
  color: 'blue'

inputs:
  token:
    description: 'GitHub token for authentication. Use secrets.GITHUB_TOKEN or a PAT.'
    required: false
    default: ${{ github.token }}
  
  citation-path:
    description: 'Path to CITATION.cff file relative to repository root'
    required: false
    default: 'CITATION.cff'
  
  target-branch:
    description: 'Branch to push updated CITATION.cff to. If empty, auto-detects default branch.'
    required: false
    default: ''
  
  update-mode:
    description: 'Update mode: "increment" (default, creates new incremented tag) or "match" (updates to match current tag without increment)'
    required: false
    default: 'increment'
  
  version-prefix:
    description: 'Expected version tag prefix (e.g., "v", "release-", "")'
    required: false
    default: 'v'
  
  version-format:
    description: 'Expected semantic version format regex. Default validates X.Y.Z format with optional pre-release suffix.'
    required: false
    default: '^([0-9]+)\.([0-9]+)\.([0-9]+)(-[a-zA-Z0-9.-]+)?$'
  
  enable-debug:
    description: 'Enable debug output'
    required: false
    default: 'false'
  
  commit-message:
    description: 'Template for commit message. Use {version} as placeholder.'
    required: false
    default: 'chore: Update CITATION.cff to version {version}'
  
  git-user-name:
    description: 'Git user name for commits'
    required: false
    default: 'github-actions[bot]'
  
  git-user-email:
    description: 'Git user email for commits'
    required: false
    default: 'github-actions[bot]@users.noreply.github.com'
  
  skip-ci:
    description: 'Add [skip ci] to commit message'
    required: false
    default: 'true'
  
  fail-on-conflict:
    description: 'Fail if incremented tag already exists (only applies in increment mode)'
    required: false
    default: 'true'
  
  validate-cff:
    description: 'Validate CITATION.cff format before and after updates'
    required: false
    default: 'true'

outputs:
  needs-update:
    description: 'Whether CITATION.cff needed updating (true/false)'
    value: ${{ steps.check-update.outputs.needs_update }}
  
  original-tag:
    description: 'The tag that triggered this action'
    value: ${{ steps.tag-info.outputs.tag_name }}
  
  new-tag:
    description: 'The new tag created (if update was needed and mode is increment)'
    value: ${{ steps.tag-info.outputs.incremented_tag_name }}
  
  new-version:
    description: 'The new version written to CITATION.cff'
    value: ${{ steps.tag-info.outputs.new_version }}
  
  commit-sha:
    description: 'The commit SHA with updated CITATION.cff (if update was needed)'
    value: ${{ steps.commit.outputs.new_commit }}
  
  target-branch:
    description: 'The branch that was updated'
    value: ${{ steps.detect-branch.outputs.target_branch }}

runs:
  using: 'composite'
  steps:
    # All the steps from example.yml, adapted for composite action
```

#### 2.2 Convert Workflow Steps to Composite Action

Key modifications needed:

1. **Change all `run:` steps to use `shell: bash`**
   - Composite actions require explicit shell specification

2. **Replace `${{ env.DEBUG }}` with `${{ inputs.enable-debug }}`**
   - Use inputs instead of workflow-level environment variables

3. **Replace hardcoded values with inputs**
   - `CITATION.cff` → `${{ inputs.citation-path }}`
   - Detect default branch automatically if `${{ inputs.target-branch }}` is empty
   - Git user config → use inputs

4. **Replace `secrets.CITATION_UPDATE_PAT || secrets.GITHUB_TOKEN` with `${{ inputs.token }}`**
   - Users pass token as input

5. **Add step IDs for all steps that produce outputs**
   - Required for output mapping in action.yml

6. **Handle the checkout step**
   - Use `actions/checkout@v5` within the composite action
   - Pass inputs.token to checkout

7. **Update conditional expressions**
   - `if: steps.check_update.outputs.needs_update == 'true'` stays the same
   - Ensure step IDs match between action.yml outputs and step definitions

8. **Implement update mode logic**
   - When `inputs.update-mode == 'match'`: Update CITATION.cff to match current tag version (no increment, no new tag)
   - When `inputs.update-mode == 'increment'`: Use existing behavior (increment patch, create new tag)

9. **Implement version tag validation**
   - Parse tag with configurable prefix (`inputs.version-prefix`)
   - Support empty, single, or multi-character prefixes
   - Strip prefix: `TAG_VERSION=${TAG_NAME#${VERSION_PREFIX}}`
   - Validate version format using regex pattern (`inputs.version-format`)
   - Support pre-release versions (e.g., `1.0.0-beta.1`)
   - Extract major.minor.patch components (ignore pre-release for incrementing)
   - Raise clear error if tag doesn't match expected format
   - Example error: `::error::Tag 'v1.0' does not match expected format. Expected: prefix='v' + version matching '^[0-9]+\.[0-9]+\.[0-9]+(-[a-zA-Z0-9.-]+)?$'`

10. **Add CITATION.cff field validation**
    - Check that `version` field appears exactly once
    - Check that `date-released` field appears exactly once
    - Raise error if duplicates found
    - Perform before update (validation guard)

11. **Add default branch detection**
    - Create new step to detect default branch
    - **Method 1 (Primary)**: Use git symbolic-ref
      ```bash
      TARGET_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
      ```
    - **Method 2 (Fallback)**: Parse git remote show origin
      ```bash
      TARGET_BRANCH=$(git remote show origin | grep 'HEAD branch' | sed 's/.*: //')
      ```
    - **Method 3 (Last Resort)**: Default to 'main'
    - Use detected branch if `inputs.target-branch` is empty
    - Store in output for user visibility
    - Error if branch doesn't exist

12. **Implement validation with rollback**
    - **Before update validation** (fail-fast):
      - YAML syntax check
      - Field uniqueness check (version, date-released)
      - Required fields check (if validate-cff=true)
      - Exit with `::error::` if any check fails
      - Don't proceed to update step
    - **After update validation** (safety check):
      - Create backup before any changes (`CITATION.cff.bak`)
      - Perform update
      - Validate YAML syntax
      - Validate version and date fields are correct
      - If validation fails:
        - Restore from backup: `mv CITATION.cff.bak CITATION.cff`
        - Exit with `::error::` explaining the rollback
        - Include suggestion to report bug if this happens
      - Delete backup if successful

#### 2.3 New Steps to Add

1. **Detect Default Branch** (new step)

   **Method 1 (Primary)**: Use git symbolic-ref
   
   ```bash
   TARGET_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
   ```

   **Method 2 (Fallback)**: Parse git remote show origin
   
   ```bash
   TARGET_BRANCH=$(git remote show origin | grep 'HEAD branch' | sed 's/.*: //')
   ```

   **Method 3 (Last Resort)**: Default to 'main'

2. **Validate Tag Format** (enhanced)
   - Strip prefix from tag: `TAG_VERSION=${TAG_NAME#${VERSION_PREFIX}}`
   - Works with empty, single, or multi-character prefixes
   - Validate against version-format regex (supports pre-release)
   - Parse major.minor.patch components using bash regex:
     ```bash
     if [[ $TAG_VERSION =~ ^([0-9]+)\.([0-9]+)\.([0-9]+)(-[a-zA-Z0-9.-]+)?$ ]]; then
       MAJOR=${BASH_REMATCH[1]}
       MINOR=${BASH_REMATCH[2]}
       PATCH=${BASH_REMATCH[3]}
       PRERELEASE=${BASH_REMATCH[4]}  # May be empty
     fi
     ```
   - For incrementing: Use only major.minor.patch, ignore pre-release
   - Error with actionable message if format doesn't match

3. **Validate CITATION.cff Fields** (enhanced)

   ```bash
   # Count occurrences of fields
   VERSION_COUNT=$(grep -c '^version:' CITATION.cff || echo "0")
   DATE_COUNT=$(grep -c '^date-released:' CITATION.cff || echo "0")
   
   if [ "$VERSION_COUNT" -ne 1 ]; then
     echo "::error::CITATION.cff must have exactly one 'version' field (found: ${VERSION_COUNT})"
     exit 1
   fi
   
   if [ "$DATE_COUNT" -ne 1 ]; then
     echo "::error::CITATION.cff must have exactly one 'date-released' field (found: ${DATE_COUNT})"
     exit 1
   fi
   ```

4. **Validate CITATION.cff Format** (with rollback support)

   **Before Update**:
   - YAML syntax validation
   - Field uniqueness validation
   - Required fields check (if validate-cff=true)
   - Hard error if any check fails

   **After Update**:
   - Create backup first: `cp CITATION.cff CITATION.cff.bak`
   - Perform update
   - Validate YAML syntax still valid
   - Verify version and date were updated correctly
   - If validation fails:
     ```bash
     echo "::error::Update validation failed. Rolling back changes."
     mv CITATION.cff.bak CITATION.cff
     echo "::error::Please report this issue - the action may have a bug."
     exit 1
     ```
   - Remove backup if successful: `rm CITATION.cff.bak`

### Phase 3: Testing & Validation

#### 3.1 Create Test Repository/Workflow

- [ ] Create `.github/workflows/test-action.yml` in this repository
  - Workflow that triggers on push to test branches
  - Creates test tags
  - Calls the action from the local repository
  - Validates outputs
  - Verifies CITATION.cff was updated correctly

- [ ] Create test fixtures
  - Sample CITATION.cff files with various formats
  - Test cases for different version formats (with/without 'v' prefix)
  - Test cases with custom prefixes (e.g., 'release-', 'ver-')
  - Edge cases (existing incremented tag, missing fields, duplicate fields, etc.)

#### 3.2 Manual Testing Checklist

**Version Tag Format Testing:**

- [ ] Test with standard 'v' prefix (v1.0.0)
- [ ] Test with custom prefix (release-1.0.0, ver-1.0.0)
- [ ] Test without prefix (1.0.0)
- [ ] Test with pre-release versions (v1.0.0-beta.1, v2.0.0-rc.2, v1.0.0-alpha)
- [ ] Test that pre-release suffix is preserved in CITATION.cff
- [ ] Test that incrementing ignores pre-release (v1.0.0-beta increments to v1.0.1, not v1.0.1-beta)
- [ ] Test with invalid format (should fail with clear error)
- [ ] Test with non-semantic version (v1.0, v1) (should fail)
- [ ] Test with multi-character prefix (release-, version-, ver-)
- [ ] Test with empty prefix and numeric-only tag (1.0.0)

**Update Mode Testing:**

- [ ] Test increment mode (default) - creates new incremented tag
- [ ] Test match mode - updates to current tag without increment
- [ ] Test match mode - verifies no new tag is created
- [ ] Test match mode - exits successfully when no update needed

**CITATION.cff Validation:**
- [ ] Test when CITATION.cff is up to date (no-op scenario)
- [ ] Test when CITATION.cff has wrong version
- [ ] Test when CITATION.cff has wrong date
- [ ] Test with duplicate version field (should fail)
- [ ] Test with duplicate date-released field (should fail)
- [ ] Test with missing version field (should fail)
- [ ] Test with missing date-released field (should fail)

**Branch Detection:**
- [ ] Test with empty target-branch (should auto-detect)
- [ ] Test with explicit target-branch
- [ ] Test with non-default branch
- [ ] Verify target-branch output is set correctly

**Other Edge Cases:**
- [ ] Test when incremented tag already exists (should fail in increment mode)
- [ ] Test with custom citation-path input
- [ ] Test with debug mode enabled
- [ ] Test permissions (ensure contents: write is sufficient)
- [ ] Test with custom commit message template
- [ ] Test with skip-ci disabled
- [ ] Test fail-on-conflict=false behavior

**CITATION.cff Format Validation (if validate-cff=true):**
- [ ] Test with valid CITATION.cff
- [ ] Test with invalid YAML syntax (should fail)
- [ ] Test with missing required CFF fields (should warn or fail)
- [ ] Test with validate-cff=false (should skip validation)

### Phase 4: Documentation

#### 4.1 README.md Structure

```markdown
# CFF Tag Sync Action

Brief description and badges (version, license, marketplace)

## Features

List key features

## Usage

### Basic Example
```yaml
name: Update CITATION.cff on Tag
on:
  push:
    tags:
      - '*'

jobs:
  update-citation:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0
          
      - uses: YourUsername/cff-tag-sync-action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced Example

Show usage with custom inputs

## Inputs

Detailed table of all inputs

## Outputs

Detailed table of all outputs

## Permissions

Required permissions explanation

## How It Works

High-level explanation (link to OVERVIEW.md)

## Troubleshooting

Common issues and solutions

## Contributing

Link to CONTRIBUTING.md

## License

License information
```

#### 4.2 Additional Documentation

- [ ] Add inline comments to action.yml
- [ ] Create examples/ directory with sample workflows
- [ ] Document the version incrementing behavior clearly
- [ ] Create FAQ section for common questions
- [ ] Add migration guide from the original workflow

### Phase 5: Publishing to Marketplace

#### 5.1 Pre-Publication Checklist

- [ ] Verify repository is public
- [ ] Ensure only one `action.yml` in root (no workflow files in `.github/workflows/` that would conflict)
- [ ] Choose unique action name (check marketplace for conflicts)
- [ ] Add clear branding (icon and color)
- [ ] Test action thoroughly
- [ ] Ensure README is comprehensive
- [ ] Add LICENSE file
- [ ] Create initial Git tag (v1.0.0)

#### 5.2 Publication Steps

1. **Create First Release**
   - Navigate to repository on GitHub
   - Go to the `action.yml` file in the repository
   - Click "Draft a release" banner
   - Check "Publish this Action to the GitHub Marketplace"
   - Select primary category (e.g., "Automation")
   - Optionally select secondary category
   - Enter tag version (v1.0.0)
   - Enter release title
   - Write release notes
   - Click "Publish release"

2. **Verify Marketplace Listing**
   - Action should appear in GitHub Marketplace
   - Verify metadata displays correctly
   - Check that usage examples work

### Phase 6: Release Management Strategy

#### 6.1 Version Tagging Strategy

Follow semantic versioning with moving tags:

- **Specific versions**: `v1.0.0`, `v1.0.1`, `v1.1.0`, `v2.0.0`
  - Never move these tags
  - Users can pin to exact versions for stability

- **Major version tags**: `v1`, `v2`
  - Move these to latest compatible release
  - Recommended for most users
  - Update using: `git tag -fa v1 -m "Update v1 to v1.2.3" && git push -f origin v1`

- **Minor version tags**: `v1.0`, `v1.1` (optional)
  - Can move to latest patch
  - For users who want minor version stability

#### 6.2 Release Workflow

- [ ] Create `.github/workflows/release.yml`
  - Triggers on published releases
  - Validates action.yml
  - Updates major version tag automatically
  - Creates release notes

- [ ] Create CHANGELOG.md
  - Document all changes between versions
  - Follow Keep a Changelog format

#### 6.3 Ongoing Maintenance Plan

- [ ] Monitor issues and respond promptly
- [ ] Review and merge pull requests
- [ ] Update dependencies (actions/checkout, etc.)
- [ ] Test with new GitHub Actions features
- [ ] Keep documentation up to date
- [ ] Security updates as needed

## Key Inputs to Expose

Based on the workflow analysis and requirements, here are the inputs users will need:

### Essential Inputs

1. **token** - GitHub token for authentication (default: github.token)
2. **citation-path** - Path to CITATION.cff (default: 'CITATION.cff')
3. **target-branch** - Branch to push updates to (default: '' = auto-detect)

### Update Behavior

4. **update-mode** - 'increment' (default, creates new tag) or 'match' (updates to match current tag)

### Version Configuration

5. **version-prefix** - Expected tag prefix (default: 'v')
6. **version-format** - Regex for semantic version validation (default: '^([0-9]+)\.([0-9]+)\.([0-9]+)$')

### Validation Options

7. **validate-cff** - Validate CITATION.cff format (default: true)

### Git Configuration

8. **git-user-name** - Name for commit author (default: 'github-actions[bot]')
9. **git-user-email** - Email for commit author (default: 'github-actions[bot]@users.noreply.github.com')
10. **commit-message** - Customizable commit message template (default: 'chore: Update CITATION.cff to version {version}')
11. **skip-ci** - Whether to add [skip ci] to commits (default: true)

### Debug & Error Handling

12. **enable-debug** - Enable verbose debugging output (default: false)
13. **fail-on-conflict** - Fail if incremented tag exists (default: true, only applies in increment mode)

### Future Considerations

These inputs might be useful in future versions:
- **increment-type** - Which version component to increment (patch/minor/major) - for now, always patch
- **dry-run** - Preview changes without committing
- **additional-fields** - Other CITATION.cff fields to update automatically
- **tag-message** - Custom message for annotated tags

## CITATION.cff Validation Strategy

### Validation Levels

The action should implement multiple levels of validation when `validate-cff: true`:

#### Level 1: Basic Structure Validation (Always Performed)

1. **YAML Syntax Validation**
   - Ensure the file is valid YAML
   - Check for parsing errors
   - Tool: `yq` or `python -c "import yaml; yaml.safe_load(open('CITATION.cff'))"`

2. **Field Uniqueness Check**
   - Verify `version` field appears exactly once
   - Verify `date-released` field appears exactly once
   - Implementation: `grep -c '^version:' CITATION.cff`

3. **Field Existence Check**
   - Ensure required fields exist: `cff-version`, `message`, `title`, `authors`
   - Ensure target fields exist: `version`, `date-released`

#### Level 2: CFF Schema Validation (Recommended)

Use existing CFF validation tools:

**Option A: cffconvert (Python tool)**
```bash
pip install cffconvert
cffconvert --validate -i CITATION.cff
```
- Pros: Official CFF tool, comprehensive validation
- Cons: Requires Python dependency, slower

**Option B: cff-validator (Docker)**
```bash
docker run --rm -v $(pwd):/app citationcff/cff-validator /app/CITATION.cff
```
- Pros: No local dependencies, comprehensive
- Cons: Requires Docker, slower startup

**Option C: Custom validation script**
```bash
# Check CFF schema version
CFF_VERSION=$(grep '^cff-version:' CITATION.cff | sed 's/cff-version:[[:space:]]*//')
if [[ ! "$CFF_VERSION" =~ ^1\.[0-9]+\.[0-9]+$ ]]; then
  echo "::warning::Unexpected cff-version: $CFF_VERSION"
fi

# Verify required fields
for field in cff-version message title authors; do
  if ! grep -q "^${field}:" CITATION.cff; then
    echo "::error::Missing required field: ${field}"
    exit 1
  fi
done

# Check authors structure
if ! grep -q "^authors:" CITATION.cff; then
  echo "::error::CITATION.cff must have an authors field"
  exit 1
fi
```
- Pros: Fast, no external dependencies
- Cons: Limited validation, may miss edge cases

#### Level 3: Semantic Validation

1. **Version Format Check**
   - Validate version follows semantic versioning
   - Already implemented in tag parsing

2. **Date Format Check**
   - Validate date-released is in YYYY-MM-DD format
   - `date -d "$DATE" 2>/dev/null` (validates date)

3. **Authors Structure**
   - Check authors is a list
   - Verify each author has required fields (family-names or name)

### Recommended Implementation

**Initial Release (MVP):**
- Level 1: Basic structure validation (YAML, field uniqueness, field existence)
- Fast, no dependencies
- Catches most common errors

**Future Enhancement:**
- Add option to use cffconvert or cff-validator
- New input: `validation-tool: 'basic' | 'cffconvert' | 'docker'`
- Document how to set up Python/Docker if needed

### Validation Timing

1. **Before Updates**
   - Validate existing CITATION.cff structure
   - Fail early if file is malformed
   - Skip update if validation fails

2. **After Updates**
   - Validate that changes didn't break the file
   - Ensure YAML is still valid
   - Verify version and date fields are correct

### Error Handling

- **Validation errors**: Fail with clear error message
- **Validation warnings**: Log warning but continue (e.g., missing optional fields)
- **When validate-cff=false**: Skip all validation except field uniqueness check

### Implementation Code Snippet

```bash
validate_citation_cff() {
  local cff_file="$1"
  local mode="$2"  # 'before' or 'after'
  
  # Check YAML syntax
  if ! python3 -c "import yaml; yaml.safe_load(open('${cff_file}'))" 2>/dev/null; then
    echo "::error::Invalid YAML syntax in ${cff_file}"
    return 1
  fi
  
  # Check field uniqueness (always performed)
  local version_count=$(grep -c '^version:' "${cff_file}" || echo "0")
  local date_count=$(grep -c '^date-released:' "${cff_file}" || echo "0")
  
  if [ "$version_count" -ne 1 ]; then
    echo "::error::CITATION.cff must have exactly one 'version' field (found: ${version_count})"
    return 1
  fi
  
  if [ "$date_count" -ne 1 ]; then
    echo "::error::CITATION.cff must have exactly one 'date-released' field (found: ${date_count})"
    return 1
  fi
  
  # Skip additional validation if disabled
  if [ "${{ inputs.validate-cff }}" != "true" ]; then
    return 0
  fi
  
  # Check required fields exist
  for field in cff-version message title authors; do
    if ! grep -q "^${field}:" "${cff_file}"; then
      echo "::error::Missing required CFF field: ${field}"
      return 1
    fi
  done
  
  # Validate version format (if after update)
  if [ "$mode" = "after" ]; then
    local version=$(grep '^version:' "${cff_file}" | sed 's/version:[[:space:]]*//' | sed 's/^"//;s/"$//')
    if [[ ! "$version" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
      echo "::error::Invalid version format: ${version}"
      return 1
    fi
  fi
  
  # Validate date format
  local date=$(grep '^date-released:' "${cff_file}" | sed 's/date-released:[[:space:]]*//' | sed 's/^"//;s/"$//')
  if ! date -d "$date" >/dev/null 2>&1; then
    echo "::error::Invalid date format: ${date}"
    return 1
  fi
  
  echo "::notice::CITATION.cff validation passed"
  return 0
}
```

### Documentation Notes

In the README, document:
1. What validation is performed by default
2. How to disable validation (`validate-cff: false`)
3. What errors users might encounter
4. How to fix common validation errors
5. Future plans for enhanced validation tools

## Important Considerations

### 1. Scope Management

- Keep the action focused on CITATION.cff synchronization
- Don't add unrelated features (stay focused)
- Consider extensibility through inputs

### 2. Error Handling

- Provide clear error messages
- Use appropriate exit codes (0 = success, non-zero = failure)
- Validate inputs early
- Handle edge cases gracefully

### 3. Security

- Use `${{ github.token }}` as default (automatic)
- Document when PAT is needed vs GITHUB_TOKEN
- Never log sensitive information
- Follow GitHub's security best practices

### 4. Compatibility

- Don't hardcode GitHub URLs (use GITHUB_API_URL environment variable)
- Support GitHub Enterprise Server users
- Use stable versions of dependent actions

### 5. Performance

- Minimize checkout depth when possible
- Only fetch what's needed
- Consider caching if applicable

## Clarification Questions - ANSWERED

All primary questions have been answered. Here's the summary:

### ✅ Answered Questions

1. **Versioning Behavior**: ✅ Keep patch increment as default, add `update-mode` input for 'match' option
2. **Action Name**: ✅ `citation-sync-action`
3. **Tag Strategy**: ✅ Keep increment behavior as default, add 'match' mode that updates without incrementing
4. **Branch Target**: ✅ Auto-detect default branch when target-branch is empty
5. **License**: ✅ MIT
6. **Author/Publisher**: ✅ Adamtaranto
7. **Additional Features**: ✅ Add CITATION.cff validation (see validation strategy section)
8. **Backward Compatibility**: ✅ Not required

### 🆕 Additional Requirements Confirmed

1. **Version Tag Format**:
   - Default format: `v0.0.0` (v + major.minor.patch)
   - Add `version-prefix` input (default: 'v')
   - Add `version-format` input (default regex for X.Y.Z)
   - Raise error if latest tag doesn't match specified format

2. **CITATION.cff Field Validation**:
   - Verify `version` field appears exactly once
   - Verify `date-released` field appears exactly once
   - Raise error if duplicates found

### ✅ All Questions Answered - Final Design

All clarification questions have been answered:

1. **Update Mode Behavior**: ✅ **Exit successfully with `needs-update: false`** when no update needed in 'match' mode
   - This is the most user-friendly approach
   - Allows workflows to continue normally
   - Users can check the output if they need to know

2. **Version Format Validation**: ✅ **Support pre-release versions**
   - Accept versions like `v1.0.0-beta.1`, `v2.0.0-rc.2`, `v1.0.0-alpha`
   - Updated default regex to: `^([0-9]+)\.([0-9]+)\.([0-9]+)(-[a-zA-Z0-9.-]+)?$`
   - Captures: major.minor.patch and optional pre-release suffix
   - Non-semantic versions (e.g., `v1.0`) still fail with clear error

3. **Tag Prefix Edge Cases**: ✅ **Support multi-character prefixes**
   - Empty prefix (`version-prefix: ''`) supported for tags like `1.0.0`
   - Single character (`v`, `r`) supported
   - Multi-character (`release-`, `ver-`, `version-`) supported
   - Implementation: Simply strip prefix from tag, validate remaining portion

4. **CITATION.cff Validation**: ✅ **Use best practices approach**
   - **Before update**: Validate structure (hard error if fails)
     - YAML syntax check
     - Field uniqueness check
     - Required fields check
     - If fails: Exit with error, don't proceed with update
   - **After update**: Validate changes (hard error with rollback if fails)
     - Verify version and date fields updated correctly
     - YAML syntax still valid
     - If fails: Restore backup, exit with error
   - **Rationale**: Fail fast before update, protect against our own bugs after update

5. **Default Branch Detection**: ✅ **Use git symbolic-ref with fallback**
   - **Primary method**: `git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'`
     - Fast, local, no API calls
     - Works in checkout with fetch-depth: 0
   - **Fallback 1**: Parse `git remote show origin` for default branch
   - **Fallback 2**: Use 'main' as last resort
   - **No GitHub API**: Avoids rate limits and token permission issues

6. **Error Messages**: ✅ **Use GitHub Actions annotations with suggestions**
   - Use `::error::` for failures
   - Use `::warning::` for non-critical issues
   - Use `::notice::` for informational messages
   - Include actionable suggestions in error messages
   - Example: `::error::Invalid version format '1.0'. Expected semantic version like '1.0.0'. Check your version-format input.`

7. **Tag Message**: ✅ **Keep simple for now, make configurable later**
   - Default message: `"Release {version}"`
   - Not configurable in v1.0.0 (keep scope manageable)
   - Add `tag-message` input in future version if requested
   - Document as future enhancement in README

## Success Criteria

The action will be considered successful when:

1. ✅ Published to GitHub Marketplace
2. ✅ Can be used with simple `uses: Adamtaranto/citation-sync-action@v1` syntax
3. ✅ Comprehensive documentation
4. ✅ Automated tests passing
5. ✅ Clear error messages for common issues
6. ✅ Works with default GitHub token (no special permissions needed beyond contents: write)
7. ✅ Supports both 'increment' and 'match' update modes
8. ✅ Validates version tag format and CITATION.cff structure
9. ✅ Auto-detects default branch
10. ✅ Handles custom version prefixes and formats

## Timeline Estimate

- **Phase 1** (Repository Setup): 2-3 hours
  - Create LICENSE (MIT)
  - Create community health files
  - Configure repository settings
  
- **Phase 2** (Action Implementation): 6-8 hours
  - Create action.yml with all inputs/outputs
  - Implement branch detection logic
  - Implement version format validation
  - Implement update-mode logic (increment vs match)
  - Implement CITATION.cff field validation
  - Convert all workflow steps to composite action
  
- **Phase 3** (Testing): 4-5 hours
  - Create comprehensive test suite
  - Test all input combinations
  - Test edge cases and error conditions
  
- **Phase 4** (Documentation): 4-5 hours
  - Write comprehensive README
  - Document all inputs/outputs with examples
  - Create troubleshooting guide
  - Write contributing guidelines
  
- **Phase 5** (Publishing): 1-2 hours
  - Prepare for marketplace publication
  - Create initial release
  
- **Phase 6** (Release Management): 2-3 hours
  - Set up release automation
  - Create changelog
  - Document versioning strategy

**Total**: ~19-26 hours of focused development work

## Next Steps After Final Clarifications

Once the outstanding questions are answered:

1. ✅ Create MIT LICENSE file
2. ✅ Create action.yml with confirmed structure
3. ✅ Implement version tag validation with configurable format
4. ✅ Implement CITATION.cff field uniqueness checks
5. ✅ Implement update-mode logic (increment/match)
6. ✅ Implement branch auto-detection
7. ✅ Implement validation strategy (basic level)
8. ✅ Create comprehensive README.md
9. ✅ Create test workflow
10. ✅ Test thoroughly with all scenarios
11. ✅ Publish to GitHub Marketplace
12. ✅ Create v1.0.0 release
