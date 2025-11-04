# GitHub Deployment Guide

This guide will help you upload this codebase to your own GitHub account and set up automatic deployment to GitHub Pages.

## Prerequisites

1. **GitHub Account**: You need a GitHub account
2. **Git Installed**: Make sure Git is installed on your system
3. **GitHub CLI (Optional)**: Can be helpful but not required

## Step 1: Prepare the Repository

### 1.1 Remove Current Remote (if needed)

If you want to start fresh with your own repo:

```bash
cd /Users/khoo/Downloads/project4/projects/project-20251103-203353-orderly-dex/temp-clone
git remote remove origin
```

### 1.2 Clean Up (Optional)

If you want to remove git history and start fresh:

```bash
# Remove .git directory to start fresh
rm -rf .git

# Initialize new git repository
git init
git branch -M main
```

Or if you want to keep history but just change remote:

```bash
# Just change the remote URL (skip if you removed it)
# git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
```

## Step 2: Create GitHub Repository

### 2.1 Create New Repository on GitHub

1. Go to [GitHub.com](https://github.com) and sign in
2. Click the **"+"** icon in the top right corner
3. Select **"New repository"**
4. Fill in the repository details:
   - **Repository name**: Choose a name (e.g., `my-orderly-dex`, `powerbrocker-dex`)
   - **Description**: Add a description (optional)
   - **Visibility**: Select **Public** (required for GitHub Pages free tier)
   - **DO NOT** initialize with README, .gitignore, or license (we already have these)
5. Click **"Create repository"**

### 2.2 Note Your Repository URL

After creating, GitHub will show you the repository URL. It will look like:
```
https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
```

## Step 3: Connect and Push Your Code

### 3.1 Add Your GitHub Remote

```bash
# Add your new repository as origin
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git

# Verify the remote was added
git remote -v
```

### 3.2 Commit All Changes (if needed)

```bash
# Check status
git status

# Add all files
git add .

# Commit (if there are uncommitted changes)
git commit -m "Initial commit: Orderly DEX template"
```

### 3.3 Push to GitHub

```bash
# Push to main branch
git push -u origin main
```

If you get authentication errors, you may need to:
- Use a Personal Access Token (PAT) instead of password
- Or configure SSH keys

## Step 4: Configure GitHub Pages

### 4.1 Enable GitHub Pages

1. Go to your repository on GitHub
2. Click on **Settings** tab
3. Scroll down to **Pages** in the left sidebar
4. Under **Source**, select:
   - **Source**: `GitHub Actions`
5. Click **Save**

### 4.2 Create GitHub Personal Access Token (PAT)

The workflow needs a Personal Access Token with write permissions:

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Or directly: https://github.com/settings/tokens
2. Click **"Generate new token"** → **"Generate new token (classic)"**
3. Configure the token:
   - **Note**: "GitHub Pages Deployment Token"
   - **Expiration**: Choose appropriate duration (90 days, 1 year, or no expiration)
   - **Scopes**: Check the following:
     - ✅ `repo` (Full control of private repositories)
     - ✅ `workflow` (Update GitHub Action workflows)
4. Click **"Generate token"**
5. **Copy the token immediately** (you won't be able to see it again!)

### 4.3 Add Secret to Repository

1. Go to your repository on GitHub
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **"New repository secret"**
4. Configure:
   - **Name**: `TEMPLATE_PAT`
   - **Secret**: Paste your Personal Access Token from step 4.2
5. Click **"Add secret"**

## Step 5: Modify Workflow for Your Repository

The workflow might need some adjustments if you're not using it as a fork:

### 5.1 Update Workflow File (Optional)

If you're not forking from the template, you can simplify the workflow:

**File**: `.github/workflows/deploy.yml`

You can remove or modify the fork-syncing logic (lines 54-99) if you don't need it:

```yaml
# Remove or comment out the "Sync with upstream" step if not forking
```

However, the current workflow should work fine even if you're not forking - it will just skip the sync step.

### 5.2 Verify Workflow Configuration

The workflow is already configured to:
- ✅ Deploy on push to `main` branch
- ✅ Deploy on manual workflow dispatch
- ✅ Build the application with correct base path
- ✅ Deploy to GitHub Pages

## Step 6: Configure Your DEX Settings

### 6.1 Update Configuration

Edit `public/config.js` with your broker details:

```javascript
window.__RUNTIME_CONFIG__ = {
  "VITE_ORDERLY_BROKER_ID": "your_broker_id",
  "VITE_ORDERLY_BROKER_NAME": "Your Broker Name",
  // ... other settings
};
```

### 6.2 Update Repository Name for Base Path

If your repository name is different from the default, the workflow automatically uses the repo name as the base path. For example:
- If repo is `my-orderly-dex`, site will be at: `https://YOUR_USERNAME.github.io/my-orderly-dex/`

## Step 7: Test the Deployment

### 7.1 Trigger Deployment

1. **Option 1: Push a change**
   ```bash
   # Make a small change (e.g., update README)
   echo "# My DEX" > README.md
   git add README.md
   git commit -m "Test deployment"
   git push origin main
   ```

2. **Option 2: Manual trigger**
   - Go to your repository on GitHub
   - Click **Actions** tab
   - Select **"Deploy to GitHub Pages"** workflow
   - Click **"Run workflow"** → **"Run workflow"**

### 7.2 Monitor Deployment

1. Go to **Actions** tab in your repository
2. You should see the workflow running
3. Click on the workflow run to see progress
4. Wait for it to complete (usually 2-5 minutes)

### 7.3 Access Your Deployed Site

Once deployment completes:
1. Go to **Settings** → **Pages**
2. Your site URL will be shown:
   - `https://YOUR_USERNAME.github.io/YOUR_REPO_NAME/`
   - Or if you have a custom domain: your custom domain

## Step 8: Custom Domain (Optional)

If you want to use a custom domain:

### 8.1 Add CNAME File

Create a file named `CNAME` in the root directory:

```bash
echo "yourdomain.com" > CNAME
git add CNAME
git commit -m "Add custom domain"
git push origin main
```

### 8.2 Configure DNS

1. Go to your domain registrar
2. Add DNS records:
   - **Type**: `CNAME`
   - **Name**: `@` or `www` (depending on your preference)
   - **Value**: `YOUR_USERNAME.github.io`
3. Wait for DNS propagation (can take up to 48 hours)

### 8.3 Configure in GitHub

1. Go to repository **Settings** → **Pages**
2. Under **Custom domain**, enter your domain
3. Check **"Enforce HTTPS"** (after DNS is configured)

The workflow will automatically detect the CNAME file and build with root path instead of repo name path.

## Step 9: Workflow Features

### 9.1 Automatic Deployment

The workflow automatically deploys when:
- ✅ Code is pushed to `main` branch
- ✅ Manual workflow dispatch is triggered
- ✅ Merge commits are skipped (prevents infinite loops)

### 9.2 Smart Build Caching

The workflow includes optimizations:
- ✅ Only rebuilds if code changes (not just config.js)
- ✅ Caches dependencies and build artifacts
- ✅ Updates config.js without full rebuild if only config changed

### 9.3 Base Path Handling

- **With repo name**: `https://YOUR_USERNAME.github.io/repo-name/`
- **With custom domain**: `https://yourdomain.com/`

The workflow automatically detects which to use based on CNAME file presence.

## Troubleshooting

### Issue: Workflow fails with "Permission denied"

**Solution**: 
- Make sure `TEMPLATE_PAT` secret is set correctly
- Verify the token has `repo` and `workflow` scopes
- Regenerate token if needed

### Issue: Site shows 404

**Solution**:
- Check the base path in the URL matches your repo name
- Verify GitHub Pages is enabled and set to "GitHub Actions"
- Check the workflow completed successfully

### Issue: Build fails

**Solution**:
- Check the Actions tab for error details
- Verify Node.js version compatibility (requires Node 20+)
- Check if all dependencies are in package.json

### Issue: Config changes not reflected

**Solution**:
- The workflow caches builds - if only config.js changed, it should update automatically
- If not updating, trigger a manual workflow run
- Or make a small code change to force rebuild

## Security Considerations

### 1. Personal Access Token

- Keep your PAT secret secure
- Never commit it to the repository
- Rotate it periodically
- Use fine-grained tokens if possible (GitHub feature)

### 2. Repository Settings

- Review who has access to your repository
- Consider using branch protection rules
- Enable security alerts for dependencies

### 3. Environment Variables

- Keep sensitive data in GitHub Secrets
- Never commit API keys or tokens
- Use `public/config.js` only for public configuration

## Next Steps

After deployment is working:

1. **Customize Your DEX**:
   - Update `public/config.js` with your broker ID and settings
   - Customize theme in `app/styles/theme.css`
   - Add your logos and branding

2. **Monitor Deployments**:
   - Set up notifications for deployment status
   - Monitor Actions tab for any issues

3. **Set Up CI/CD**:
   - Consider adding tests before deployment
   - Add linting checks
   - Set up branch protection

4. **Documentation**:
   - Update README.md with your project details
   - Document any custom configurations

## Quick Reference Commands

```bash
# Initialize and push to new repo
git init
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git add .
git commit -m "Initial commit"
git push -u origin main

# Update config and redeploy
# Edit public/config.js
git add public/config.js
git commit -m "Update configuration"
git push origin main

# Check deployment status
# Visit: https://github.com/YOUR_USERNAME/YOUR_REPO/actions
```

## Support

If you encounter issues:
1. Check the GitHub Actions logs for detailed error messages
2. Verify all secrets are configured correctly
3. Ensure GitHub Pages is enabled
4. Check the workflow file syntax is valid

## Workflow File Reference

The workflow file (`.github/workflows/deploy.yml`) handles:
- ✅ Detecting if repo is a fork
- ✅ Syncing with upstream (if fork)
- ✅ Building the application
- ✅ Caching for faster builds
- ✅ Deploying to GitHub Pages
- ✅ Handling custom domains

You can customize it further if needed, but the default configuration should work for most use cases.

