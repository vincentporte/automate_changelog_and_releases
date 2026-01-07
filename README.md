# Automated Changelog and Releases

This repository demonstrates how to automate changelog generation and releases using [git-cliff](https://git-cliff.org/) and GitHub Actions. The setup provides a complete workflow for managing semantic versioning, changelog generation, and automated releases.

## Overview

The automation consists of three main components:

1. **git-cliff** - A changelog generator that parses commit messages and PR labels
2. **GitHub Actions workflows** - Automated workflows for changelog generation and release creation
3. **Label-based categorization** - PR labels that determine changelog sections

## Setup

### 1. Install git-cliff

Add git-cliff to your development dependencies:

```toml
[dependency-groups]
dev = [
    "git-cliff>=2.11.0",
]
```

Or install globally:
```bash
# Using cargo
cargo install git-cliff

# Using homebrew (macOS)
brew install git-cliff

# Using pip
pip install git-cliff
```

### 2. Configure git-cliff

Create a `cliff.toml` configuration file in your repository root. The configuration includes:

- **Changelog format** - Header, body, and footer templates
- **Commit parsing** - Rules for categorizing commits based on PR labels
- **Repository settings** - GitHub owner and repo information

Key configuration sections:
- `[changelog]` - Defines the changelog format and templates
- `[git]` - Commit parsing and preprocessing rules
- `[remote.github]` - Repository information for generating links

### 3. GitHub Actions Workflows

The automation uses three GitHub Actions workflows:

#### Label Checks (`check_labels.yml`)
- **Trigger**: On PR events (opened, labeled, etc.)
- **Purpose**: Ensures every PR has exactly one changelog label
- **Required labels**: `added`, `modified`, `removed`, `dependencies`, `documentation`

#### Changelog PR Generation (`create_pr_for_changelog.yml`)
- **Trigger**: Push to main branch or manual dispatch
- **Purpose**: Generates changelog and creates a PR for the next release
- **Actions**:
  1. Generates full changelog using git-cliff
  2. Creates a PR with updated `CHANGELOG.md`
  3. Labels the PR as `pending-release` and `automated`

#### Release Creation (`create_release.yml`)
- **Trigger**: When changelog PR is merged or manual dispatch
- **Purpose**: Creates git tags and GitHub releases
- **Actions**:
  1. Determines next version using git-cliff
  2. Creates and pushes a git tag
  3. Creates a GitHub release with changelog content

## Usage

### Daily Workflow

1. **Create PRs with proper labels**:
   - Add one of the required labels: `breaking`, `added`, `modified`, `removed`, `bugfix`, `dependencies`, `documentation`
   - The label determines which section the change appears in the changelog

2. **Merge PRs to main**:
   - After merging, the changelog workflow automatically runs
   - A new PR is created with the updated changelog
   - When using `squash and merge`, PR id in commit message is converted into a link to the PR in the changelog file

3. **Review and merge changelog PR**:
   - Review the generated changelog
   - Merge the changelog PR to trigger release creation
   - A new tag and GitHub release are automatically created

### Manual Release

You can manually trigger a release using GitHub's workflow dispatch:

1. Go to Actions → "Create tag and release"
2. Click "Run workflow"
3. The workflow will create a tag and release based on current changes

### Local Development

Generate changelog locally:

```bash
# Generate full changelog
git cliff --output CHANGELOG.md

# Preview next version
git cliff --latest --bump --unreleased

# Generate changelog for specific version
git cliff --tag v1.0.0
```

## Changelog Categories

Changes are automatically categorized based on PR labels:

- **Breaking Changes** (`breaking` label) - Changes that are not backward compatible
- **Added** (`added` label) - New features
- **Modified** (`modified` label) - Changes to existing functionality  
- **Removed** (`removed` label) - Removed features
- **Bug Fixes** (`bugfix` label) - Fix a bug
- **Dependencies** (`dependencies` label) - Dependency updates
- **Documentation** (`documentation` label) - Documentation changes

## Version Bump Rules

By default, git-cliff performs patch version bumps (e.g., 1.0.0 → 1.0.1). To control major and minor version bumps, you can configure bump rules in your `cliff.toml`:

To implement version control through PR labels, add these labels to your repository:

- **`breaking`** - Major version bump (1.0.0 → 2.0.0)
  - Breaking changes that are not backward compatible
  - API changes that require user code updates
  
- **`added`** or **`modified`** or `removed`** - Minor version bump (1.0.0 → 1.1.0)
  - Backward compatible changes
  
- **`bugfix`** - Patch version bump (1.0.0 → 1.0.1)
  - Bug fixes and _very_ small improvements

> As long as major version is 0, `breaking` will bump version from 0.1.0 to 0.2.0, and `added`|`modified`|`removed` will bump version from 0.1.0 to 0.1.1

## Configuration Customization

### Modify Changelog Format

Edit the templates in `cliff.toml`:
- `header` - Changelog header
- `body` - Format for each release section
- `footer` - Links to releases and comparisons

### Add Custom Labels

Update both `cliff.toml` and `.github/workflows/check_labels.yml`:

1. Add new commit parser in `cliff.toml`:
```toml
{ field = "github.pr_labels", pattern = "bugfix", group = "<!-- 4 --> Fixed" },
```

2. Add label to required list in `check_labels.yml`:
```yaml
labels: |
  added
  modified
  removed
  dependencies
  documentation
  bugfix
```

### Repository Settings

Update the repository information in `cliff.toml`:
```toml
[remote.github]
owner = "your-username"
repo = "your-repo-name"
```

## Troubleshooting

### Common Issues

1. **Missing labels**: Ensure PRs have exactly one changelog label
2. **Workflow permissions**: Verify workflows have `contents: write` and `pull-requests: write` permissions
3. **Git configuration**: Workflows use `github-actions[bot]` for commits and tags

### Debug Mode

Enable verbose logging in workflows:
```yaml
args: -vv --latest --bump --unreleased
```

## Benefits

- **Automated changelog generation** - No manual changelog maintenance
- **Consistent formatting** - Standardized changelog format
- **Semantic versioning** - Automatic version bumping
- **GitHub integration** - Seamless integration with GitHub releases
- **PR-based workflow** - Review changes before release
- **Flexible categorization** - Label-based change categorization
