# Citation Sync Action

[![GitHub release](https://img.shields.io/github/v/release/Adamtaranto/citation-sync-action)](https://github.com/Adamtaranto/citation-sync-action/releases)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Citation%20Sync%20Action-blue.svg)](https://github.com/marketplace/actions/citation-sync-action)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Automatically synchronize your `CITATION.cff` file version and date with Git tags. Keep your citation metadata accurate and up-to-date without manual effort!

## Features

- 🏷️ **Automatic Synchronization**: Updates `CITATION.cff` when you push version tags
- 🔀 **Two Update Modes**:
  - `increment`: Creates new incremented tag with updated citation (default)
  - `match`: Updates citation to match current tag without creating new tag
- 🎯 **Flexible Version Formats**: Supports custom prefixes (`v`, `release-`, etc.) and pre-release versions
- 🌿 **Smart Branch Detection**: Auto-detects your default branch
- ✅ **Validation**: Validates `CITATION.cff` structure before and after updates
- 🔄 **Automatic Rollback**: Restores original file if validation fails
- 📝 **Clear Error Messages**: Provides actionable suggestions when issues occur
- 🎨 **Customizable**: Configure commit messages, version formats, and more

## Quick Start

### Basic Usage

Add this workflow to `.github/workflows/citation-sync.yml`:

```yaml
name: Update CITATION.cff on Tag
on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  update-citation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: Adamtaranto/citation-sync-action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

That's it! Now whenever you push a version tag like `v1.0.0`, the action will:

1. Check if `CITATION.cff` needs updating
2. Update version and date-released fields
3. Create a new incremented tag (e.g., `v1.0.1`) with the changes

## Usage

### Update Modes

#### Increment Mode (Default)

Creates a new incremented version tag with updated citation:

```yaml
- uses: Adamtaranto/citation-sync-action@v1
  with:
    update-mode: increment  # default
```

**Example Flow**:

- You push tag `v1.0.0`
- `CITATION.cff` has version `0.9.0`
- Action updates to version `1.0.1` and creates new tag `v1.0.1`
- Original tag `v1.0.0` remains unchanged

#### Match Mode

Updates citation to match the current tag without incrementing:

```yaml
- uses: Adamtaranto/citation-sync-action@v1
  with:
    update-mode: match
```

**Example Flow**:

- You push tag `v1.0.0`
- `CITATION.cff` has version `0.9.0`
- Action updates to version `1.0.0`
- No new tag is created

### Advanced Configuration

```yaml
- uses: Adamtaranto/citation-sync-action@v1
  with:
    # GitHub token (default: github.token)
    token: ${{ secrets.GITHUB_TOKEN }}

    # Path to CITATION.cff (default: 'CITATION.cff')
    citation-path: 'CITATION.cff'

    # Target branch (default: auto-detect)
    target-branch: 'main'

    # Update mode: 'increment' or 'match' (default: 'increment')
    update-mode: 'increment'

    # Version tag prefix (default: 'v')
    version-prefix: 'v'

    # Version format regex (default: semantic versioning with optional pre-release)
    version-format: '^([0-9]+)\.([0-9]+)\.([0-9]+)(-[a-zA-Z0-9.-]+)?$'

    # Enable debug output (default: 'false')
    enable-debug: 'false'

    # Commit message template (default shown, use {version} placeholder)
    commit-message: 'chore: Update CITATION.cff to version {version}'

    # Git user configuration
    git-user-name: 'github-actions[bot]'
    git-user-email: 'github-actions[bot]@users.noreply.github.com'

    # Add [skip ci] to commit (default: 'true')
    skip-ci: 'true'

    # Fail if incremented tag exists (default: 'true')
    fail-on-conflict: 'true'

    # Validate CITATION.cff format (default: 'true')
    validate-cff: 'true'
```

### Custom Version Prefixes

Support for different version tag formats:

```yaml
# Standard 'v' prefix (default)
- uses: Adamtaranto/citation-sync-action@v1
  with:
    version-prefix: 'v'  # Tags: v1.0.0, v2.0.0

# Release prefix
- uses: Adamtaranto/citation-sync-action@v1
  with:
    version-prefix: 'release-'  # Tags: release-1.0.0

# No prefix
- uses: Adamtaranto/citation-sync-action@v1
  with:
    version-prefix: ''  # Tags: 1.0.0, 2.0.0
```

### Pre-Release Versions

Pre-release versions are fully supported:

```yaml
on:
  push:
    tags:
      - 'v*'

jobs:
  update-citation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: Adamtaranto/citation-sync-action@v1
```

**Example**:

- Tag: `v1.0.0-beta.1` → CITATION.cff version: `1.0.0-beta.1`
- Tag: `v2.0.0-rc.2` → CITATION.cff version: `2.0.0-rc.2`
- When incrementing from `v1.0.0-beta`, new version is `1.0.1` (pre-release suffix dropped)

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `token` | GitHub token for authentication | No | `${{ github.token }}` |
| `citation-path` | Path to CITATION.cff file | No | `CITATION.cff` |
| `target-branch` | Branch to push updates to (empty = auto-detect) | No | `''` |
| `update-mode` | Update mode: `increment` or `match` | No | `increment` |
| `version-prefix` | Expected version tag prefix | No | `v` |
| `version-format` | Semantic version regex pattern | No | `^([0-9]+)\.([0-9]+)\.([0-9]+)(-[a-zA-Z0-9.-]+)?$` |
| `enable-debug` | Enable debug output | No | `false` |
| `commit-message` | Commit message template (use `{version}`) | No | `chore: Update CITATION.cff to version {version}` |
| `git-user-name` | Git user name for commits | No | `github-actions[bot]` |
| `git-user-email` | Git user email for commits | No | `github-actions[bot]@users.noreply.github.com` |
| `skip-ci` | Add [skip ci] to commit message | No | `true` |
| `fail-on-conflict` | Fail if incremented tag already exists | No | `true` |
| `validate-cff` | Validate CITATION.cff format | No | `true` |
| `use-pull-request` | Create PR instead of direct push (for protected branches) | No | `false` |
| `pr-branch-prefix` | Prefix for PR branch names | No | `citation-sync-` |
| `pr-title` | PR title template (use `{version}`) | No | `Update CITATION.cff to version {version}` |
| `pr-body` | PR body template (use `{version}`, `{date}`, `{original_tag}`) | No | `This PR updates CITATION.cff based on tag {original_tag}.` |

## Outputs

| Output | Description |
|--------|-------------|
| `needs-update` | Whether CITATION.cff needed updating (`true`/`false`) |
| `original-tag` | The tag that triggered this action |
| `new-tag` | The new tag created (increment mode only) |
| `new-version` | The new version written to CITATION.cff |
| `commit-sha` | The commit SHA with updated CITATION.cff |
| `target-branch` | The branch that was updated |
| `pull-request-number` | The PR number (if `use-pull-request: true`) |
| `pull-request-url` | The PR URL (if `use-pull-request: true`) |

### Using Outputs

```yaml
- uses: Adamtaranto/citation-sync-action@v1
  id: citation-sync
  with:
    token: ${{ secrets.GITHUB_TOKEN }}

- name: Check if update was needed
  if: steps.citation-sync.outputs.needs-update == 'true'
  run: |
    echo "Updated to version: ${{ steps.citation-sync.outputs.new-version }}"
    echo "New commit: ${{ steps.citation-sync.outputs.commit-sha }}"
```

## Requirements

### Permissions

The action requires `contents: write` permission to:

- Commit changes to CITATION.cff
- Create and push tags (increment mode)
- Push to the default branch

```yaml
permissions:
  contents: write
```

If using `use-pull-request: true`, also add `pull-requests: write`:

```yaml
permissions:
  contents: write
  pull-requests: write  # Required for creating pull requests
```

### Branch Protection

**⚠️ Important**: This action requires direct push access to your target branch (usually `main`).

#### If Your Branch is Protected

If your default branch has protection rules enabled (requires PR reviews, status checks, etc.), you have several options:

##### Option 1: Use Pull Request Mode (Recommended)

Enable PR mode to create a pull request instead of pushing directly:

```yaml
- uses: Adamtaranto/citation-sync-action@v1
  with:
    use-pull-request: true
    token: ${{ secrets.GITHUB_TOKEN }}
```

The action will:

- Create a new branch with the citation updates
- Open a pull request to your default branch
- You can then review and merge through your normal PR workflow

⚠️ **Note**: In `increment` mode with `use-pull-request: true`, you'll need to manually create the new version tag after merging the PR.

##### Option 2: Use a Different Branch

Configure the action to push to a non-protected branch:

```yaml
- uses: Adamtaranto/citation-sync-action@v1
  with:
    target-branch: citation-updates
```

##### Option 3: Use Admin Token

Use a Personal Access Token (PAT) with admin privileges or bypass permissions:

```yaml
- uses: Adamtaranto/citation-sync-action@v1
  with:
    token: ${{ secrets.ADMIN_PAT }}
```

##### Option 4: Configure Branch Protection Bypass

In your repository settings, configure branch protection to allow this workflow to bypass restrictions:

- Go to Settings → Branches → Branch protection rules
- Edit your protection rule
- Under "Allow specified actors to bypass required pull requests", add the GitHub Actions app

#### Branch Protection Detection

The action automatically detects if your target branch is protected and will:

- ✅ Proceed normally if `use-pull-request: true`
- ❌ Fail with a helpful error message if `use-pull-request: false` (with solutions)
- ⚠️ Warn if protection status cannot be determined

### Checkout Configuration

You must checkout with sufficient history to access tag information:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0  # Fetch all history for all tags
```

### CITATION.cff Requirements

Your `CITATION.cff` must:

1. Be valid YAML
2. Have exactly one `version` field
3. Have exactly one `date-released` field
4. Include required CFF fields: `cff-version`, `message`, `title`, `authors`

## How It Works

For a detailed explanation of the action's behavior, see [OVERVIEW.md](OVERVIEW.md).

**Quick Summary**:

1. **Validation**: Checks CITATION.cff exists and is valid
2. **Tag Parsing**: Extracts version from tag using configured prefix/format
3. **Branch Detection**: Auto-detects default branch if not specified
4. **Comparison**: Checks if version and date need updating
5. **Update**: Creates backup, updates fields, validates changes
6. **Commit**: Commits changes with configured message
7. **Tag/Push**: Creates new tag (increment mode) or pushes commit (match mode)
8. **Rollback**: Automatically restores backup if validation fails

## Examples

### Example 1: Standard Workflow

```yaml
name: Sync CITATION.cff
on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: Adamtaranto/citation-sync-action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

### Example 2: Match Mode with Custom Commit Message

```yaml
name: Update Citation Metadata
on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: Adamtaranto/citation-sync-action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          update-mode: match
          commit-message: 'docs: Update CITATION.cff for release {version}'
```

### Example 3: Protected Branch with Pull Request Mode

```yaml
name: Update Citation via PR
on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write
  pull-requests: write  # Required for creating PRs

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: Adamtaranto/citation-sync-action@v1
        id: sync
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          use-pull-request: true
          pr-title: 'chore: Update CITATION.cff to {version}'
          pr-body: |
            Automated citation update triggered by tag {original_tag}.

            This PR updates the CITATION.cff file with:
            - Version: {version}
            - Date: {date}

            Please review and merge.

      - name: Comment on PR
        if: steps.sync.outputs.pull-request-number != ''
        run: |
          echo "Created PR #${{ steps.sync.outputs.pull-request-number }}"
          echo "URL: ${{ steps.sync.outputs.pull-request-url }}"
```

### Example 4: Custom Prefix and Branch

```yaml
name: Citation Sync with Custom Settings
on:
  push:
    tags:
      - 'release-*'

permissions:
  contents: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: Adamtaranto/citation-sync-action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          version-prefix: 'release-'
          target-branch: 'develop'
```

### Example 5: With Conditional Actions

```yaml
name: Citation Sync with Notifications
on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: Adamtaranto/citation-sync-action@v1
        id: sync
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Notify on update
        if: steps.sync.outputs.needs-update == 'true'
        run: |
          echo "::notice::Created new version ${{ steps.sync.outputs.new-version }}"
          # Add your notification logic here (Slack, email, etc.)
```

## Troubleshooting

### Common Issues

#### "CITATION.cff file not found"

**Cause**: The file doesn't exist at the specified path.

**Solution**: Ensure `CITATION.cff` exists in your repository root, or specify the correct path:

```yaml
with:
  citation-path: 'path/to/CITATION.cff'
```

#### "Tag does not match expected format"

**Cause**: Your tag doesn't follow the expected version format.

**Solution**: Check your `version-prefix` and `version-format` inputs match your tags:

```yaml
with:
  version-prefix: 'v'  # or 'release-', '', etc.
  version-format: '^([0-9]+)\.([0-9]+)\.([0-9]+)(-[a-zA-Z0-9.-]+)?$'
```

#### "Incremented tag already exists"

**Cause**: In increment mode, the next version tag already exists.

**Solution**: Either:

- Use `update-mode: match` to update without creating a new tag
- Set `fail-on-conflict: false` to proceed anyway
- Manually resolve the tag conflict

#### "CITATION.cff must have exactly one 'version' field"

**Cause**: Duplicate or missing `version` field in CITATION.cff.

**Solution**: Ensure your CITATION.cff has exactly one `version:` line.

### Debug Mode

Enable debug output for troubleshooting:

```yaml
- uses: Adamtaranto/citation-sync-action@v1
  with:
    enable-debug: 'true'
```

### Getting Help

If you encounter issues:

1. Check the [troubleshooting section](#troubleshooting) above
2. Review the [action logs](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/using-workflow-run-logs) with debug mode enabled
3. Search [existing issues](https://github.com/Adamtaranto/citation-sync-action/issues)
4. [Open a new issue](https://github.com/Adamtaranto/citation-sync-action/issues/new) with:
   - Your workflow file
   - Error messages
   - Repository details (if public)

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Citation

If you use this action in your research software, you can cite it as:

```bibtex
@software{citation_sync_action,
  author = {Taranto, Adam},
  title = {Citation Sync Action},
  year = {2025},
  url = {https://github.com/Adamtaranto/citation-sync-action}
}
```

## Related

- [Citation File Format (CFF)](https://citation-file-format.github.io/)
- [CFF Validator](https://github.com/citation-file-format/cff-validator-python)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## Acknowledgments

This action was developed to simplify citation management for research software projects. Special thanks to the Citation File Format community for creating and maintaining the CFF specification.

---
