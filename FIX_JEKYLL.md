# Fix Jekyll Issue - Step by Step

## The Problem
GitHub Pages is building your site with Jekyll instead of using the pre-built MkDocs site.

## Solution

### Step 1: Verify GitHub Pages Settings

1. Go to: https://github.com/sunilkumartc/data-platform-playbook/settings/pages
2. **CRITICAL**: Make sure:
   - **Source**: "Deploy from a branch" (NOT "GitHub Actions")
   - **Branch**: `gh-pages` (NOT `main`)
   - **Folder**: `/ (root)`
3. Click **Save**

### Step 2: Manually Add .nojekyll to gh-pages Branch

Run these commands in your terminal:

```bash
cd /Users/sk040d/myprojects/sunil/data-platform-playbook

# Fetch the gh-pages branch
git fetch origin gh-pages

# Checkout gh-pages branch
git checkout gh-pages

# Create .nojekyll file (this tells GitHub Pages to skip Jekyll)
touch .nojekyll

# Commit and push
git add .nojekyll
git commit -m "Disable Jekyll processing"
git push origin gh-pages

# Go back to main branch
git checkout main
```

### Step 3: Verify .nojekyll Exists

1. Go to: https://github.com/sunilkumartc/data-platform-playbook/tree/gh-pages
2. You should see `.nojekyll` file in the root of the gh-pages branch
3. If you don't see it, repeat Step 2

### Step 4: Wait and Test

1. Wait 2-3 minutes for GitHub Pages to rebuild
2. Clear your browser cache (Ctrl+Shift+R or Cmd+Shift+R)
3. Visit: https://sunilkumartc.github.io/data-platform-playbook/
4. You should see your MkDocs Material theme (Deep Purple/Orange), NOT Jekyll theme

## Why This Happens

- GitHub Pages automatically uses Jekyll when it sees markdown files
- The `.nojekyll` file tells GitHub Pages: "Don't use Jekyll, just serve the static files"
- Your site is built by MkDocs and deployed to `gh-pages` branch
- GitHub Pages should serve from `gh-pages` branch, not build from `main`

## After Fix

The updated GitHub Actions workflow will automatically create `.nojekyll` on future deployments. But for now, use the manual steps above to fix it immediately.

## Troubleshooting

If it still doesn't work:

1. **Check Actions tab**: Make sure the workflow completed successfully
2. **Check gh-pages branch**: Verify `.nojekyll` exists
3. **Check Pages settings**: Make sure source is `gh-pages` branch
4. **Wait longer**: GitHub Pages can take up to 5 minutes to update
5. **Try incognito mode**: Rule out browser cache issues

