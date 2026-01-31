# Deployment Guide

This repository includes a GitHub Actions workflow for automated deployment to AUR and creating GitHub releases.

## Workflow: Deploy to AUR and Create Release

### Location
`.github/workflows/deploy.yml`

### Purpose
This workflow automates the process of:
1. Bumping the version in the `hyproled` script
2. Creating a Git tag
3. Creating a GitHub release
4. Deploying the update to the AUR (Arch User Repository)

### How to Use

1. Go to the **Actions** tab in the GitHub repository
2. Select the **Deploy to AUR and Create Release** workflow
3. Click **Run workflow**
4. Choose the version bump type:
   - **patch**: Increment the patch version (0.1.3 → 0.1.4)
   - **minor**: Increment the minor version (0.1.3 → 0.2.0)
   - **major**: Increment the major version (0.1.3 → 1.0.0)
5. Click **Run workflow** to start the deployment

### Required Secrets

For the workflow to function properly, you need to configure the following secret in your GitHub repository:

#### AUR_SSH_KEY

This is the SSH private key used to authenticate with the AUR repository.

**Setup Instructions:**

1. Generate an SSH key pair (if you don't have one):
   ```bash
   ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/aur
   ```

2. Add the public key to your AUR account:
   - Log in to https://aur.archlinux.org
   - Go to **My Account** → **SSH Public Keys**
   - Add the contents of `~/.ssh/aur.pub`

3. Add the private key as a GitHub secret:
   - Go to your repository's **Settings** → **Secrets and variables** → **Actions**
   - Click **New repository secret**
   - Name: `AUR_SSH_KEY`
   - Value: Paste the contents of `~/.ssh/aur` (the private key)

### What the Workflow Does

1. **Version Bump**: Updates the version number in the `hyproled` script based on your selection
2. **Git Tag**: Creates and pushes a new Git tag (e.g., `v0.1.4`)
3. **GitHub Release**: Creates a new GitHub release with:
   - Release notes
   - Installation instructions
   - Links to AUR package
4. **AUR Deployment**: 
   - Clones the AUR repository
   - Updates the `PKGBUILD` file (increments `pkgrel`)
   - Updates `.SRCINFO`
   - Pushes changes to AUR

### Manual Deployment (Alternative)

If you prefer to deploy manually:

1. Update version in `hyproled`:
   ```bash
   sed -i 's/version: 0.1.3/version: 0.1.4/' hyproled
   git commit -am "Bump version to 0.1.4"
   git tag -a v0.1.4 -m "Release v0.1.4"
   git push && git push --tags
   ```

2. Create GitHub release through the web interface

3. Deploy to AUR:
   ```bash
   git clone ssh://aur@aur.archlinux.org/hyproled-git.git
   cd hyproled-git
   # Update PKGBUILD
   makepkg --printsrcinfo > .SRCINFO
   git add PKGBUILD .SRCINFO
   git commit -m "Update to version 0.1.4"
   git push
   ```

### Troubleshooting

- **Workflow fails at AUR deployment**: Make sure the `AUR_SSH_KEY` secret is correctly configured
- **Version not updating**: Check that the version format in `hyproled` follows `version: X.Y.Z`
- **Push fails**: Ensure the GitHub Actions bot has permission to push to the repository (this should be enabled by default)

### Notes

- The workflow uses `workflow_dispatch`, meaning it only runs when manually triggered
- The AUR package is `hyproled-git`, which tracks the Git repository
- For `-git` AUR packages, we increment `pkgrel` rather than changing `pkgver`
- The workflow requires `makepkg` to be available for generating `.SRCINFO`
