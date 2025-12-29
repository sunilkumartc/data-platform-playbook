# Fix GitHub Pages Deployment Issue

## Problem
GitHub Pages is trying to build your site with Jekyll instead of using the pre-built MkDocs site.

## Solution

### Step 1: Commit the `.nojekyll` file
```bash
git add .nojekyll
git commit -m "Add .nojekyll to disable Jekyll"
git push origin main
```

### Step 2: Manually add .nojekyll to gh-pages branch (if needed)

If the automatic deployment doesn't work, manually add it:

```bash
# Clone the gh-pages branch
git clone -b gh-pages https://github.com/sunilkumartc/data-platform-playbook.git gh-pages-temp
cd gh-pages-temp

# Create .nojekyll file
touch .nojekyll

# Commit and push
git add .nojekyll
git commit -m "Disable Jekyll processing"
git push origin gh-pages

# Clean up
cd ..
rm -rf gh-pages-temp
```

### Step 3: Verify GitHub Pages Settings

1. Go to your repo: https://github.com/sunilkumartc/data-platform-playbook
2. Settings → Pages
3. Source: "Deploy from a branch"
4. Branch: `gh-pages` (NOT `main`)
5. Folder: `/ (root)`

### Step 4: Wait and Clear Cache

- Wait 1-2 minutes for GitHub Pages to rebuild
- Clear browser cache (Ctrl+Shift+R or Cmd+Shift+R)
- Or use incognito mode

## Why This Happens

GitHub Pages automatically tries to build sites with Jekyll if it detects markdown files. The `.nojekyll` file tells GitHub Pages to skip Jekyll and serve the static files directly (which is what MkDocs generates).

