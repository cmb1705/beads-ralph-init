# Distribution & Packaging Guide

## Current Location

The plugin is installed at:
```
C:\Users\cmb17\.claude\plugins\local\beads-ralph-init\
```

## Distribution Options

### Option 1: GitHub Repository (Recommended)

**Best for**: Public sharing, version control, community contributions

1. Create a GitHub repository:
   ```bash
   cd "C:\Users\cmb17\.claude\plugins\local\beads-ralph-init"
   git init
   git add .
   git commit -m "Initial release v1.0.0"
   gh repo create beads-ralph-init --public --source=. --push
   ```

2. Users install via:
   ```bash
   /plugin install beads-ralph-init@github:yourusername/beads-ralph-init
   ```

### Option 2: Claude Code Marketplace

**Best for**: Maximum visibility, official distribution

1. Fork the marketplace repository:
   ```bash
   gh repo fork anthropics/claude-code-plugins
   ```

2. Add your plugin to `plugins/` directory

3. Create a pull request following marketplace guidelines

4. Users install via:
   ```bash
   /plugin install beads-ralph-init@claude-code-marketplace
   ```

### Option 3: Private Marketplace

**Best for**: Team/organization internal distribution

1. Create a private git repository with marketplace.json:
   ```json
   {
     "name": "my-team-plugins",
     "plugins": {
       "beads-ralph-init": {
         "source": "./plugins/beads-ralph-init"
       }
     }
   }
   ```

2. Users add marketplace and install:
   ```bash
   /plugin marketplace add my-team https://github.com/my-org/my-plugins
   /plugin install beads-ralph-init@my-team
   ```

### Option 4: Direct Path (Development/Testing)

**Best for**: Quick testing, development

```bash
claude --plugin-dir "C:\Users\cmb17\.claude\plugins\local\beads-ralph-init"
```

## Packaging Checklist

Before publishing, ensure:

- [x] `plugin.json` has name, version, description
- [x] `README.md` includes installation and usage
- [x] `CHANGELOG.md` documents changes
- [ ] All commands tested
- [ ] Skill triggers verified
- [ ] Cross-platform compatibility checked (Windows/Mac/Linux)

## Recommended Next Steps

1. **Create GitHub repo** for version control and sharing
2. **Add LICENSE file** (MIT recommended for open source)
3. **Submit to claude-code-marketplace** for wider distribution
4. **Create GitHub releases** for version tracking

## Version Management

Update these files when releasing new versions:
1. `.claude-plugin/plugin.json` - version field
2. `CHANGELOG.md` - document changes
3. Git tag: `git tag v1.0.1 && git push --tags`
