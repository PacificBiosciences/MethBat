# GitHub Actions Workflows

This directory contains GitHub Actions workflows for automated CI/CD processes.

## Release Workflows

### `simple-release.yml` - Update Release with Changelog
This workflow updates an existing release with changelog notes:

**Triggers:** When a tag matching `v*` is pushed

**What it does:**
1. Finds the existing GitHub release for the tag
2. Extracts changelog notes from `CHANGELOG.md`
3. Updates the release description with the latest changelog content

## How to Use

### Creating/Updating a Release

1. **Update your CHANGELOG.md** with the new version's changes
2. **Commit and push** your changes
3. **Create and push a tag:**
   ```bash
   git tag v0.8.1
   git push origin v0.8.1
   ```
4. **The workflow will automatically:**
   - Find the existing release for that tag
   - Extract the changelog section for that version
   - Update the release description with the latest changelog content

**Note:** You can push the same tag multiple times to update the release description with the latest changelog content.

### Changelog Format

The workflow expects your `CHANGELOG.md` to follow this format:
```markdown
# v0.8.1
## Changes
- New feature 1
- New feature 2

## Fixed
- Bug fix 1

# v0.8.0
## Changes
- Previous version changes...
```

The workflow will extract everything from the version header to the next version header (or end of file).

## Requirements

- `GITHUB_TOKEN` secret (automatically provided by GitHub)
- Your `CHANGELOG.md` should follow the expected format
- Tags should follow semantic versioning (e.g., `v0.8.1`)
- The release must already exist (you'll need to create it manually the first time)

## Notes

- The workflow automatically detects the version from the tag name
- It strips the `v` prefix when looking for changelog sections
- If no changelog is found for a version, it will note this in the release
- This workflow updates existing releases rather than creating new ones
- You can push the same tag multiple times to refresh the release content
