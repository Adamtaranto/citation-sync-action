# CITATION.cff Tag Sync Action - Overview

## Purpose

This GitHub Action automatically synchronizes CITATION.cff metadata with Git tags in your repository. When a new version tag is pushed to your repository, the action checks if the CITATION.cff file contains the correct version number and release date. If the CITATION.cff is out of sync, the action updates it and creates a new incremented tag with the corrected citation metadata.

## Problem Being Solved

When maintaining scientific software or research code that uses CITATION.cff files (Citation File Format), keeping version numbers and release dates synchronized between Git tags and the citation file is manual and error-prone. Developers may forget to update CITATION.cff when creating a new release, leading to:

- Incorrect version numbers in citations
- Outdated release dates in citation metadata
- Manual effort to maintain consistency between tags and citation files

This action automates this synchronization process, ensuring that citation metadata always reflects the actual release information.

## How It Works

### Trigger

The workflow is triggered when any tag is pushed to the repository:
```yaml
on:
  push:
    tags:
      - '*'
```

### Process Flow

1. **Initial Checks**
   - Skips execution if the last commit was made by the action itself (prevents infinite loops)
   - Validates that a CITATION.cff file exists in the repository root

2. **Extract Tag Information**
   - Captures the tag name (e.g., `v1.0.0`)
   - Extracts the tag creation date in YYYY-MM-DD format
   - Removes any 'v' prefix to get the semantic version number (e.g., `1.0.0`)
   - Calculates an incremented version (patch version +1) for potential updates
   - Preserves any 'v' prefix in the incremented tag name

3. **Parse Current CITATION.cff**
   - Extracts the current `version` field from CITATION.cff
   - Extracts the current `date-released` field from CITATION.cff

4. **Compare and Determine Update Need**
   - Compares the tag version with the CITATION.cff version
   - Compares the tag date with the CITATION.cff release date
   - If there's a mismatch:
     - Checks if the incremented tag already exists (to prevent conflicts)
     - Sets a flag indicating an update is needed
     - Creates a summary of changes to be made

5. **Update CITATION.cff** (if needed)
   - Updates the `version` field to the **incremented version** (not the triggering tag version)
   - Updates the `date-released` field to the tag's creation date
   - Creates a backup before updating
   - Validates that the update was successful
   - Shows a diff of the changes

6. **Commit and Create New Tag**
   - Configures Git with the github-actions bot identity
   - Commits the updated CITATION.cff with a descriptive message including `[skip ci]`
   - Creates a new annotated tag with the incremented version
   - Pushes the commit to the main branch
   - Pushes the new incremented tag

7. **Generate Summary**
   - Creates a workflow summary showing what was done
   - Reports whether an update was applied or if everything was already in sync

## Key Features

### Version Incrementing

The action doesn't simply update CITATION.cff to match the triggering tag. Instead:
- It increments the patch version (third number in semantic versioning)
- Creates a new tag with this incremented version
- This ensures the original tag remains as a "checkpoint" and the new tag contains the corrected metadata

**Example:**
- User pushes tag `v1.2.3`
- CITATION.cff has version `1.2.2` and old date
- Action updates CITATION.cff to version `1.2.4` (not 1.2.3)
- Action creates new tag `v1.2.4` with the updated file

### Safety Mechanisms

1. **Loop Prevention**: Skips execution if triggered by its own commits
2. **Conflict Detection**: Fails if the incremented tag already exists
3. **Validation**: Verifies CITATION.cff exists before proceeding
4. **Backup and Verify**: Creates backup before updates and validates changes
5. **Skip CI**: Includes `[skip ci]` in commit messages to prevent unnecessary CI runs

### Semantic Versioning Support

- Handles versions with or without 'v' prefix (`v1.0.0` or `1.0.0`)
- Preserves the prefix style in generated tags
- Validates semantic version format (major.minor.patch)

### Flexible Authentication

Uses either a custom Personal Access Token (PAT) or the default GITHUB_TOKEN:
```yaml
token: ${{ secrets.CITATION_UPDATE_PAT || secrets.GITHUB_TOKEN }}
```

This allows for better permissions if needed while maintaining a fallback.

## Configuration Options

### Environment Variables

- **DEBUG**: Set to `true` to see additional debug output during execution

### Required Permissions

```yaml
permissions:
  contents: write
```

The action needs write access to:
- Commit changes to CITATION.cff
- Create and push tags
- Push to the main branch

### Secrets

- **CITATION_UPDATE_PAT** (optional): A Personal Access Token with repository write permissions
- **GITHUB_TOKEN** (automatic): The default GitHub Actions token (fallback)

## Workflow Output

The action provides detailed output including:
- Notices about tag processing and version calculations
- Comparison of current vs. expected values
- Diffs showing changes to CITATION.cff
- Success confirmation with new tag information
- Job summary with update details or confirmation that no update was needed

## Limitations and Considerations

1. **Semantic Versioning Only**: Requires semantic versioning format (major.minor.patch)
2. **Main Branch**: Currently hardcoded to push to the `main` branch
3. **File Location**: CITATION.cff must be in the repository root
4. **Field Requirements**: Assumes CITATION.cff has `version` and `date-released` fields
5. **Incrementation Logic**: Always increments patch version by 1
6. **Tag Conflicts**: Fails if the calculated incremented tag already exists

## Use Cases

This action is ideal for:
- Research software repositories that use CITATION.cff
- Projects where reproducible citations are critical
- Teams that want to automate citation metadata maintenance
- Repositories with frequent releases requiring citation updates
