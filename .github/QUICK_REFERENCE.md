# Quick Reference - Auto Rebuild Setup

This is a quick reference for the automatic website rebuild system.

## 📦 What Was Set Up

### In the `edge-docs` repository:

```
.github/
├── workflows/
│   └── trigger-website-rebuild.yml   # Triggers rebuild when docs change
├── SETUP_GUIDE.md                     # Detailed setup instructions
└── QUICK_REFERENCE.md                 # This file
```

### In the `edge-nextjs-website` repository:

```
.github/
└── workflows/
    └── rebuild-on-docs-change.yml     # Responds to rebuild trigger

scripts/
└── sync-github-docs.js                # Syncs docs from GitHub repo
```

## ⚙️ Configuration Checklist

- [ ] **Create GitHub Personal Access Token** (with `repo` and `workflow` scopes)
- [ ] **Add `WEBSITE_REBUILD_TOKEN` secret** to edge-docs repository
- [ ] **Add `NEXT_PUBLIC_DIRECTUS_URL` secret** to edge-test repository (if using Directus)
- [ ] **Add `DIRECTUS_BOT_SECRET` secret** to edge-test repository (if using Directus)
- [ ] **Add Vercel secrets** to edge-test repository (if auto-deploying to Vercel)

## 🚀 How to Use

### Automatic Trigger

Just push changes to the `master` branch in the `content/` directory:

```bash
cd /home/enrypase/Desktop/Work/Marketized/Edge/docs
# Make your changes to files in content/
git add content/
git commit -m "docs: update documentation"
git push origin master
```

The website will automatically rebuild within a few minutes.

### Manual Trigger

Go to the GitHub Actions page and manually trigger the workflow:

1. **Docs repo**: https://github.com/Enrypase/edge-docs/actions
2. Click "Trigger Website Rebuild" workflow
3. Click "Run workflow" button

## 📊 Monitoring

### Check Workflow Status

- **Docs trigger**: https://github.com/Enrypase/edge-docs/actions
- **Website rebuild**: https://github.com/Enrypase/edge-test/actions

### Workflow Statuses

- ✅ **Success** - Everything worked correctly
- ⚠️ **Warning** - Workflow completed with warnings
- ❌ **Failed** - Check the logs for errors

## 🔧 Common Commands

### Test sync locally (website repo):

```bash
cd /home/enrypase/Desktop/Work/Marketized/Edge/edge-nextjs-website

# Sync docs from GitHub
yarn sync-github-docs

# Sync docs from Directus
yarn sync-docs

# Process MDX files
yarn docs

# Build the site
yarn build
```

## 🐛 Troubleshooting

| Problem                  | Solution                                                    |
| ------------------------ | ----------------------------------------------------------- |
| Workflow doesn't trigger | Check `WEBSITE_REBUILD_TOKEN` is set correctly              |
| Build fails              | Check logs in website repo Actions tab                      |
| Docs not syncing         | Verify branch name is `master` and path is `content/`       |
| Directus sync fails      | Verify `NEXT_PUBLIC_DIRECTUS_URL` and `DIRECTUS_BOT_SECRET` |

## 📝 File Locations

### Docs Repository (edge-docs)

- Local: `/home/enrypase/Desktop/Work/Marketized/Edge/docs`
- GitHub: https://github.com/Enrypase/edge-docs
- Trigger: Changes to `content/**` on `master` branch

### Website Repository (edge-test)

- Local: `/home/enrypase/Desktop/Work/Marketized/Edge/edge-nextjs-website`
- GitHub: https://github.com/Enrypase/edge-test
- Branch: `fumadocs`
- Receives: `docs-updated` repository_dispatch events

## 🔗 Useful Links

- [Detailed Setup Guide](.github/SETUP_GUIDE.md)
- [GitHub Actions - repository_dispatch](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch)
- [Fumadocs Documentation](https://fumadocs.vercel.app/)
- [Vercel Deployment Docs](https://vercel.com/docs/cli/deploying)

## 💡 Tips

- The workflow only triggers for changes in the `content/` directory
- You can manually trigger the workflow anytime from the Actions tab
- The temporary clone directory (`.temp-docs`) is kept between runs for faster syncs
- Both GitHub and Directus docs sources are merged in the final build
- The script automatically converts `.md` files to `.mdx` and adds frontmatter if missing

## 🔒 Security Notes

- Never commit secrets or tokens to the repository
- Rotate the `WEBSITE_REBUILD_TOKEN` every 6-12 months
- The PAT has full repository access - keep it secure
- Use GitHub's fine-grained tokens for better security if possible
