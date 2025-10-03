# GitHub Actions Setup Guide

This guide explains how to set up the automatic website rebuild trigger when documentation changes.

## Overview

When changes are pushed to the `docs` repository (specifically in the `content/` directory), a GitHub Actions workflow automatically triggers a rebuild of the `edge-nextjs-website` repository.

## Required Setup

### 1. Create a Personal Access Token (PAT)

You need to create a GitHub Personal Access Token that allows the docs repository to trigger workflows in the website repository.

1. Go to GitHub Settings → Developer settings → Personal access tokens → **Tokens (classic)**

   - URL: https://github.com/settings/tokens

2. Click **"Generate new token"** → **"Generate new token (classic)"**

3. Configure the token:

   - **Name**: `Docs to Website Rebuild Trigger`
   - **Expiration**: Choose an appropriate expiration (recommended: 1 year)
   - **Scopes**: Select the following:
     - ✅ `repo` (Full control of private repositories)
     - ✅ `workflow` (Update GitHub Action workflows)

4. Click **"Generate token"** and **copy the token immediately** (you won't be able to see it again)

### 2. Add the Secret to the Docs Repository

1. Go to the docs repository settings:

   - URL: https://github.com/Enrypase/edge-docs/settings/secrets/actions

2. Click **"New repository secret"**

3. Add the secret:

   - **Name**: `WEBSITE_REBUILD_TOKEN`
   - **Value**: Paste the Personal Access Token you created in step 1

4. Click **"Add secret"**

### 3. Configure Directus Environment Variables (Required for CMS sync)

If you're using Directus CMS for documentation, add these secrets to the **website repository** (Enrypase/edge-test):

1. Go to: https://github.com/Enrypase/edge-test/settings/secrets/actions

2. Add the following secrets:

   - **Name**: `NEXT_PUBLIC_DIRECTUS_URL`

     - **Value**: Your Directus instance URL (e.g., `https://your-directus-instance.com`)

   - **Name**: `DIRECTUS_BOT_SECRET`
     - **Value**: Your Directus API token with read access to the `Doc` and `Source` collections

> **Note**: If you're only using the GitHub docs repository (not Directus), you can skip this step, but you'll need to modify the workflow to comment out the Directus sync step.

### 4. Optional: Configure Vercel Deployment (if using Vercel)

If the website is deployed on Vercel and you want automatic deployments, add these additional secrets to the **website repository** (Enrypase/edge-test):

1. Go to: https://github.com/Enrypase/edge-test/settings/secrets/actions

2. Add the following secrets:

   - **Name**: `VERCEL_TOKEN`

     - **Value**: Get from Vercel dashboard → Settings → Tokens

   - **Name**: `VERCEL_ORG_ID`

     - **Value**: Get from your Vercel project settings (`.vercel/project.json`)

   - **Name**: `VERCEL_PROJECT_ID`
     - **Value**: Get from your Vercel project settings (`.vercel/project.json`)

## How It Works

### Workflow Trigger (docs repository)

1. When you push changes to the `master` or `main` branch in the `content/` directory
2. The workflow `.github/workflows/trigger-website-rebuild.yml` runs
3. It sends a `repository_dispatch` event to the website repository with metadata about the change

### Workflow Response (website repository)

1. The website repository receives the `docs-updated` event
2. The workflow `.github/workflows/rebuild-on-docs-change.yml` runs
3. It performs the following steps:
   - Checks out the website code (from the `fumadocs` branch)
   - Installs dependencies
   - Runs `yarn sync-github-docs` to pull the latest documentation from the edge-docs GitHub repository
   - Runs `yarn sync-docs` to pull documentation from Directus CMS (if configured)
   - Runs `yarn docs` (fumadocs-mdx) to process the MDX files
   - Builds the website with `yarn build`
   - (Optional) Deploys to Vercel if configured

### Documentation Sync Flow

The website pulls documentation from two sources:

1. **GitHub (edge-docs repository)**: Markdown files from the `content/` directory are cloned and converted to MDX format
2. **Directus CMS**: Additional documentation managed in the Directus headless CMS

Both sources are merged into the final website build.

## Testing the Setup

### Manual Trigger

You can manually trigger the workflow to test it:

1. Go to the docs repository actions:

   - URL: https://github.com/Enrypase/edge-docs/actions

2. Select the "Trigger Website Rebuild" workflow

3. Click **"Run workflow"** → Select the `master` branch → Click **"Run workflow"**

### Automatic Trigger

Make any change to a file in the `content/` directory:

```bash
# Example: Update a documentation file
cd /home/enrypase/Desktop/Work/Marketized/Edge/docs
echo "# Test update" >> content/documentation/README.md
git add .
git commit -m "test: trigger website rebuild"
git push origin master
```

Then check:

- Docs repository actions: https://github.com/Enrypase/edge-docs/actions
- Website repository actions: https://github.com/Enrypase/edge-test/actions

## Workflow Files

### Docs Repository

- **File**: `.github/workflows/trigger-website-rebuild.yml`
- **Purpose**: Detects changes and sends notification to website repo
- **Triggers on**: Push to `master`/`main` branch (only `content/` directory changes)

### Website Repository (edge-test)

- **File**: `.github/workflows/rebuild-on-docs-change.yml`
- **Purpose**: Receives notification and rebuilds the website
- **Triggers on**: `repository_dispatch` event with type `docs-updated`

## Troubleshooting

### Workflow doesn't trigger

1. **Check the PAT token**:

   - Ensure `WEBSITE_REBUILD_TOKEN` is set in the docs repository secrets
   - Verify the token has `repo` and `workflow` scopes
   - Check if the token has expired

2. **Check branch names**:

   - Ensure you're pushing to `master` or `main` branch
   - Update the workflow if your default branch has a different name

3. **Check file paths**:
   - The trigger only works for changes in the `content/` directory
   - If you need to trigger on other paths, update the `paths` filter in the workflow

### Build fails in website repository

1. **Check the sync-docs script**:

   - Ensure `yarn sync-docs` works locally
   - Verify environment variables are set correctly

2. **Check dependencies**:

   - Ensure `package.json` is up to date
   - Verify Node.js version compatibility (requires Node 22+)

3. **Check the logs**:
   - Go to Actions tab in the website repository
   - Click on the failed workflow run
   - Review the logs for specific error messages

## Security Notes

- The PAT token has access to private repositories - keep it secure
- Never commit the token to the repository
- Regularly rotate the token (recommended: every 6-12 months)
- Use the minimum required scopes for the token
- Consider using fine-grained PATs for better security (GitHub's newer token type)

## Need Help?

If you encounter issues:

1. Check the workflow logs in the Actions tab
2. Verify all secrets are configured correctly
3. Test the `yarn sync-docs` script locally first
4. Check that both repositories are accessible with the PAT
