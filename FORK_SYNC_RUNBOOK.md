# Fork Sync Runbook

This document describes the branch synchronization workflow for this fork of [THUDM/slime](https://github.com/THUDM/slime).

## Branch Structure

### `main`
- Contains fork-specific files, including [.github/workflows/sync-upstream.yml](.github/workflows/sync-upstream.yml)
- Automatically synced with upstream daily at 00:00 UTC
- Not used for day-to-day development work

### `upstream_sync`
- Clean replica of upstream (no fork-specific workflow files)
- Automatically synced with upstream daily at 00:00 UTC
- Replicated to `GymPod/slime` repository
- Base branch for all feature branches

### Feature branches
- Created from `upstream_sync` for all development work
- Used to create pull requests to upstream
- Merged to `upstream_sync` when changes are needed immediately

## Automatic Synchronization

Both `main` and `upstream_sync` branches are automatically synced with upstream daily via GitHub Actions:
- **Schedule**: Daily at 00:00 UTC
- **Workflow**: [.github/workflows/sync-upstream.yml](.github/workflows/sync-upstream.yml)
- **Source**: [https://github.com/THUDM/slime](https://github.com/THUDM/slime)
- **Manual trigger**: Can be triggered manually from GitHub Actions tab

If merge conflicts occur, the workflow will fail and require manual intervention.

## Development Workflow

### Standard workflow
1. Create a feature branch from `upstream_sync`:
   ```bash
   git checkout upstream_sync
   git pull origin upstream_sync
   git checkout -b feature/your-feature-name
   ```

2. Make and test your changes:
   ```bash
   # make your changes
   git add .
   git commit -m "Your changes"
   git push origin feature/your-feature-name
   ```

3. Create a pull request to upstream from your feature branch

4. Wait for PR to merge - changes will automatically sync back to both `main` and `upstream_sync`

### Fast-track workflow (when you need changes immediately)
If you have a PR to upstream but need the changes available in `GymPod/slime` before it merges:

1. Create and push your feature branch as above

2. Merge feature branch to `upstream_sync`:
   ```bash
   git checkout upstream_sync
   git merge feature/your-feature-name
   git push origin upstream_sync
   ```

3. Push `upstream_sync` to `GymPod/slime`:
   Currently set up as a Claude Code hook
   [TODO] Add detailed instructions

4. Continue with your PR to upstream as normal

5. When your PR merges to upstream, it will automatically sync back to both branches

## Contributing Changes to Upstream

All pull requests to upstream should be created from feature branches, not from `main` or `upstream_sync`.

**Example:**
```bash
# Create feature branch
git checkout upstream_sync
git checkout -b feature/add-new-feature

# Make changes and push
git add .
git commit -m "Add new feature"
git push origin feature/add-new-feature

# Create PR to THUDM/slime from feature/add-new-feature

# If needed immediately in GymPod/slime:
git checkout upstream_sync
git merge feature/add-new-feature
git push origin upstream_sync
```

## Troubleshooting

### Merge conflicts during automatic sync
1. Check the failed workflow run in GitHub Actions
2. Manually resolve conflicts:
   ```bash
   git fetch upstream
   git checkout main  # or upstream_sync
   git merge upstream/main
   # resolve conflicts
   git add .
   git commit
   git push origin main  # or upstream_sync
   ```

### Resetting branches to match upstream
If branches diverge significantly and you want to reset to match upstream:

```bash
# For main (will lose fork-specific changes except workflow file)
git checkout main
git fetch upstream
git reset --hard upstream/main
git checkout HEAD -- .github/workflows/sync-upstream.yml
git push --force origin main

# For upstream_sync (complete reset)
git checkout upstream_sync
git fetch upstream
git reset --hard upstream/main
git push --force origin upstream_sync
```

### Cleaning up old feature branches
After your PR merges to upstream:
```bash
# Delete local branch
git branch -d feature/your-feature-name

# Delete remote branch
git push origin --delete feature/your-feature-name
```

## Remotes Configuration

Expected git remotes:
- `origin`: Your fork (GymPod/slime-public-fork)
- `upstream`: Original repository (THUDM/slime)

Verify with:
```bash
git remote -v
```

Add missing remotes:
```bash
# Add upstream if not present
git remote add upstream https://github.com/THUDM/slime.git
```
