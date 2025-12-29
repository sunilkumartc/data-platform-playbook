# Quick Fix for Jekyll Issue

## The Problem
GitHub Pages is building your site with Jekyll instead of using the pre-built MkDocs site.

## Immediate Solution

### Option 1: Manual Fix (Fastest)

1. **Go to your GitHub repo** → Settings → Pages
2. **Change the source**:
   - Source: "Deploy from a branch"
   - Branch: `gh-pages` (NOT `main`)
   - Folder: `/ (root)`
   - Click Save

3. **Manually add .nojekyll to gh-pages branch**:

```bash
# Clone just the gh-pages branch
git clone -b gh-pages https://github.com/sunilkumartc/data-platform-playbook.git temp-gh-pages
cd temp-gh-pages

# Create .nojekyll file
touch .nojekyll

# Commit and push
git add .nojekyll
git commit -m "Disable Jekyll processing"
git push origin gh-pages

# Clean up
cd ..
rm -rf temp-gh-pages
```

4. **Wait 1-2 minutes** and refresh your site

### Option 2: Use GitHub Actions (Automatic)

The updated workflow will automatically create `.nojekyll` on the next deployment.

1. Commit and push the updated workflow:
```bash
git add .github/workflows/ci.yml
git commit -m "Fix: Ensure .nojekyll in gh-pages branch"
git push origin main
```

2. Wait for the GitHub Action to complete (check Actions tab)

3. Verify `.nojekyll` exists in `gh-pages` branch

## Verify It's Fixed

1. Go to: https://github.com/sunilkumartc/data-platform-playbook/tree/gh-pages
2. Check if `.nojekyll` file exists in the root
3. Visit: https://sunilkumartc.github.io/data-platform-playbook/
4. The site should show your MkDocs theme (Deep Purple/Orange) instead of Jekyll theme

## Why This Happens

- GitHub Pages automatically uses Jekyll when it sees markdown files
- The `.nojekyll` file tells GitHub Pages to skip Jekyll
- Your site should be built by MkDocs and deployed to `gh-pages` branch
- GitHub Pages should serve from `gh-pages`, not `main`

